mysql下知识点补充:

```sql
<!--逻辑库--> 
<schema name="lg_edu_order" checkSQLschema="true" sqlMaxLimit="100" 
dataNode="dn1,dn2" randomDataNode="dn1">
</schema> 
```

dataNode: 可以设置针对整个库的读写分离设置

 randomDataNode:  是新加的属性，也就是在配置上有dataNode属性也有randomDataNode属性，一些非DQL语句在在没有randomDataNode属性前是随机发送，有了randomDataNode语句可以指定一个节点而不是随机发送 。（dataNode设置对整个库读写分离，randomDataNode可以指定写操作只在dn1上）。





```sql
<dataHost name="localhost1" maxCon="1000" minCon="10" balance="1" writeType="0"  swichType="1" dbType="mysql" dbDriver="native"> 

  <heartbeat>select user()</heartbeat> 
  <!-- can have multi write hosts --> 
  <writeHost host="M1" url="localhost:3306" user="root" password="123456"> 
	  <readHost host="S1" url="localhost:3307" user="root" password="123456" weight="1"/> 
  </writeHost> 

</dataHost>
```

**balance参数：** 

- 0 ： 所有读操作都发送到当前可用的writeHost 
- 1 ：所有读操作都随机发送到readHost和stand by writeHost
- 2 ：所有读操作都随机发送到writeHost和readHost 
- 3 ：所有读操作都随机发送到writeHost对应的readHost上，但是writeHost不负担读压力(**与2的区别是，没有stand by writeHost 进行分担读，只有writeHost进行读操作**)

**writeType参数：** 

- 0 ： 所有写操作都发送到可用的writeHost 
- 1 ：所有写操作都随机发送到readHost （**与balance=0 搭配，实现主从模式，但读库崩溃，写库也不能用了，不建议使用**）
- 2 ：所有写操作都随机发送到writeHost，readHost （**需要要配置双主双写模式了，不建议使用**）

**switchType参数：** 

- -1： 表示不自动切换 
- 1 ：表示自动切换 （**主库挂了，自动进行切换，需要配多writeHost**）
- 2 ：基于MySQL主从同步状态决定是否切换 (**通过心跳检测 show slave status ，进行判断是否切换在主库进行读操作**)
- 3 ：基于MySQL cluster集群切换机制（**开始支持 MySQL 集群模式，让读更加安全可靠，配置如下: MyCAT 心跳检查语句配置为 show status like ‘wsrep%’** ）



## MyCAT中的弱XA事务

### 一、**XA事务原理** 

分布式事务处理（ Distributed Transaction Processing ， DTP ）指一个程序或程序段，在一个或多个资源如数据库或文件上为完成某些功能的执行过程的集合，分布式事务处理的关键是必须有一种方法可以知道事务在任何地方所做的所有动作，提交或回滚事务的决定必须产生统一的结果（全部提交或全部回滚）。X/Open 组织（即现在的 Open Group ）定义了分布式事务处理模型。 X/Open DTP 模型（ 1994 ）包括应用程序（ AP ）、事务管理器（ TM ）、资源管理器（ RM ）、通信资源管理器（ CRM ）四部分。一般，常见的事务管理器（ TM ）是交易中间件，常见的资源管理器（ RM ）是数据库，常见的通信资源管理器（ CRM ）是消息中间件，下图是X/Open DTP模型

