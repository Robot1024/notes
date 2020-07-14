# **第一部分：项目架构演变过程**

单体架构     微服务架构的演变    拉钩网架构演变

Dubbo 提供了以下能力：

1. 面向接口远程方法调用
2. 智能容错和负载均衡
3. 服务的注册与发现
4. 高度可扩展能力
5. 运行期流量调度
6. 可视化的服务治理与运维

# 第二部分：Dubbo架构与实战

Dubbo的架构（调用流程 特性） 注册中心  dubbo的开发案例（注册 和 xml） Dubbo的管理控制台 Dubbo的相关配置

![image-20200710194342273](https://tva1.sinaimg.cn/large/007S8ZIlly1ggm456tnefj31600kudq1.jpg)

节点说明：

| **节点**  | **角色名称**                              |
| --------- | ----------------------------------------- |
| Provider  | 暴露服务的服务提供方                      |
| Consumer  | 调用远程服务的服务消费方                  |
| Registry  | 服务注册与发现的注册中心                  |
| Monitor   | 统计服务的调用次数和调用时间的监控中心    |
| Container | 服务运行容器 负责启动 加载 运行服务提供者 |

**调用关系说明:**
   虚线                代表异步调用
   蓝色虚线        是在启动时完成的功能 
   红色虚线        是程序运行中执行的功能
   红色实现        程序的同步调用

**调用流程:**
   服务提供者在服务容器启动时 向注册中心 注册自己提供的服务
   服务消费者在启动时 向注册中心订阅自己所需的服务
   注册中心返回服务提供者地址列表给消费者 如果有变更 注册中心会基于长连接推送变更数据给消费者
   服务消费者 从提供者地址列表中 基于软负载均衡算法 选一台提供者进行调用 如果调用失败 则重新选 择一台
   服务提供者和消费者 在内存中的调用次数 和 调用时间 定时每分钟发送给监控中心

**Dubbo管理控制台dubbo-admin**

**主要包含**:服务管理 、 路由规则、动态配置、服务降级、访问控制、权重调整、负载均衡等管理功能

如我们在开发时，需要知道Zookeeper注册中心都注册了哪些服务，有哪些消费者来消费这些服务。我 们可以通过部署一个管理中心来实现。其实管理中心就是一个web应用，原来是war(2.6版本以前)包需 要部署到tomcat即可。现在是jar包可以直接通过java命令运行。



**Dubbo配置项说明**

**1** **dubbo:application**

**对应 org.apache.dubbo.config.ApplicationConfig, 代表当前应用的信息**

1. name: 当前应用程序的名称，在dubbo-admin中我们也可以看到，这个代表这个应用名称。我们 在真正时是时也会根据这个参数来进行聚合应用请求。
2. owner: 当前应用程序的负责人，可以通过这个负责人找到其相关的应用列表，用于快速定位到责 任人。
3. qosEnable : 是否启动QoS 默认true
4. qosPort : 启动QoS绑定的端口 默认22222
5. qosAcceptForeignIp: 是否允许远程访问 默认是false

**2** **dubbo:registtry**

**org.apache.dubbo.config.RegistryConfig, 代表该模块所使用的注册中心。一个模块中的服务可以将 其注册到多个注册中心上，也可以注册到一个上。后面再service和reference也会引入这个注册中心。**

1.id:当当前服务中provider 或者 cunsumer 中存在多个注册中心时，则使用需要增加该配置。在一些公司，会通过业务线的不同选择不同的注册中心，所以一般都会配置该值

2.address:当前配置中心的访问地址

3.protocol:当前注册中心使用的协议是什么。也可以直接在address中写入，比如使用zookeeper,就可以写成zookpeer://xx.xx.xx.xx:2181

4.timeout:当与注册中心不在同一个机房时，大多会把该参数延迟

**3** **dubbo:protocol**

**org.apache.dubbo.config.ProtocolConfig, 指定服务在进行数据传输所使用的协议。**

1. id : 在大公司，可能因为各个部门技术栈不同，所以可能会选择使用不同的协议进行交互。这里

在多个协议使用时，需要指定。

2. name : 指定协议名称。默认使用 dubbo 。

**4** **dubbo:service**

**org.apache.dubbo.config.ServiceConfig, 用于指定当前需要对外暴露的服务信息，后面也会具体讲 解。和 dubbo:reference 大致相同。**

1. interface : 指定当前需要进行对外暴露的接口是什么。

2. ref : 具体实现对象的引用，一般我们在生产级别都是使用Spring去进行Bean托管的，所以这里面 一般也指的是Spring中的BeanId。

3. version : 对外暴露的版本号。不同的版本号，消费者在消费的时候只会根据固定的版本号进行消 费。

**5** **dubbo:reference**

**org.apache.dubbo.config.ReferenceConfig, 消费者的配置，这里只做简单说明，后面会具体讲解。**

1. id : 指定该Bean在注册到Spring中的id。
2. interface: 服务接口名
3. version : 指定当前服务版本，与服务提供者的版本一致。
4. registry : 指定所具体使用的注册中心地址。这里面也就是使用上面在 dubbo:registry 中所声明的id。

**6 dubbo:method**

**org.apache.dubbo.config.MethodConfig, 用于在制定的 dubbo:service 或者 dubbo:reference 中的 更具体一个层级，指定具体方法级别在进行RPC操作时候的配置，可以理解为对这上面层级中的配置针 对于具体方法的特殊处理。**

1. name : 指定方法名称，用于对这个方法名称的RPC调用进行特殊配置。 

2. async: 是否异步 默认false

**7 dubbo:service 和 dubbo:reference 详解**

**这两个在dubbo中是我们最为常用的部分，其中有一些我们必然会接触到的属性。并且这里会讲到一些**

**设置上的使用方案。**

1. mock: 用于在方法调用出现错误时，当做服务降级来统一对外返回结果，后面我们也会对这个方 法做更多的介绍。

2. timeout: 用于指定当前方法或者接口中所有方法的超时时间。我们一般都会根据提供者的时长来 具体规定。比如我们在进行第三方服务依赖时可能会对接口的时长做放宽，防止第三方服务不稳定 导致服务受损。

3. check: 用于在启动时，检查生产者是否有该服务。我们一般都会将这个值设置为false，不让其进 行检查。因为如果出现模块之间循环引用的话，那么则可能会出现相互依赖，都进行check的话， 那么这两个服务永远也启动不起来。

4. retries: 用于指定当前服务在执行时出现错误或者超时时的重试机制。

   1. 注意提供者是否有幂等，否则可能出现数据一致性问题

   2. 注意提供者是否有类似缓存机制，如出现大面积错误时，可能因为不停重试导致雪崩

5. executes: 用于在提供者做配置，来确保最大的并行度。

   1. 可能导致集群功能无法充分利用或者堵塞 2. 但是也可以启动部分对应用的保护功能 3. 可以不做配置，结合后面的熔断限流使用



# 第三部分：Dubbo高级应用实战

SPI 负载均衡  异步调用  自定义线程池  路由规则  服务降级

**SPI****简介**

SPI 全称为 (Service Provider Interface) ，是JDK内置的一种服务提供发现机制。 目前有不少框架用它 来做服务的扩展发现，简单来说，它就是一种动态替换发现的机制。使用SPI机制的优势是实现解耦， 使得第三方服务模块的装配控制逻辑与调用者的业务代码分离。

![image-20200714161523937](https://tva1.sinaimg.cn/large/007S8ZIlly1ggqklp52y6j30k80fy0vb.jpg)

![image-20200714161612689](https://tva1.sinaimg.cn/large/007S8ZIlly1ggqkmh6inej30kc0fzq56.jpg)

![image-20200714161640247](https://tva1.sinaimg.cn/large/007S8ZIlly1ggqkn0cm87j30k50eqdif.jpg)

![image-20200714161735296](https://tva1.sinaimg.cn/large/007S8ZIlly1ggqknwpfwgj30k70h50vb.jpg)

**异步调用**

Dubbo不只提供了堵塞式的的同步调用，同时提供了异步调用的方式。这种方式主要应用于提供者接口 响应耗时明显，消费者端可以利用调用接口的时间去做一些其他的接口调用,利用 Future 模式来异步等 待和获取结果即可。这种方式可以大大的提升消费者端的利用率。 目前这种方式可以通过XML的方式进 行引入。

**异步调用实现
** (1)为了能够模拟等待，通过 int timeToWait参数，标明需要休眠多少毫秒后才会进行返回。

(2)接口实现 为了模拟调用耗时 可以让线程等待一段时间
 (3)在消费者端，配置异步调用 注意消费端默认超时时间1000毫秒 如果提供端耗时大于1000毫秒会

出现超时
 可以通过改变消费端的超时时间 通过timeout属性设置即可单位毫秒

(4)测试，我们休眠100毫秒，然后再去进行获取结果。方法在同步调用时的返回值是空，我们可以通 过 RpcContext.getContext().getFuture() 来进行获取Future对象来进行后续的结果等待操作。

**Dubbo****已有线程池**

dubbo在使用时，都是通过创建真实的业务线程池进行操作的。目前已知的线程池模型有两个和java中 的相互对应:

- fix: 表示创建固定大小的线程池。也是Dubbo默认的使用方式，默认创建的执行线程数为200，并 且是没有任何等待队列的。所以再极端的情况下可能会存在问题，比如某个操作大量执行时，可能 存在堵塞的情况。后面也会讲相关的处理办法。

-  cache: 创建非固定大小的线程池，当线程不足时，会自动创建新的线程。但是使用这种的时候需 要注意，如果突然有高TPS的请求过来，方法没有及时完成，则会造成大量的线程创建，对系统的 CPU和负载都是压力，执行越多反而会拖慢整个系统。

**路由规则**

路由是决定一次请求中需要发往目标机器的重要判断，通过对其控制可以决定请求的目标机器。我们可 以通过创建这样的规则来决定一个请求会交给哪些服务器去处理。

**路由规则快速入门**

 (1)提供两个提供者(一台本机作为提供者，一台为其他的服务器)，每个提供者会在调用时可以返回不

同的信息 以区分提供者。

(2)针对于消费者，我们这里通过一个死循环，每次等待用户输入，再进行调用，来模拟真实的请求 情况。

通过调用的返回值 确认具体的提供者。

(3)我们通过ipconfig来查询到我们的IP地址，并且单独启动一个客户端，来进行如下配置(这里假设 我们希望隔离掉本机的请求，都发送到另外一台机器上)。



**路由规则详解**

通过上面的程序，我们实际本质上就是通过在zookeeper中保存一个节点数据，来记录路由规则。消费 者会通过监听这个服务的路径，来感知整个服务的路由规则配置，然后进行适配。这里主要介绍路由配 置的参数。

- route:// 表示路由规则的类型，支持条件路由规则和脚本路由规则，可扩展，**必填**。

- 0.0.0.0 表示对所有 IP 地址生效，如果只想对某个 IP 的生效，请填入具体 IP，**必填**。 

- com.lagou.service.HelloService 表示只对指定服务生效，**必填**。 

- category=routers 表示该数据为动态配置类型，**必填**。

- dynamic : 是否为持久数据，当指定服务重启时是否继续生效。**必填**。
-  runtime : 是否在设置规则时自动缓存规则，如果设置为true则会影响部分性能。
-  rule : 是整个路由最关键的配置，用于配置路由规则。
   ... => ... 在这里 => 前面的就是表示消费者方的匹配规则，可以不填(代表全部)。 => 后方则必

​        须填写，表示当请求过来时，如果选择提供者的配置。官方这块儿也给出了详细的示例，可以按照

​        那里来讲。

​        其中使用最多的便是 host 参数。 **必填**。

**服务动态降级**

**第一种 在 dubbo 管理控制台配置服务降级**

屏蔽和容错

mock=force:return+null
 表示消费方对该服务的方法调用都直接返回 null 值，不发起远程调用。用来屏蔽不重要服务不可 用时对调用方的影响。
 mock=fail:return+null
 表示消费方对该服务的方法调用在失败后，再返回 null 值，不抛异常。用来容忍不重要服务不稳 定时对调用方的影响。

**第二种 指定返回简单值或者null**

```xml
<dubbo:reference id="xxService" check="false" interface="com.xx.XxService" timeout="3000" mock="return null" />

<dubbo:reference id="xxService2" check="false" interface="com.xx.XxService2" timeout="3000" mock="return 1234" />
```

如果是标注 则使用@Reference(mock="return null") @Reference(mock="return 简单值") 也支持 @Reference(mock="force:return null")

**第三种 使用java代码 动态写入配置中心**

```java
RegistryFactory registryFactory =

ExtensionLoader.getExtensionLoader(RegistryFactory.class).getAdaptiveExtension() ;
 Registry registry = registryFactory.getRegistry(URL.valueOf("zookeeper://IP:端 口"));

registry.register(URL.valueOf("override://0.0.0.0/com.foo.BarService? category=configurators&dynamic=false&application=foo&mock=force:return+null"));
```

**第四种 整合整合 hystrix 会在后期SpringCloud课程中详细讲解**

# 第四部分：Dubbo源码分析

Dubbo的整体设计    服务注册与发现的源码剖析  Dubbo扩展SPI分析  集群容器的源码分析   网络通信原理



**源码下载和编译**

源码下载、编译和导入步骤如下:
 (1)dubbo的项目在github中的地址为: https://github.com/apache/dubbo

 (2)进入需要进行下载的地址，执行 git clone https://github.com/apache/dubbo.git 

(3)为了防止master中代码不稳定，进入dubbo项目 cd dubbo 可以切入到最近的release分支

(4)进行本地编译，进入dubbo项目 cd dubbo , 进行编译操作 mvn clean install -DskipTests

 (5)使用IDE引入项目。

![image-20200714162738522](https://tva1.sinaimg.cn/large/007S8ZIlly1ggqkydp3qpj30h20c8aeu.jpg)

![image-20200714162813977](https://tva1.sinaimg.cn/large/007S8ZIlly1ggqkyzxamgj30ng0lc7i9.jpg)

![image-20200714162832026](https://tva1.sinaimg.cn/large/007S8ZIlly1ggqkzbelolj30n00icat4.jpg)

![image-20200714162852063](https://tva1.sinaimg.cn/large/007S8ZIlly1ggqkznihkbj30ks0bo0yf.jpg)

