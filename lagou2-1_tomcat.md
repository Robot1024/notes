# Tomcat系统架构与原理剖析

1.tomcat 目录结构

![image-20200531223448613](https://tva1.sinaimg.cn/large/007S8ZIlly1gfc0ao4gvzj30c00fgta5.jpg)

bin:命令文件，conf:配置文件server.xml ,web.xml 。lib 依赖包。temp 临时文件。logs 存放日志的。webapps 存放项目的。work jsp 生成中间文件的。

![image-20200601030013838](https://tva1.sinaimg.cn/large/007S8ZIlly1gfc7yrlkdnj314u0osgrz.jpg)

tomcat 是一个http 服务器：

![image-20200601030226168](https://tva1.sinaimg.cn/large/007S8ZIlly1gfc812hn2aj315w0rugue.jpg)Tomcat 的两个重要身份：

1.HTTP 服务器

2.是一个Servlet容器

![image-20200601032721411](https://tva1.sinaimg.cn/large/007S8ZIlly1gfc8qzqwnxj314k0lmq5w.jpg)

![image-20200601034525180](https://tva1.sinaimg.cn/large/007S8ZIlly1gfc99si2mdj31640puk00.jpg)



![image-20200601034803313](https://tva1.sinaimg.cn/large/007S8ZIlly1gfc9cj8qs6j316e0do790.jpg)



![image-20200601034818389](https://tva1.sinaimg.cn/large/007S8ZIlly1gfc9csg9znj311s0nsgnn.jpg)



Coyote 连接器：

Coyote 是Tomcat 中连接器的组件名称 , 是对外的接口。客户端通过Coyote与服务器建立连接、发送请 求并接受响应 。 

(1)Coyote 封装了底层的网络通信(Socket 请求及响应处理)
 (2)Coyote 使Catalina 容器(容器组件)与具体的请求协议及IO操作方式完全解耦 

(3)Coyote 将Socket 输入转换封装为 Request 对象，进一步封装后交由Catalina 容器进行处理，处 理请求完成后, Catalina 通过Coyote 提供的Response 对象将结果写入输出流 

(4)Coyote 负责的是具体协议(应用层)和IO(传输层)相关内容 

![image-20200603184841071](https://tva1.sinaimg.cn/large/007S8ZIlly1gffamct0v2j316w0qcmzq.jpg)



Tomcat Coyote 支持的 IO模型与协议
Tomcat支持多种应用层协议和I/O模型，如下:

![image-20200603184925963](https://tva1.sinaimg.cn/large/007S8ZIlly1gffan3e5l0j31aa0ay440.jpg)

![image-20200603184941904](https://tva1.sinaimg.cn/large/007S8ZIlly1gffanddsykj315q0a0qa5.jpg)

在 8.0 之前 ，Tomcat 默认采用的I/O方式为 BIO，之后改为 NIO。 无论 NIO、NIO2 还是 APR， 在性
能方面均优于以往的BIO。 如果采用APR， 甚至可以达到 Apache HTTP Server 的影响性能。



| 组件            | 作用描述                                                     |
| --------------- | ------------------------------------------------------------ |
| EndPoint        | EndPoint 是 Coyote 通信端点，即通信监听的接口，是具体Socket接收和发 送处理器，是对传输层的抽象，因此EndPoint用来实现TCP/IP协议的 |
| Processor       | Processor 是Coyote 协议处理接口 ，如果说EndPoint是用来实现TCP/IP协 议的，那么Processor用来实现HTTP协议，Processor接收来自EndPoint的 Socket，读取字节流解析成Tomcat Request和Response对象，并通过 Adapter将其提交到容器处理，Processor是对应用层协议的抽象 |
| ProtocolHandler | Coyote 协议接口， 通过Endpoint 和 Processor ， 实现针对具体协议的处 理能力。Tomcat 按照协议和I/O 提供了6个实现类 : AjpNioProtocol ， AjpAprProtocol， AjpNio2Protocol ， Http11NioProtocol ， Http11Nio2Protocol ，Http11AprProtocol |
| Adapter         | 由于协议不同，客户端发过来的请求信息也不尽相同，Tomcat定义了自己的 Request类来封装这些请求信息。ProtocolHandler接口负责解析请求并生成 Tomcat Request类。但是这个Request对象不是标准的ServletRequest，不 能用Tomcat Request作为参数来调用容器。Tomcat设计者的解决方案是引 入CoyoteAdapter，这是适配器模式的经典运用，连接器调用 CoyoteAdapter的Sevice方法，传入的是Tomcat Request对象， CoyoteAdapter负责将Tomcat Request转成ServletRequest，再调用容器 |



Tomcat Servlet 容器 Catalina 4.1 **Tomcat 模块分层结构图及Catalina位置** 

Tomcat是一个由一系列可配置(conf/server.xml)的组件构成的Web容器，而Catalina是Tomcat的 servlet容器。 

从另一个⻆度来说，Tomcat 本质上就是一款 Servlet 容器， 因为 Catalina 才是 Tomcat 的核心 ， 其 他模块都是为Catalina 提供支撑的。 比如 : 通过 Coyote 模块提供链接通信，Jasper 模块提供 JSP 引 擎，Naming 提供JNDI 服务，Juli 提供日志服务。 



**Catalina 是tomcat 的核心。**

![image-20200603185129937](https://tva1.sinaimg.cn/large/007S8ZIlly1gffap8nyvvj318m0u079l.jpg)



其实，可以认为整个Tomcat就是一个Catalina实例，Tomcat 启动的时候会初始化这个实例，Catalina 实例通过加载server.xml完成其他实例的创建，创建并管理一个Server，Server创建并管理多个服务， 每个服务又可以有多个Connector和一个Container。 

一个Catalina实例(容器)
 一个 Server实例(容器)
 多个Service实例(容器) 每一个Service实例下可以有多个Connector实例和一个Container实例 



Container 组件的具体结构

 Container组件下有几种具体的组件，分别是Engine、Host、Context和Wrapper。这4种组件(容器) 

是父子关系。Tomcat通过一种分层的架构，使得Servlet容器具有很好的灵活性。 

**Engine** 

表示整个Catalina的Servlet引擎，用来管理多个虚拟站点，一个Service最多只能有一个Engine， 但是一个引擎可包含多个Host 

**Host** 

代表一个虚拟主机，或者说一个站点，可以给Tomcat配置多个虚拟主机地址，而一个虚拟主机下 

可包含多个Context
 **Context**
 表示一个Web应用程序， 一个Web应用可包含多个Wrapper

 **Wrapper**
 表示一个Servlet，Wrapper 作为容器中的最底层，不能包含子容器 上述组件的配置其实就体现在conf/server.xml中。 



# 第二部分 Tomcat 服务器核心配置详解

conf/server.xml文件  的配置

**主要标签结构如下:**

```xml
<!--
 Server 根元素，创建一个Server实例，子标签有 Listener、GlobalNamingResources、 

Service
-->
<Server>

<!--定义监听器-->
 <Listener/> <!--定义服务器的全局JNDI资源 --> <GlobalNamingResources/> <!-- 

定义一个Service服务，一个Server标签可以有多个Service服务实例 --> 

  <Service/>
</Server>
```

**Server 标签**

**Service 标签**

**Executor 标签**

**Connector 标签** 

**可以使用共享线程池**

**Engine 标签** 

**Host 标签** 

**Context 标签** 

# 第三部分 手写实现迷你版 Tomcat

# 第四部分 Tomcat 源码构建及核心流程源码剖析



核心流程源码剖析 

源码追踪部分我们关注两个流程:Tomcat启动流程和Tomcat请求处理流程 

Tomcat启动流程 

![image-20200603185929178](https://tva1.sinaimg.cn/large/007S8ZIlly1gffaxqvhegj31a40qm0yz.jpg)

![image-20200603185947638](https://tva1.sinaimg.cn/large/007S8ZIlly1gffay25z2cj31a00r8dpb.jpg)

请求处理流程示意图:

![image-20200603190044697](https://tva1.sinaimg.cn/large/007S8ZIlly1gffaz1xxtrj30vh0u0n0w.jpg)



![image-20200603190104100](https://tva1.sinaimg.cn/large/007S8ZIlly1gffazdl6v4j31900o2jun.jpg)



# 第五部分 Tomcat 类加载机制剖析

Java类(.java)—> 字节码文件(.class) —> 字节码文件需要被加载到jvm内存当中(这个过程就是一个 类加载的过程) 

类加载器(ClassLoader，说白了也是一个类，jvm启动的时候先把类加载器读取到内存当中去，其他的 类(比如各种jar中的字节码文件，自己开发的代码编译之后的.class文件等等)) 

要说 Tomcat 的类加载机制，首先需要来看看 Jvm 的类加载机制，因为 Tomcat 类加载机制是在 Jvm 类 加载机制基础之上进行了一些变动。 

**第 1 节 JVM 的类加载机制** 

JVM 的类加载机制中有一个非常重要的⻆色叫做类加载器(ClassLoader)，类加载器有自己的体系， Jvm内置了几种类加载器，包括:引导类加载器、扩展类加载器、系统类加载器，他们之间形成父子关 系，通过 Parent 属性来定义这种关系，最终可以形成树形结构。 

![image-20200603190241897](https://tva1.sinaimg.cn/large/007S8ZIlly1gffb13odkhj311c0u00zi.jpg)



| 类加载器                                                     | 作用                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 引导启动类加载器  								BootstrapClassLoader | c++编写，加载java核心库 java.*,比如rt.jar中的类，构 造ExtClassLoader和AppClassLoader |
| 扩展类加载器 ExtClassLoader                                  | java编写，加载扩展库 JAVA_HOME/lib/ext目录下的jar 中的类，如classpath中的jre ，javax.*或者java.ext.dir 指定位置中的类 |
| 系统类加载器  								SystemClassLoader/AppClassLoader | 默认的类加载器，搜索环境变量 classpath 中指明的路 径         |

另外:用户可以自定义类加载器(Java编写，用户自定义的类加载器，可加载指定路径的 class 文件)   当 JVM 运行过程中，用户自定义了类加载器去加载某些类时，会按照下面的步骤(父类委托机制) 

  1) 用户自己的类加载器，把加载请求传给父加载器，父加载器再传给其父加载器，一直到加载器 树的顶层 

  2 )最顶层的类加载器首先针对其特定的位置加载，如果加载不到就转交给子类
   3 )如果一直到底层的类加载都没有加载到，那么就会抛出异常 ClassNotFoundException 

  因此，按照这个过程可以想到，如果同样在 classpath 指定的目录中和自己工作目录中存放相同的 class，会优先加载 classpath 目录中的文件 

**第 2 节 双亲委派机制** 

2.1 什么是双亲委派机制 当某个类加载器需要加载某个.class文件时，它首先把这个任务委托给他的上级类加载器，递归这个操 

作，如果上级的类加载器没有加载，自己才会去加载这个类。

2.2 双亲委派机制的作用 防止重复加载同一个.class。通过委托去向上面问一问，加载过了，就不用再加载一遍。保证数据 

安全。 

保证核心.class不能被篡改。通过委托方式，不会去篡改核心.class，即使篡改也不会去加载，即使 加载也不会是同一个.class对象了。不同的加载器加载同一个.class也不是同一个.class对象。这样 保证了class执行安全(如果子类加载器先加载，那么我们可以写一些与java.lang包中基础类同名 的类， 然后再定义一个子类加载器，这样整个应用使用的基础类就都变成我们自己定义的类了。 ) 

Object类 -----> 自定义类加载器(会出现问题的，那么真正的Object类就可能被篡改了) 

**第 3 节 Tomcat 的类加载机制** 

Tomcat 的类加载机制相对于 Jvm 的类加载机制做了一些改变。

 没有严格的遵从双亲委派机制，也可以说打破了双亲委派机制

 比如:有一个tomcat，webapps下部署了两个应用 

app1/lib/a-1.0.jar com.lagou.edu.Abc 

app2/lib/a-2.0.jar com.lagou.edu.Abc 

不同版本中Abc类的内容是不同的，代码是不一样的 

![image-20200603190448066](https://tva1.sinaimg.cn/large/007S8ZIlly1gffb39ho1sj30tm0t0wi7.jpg)



引导类加载器 和 扩展类加载器 的作用不变 

系统类加载器正常情况下加载的是 CLASSPATH 下的类，但是 Tomcat 的启动脚本并未使用该变 量，而是加载tomcat启动的类，比如bootstrap.jar，通常在catalina.bat或者catalina.sh中指定。 位于CATALINA_HOME/bin下 

Common 通用类加载器加载Tomcat使用以及应用通用的一些类，位于CATALINA_HOME/lib下， 比如servlet-api.jar 

Catalina ClassLoader 用于加载服务器内部可⻅类，这些类应用程序不能访问 

Shared ClassLoader 用于加载应用程序共享类，这些类服务器不会依赖 

Webapp ClassLoader，每个应用程序都会有一个独一无二的Webapp ClassLoader，他用来加载 本应用程序 /WEB-INF/classes 和 /WEB-INF/lib 下的类。 

tomcat 8.5 默认改变了严格的双亲委派机制 

首先从 Bootstrap Classloader加载指定的类 

如果未加载到，则从 /WEB-INF/classes加载 

如果未加载到，则从 /WEB-INF/lib/*.jar 加载 

如果未加载到，则依次从 System、Common、Shared 加载(在这最后一步，遵从双亲委派 机制) 

# 第六部分 Tomcat 对 Https 的支持及 Tomcat 性能优化策略





# Nginx课程笔记

# 第一部分 Nginx基础回顾

**第一部分:Nginx基础回顾(Nginx是什么?能做什么事情(应用在什么场合)?常用命令是什么?)**

 **第二部分:Nginx核心配置文件解读**
 **第三部分:Nginx应用场景之反向代理**
 **第四部分:Nginx应用场景之负载均衡** 

**第五部分:Nginx应用场景之动静分离**
 **第六部分:Nginx底层进程机制剖析 Nginx源代码是使用C语言开发的，所以呢我们不会再去追踪分析它的源代码了。** 

# 第二部分 Nginx核心配置文件解读

Nginx的核心配置文件conf/nginx.conf包含三块内容:全局块、events块、http块 

**全局块** 

从配置文件开始到events块之间的内容，此处的配置影响nginx服务器整体的运行，比如worker进 程的数量、错误日志的位置等

**events块**
 events块主要影响nginx服务器与用户的网络连接，比如worker_connections 1024，标识每个 

workderprocess支持的最大连接数为1024  

**http块**

http块是配置最频繁的部分，虚拟主机的配置，监听端口的配置，请求转发、反向代理、负载均衡
等

# 第三部分 Nginx应用场景之反向代理

# 第四部分 Nginx应用场景之负载均衡

# 第五部分 Nginx应用场景之动静分离

# 第六部分 Nginx底层进程机制剖析

![image-20200603191128883](https://tva1.sinaimg.cn/large/007S8ZIlly1gffba7v020j31540u0dpj.jpg)



![image-20200603191144329](https://tva1.sinaimg.cn/large/007S8ZIlly1gffbakfl9tj314c0tcjwl.jpg)



![image-20200603191211293](https://tva1.sinaimg.cn/large/007S8ZIlly1gffbaykr02j31440mg494.jpg)

![image-20200603191245208](https://tva1.sinaimg.cn/large/007S8ZIlly1gffbbjut57j313u0p27cv.jpg)