![image-20210125063612505](https://tva1.sinaimg.cn/large/008eGmZEly1gmzjp4mc5aj30rc0fytba.jpg)

一般的编程方式是这样的

- 配置TM，通过TM或者RM提供的方式，把RM注册到TM。可以理解为给TM注册RM作为数据源。一个TM可以注册多个RM。

- AP从TM获取资源管理器的代理（例如：使用JTA接口，从TM管理的上下文中，获取出这个TM所管理的RM的JDBC连接或JMS连接）。

- AP向TM发起一个全局事务。这时，TM会通知各个RM。XID（全局事务ID）会通知到各个RM。

- AP通过1中获取的连接，直接操作RM进行业务操作。这时，AP在每次操作时把XID(包括所属分支的信息)传递给RM，RM正是通过这个XID与2步中的XID关联来知道操作和事务的关系的。

- AP结束全局事务。此时TM会通知RM全局事务结束。

- 开始二段提交，也就是prepare - commit的过程


​       XA协议(XA Specification)，指的是TM和RM之间的接口，其实这个协议只是定义了xa_和ax_系列的函数原型以及功能描述、约束和实施规范等。至于RM和TM之间通过什么协议通信，则没有提及，目前知名的数据库，如Oracle, DB2,Mysql等，都是实现了XA接口的，都可以作为RM。Tuxedo、TXseries、Atomikos、Bitronix、Narayana等事务中间件可以通过XA协议跟这些数据源进行对接。JTA(Java Transaction API)是符合X/Open DTP的一个编程模型，事务管理和资源管理器支架也是用了XA协议。



下面两个图片分别给出了XA成功与失败的两种情况，首先是XA事务成功的流程图：

![image-20210125064441700](https://tva1.sinaimg.cn/large/008eGmZEly1gmzjxww9muj30te0dkdio.jpg)

![image-20210125064515119](https://tva1.sinaimg.cn/large/008eGmZEly1gmzjyhz6hxj30ta0d077g.jpg)

然后，是XA事务失败的流程图：

![image-20210125064609377](https://tva1.sinaimg.cn/large/008eGmZEly1gmzjzfo5i9j30t60fu77o.jpg)

![image-20210125064624474](https://tva1.sinaimg.cn/large/008eGmZEly1gmzjzp1aeoj30uq0dkjtr.jpg)

  XA事务的关键在于TM组件，其中的难点技术点如下：

第二段提交时，当RM1 commit完成了，而RM2 commit还没有完成，这时TM需要进行协调，当RM2恢复以后，**重新提交**之前没有Commit的事务，或者自动回滚之前Rollback的事务。

因此**TM需要记录XA事务的状态**，以及在各个**RM上的执行情况**，这个**日志文件**需要**存储在可靠**的地方，用来进行XA事务异常之后的补救工作。

**TM是一定要把事务的信息，比如XID，哪个RM已经完成了等保存起来的。只有当全部的RM提交或者回滚完后，才能丢弃这些事务的信息。**

 于是我们明白**TM是一个单点**，要非常可靠才行。

以Java分布式事务的开源TM组件atomikos为例，它是通过在应用的目录下生成日志文件来保证，如果失败，在重启后可以通过日志来完成未完成的事务。

mycat未来计划以Zookeeper作为XA事务的日志存储手段，实现TM角色以支持XA事务(**的强一致性，现在mycat是弱一致性的**).

### **二、XA事务的问题和MySQL的局限**

XA事务的明显问题是timeout问题，比如当一个RM出问题了，那么整个事务只能处于等待状态。这样可以会连锁反应，导致整个系统都很慢，最终不可用，另外2阶段提交也大大增加了XA事务的时间，使得XA事务无法支持高并发请求。

避免使用XA事务的方法通常是最终一致性。

举个例子，比如一个业务逻辑中，最后一步是用户账号增加300元，为了减少DB的压力，先把这个放到消息队列里，然后后端再从消息队列里取出消息，更新DB。那么如何保证，这条消息不会被重复消费？或者重复消费后，仍能保证结果是正确的？在消息里带上用户帐号在数据库里的版本，在更新时比较数据的版本，如果相同则加上300；比如用户本来有500元，那么消息是更新用户的钱数为800，而不是加上300；

另外一个方式是，建一个消息是否被消费的表，记录消息ID，在事务里，先判断消息是否已经消费过，如果没有，则更新数据库，加上300,否则说明已经消费过了，丢弃。

前面两种方法都必须从流程上保证是单方向的。

其实严格意义上，用消息队列来实现最终一致性仍然有漏洞，因为消息队列跟当前操作的数据库是两个不同的资源，仍然存在消息队列失败导致这个账号增加300元的消息没有被存储起来（当然复杂的高级的消息队列产品可以避免这种现象，但仍然存在风险），而第二种方式则由于**新的表跟之前的事务操作的表示在一个Database中**，因此不存在上述的可能性。

MySQL的XA事务，长期以来都存在一个缺陷：

MySQL数据库的主备数据库的同步，通过Binlog的复制完成。而Binlog是MySQL数据库内部XA事务的协调者，并且MySQL数据库为binlog做了优化——**binlog不写prepare日志**，**只写commit日志**。所有的参与节点prepare完成，在进行xa commit前crash。crash recover如果选择commit此事务。由于binlog在prepare阶段未写，因此主库中看来，此分布式事务最终提交了，但是此事务的操作并未写到binlog中，因此也就未能成功复制到备库，从而导致**主备库数据不一致的情况出现**。

### 二、MyCAT分布式事务

 Mycat里的事务包括以下几种情况：

- SQL不垮分片：事务中的SQL在单个节点上执行

- SQL跨分片：事务中的SQL在多个节点上执行

其中，第一种情况，SQL仅仅在一个dataNode上执行，此时Mycat事务模式跟标准的数据库事务模式一样，要么提交要么回滚；而对于第二种事务，Mycat执行的一种”弱XA事务“模式，此模式的逻辑如下：

​       首先事务内的SQL在各自的分片上执行并返回状态码，若某个分片上的返回码为ERROR，则Mycat认为事务失败，应用端只能回滚（rollback）事务，Mycat收到回滚指令后，依次回滚事务中涉及到的所有分片；若事务中的所有SQL的执行都返回成功（OK）的返回码，则应用程序提交事务的时候，Mycat会同时向事务中涉及到的节点发送提交事务的指令。(**弱一致性体现在，mycat不记录每个分片状态，也就不保存rm信息，一个分片失败也不进行多次尝试和恢复后继续提交，而是简单的进行一个失败，全部回滚**)

​        这里称之为弱XA，是因为第二阶段Commit的时候，若某个节点出错了，也无法等节点恢复以后去做Recover操作重新commit（只能回滚，而不会等其恢复后再次提交。），但考虑到所有的节点都执行成功，但Commit指令失败的概率很小，因此这种弱XA事务也已经满足大多数应用的需求，而且性能接近普通事务。

   XA本质上还是一个二阶段提交，二阶段提交看起来确实能够提供原子性的操作，但是不幸的事，二阶段提交还是有几个缺点的：

> 1、同步阻塞问题。执行过程中，所有参与节点都是事务阻塞型的。当参与者占有公共资源时，其他第三方节点访问公共资源不得不处于阻塞状态。
>
> 2、单点故障。由于协调者的重要性，一旦协调者发生故障。参与者会一直阻塞下去。尤其在第二阶段，协调者发生故障，那么所有的参与者还都处于锁定事务资源的状态中，而无法继续完成事务操作。（如果是协调者挂掉，可以重新选举一个协调者，但是无法解决因为协调者宕机导致的参与者处于阻塞状态的问题）
>
> 3、数据不一致。在二阶段提交的阶段二中，当协调者向参与者发送commit请求之后，发生了局部网络异常或者在发送commit请求过程中协调者发生了故障，这回导致只有一部分参与者接受到了commit请求。而在这部分参与者接到commit请求之后就会执行commit操作。但是其他部分未接到commit请求的机器则无法执行事务提交。于是整个分布式系统便出现了数据部一致性的现象。
>
> 4、二阶段无法解决的问题：协调者再发出commit消息之后宕机，而唯一接收到这条消息的参与者同时也宕机了。那么即使协调者通过选举协议产生了新的协调者，这条事务的状态也是不确定的，没人知道事务是否被已经提交。