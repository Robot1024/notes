# **Spring Cloud** 微服务课程笔记

SOA—>Dubbo （自建体系，定义协议的面向服务的架构，也是为了满足服务的远程调用，服务独立于硬件，操作系统，编程语言，比如web Service）
 **微**服务架构—>Spring Cloud提供了一个一站式的微服务解决方案 （Spring Cloud 也是SOA 的一种实现，不同在于两点，1.拆分的更细，是一站式的解决方案，提供了周边的组件，以spring boot 为开发基础的。2.不需要自定义协议，是在原有的http的基础上实现的，也可以使用自定义协议）

## **主要课程内容**

- 第一部分:微服务架构(回顾)
  - 单体 
  - 垂直
  -  SOA
  -  微服务
- 第二部分:SpringCloud概述
  - 是什么
  - 解决什么问题
  -  架构及其核心组件(框架)
  -  Spring Cloud与Dubbo的对比
  -  Spring Cloud与Spring Boot的对比
- 第三部分:案例准备

- 第四部分:第一代 Spring Cloud 核心组件 
- 第五部分:常⻅问题及解决方案 
- 第六部分:Spring Cloud 高级进阶
  - 分布式链路追踪技术(Sleuth+Zipkin)
  - 统一认证(Spring Cloud OAuth2 + Jwt)

- 第七部分:第二代 Spring Cloud 核心组件(Spring Cloud Alibaba)

## 第一部分 微服务架构

**微服务应用架构**

微服务架构可以说是SOA架构的一种拓展，这种架构模式下它拆分粒度更小、服务更独立。把应用拆分 成为一个个微小的服务，不同的服务可以使用不同的开发语言和存储，服务之间往往通过Restful等轻量 级通信。微服务架构关键在于**微小、独立、轻量级通信**。

微服务是在 SOA 上做的升华粒度更加细致，微服务架构强调的一个重点是**“业务需要彻底的组件化 和服 务化”**



**微服务架构中的一些概念**

- 服务注册与服务发现

- 负载均衡

- 熔断

- 链路追踪

- **API** 网关

  微服务架构下，不同的微服务往往会有不同的访问地址，客户端可能需要调用多个服务的接口才能完成

  一个业务需求，如果让客户端直接与各个微服务通信可能出现: 

  ​	1)客户端需要调用不同的url地址，增加了维护调用难度

  ​	2)在一定的场景下，也存在跨域请求的问题(前后端分离就会碰到跨域问题，原本我们在后端采用 Cors就能解决，现在利用网关，那么就放在网关这层做好了)

  ​	3)每个微服务都需要进行单独的身份认证

  那么，API网关就可以较好的统一处理上述问题，API请求调用统一接入API网关层，由网关转发请求。 API网关更专注在安全、路由、流量等问题的处理上(微服务团队专注于处理业务逻辑即可)，它的功 能比如

  ​	1)统一接入(路由) 

  ​	2)安全防护(统一鉴权，负责网关访问身份认证验证，与“访问认证中心”通信，实际认证业务逻辑交移

  “访问认证中心”处理) 

  ​	3)黑白名单(实现通过IP地址控制禁止访问网关功能，控制访问) 

  ​	4)协议适配(实现通信协议校验、适配转换的功能) 

  ​	5)流量管控(限流)
  ​    6)⻓短链接支持
  ​    7)容错能力(负载均衡)

## 第二部分 **Spring Cloud** 综述

### **第 **1节 Spring Cloud 是什么

**[**百度百科**]**Spring Cloud是一系列框架的有序集合。它利用Spring Boot的开发便利性巧妙地简化了分布 式系统基础设施的开发，如服务发现注册、配置中心、消息总线、负载均衡、断路器、数据监控等，都 可以用 Spring Boot的开发⻛格做到一键启动和部署。**Spring Cloud并没有重复制造轮子，它只是将目 前各家公司开发的比较成熟、经得起实际考验的服务框架组合起来，通过Spring Boot⻛格进行再封装 屏蔽掉了复杂的配置和实现原理，最终给开发者留出了一套简单易懂、易部署和易维护的分布式系统开 发工具包**。

Spring Cloud是一系列框架的有序集合(Spring Cloud是一个规范) 

开发服务发现注册、配置中心、消息总线、负载均衡、断路器、数据监控等 

利用Spring Boot的开发便利性简化了微服务架构的开发(自动装配)

> 这里，我们需要注意，Spring Cloud其实是一套规范，是一套用于构建微服务架构的规范，而不是一个 可以拿来即用的框架(所谓规范就是应该有哪些功能组件，然后组件之间怎么配合，共同完成什么事 情)。在这个规范之下第三方的Netflix公司开发了一些组件、Spring官方开发了一些框架/组件，包括 第三方的阿里巴巴开发了一套框架/组件集合Spring Cloud Alibaba，这些才是Spring Cloud规范的实 现。

Netflix搞了一套 简称SCN

Spring Cloud 吸收了Netflix公司的产品基础之上自己也搞了几个组件

阿里巴巴在之前的基础上搞出了一堆微服务组件,Spring Cloud Alibaba(SCA)

### **第 **2 节Spring Cloud 解决什么问题

Spring Cloud 规范及实现意图要解决的问题其实就是微服务架构实施过程中存在的一些问题，比如微服 务架构中的服务注册发现问题、网络问题(比如熔断场景)、统一认证安全授权问题、负载均衡问题、 链路追踪等问题。

### 第 **3** 节 **Spring Cloud** 架构

如前所述，Spring Cloud是一个微服务相关规范，这个规范意图为搭建微服务架构提供一站式服务，采 用组件(框架)化机制定义一系列组件，各类组件针对性的处理微服务中的特定问题，这些组件共同来 构成**Spring Cloud微服务技术栈。**

#### **3.1 Spring Cloud 核心组件**

Spring Cloud 生态圈中的组件，按照发展可以分为第一代 Spring Cloud组件和第二代 Spring Cloud组 件。

|                 | 第一代 **Spring Cloud**(Netflix**， **SCN)            | 第二代 **Spring Cloud**(主要就是**Spring Cloud Alibaba**，**SCA**) |
| --------------- | ----------------------------------------------------- | ------------------------------------------------------------ |
| 注册中心        | Netflix Eureka                                        | 阿里巴巴 Nacos                                               |
| 客户端负 载均衡 | Netflix Ribbon                                        | 阿里巴巴 Dubbo LB、Spring Cloud Loadbalancer                 |
| 熔断器          | Netflix Hystrix                                       | 阿里巴巴 Sentinel                                            |
| 网关            | Netflix Zuul:性能一般，未来将退出 Spring Cloud 生态圈 | 官方 **Spring Cloud Gateway**                                |
| 配置中心        | 官方 Spring Cloud Config                              | 阿里巴巴 Nacos、携程 Apollo                                  |
| 服务调用        | Netflix Feign                                         | 阿里巴巴 Dubbo RPC                                           |
| 消息驱动        | 官方 **Spring Cloud Stream**                          |                                                              |
| 链路追踪        | 官方 **Spring Cloud Sleuth/Zipkin**                   |                                                              |
|                 |                                                       | 阿里巴巴 **seata** 分布式事务方案                            |

![image-20200717200745786](https://tva1.sinaimg.cn/large/007S8ZIlly1ggu86dphl4j312u0ncgxj.jpg)

#### **3.2 Spring Cloud** 体系结构(组件协同工作机制)

### 第 **4** 节 **Spring Cloud** 与 **Dubbo** 对比

Dubbo是阿里巴巴公司开源的一个高性能优秀的服务框架，基于RPC调用，对于目前使用率较高的 Spring Cloud Netflix来说，它是基于HTTP的，所以效率上没有Dubbo高，但问题在于Dubbo体系的组 件不全，不能够提供一站式解决方案，比如服务注册与发现需要借助于Zookeeper等实现，而Spring Cloud Netflix则是真正的提供了一站式服务化解决方案，且有Spring大家族背景。

前些年，Dubbo使用率高于SpringCloud，但目前Spring Cloud在服务化/微服务解决方案中已经有了非 常好的发展趋势。

### 第 **5** 节 **Spring Cloud** 与 **Spring Boot** 的关系

Spring Cloud 只是利用了Spring Boot 的特点，让我们能够快速的实现微服务组件开发，否则不使用 Spring Boot的话，我们在使用Spring Cloud时，每一个组件的相关Jar包都需要我们自己导入配置以及 需要开发人员考虑兼容性等各种情况。所以Spring Boot是我们快速把Spring Cloud微服务技术应用起 来的一种方式。	

## 第三部分 案例准备

**案例代码问题分析**

我们在自动投递微服务中使用RestTemplate调用简历微服务的简历状态接口时(Restful API 接口)。

在微服务分布式集群环境下会存在什么问题呢?怎么解决?
 存在的问题:
 1)在服务消费者中，我们把url地址硬编码到代码中，不方便后期维护。 

 2)服务提供者只有一个服务，即便服务提供者形成集群，服务消费者还需要自己实现负载均衡。 

 3)在服务消费者中，不清楚服务提供者的状态。 

 4)服务消费者调用服务提供者时候，如果出现故障能否及时发现不向用户抛出异常⻚面? 

 5)RestTemplate这种请求调用方式是否还有优化空间?能不能类似于Dubbo那样玩?

 6)这么多的微服务统一认证如何实现? 7)配置文件每次都修改好多个很麻烦!?

 8).... 上述分析出的问题。

其实就是微服务架构中必然面临的一些问题: 

1)服务管理:自动注册与发现、状态监管

2)服务负载均衡

3)熔断

4)远程过程调用

5)网关拦截、路由转发

6)统一认证

7)集中式配置管理，配置信息实时自动更新

这些问题，Spring Cloud 体系都有解决方案，后续我们会逐个学习。

## 第四部分 第一代 **Spring Cloud** 核心组件

说明:上面提到网关组件Zuul性能一般，未来将退出Spring Cloud 生态圈，所以我们直接讲解

GateWay，在课程章节规划时，我们就把GateWay划分到第一代Spring Cloud 核心组件这一部分了。 各组件整体结构如下:

![image-20200717210819844](https://tva1.sinaimg.cn/large/007S8ZIlly1ggu9xcr80tj319a0no77m.jpg)

从形式上来说，Feign一个顶三，Feign = RestTemplate + Ribbon + Hystrix 

### 第 **1** 节 **Eureka**服务注册中心

#### **1.1** 关于服务注册中心

对于任何一个微服务，原则上都应存在或者支持多个提供者(比如简历微服务部署多个实例)，这是由 微服务的**分布式属性**决定的。

更进一步，为了支持弹性扩缩容特性，一个微服务的提供者的数量和分布往往是动态变化的，也是无法 预先确定的。因此，原本在单体应用阶段常用的静态LB机制就不再适用了，需要引入额外的组件来管理 微服务提供者的注册与发现，而这个组件就是服务注册中心。

##### **1.1.1** 服务注册中心一般原理

![image-20200717213138772](https://tva1.sinaimg.cn/large/007S8ZIlly1ggualmi29wj31980mcn1u.jpg)

![image-20200717213200506](https://tva1.sinaimg.cn/large/007S8ZIlly1ggualzz3qxj317i0o4qay.jpg)

分布式微服务架构中，服务注册中心用于存储服务提供者地址信息、服务发布相关的属性信息，消费者 通过主动查询和被动通知的方式获取服务提供者的地址信息，而不再需要通过硬编码方式得到提供者的 地址信息。消费者只需要知道当前系统发布了那些服务，而不需要知道服务具体存在于什么位置，这就 是透明化路由。

1)服务提供者启动

2)服务提供者将相关服务信息主动注册到注册中心

3)服务消费者获取服务注册信息:

poll模式:服务消费者可以主动拉取可用的服务提供者清单

push模式:服务消费者订阅服务(当服务提供者有变化时，注册中心也会主动推送更新后的服务清单给 消费者

4)服务消费者直接调用服务提供者 另外，注册中心也需要完成服务提供者的健康监控，当发现服务提供者失效时需要及时剔除;

##### **1.1.2 主流服务中心对比**

**Zookeeper**

Zookeeper它是一个分布式服务框架，是Apache Hadoop 的一个子项目，它主要是用来解决分布 式应 用中经常遇到的一些数据管理问题，如:统一命名服务、状态同步服务、集群管理、分布式 应用配置项的管理等。

简单来说zookeeper本质=存储+监听通知。 znode

Zookeeper 用来做服务注册中心，主要是因为它具有节点变更通知功能，只要客户端监听相关服 务节点，服务节点的所有变更，都能及时的通知到监听客户端，这样作为调用方只要使用 Zookeeper 的客户端就能实现服务节点的订阅和变更通知功能了，非常方便。另外，Zookeeper 可用性也可以，因为只要半数以上的选举节点存活，整个集群就是可用的。3

**Eureka**

由Netflix开源，并被Pivatal集成到SpringCloud体系中，它是基于 RestfulAPI ⻛格开发的服务注册 与发现组件。

**Consul**

Consul是由HashiCorp基于Go语言开发的支持多数据中心分布式高可用的服务发布和注册服务软 件， 采用Raft算法保证服务的一致性，且支持健康检查。

**Nacos**

Nacos是一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。简单来说 Nacos 就是 注册中心 + 配置中心的组合，帮助我们解决微服务开发必会涉及到的服务注册 与发现，服务 配置，服务管理等问题。Nacos 是 Spring Cloud Alibaba 核心组件之一，负责服务注册与发现， 还有配置。

| 组件名    | 语言 | **CAP**                    | 对外暴露接口 |
| --------- | ---- | -------------------------- | ------------ |
| Eureka    | Java | AP(自我保护机制，保证可用) | HTTP         |
| Consul    | Go   | CP                         | HTTP/DNS     |
| Zookeeper | Java | CP                         | 客户端       |
| Nacos     | Java | 支持AP/CP切换              | HTTP         |

P:分区容错性(一定的要满足的) 

C:数据一致性

A:高可用

CAP不可能同时满足三个，要么是AP，要么是CP

#### **1.2** 服务注册中心组件 **Eureka**

服务注册中心的一般原理、对比了主流的服务注册中心方案 

目光聚焦Eureka

![image-20200717220224103](https://tva1.sinaimg.cn/large/007S8ZIlly1ggubhnubphj312z0u0k54.jpg)

Eureka 交互流程及原理 

下图是官网描述的一个架构图

![image-20200717220332238](https://tva1.sinaimg.cn/large/007S8ZIlly1ggublspz0cj30ww0u048b.jpg)



Eureka 包含两个组件:Eureka Server 和 Eureka Client，Eureka Client是一个Java客户端，用于 简化与Eureka Server的交互;Eureka Server提供服务发现的能力，各个微服务启动时，会通过 Eureka Client向Eureka Server 进行注册自己的信息(例如网络信息)，Eureka Server会存储该 服务的信息;

1)图中us-east-1c、us-east-1d，us-east-1e代表不同的区也就是不同的机房

2)图中每一个Eureka Server都是一个集群。

3)图中Application Service作为服务提供者向Eureka Server中注册服务，Eureka Server接受到 注册事件会在集群和分区中进行数据同步，Application Client作为消费端(服务消费者)可以从 Eureka Server中获取到服务注册信息，进行服务调用。

4)微服务启动后，会周期性地向Eureka Server发送心跳(默认周期为30秒)以续约自己的信息 5)Eureka Server在一定时间内没有接收到某个微服务节点的心跳，Eureka Server将会注销该微

服务节点(默认90秒)
 6)每个Eureka Server同时也是Eureka Client，多个Eureka Server之间通过复制的方式完成服务

注册列表的同步

7)Eureka Client会缓存Eureka Server中的信息。即使所有的Eureka Server节点都宕掉，服务消 费者依然可以使用缓存中的信息找到服务提供者

**Eureka通过心跳检测、健康检查和客户端缓存等机制，提高系统的灵活性、可伸缩性和可用性。**



#### 1.3 Eureka应用及高可用集群

1)单实例Eureka Server—>访问管理界面—>Eureka Server集群 

2)服务提供者(简历微服务注册到集群) 

3)服务消费者(自动投递微服务注册到集群/从Eureka Server集群获取服务信息) 

4)完成调用

##### **1.3.1** 搭建单例**Eureka Server**服务注册中心

agou-service-resume 8080-----

lagou-service-autodeliver 8090---- 

lagou-cloud-eureka-server 8761----

基于Maven构建SpringBoot工程，在SpringBoot工程之上搭建EurekaServer服务(lagou-cloud- eureka-server-8761)



注意:在父工程的**pom**文件中手动引入**jaxb**的**jar**，因为**Jdk9**之后默认没有加载该模块， **EurekaServer**使用到，所以需要手动导入，否则**EurekaServer**服务无法启动



- 执行启动类LagouCloudEurekaServerApplication的main函数
- 访问http://127.0.0.1:8761，如果看到如下⻚面(Eureka注册中心后台)，则表明EurekaServer 发布成功

![image-20200717224906724](https://tva1.sinaimg.cn/large/007S8ZIlly1ggucu8afa7j315k0lo79u.jpg)

![image-20200717224954345](https://tva1.sinaimg.cn/large/007S8ZIlly1ggucv23w2ij315i0byq5y.jpg)

![image-20200717225013248](https://tva1.sinaimg.cn/large/007S8ZIlly1ggucvdrp4rj315k0e4dim.jpg)

##### **1.3.2** 搭建**Eureka Server HA**高可用集群

在互联网应用中，服务实例很少有单个的。

即使微服务消费者会缓存服务列表，但是如果EurekaServer只有一个实例，该实例挂掉，正好微服务消 费者本地缓存列表中的服务实例也不可用，那么这个时候整个系统都受影响。

在生产环境中，我们会配置Eureka Server集群实现高可用。Eureka Server集群之中的节点通过点对点 (P2P)通信的方式共享服务注册表。我们开启两台 Eureka Server 以搭建集群。

![image-20200717230028648](https://tva1.sinaimg.cn/large/007S8ZIlly1ggud61ysifj315m0o40yl.jpg)

- 启动类添加注解

  ![image-20200720153208220](https://tva1.sinaimg.cn/large/007S8ZIlly1ggxh2iov4aj313s07qgo5.jpg)				 		

**注意:**

1)从Spring Cloud Edgware版本开始，@EnableDiscoveryClient 或 @EnableEurekaClient 可省 略。只需加 上相关依赖，并进行相应配置，即可将微服务注册到服务发现组件上。

2)@EnableDiscoveryClient和@EnableEurekaClient二者的功能是一样的。但是如果选用的是 eureka服务器，那么就推荐@EnableEurekaClient，如果是其他的注册中心，那么推荐使用 @EnableDiscoveryClient，考虑到通用性，后期我们可以使用@EnableDiscoveryClient

#### **1.4 Eureka**细节详解 

##### **1.4.1 Eureka**元数据详解

Eureka的元数据有两种:标准元数据和自定义元数据。 

**标准元数据**:主机名、IP地址、端口号等信息，这些信息都会被发布在服务注册表中，用于服务之间的

调用。

**自定义元数据**:可以使用eureka.instance.metadata-map配置，符合KEY/VALUE的存储格式。这 些元 数据可以在远程客户端中访问。

类似于

```yaml
instance: 
	prefer-ip-address:
    true metadata-map:
      /#自定义元数据(kv自定义) 
      cluster: cl1
      region: rn1
```

##### 1.4.2 Eureka客户端详解 

服务提供者(也是Eureka客户端)要向EurekaServer注册服务，并完成服务续约等工作

**服务注册详解(服务提供者)**

1)当我们导入了eureka-client依赖坐标，配置Eureka服务注册中心地址 

2)服务在启动时会向注册中心发起注册请求，携带服务元数据信息 

3)Eureka注册中心会把服务的信息保存在Map中。

**服务续约详解(服务提供者)**

服务每隔30秒会向注册中心续约(心跳)一次(也称为报活)，如果没有续约，租约在90秒后到期，然后 服务会被失效。每隔30秒的续约操作我们称之为心跳检测

往往不需要我们调整这两个配置

![image-20200720154839810](https://tva1.sinaimg.cn/large/007S8ZIlly1ggxhjnr23mj315c0c4whb.jpg)

**获取服务列表详解(服务消费者)**

每隔30秒服务会从注册中心中拉取一份服务列表，这个时间可以通过配置修改。往往不需要我们调整

![image-20200720154933666](https://tva1.sinaimg.cn/large/007S8ZIlly1ggxhklfwvej315g07y75j.jpg)

1)服务消费者启动时，从 EurekaServer服务列表获取只读备份，缓存到本地 

2)每隔30秒，会重新获取并更新数据 

3)每隔30秒的时间可以通过配置eureka.client.registry-fetch-interval-seconds修改

##### **1.4.3 Eureka服务端详解**

**服务下线**

1)当服务正常关闭操作时，会发送服务下线的REST请求给EurekaServer。 

2)服务中心接受到请求后，将该服务置为下线状态

**失效剔除**

Eureka Server会定时(间隔值是eureka.server.eviction-interval-timer-in-ms，默认60s)进行检查， 如果发现实例在在一定时间(此值由客户端设置的eureka.instance.lease-expiration-duration-in- seconds定义，默认值为90s)内没有收到心跳，则会注销此实例。

**自我保护**

服务提供者 —> 注册中心

定期的续约(服务提供者和注册中心通信)，假如服务提供者和注册中心之间的网络有点问题，不代表 服务提供者不可用，不代表服务消费者无法访问服务提供者

**如果在15分钟内超过85%的客户端节点**都没有正常的心跳，那么Eureka就认为客户端与注册中心出现了 网络故障，Eureka Server自动进入自我保护机制。

为什么会有自我保护机制?

默认情况下，如果Eureka Server在一定时间内(默认90秒)没有接收到某个微服务实例的心跳， Eureka Server将会移除该实例。但是当网络分区故障发生时，微服务与Eureka Server之间无法正常通 信，而微服务本身是正常运行的，此时不应该移除这个微服务，所以引入了自我保护机制。

服务中心⻚面会显示如下提示信息

![image-20200720155707363](https://tva1.sinaimg.cn/large/007S8ZIlly1ggxhsgrsmdj315405mn2d.jpg)

当处于自我保护模式时

1)不会剔除任何服务实例(可能是服务提供者和EurekaServer之间网络问题)，保证了大多数服务依 然可用

2)Eureka Server仍然能够接受新服务的注册和查询请求，但是不会被同步到其它节点上，保证当前节 点依然可用，当网络稳定时，当前Eureka Server新的注册信息会被同步到其它节点中。

3)在Eureka Server工程中通过eureka.server.enable-self-preservation配置可用关停自我保护，默认 值是打开

![image-20200720155757763](https://tva1.sinaimg.cn/large/007S8ZIlly1ggxhtcgnxoj315a05gq3o.jpg)

经验:建议生产环境打开自我保护机制

#### **1.5 Eureka**核心源码剖析

#####  **1.5.1 Eureka Server**启动过程

入口:SpringCloud充分利用了SpringBoot的自动装配的特点

- 观察eureka-server的jar包，发现在META-INF下面有配置文件spring.factories

  ![image-20200720185949280](https://tva1.sinaimg.cn/large/007S8ZIlly1ggxn2kkwwrj31580hiqdw.jpg)

springboot应用启动时会加载EurekaServerAutoConfiguration自动配置类

- EurekaServerAutoConfiguration类

  首先观察类头分析

![image-20200720190105811](https://tva1.sinaimg.cn/large/007S8ZIlly1ggxn3w7ymaj31540aa7bm.jpg)

图中的 **1**)需要有一个marker bean，才能装配Eureka Server，那么这个marker 其实是由 @EnableEurekaServer注解决定的

![image-20200720190130315](https://tva1.sinaimg.cn/large/007S8ZIlly1ggxn4bgd0zj315w0q4wox.jpg)

### 第 **2** 节 **Ribbon**负载均衡

#### **2.1** 关于负载均衡

负载均衡一般分为**服务器端负载均衡和客户端负载均衡**

所谓**服务器端负载均衡**，比如Nginx、F5这些，请求到达服务器之后由这些负载均衡器根据一定的算法 将请求路由到目标服务器处理。

所谓**客户端负载均衡**，比如我们要说的Ribbon，服务消费者客户端会有一个服务器地址列表，调用方在 请求前通过一定的负载均衡算法选择一个服务器进行访问，负载均衡算法的执行是在请求客户端进行。

Ribbon是Netflix发布的负载均衡器。Eureka一般配合Ribbon进行使用，Ribbon利用从Eureka中读取到 服务信息，在调用服务提供者提供的服务时，会根据一定的算法进行负载。

#### **2.2 Ribbon**高级应用

不需要引入额外的Jar坐标，因为在服务消费者中我们引入过eureka-client，它会引入Ribbon相关Jar

![image-20200720211340428](https://tva1.sinaimg.cn/large/007S8ZIlly1ggxqxufeldj315o0h8ti9.jpg)

代码中使用如下，在RestTemplate上添加对应注解即可

```java
 @Bean
// Ribbon负载均衡
 @LoadBalanced
 public RestTemplate getRestTemplate() {

  return new RestTemplate();
}
```

#### **2.3 Ribbon**负载均衡策略

Ribbon内置了多种负载均衡策略，内部负责复杂均衡的顶级接口为 com.netflix.loadbalancer.IRule ， 类树如下



![image-20200720211505434](https://tva1.sinaimg.cn/large/007S8ZIlly1ggxqzbeqzjj315c0k2tmt.jpg)

![image-20200720211523942](https://tva1.sinaimg.cn/large/007S8ZIlly1ggxqzn7q9bj310b0u07e1.jpg)

修改负载均衡策略

```yaml
#针对的被调用方微服务名称,不加就是全局生效 
lagou-service-resume:
   ribbon:
      NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule 
/#负载策略调整
```

#### **2.4 Ribbon**核心源码剖析

**Ribbon**工作原理

![image-20200720213300722](https://tva1.sinaimg.cn/large/007S8ZIlly1ggxrhyr0ipj315o0n4tg3.jpg)

重点:**Ribbon**给**restTemplate**添加了一个拦截器 

思考:Ribbon在做什么:

当我们访问http://lagou-service-resume/resume/openstate/的时候，ribbon应该根据服务名lagou- service-resume获取到该服务的实例列表并按照一定的负载均衡策略从实例列表中获取一个实例 Server，并最终通过RestTemplate进行请求访问

**Ribbon细节结构图(涉及到底层的一些组件/类的描述)**

1)获取服务实例列表 2)从列表中选择一个server

![image-20200720213542377](https://tva1.sinaimg.cn/large/007S8ZIlly1ggxrkrf1ooj313w0ng42m.jpg)



图中核心是负载均衡管理器**LoadBalancer(总的协调者，相当于大脑，为了做事情，协调四肢)**，围 绕它周围的多有IRule、IPing等

- IRule:是在选择实例的时候的负载均衡策略对象 
- IPing:是用来向服务发起心跳检测的，通过心跳检测来判断该服务是否可用 
- ServerListFilter:根据一些规则过滤传入的服务实例列表 
- ServerListUpdater:定义了一系列的对服务列表的更新操作

##### 2.4.1 @LoadBalanced源码剖析

我们在RestTemplate实例上添加了一个@LoadBalanced注解，就可以实现负载均衡，很神奇，我们接

下来分析这个注解背后的操作(负载均衡过程) 

- 查看@LoadBalanced注解，那这个注解是在哪里被识别到的呢?

  ![image-20200720225213782](https://tva1.sinaimg.cn/large/007S8ZIlly1ggxtse3e7sj317k0doafs.jpg)

  

- LoadBalancerClient类(实现类RibbonLoadBalancerClient，待用)

![image-20200720225356744](https://tva1.sinaimg.cn/large/007S8ZIlly1ggxtu6mujdj316e0todrl.jpg)

**1)研究LoadBalancerAutoConfiguration**

![image-20200720225525934](https://tva1.sinaimg.cn/large/007S8ZIlly1ggxtvpzqluj315i0aytf8.jpg)

第一处:注入resttemplate对象到集合待用

![image-20200720225551227](https://tva1.sinaimg.cn/large/007S8ZIlly1ggxtw5xsl1j315m04qae5.jpg)

第二处:注入resttemplate定制器

![image-20200720225615595](https://tva1.sinaimg.cn/large/007S8ZIlly1ggxtwl9bpmj315q0f610g.jpg)

第三处:使用定制器给集合中的每一个resttemplate对象添加一个拦截器

![image-20200720225654556](https://tva1.sinaimg.cn/large/007S8ZIlly1ggxtx96564j315c0c4wjx.jpg)

![image-20200720225709238](https://tva1.sinaimg.cn/large/007S8ZIlly1ggxtxifbqrj315k09mjtf.jpg)

### 第 **3** 节 **Hystrix**熔断器 

属于一种容错机制

#### **3.1** 微服务中的雪崩效应

 什么是微服务中的雪崩效应呢? 

微服务中，一个请求可能需要多个微服务接口才能实现，会形成复杂的调用链路。

![image-20200720225931953](https://tva1.sinaimg.cn/large/007S8ZIlly1ggxtzzo1f3j315y0gsag3.jpg)

扇入:代表着该微服务被调用的次数，扇入大，说明该模块复用性好

扇出:该微服务调用其他微服务的个数，扇出大，说明业务逻辑复杂 

扇入大是一个好事，扇出大不一定是好事

在微服务架构中，一个应用可能会有多个微服务组成，微服务之间的数据交互通过远程过程调用完成。 这就带来一个问题，假设微服务A调用微服务B和微服务C，微服务B和微服务C又调用其它的微服务，这 就是所谓的“扇出”。如果扇出的链路上某个微服务的调用响应时间过⻓或者不可用，对微服务A的调用就 会占用越来越多的系统资源，进而引起系统崩溃，所谓的“雪崩效应”。

如图中所示，最下游**简历微服务**响应时间过⻓，大量请求阻塞，大量线程不会释放，会导致服务器资源 耗尽，最终导致上游服务甚至整个系统瘫痪。

#### **3.2** 雪崩效应解决方案

从可用性可靠性着想，为防止系统的整体缓慢甚至崩溃，采用的技术手段;

下面，我们介绍三种技术手段应对微服务中的雪崩效应，这三种手段都是从系统可用性、可靠性⻆度出 发，尽量防止系统整体缓慢甚至瘫痪。

**服务熔断**

熔断机制是应对雪崩效应的一种微服务链路保护机制。我们在各种场景下都会接触到熔断这两个字。高 压电路中，如果某个地方的电压过高，熔断器就会熔断，对电路进行保护。股票交易中，如果股票指数 过高，也会采用熔断机制，暂停股票的交易。同样，在微服务架构中，熔断机制也是起着类似的作用。 当扇出链路的某个微服务不可用或者响应时间太⻓时，熔断该节点微服务的调用，进行服务的降级，快 速返回错误的响应信息。当检测到该节点微服务调用响应正常后，恢复调用链路。

**注意:**

1)服务熔断重点在**“断”**，切断对下游服务的调用 

2)服务熔断和服务降级往往是一起使用的，Hystrix就是这样。

**服务降级**

通俗讲就是整体资源不够用了，先将一些不关紧的服务停掉(调用我的时候，给你返回一个预留的值， 也叫做兜底数据)，待渡过难关高峰过去，再把那些服务打开。

服务降级一般是从整体考虑，就是当某个服务熔断之后，服务器将不再被调用，此刻客户端可以自己准 备一个本地的fallback回调，返回一个缺省值，这样做，虽然服务水平下降，但好歹可用，比直接挂掉 要强。

**服务限流**

服务降级是当服务出问题或者影响到核心流程的性能时，暂时将服务屏蔽掉，待高峰或者问题解决后再 打开;但是有些场景并不能用服务降级来解决，比如秒杀业务这样的核心功能，这个时候可以结合服务 限流来限制这些场景的并发/请求量

限流措施也很多，比如

- 限制总并发数(比如数据库连接池、线程池)
- 限制瞬时并发数(如nginx限制瞬时并发连接数)
- 限制时间窗口内的平均速率(如Guava的RateLimiter、nginx的limit_req模块，限制每秒的平均速 率)
- 限制远程接口调用速率、限制MQ的消费速率等

#### **3.3 Hystrix**简介

**[来自官网]**Hystrix(豪猪----->刺)，宣言“defend your app”是由Netflix开源的一个延迟和容错库，用 于隔离访问远程系统、服务或者第三方库，防止级联失败，从而提升系统的可用性与容错性。Hystrix主 要通过以下几点实现延迟和容错。

- 包裹请求:使用HystrixCommand包裹对依赖的调用逻辑。 自动投递微服务方法 (@HystrixCommand 添加Hystrix控制) ——调用简历微服务 
- 跳闸机制:当某服务的错误率超过一定的阈值时，Hystrix可以跳闸，停止请求该服务一段时间。 
- 资源隔离:Hystrix为每个依赖都维护了一个小型的线程池(舱壁模式)(或者信号量)。如果该线程池已满， 发往该依赖的请求就被立即拒绝，而不是排队等待，从而加速失败判定。 
- 监控:Hystrix可以近乎实时地监控运行指标和配置的变化，例如成功、失败、超时、以及被拒绝 的请求等。 
- 回退机制:当请求失败、超时、被拒绝，或当断路器打开时，执行回退逻辑。回退逻辑由开发人员 自行提供，例如返回一个缺省值。
- 自我修复:断路器打开一段时间后，会自动进入“半开”状态。

#### **3.4 Hystrix**熔断应用

目的:简历微服务⻓时间没有响应，服务消费者—>**自动投递微服务**快速失败给用户提示

![image-20200720232204351](https://tva1.sinaimg.cn/large/007S8ZIlly1ggxunfnph7j31480my7a5.jpg)

#### **3.5 Hystrix**舱壁模式(线程池隔离策略)

![image-20200720234950992](https://tva1.sinaimg.cn/large/007S8ZIlly1ggxvgc5vlkj31480qegsw.jpg)

如果不进行任何设置，所有熔断方法使用一个Hystrix线程池(10个线程)，那么这样的话会导致问 题，这个问题并不是扇出链路微服务不可用导致的，而是我们的线程机制导致的，如果方法A的请求把 10个线程都用了，方法2请求处理的时候压根都没法去访问B，因为没有线程可用，并不是B服务不可 用。

![image-20200720235044978](https://tva1.sinaimg.cn/large/007S8ZIlly1ggxvha9gusj312k0sqgqg.jpg)



为了避免问题服务请求过多导致正常服务无法访问，Hystrix 不是采用增加线程数，而是单独的为每一 个控制方法创建一个线程池的方式，这种模式叫做“舱壁模式"，也是线程隔离的手段。

**我们可以使用一些手段查看线程情况**

![image-20200721122647257](https://tva1.sinaimg.cn/large/007S8ZIlly1ggyhbx92asj31560fqn5j.jpg)

发起请求，可以使用PostMan模拟批量请求

![image-20200721122721408](https://tva1.sinaimg.cn/large/007S8ZIlly1ggyhciva8zj31580f4aky.jpg)

#### **3.6 Hystrix**工作流程与高级应用

![image-20200721125249559](https://tva1.sinaimg.cn/large/007S8ZIlly1ggyi32xr7hj313m0najw4.jpg)

1)当调用出现问题时，开启一个时间窗(10s) 

2)在这个时间窗内，统计调用次数是否达到最小请求数?

 如果没有达到，则重置统计信息，回到第1步 

如果达到了，则统计失败的请求数占所有请求数的百分比，是否达到阈值? 

如果达到，则跳闸(不再请求对应服务) 

如果没有达到，则重置统计信息，回到第1步

3)如果跳闸，则会开启一个活动窗口(默认5s)，每隔5s，Hystrix会让一个请求通过,到达那个问题服 务，看 是否调用成功，如果成功，重置断路器回到第1步，如果失败，回到第3步

**基于springboot的健康检查观察跳闸状态(自动投递微服务暴露健康检查细节)**

```yaml
# springboot中暴露健康检查等断点接口 
management:
  endpoints:
    web:
exposure: 
  include: "*"

\# 暴露健康接口的细节 
endpoint:
  health:
    show-details: always
```

访问健康检查接口:http://localhost:8090/actuator/health

hystrix正常工作状态

![image-20200721130252472](https://tva1.sinaimg.cn/large/007S8ZIlly1ggyidgvwqij30f4058weu.jpg)

跳闸状态

![image-20200721130312003](https://tva1.sinaimg.cn/large/007S8ZIlly1ggyidtjr6sj315w0b4q5d.jpg)

活动窗口内自我修复

![image-20200721130332044](https://tva1.sinaimg.cn/large/007S8ZIlly1ggyie5oztwj30je06q3yx.jpg)

#### **3.7 Hystrix Dashboard**断路监控仪表盘

正常状态是UP，跳闸是一种状态CIRCUIT_OPEN，可以通过/health查看，前提是工程中需要引入 SpringBoot的actuator(健康监控)，它提供了很多监控所需的接口，可以对应用系统进行配置查看、 相关功能统计等。

已经统一添加在父工程中

```xml
<dependency>
 <groupId>org.springframework.boot</groupId>
 <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

如果我们想看到Hystrix相关数据，比如有多少请求、多少成功、多少失败、多少降级等，那么引入 SpringBoot健康监控之后，访问/actuator/hystrix.stream接口可以获取到监控的文字信息，但是不直 观，所以Hystrix官方还提供了基于图形化的DashBoard(仪表板)监控平 台。Hystrix仪表板可以显示 每个断路器(被@HystrixCommand注解的方法)的状态。

![image-20200721131842507](https://tva1.sinaimg.cn/large/007S8ZIlly1ggyitykiq0j31620lm7da.jpg)

1)新建一个监控服务工程，导入依赖

```xml
<!--hystrix-->

<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>

<!--hystrix 仪表盘-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>

<dependency>
	<groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId> 
</dependency>
```

2)启动类添加@EnableHystrixDashboard激活仪表盘

![image-20200721132748381](https://tva1.sinaimg.cn/large/007S8ZIlly1ggyj3f5hgcj315c0pa48h.jpg)

输入监控的微服务端点地址，展示监控的详细数据，比如监控服务消费者http://localhost:8090/actuat or/hystrix.stream

![image-20200721132811744](/Users/leipeng/Library/Application Support/typora-user-images/image-20200721132811744.png)

百分比，10s内错误请求百分比

实心圆:

- 大小:代表请求流量的大小，流量越大球越大

- 颜色:代表请求处理的健康状态，从绿色到红色递减，绿色代表健康，红色就代表很不健康 

曲线波动图:

记录了2分钟内该方法上流量的变化波动图，判断流量上升或者下降的趋势

#### **3.8 Hystrix Turbine**聚合监控

之前，我们针对的是一个微服务实例的Hystrix数据查询分析，在微服务架构下，一个微服务的实例往往 是多个(集群化)

比如自动投递微服务
 实例1(hystrix) ip1:port1/actuator/hystrix.stream
 实例2(hystrix) ip2:port2/actuator/hystrix.stream
 实例3(hystrix) ip3:port3/actuator/hystrix.stream 

按照已有的方法，我们就可以结合dashboard仪表盘每次输入一个监控数据流url，进去查看

手工操作能否被自动功能替代?Hystrix Turbine聚合(聚合各个实例上的hystrix监控数据)监控 

Turbine(涡轮)

思考:微服务架构下，一个微服务往往部署多个实例，如果每次只能查看单个实例的监控，就需要经常 切换很不方便，在这样的场景下，我们可以使用 Hystrix Turbine 进行聚合监控，它可以把相关微服务 的监控数据聚合在一起，便于查看。

![image-20200721134513318](https://tva1.sinaimg.cn/large/007S8ZIlly1ggykon7gwtj31680c6wjf.jpg)

#### **3.8 Hystrix**核心源码剖析

springboot装配、面向切面编程、RxJava响应式编程的知识等等，我们剖析下主体脉络。 

分析入口:@EnableCircuitBreaker注解激活了熔断功能，那么该注解就是Hystrix源码追踪的入口.

- @EnableCircuitBreaker注解激活熔断器

### 第 **4** 节 **Feign**远程调用组件

服务消费者调用服务提供者的时候使用RestTemplate技术

![image-20200721150053668](https://tva1.sinaimg.cn/large/007S8ZIlly1ggylsatuggj319u07kwk2.jpg)

存在不便之处
 1)拼接url 2)restTmplate.getForObJect 

这两处代码都比较模板化，能不能不让我我们来写这种模板化的东⻄ 

另外来说，拼接url非常的low，拼接字符串，拼接参数，很low还容易出错

#### **4.1 Feign**简介

Feign**是Netflix开发的一个轻量级RESTful的HTTP服务客户端(用它来发起请求，远程调用的)**，是以 Java接口注解的方式调用Http请求，而不用像Java中通过封装HTTP请求报文的方式直接调用，Feign被 广泛应用在Spring Cloud 的解决方案中。

类似于Dubbo，服务消费者拿到服务提供者的接口，然后像调用本地接口方法一样去调用，实际发出的 是远程的请求。

- Feign可帮助我们更加便捷，优雅的调用HTTP API:不需要我们去拼接url然后呢调用 restTemplate的api，在SpringCloud中，使用Feign非常简单，创建一个接口(在消费者--服务调 用方这一端)，并在接口上添加一些注解，代码就完成了 

- SpringCloud对Feign进行了增强，使Feign支持了SpringMVC注解(OpenFeign)

**本质:封装了Http调用流程，更符合面向接口化的编程习惯，类似于Dubbo的服务调用**

Dubbo的调用方式其实就是很好的面向接口编程

#### **4.2 Feign**配置应用

在服务调用者工程(消费)创建接口(添加注解)

 (效果)Feign = RestTemplate+Ribbon+Hystrix

- 服务消费者工程(自动投递微服务)中引入Feign依赖(或者父类工程)

```xml
<dependency>
 <groupId>org.springframework.cloud</groupId> 
 <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

- 服务消费者工程(自动投递微服务)启动类使用注解@EnableFeignClients添加Feign支持

**注意:**

1)@FeignClient注解的name属性用于指定要调用的服务提供者名称，和服务提供者yml文件中 spring.application.name保持一致

2)接口中的接口方法，就好比是远程服务提供者Controller中的Hander方法(只不过如同本地调 用了)，那么在进行参数绑定的时，可以使用@PathVariable、@RequestParam、 @RequestHeader等，这也是OpenFeign对SpringMVC注解的支持，但是需要注意value必须设 置，否则会抛出异常

- 使用接口中方法完成远程调用(注入接口即可，实际注入的是接口的实现)

#### **4.3 Feign**对负载均衡的支持

Feign 本身已经集成了Ribbon依赖和自动配置，因此我们不需要额外引入依赖，可以通过 ribbon.xx 来

进 行全局配置,也可以通过服务名.ribbon.xx 来对指定服务进行细节配置配置(参考之前，此处略) Feign默认的请求处理超时时⻓1s，有时候我们的业务确实执行的需要一定时间，那么这个时候，我们

就需要调整请求处理超时时⻓，Feign自己有超时设置，如果配置Ribbon的超时，则会以Ribbon的为准

**Ribbon**设置

\#针对的被调用方微服务名称,不加就是全局生效 lagou-service-resume:

```yaml
ribbon:
 \#请求连接超时时间
 \#ConnectTimeout: 2000
 \#请求处理超时时间
 \#ReadTimeout: 5000
 \#对所有操作都进行重试
 OkToRetryOnAllOperations: true ####根据如上配置，当访问到故障请求的时候，它会再尝试访问一次当前实例(次数由

MaxAutoRetries配置)， ####如果不行，就换一个实例进行访问，如果还不行，再换一次实例访问(更换次数由

MaxAutoRetriesNextServer配置)，
 \####如果依然不行，返回失败信息。
 MaxAutoRetries: 0 #对当前选中实例重试次数，不包括第一次调用 MaxAutoRetriesNextServer: 0 #切换实例的重试次数
 NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RoundRobinRule #负载

策略调整
```

#### **4.4 Feign**对熔断器的支持

1)在Feign客户端工程配置文件(application.yml)中开启Feign对熔断器的支持

```yaml
# 开启Feign的熔断功能 feign:

  hystrix:
    enabled: true
```

Feign的超时时⻓设置那其实就上面Ribbon的超时时⻓设置

Hystrix超时设置(就按照之前Hystrix设置的方式就OK了)

注意:

1)开启Hystrix之后，Feign中的方法都会被进行一个管理了，一旦出现问题就进入对应的回退逻辑处理

2)针对超时这一点，当前有两个超时时间设置(Feign/hystrix)，熔断的时候是根据这两个时间的最 小值来进行的，即处理时⻓超过最短的那个超时时间了就熔断进入回退降级逻辑

3)自定义FallBack处理类(需要实现FeignClient接口)

4)在@FeignClient注解中关联2)中自定义的处理类

#### **4.5 Feign**对请求压缩和响应压缩的支持

Feign 支持对请求和响应进行GZIP压缩，以减少通信过程中的性能损耗。通过下面的参数 即可开启请求

与响应的压缩功能

#### **4.6 Feign**的日志级别配置

Feign是http请求客户端，类似于咱们的浏览器，它在请求和接收响应的时候，可以打印出比较详细的

一些日志信息(响应头，状态码等等) 如果我们想看到Feign请求时的日志，我们可以进行配置，默认情况下Feign的日志没有开启。 

1) 开启Feign日志功能及级别

2) 配置log日志级别为debug

#### **4.7 Feign**核心源码剖析

 思考一个问题:只定义了接口，添加上@FeignClient，真的没有实现的话，能完成远程请求么?

不能，考虑是做了代理了。

### 第 **5** 节 **GateWay**网关组件

网关(翻译过来就叫做GateWay):微服务架构中的重要组成部分

局域网中就有网关这个概念，局域网接收或者发送数据出去通过这个网关，比如用Vmware虚拟机软件 搭建虚拟机集群的时候，往往我们需要选择IP段中的一个IP作为网关地址。

我们学习的GateWay-->Spring Cloud GateWay(它只是众多网关解决方案中的一种)

#### **5.1 GateWay**简介

Spring Cloud GateWay是Spring Cloud的一个全新项目，目标是取代Netflix Zuul，它基于 Spring5.0+SpringBoot2.0+WebFlux(基于高性能的Reactor模式响应式通信框架Netty，异步非阻塞模 型)等技术开发，性能高于Zuul，官方测试，GateWay是Zuul的1.6倍，旨在为微服务架构提供一种简 单有效的统一的API路由管理方式。

Spring Cloud GateWay不仅提供统一的路由方式(反向代理)并且基于 Filter(定义过滤器对请求过滤， 完成一些功能) 链的方式提供了网关基本的功能，例如:鉴权、流量控制、熔断、路径重写、日志监控 等。

![image-20200721215329515](https://tva1.sinaimg.cn/large/007S8ZIlly1ggyxplx87jj314u0u0qhn.jpg)

#### **5.2 GateWay**核心概念

Zuul1.x 阻塞式IO 2.x 基于Netty
Spring Cloud GateWay天生就是异步非阻塞的，基于Reactor模型

一个请求—>网关根据一定的条件匹配—匹配成功之后可以将请求转发到指定的服务地址;而在这个过 程中，我们可以进行一些比较具体的控制(限流、日志、黑白名单)

- 路由(route): 网关最基础的部分，也是网关比较基础的工作单元。路由由一个ID、一个目标 URL(最终路由到的地址)、一系列的断言(匹配条件判断)和Filter过滤器(精细化控制)组 成。如果断言为true，则匹配该路由。

- 断言(predicates):参考了Java8中的断言java.util.function.Predicate，开发人员可以匹配Http 请求中的所有内容(包括请求头、请求参数等)(类似于nginx中的location匹配一样)，如果断 言与请求相匹配则路由。

- 过滤器(filter):一个标准的Spring webFilter，使用过滤器，可以在请求之前或者之后执行业务 逻辑。

![image-20200721215910149](https://tva1.sinaimg.cn/large/007S8ZIlly1ggyxvi6guwj30ym0j8dlx.jpg)

其中，Predicates断言就是我们的匹配条件，而Filter就可以理解为一个无所不能的拦截器，有了 这两个元素，结合目标URL，就可以实现一个具体的路由转发。

#### **5.3 GateWay**工作过程(**How It Works**)

![image-20200721220045927](https://tva1.sinaimg.cn/large/007S8ZIlly1ggyxx5ro94j30mq0uimzr.jpg)

来自官方的描述图

客户端向Spring Cloud GateWay发出请求，然后在GateWay Handler Mapping中找到与请求相匹配的 路由，将其发送到GateWay Web Handler;Handler再通过指定的过滤器链来将请求发送到我们实际的 服务执行业务逻辑，然后返回。过滤器之间用虚线分开是因为过滤器可能会在发送代理请求之前 (pre)或者之后(post)执行业务逻辑。

Filter在“pre”类型过滤器中可以做参数校验、权限校验、流量监控、日志输出、协议转换等，在“post”类 型的过滤器中可以做响应内容、响应头的修改、日志的输出、流量监控等。

**GateWay核心逻辑:路由转发+执行过滤器链**

**5.4 GateWay**应用

使用网关对自动投递微服务进行代理(添加在它的上游，相当于隐藏了具体微服务的信息，对外暴露的 是网关)

- 创建工程lagou-cloud-gateway-server-9002导入依赖

GateWay不需要使用web模块，它引入的是WebFlux(类似于SpringMVC)

**注意:不要引入starter-web模块，需要引入web-flux application.yml 配置文件部分内容**

- application.yml 配置文件部分内容

上面这段配置的意思是，配置了一个 id 为 service-autodeliver-router 的路由规则，当向网关发起 请求 http://localhost:9002/autodeliver/checkAndBegin/1545132，请求会被分发路由到对应的 微服务上

#### **5.4 GateWay**路由规则详解

Spring Cloud GateWay 帮我们内置了很多 Predicates功能，实现了各种路由匹配规则(通过 Header、请求参数等作为条件)匹配到对应的路由。

![image-20200721221738365](https://tva1.sinaimg.cn/large/007S8ZIlly1ggyyepstgxj317u0ky7ao.jpg)

#### **5.5 GateWay**动态路由详解

GateWay支持自动从注册中心中获取服务列表并访问，即所谓的动态路由
 实现步骤如下 

1)pom.xml中添加注册中心客户端依赖(因为要获取注册中心服务列表，eureka客户端已经引入) 

2)动态路由配置

**注意:动态路由设置时，uri以 lb: //开头(lb代表从注册中心获取服务)，后面是需要转发到的服务名 称**

#### **5.6 GateWay**过滤器

##### **5.5.1 GateWay**过滤器简介

从过滤器生命周期(影响时机点)的⻆度来说，主要有两个pre和post:

![image-20200721223148385](https://tva1.sinaimg.cn/large/007S8ZIlly1ggyytfu695j318i0p40x7.jpg)

如Gateway Filter - **StripPrefix**可以去掉url中的占位后转发路由，比如

```yaml
predicates:
 \- Path=/resume/**
filters:
 \- StripPrefix=1 # 可以去掉resume之后转发
```

**注意:GlobalFilter全局过滤器是程序员使用比较多的过滤器，我们主要讲解这种类型**

##### **5.5.2** 自定义全局过滤器实现**IP**访问限制(黑白名单)

请求过来时，判断发送请求的客户端的ip，如果在黑名单中，拒绝访问

自定义GateWay全局过滤器时，我们实现Global Filter接口即可，通过全局过滤器可以实现黑白名单、 限流等功能。

#### **5.7 GateWay**高可用

网关作为非常核心的一个部件，如果挂掉，那么所有请求都可能无法路由处理，因此我们需要做

GateWay的高可用。 **GateWay**的高可用很简单:可以启动多个GateWay实例来实现高可用，在GateWay的上游使用Nginx

等负载均衡设备进行负载转发以达到高可用的目的。

启动多个GateWay实例(假如说两个，一个端口9002，一个端口9003)，剩下的就是使用Nginx等完 成负载代理即可。示例如下:

```xml
#配置多个GateWay实例 
upstream gateway {
	server 127.0.0.1:9002;
	server 127.0.0.1:9003; 
}
location / {
 	proxy_pass http://gateway;
}
```

### 第 6 节 Spring Cloud Config 分布式配置中心

#### **6.1** 分布式配置中心应用场景

往往，我们使用配置文件管理一些配置信息，比如application.yml 

**单体应用架构**，配置信息的管理、维护并不会显得特别麻烦，手动操作就可以，因为就一个工程;

**微服务架构**，因为我们的分布式集群环境中可能有很多个微服务，我们不可能一个一个去修改配置然后 重启生效，在一定场景下我们还需要在运行期间动态调整配置信息，比如:根据各个微服务的负载情 况，动态调整数据源连接池大小，我们希望配置内容发生变化的时候，微服务可以自动更新。

场景总结如下:

1)集中配置管理，一个微服务架构中可能有成百上千个微服务，所以集中配置管理是很重要的(一次 修改、到处生效)

2)不同环境不同配置，比如数据源配置在不同环境(开发dev,测试test,生产prod)中是不同的 3)运行期间可动态调整。例如，可根据各个微服务的负载情况，动态调整数据源连接池大小等配置修改后可自动更新

4)如配置内容发生变化，微服务可以自动更新配置

那么，我们就需要对配置文件进行集中式管理，这也是分布式配置中心的作用。

#### 6.2 Spring Cloud Config

##### 6.2.1 Config简介

Spring Cloud Config是一个分布式配置管理方案，包含了 Server端和 Client端两个部分。

![image-20200722123429023](https://tva1.sinaimg.cn/large/007S8ZIlly1ggzn68x9l3j316y0gmth4.jpg)

- Server 端:提供配置文件的存储、以接口的形式将配置文件的内容提供出去，通过使用 @EnableConfigServer注解在 Spring boot 应用中非常简单的嵌入
- Client 端:通过接口获取配置数据并初始化自己的应用

##### 6.2.2 Config分布式配置应用

**说明:Config Server是集中式的配置服务，用于集中管理应用程序各个环境下的配置。 默认使用Git存**

**储配置文件内容，也可以SVN。**

比如，我们要对“简历微服务”的application.yml进行管理(区分开发环境、测试环境、生产环境)

1)登录码云，创建项目lagou-config-repo

2)上传yml配置文件，命名规则如下:

{application}-{profile}.yml 或者 {application}-{profile}.properties

其中，application为应用名称，profile指的是环境(用于区分开发环境，测试环境、生产环境等)

示例:lagou-service-resume-dev.yml、lagou-service-resume-test.yml、lagou-service-resume- prod.yml

**3)构建Config Server统一配置中心**

**新建SpringBoot工程，引入依赖坐标**(需要注册自己到Eureka)

测试访问:http://localhost:9006/master/lagou-service-resume-dev.yml，查看到配置文件内容

**4)构建Client客户端(在已有简历微服务基础上)**

已有工程中添加依赖坐标

```xml
<dependency> 
  <groupId>org.springframework.cloud</groupId> 
  <artifactId>spring-cloud-config-client</artifactId>
</dependency>
```

**application.yml**修改为**bootstrap.yml**配置文件

bootstrap.yml是系统级别的，优先级比application.yml高，应用启动时会检查这个配置文件，在这个 配置文件中指定配置中心的服务地址，会自动拉取所有应用配置并且启用。

(主要是把与统一配置中心连接的配置信息放到bootstrap.yml) 

注意:需要统一读取的配置信息，从集中配置中心获取

#### 6.3 Config配置手动刷新

不用重启微服务，只需要手动的做一些其他的操作(访问一个地址/refresh)刷新，之后再访问即可

此时，客户端取到了配置中心的值，但当我们修改GitHub上面的值时，服务端(Config Server)能实 时获取最新的值，但客户端(Config Client)读的是缓存，无法实时获取最新值。Spring Cloud已 经为 我们解决了这个问题，那就是客户端使用post去触发refresh，获取最新数据。

1)Client客户端添加依赖springboot-starter-actuator(已添加) 

2)Client客户端bootstrap.yml中添加配置(暴露通信端点)

3)Client客户端使用到配置信息的类上添加@RefreshScope

4)手动向Client客户端发起POST请求，http://localhost:8080/actuator/refresh，刷新配置信息

注意:手动刷新方式避免了服务重启(流程:**Git**改配置**—>for**循环脚本手动刷新每个微服务) 

思考:可否使用广播机制，一次通知，处处生效，方便大范围配置刷新?

#### 6.4 Config配置自动更新

实现一次通知处处生效

拉勾内部做分布式配置，用的是zk(存储+通知)，zk中数据变更，可以通知各个监听的客户端，客户 端收到通知之后可以做出相应的操作(内存级别的数据直接生效，对于数据库连接信息、连接池等信息 变化更新的，那么会在通知逻辑中进行处理，比如重新初始化连接池)

在微服务架构中，我们可以结合消息总线(Bus)实现分布式配置的自动更新(Spring Cloud Config+Spring Cloud Bus)

#### 6.4.1 消息总线Bus

所谓消息总线Bus，即我们经常会使用MQ消息代理构建一个共用的Topic，通过这个Topic连接各个微服 务实例，MQ广播的消息会被所有在注册中心的微服务实例监听和消费。换言之就是通过一个主题连接 各个微服务，打通脉络。

Spring Cloud Bus(基于MQ的，支持RabbitMq/Kafka) 是Spring Cloud中的消息总线方案，Spring Cloud Config + Spring Cloud Bus 结合可以实现配置信息的自动更新。

![image-20200722131138098](https://tva1.sinaimg.cn/large/007S8ZIlly1ggzo8ypzuoj316u0q4gtp.jpg)

**6.4.2 Spring Cloud Config+Spring Cloud Bus** 实现自动刷新

MQ消息代理，我们还选择使用RabbitMQ，ConfigServer和ConfigClient都添加都消息总线的支持以及

与RabbitMq的连接信息

1)Config Server服务端添加消息总线支持

2)ConfigServer添加配置

3)微服务暴露端口

4)重启各个服务，更改配置之后，向配置中心服务端发送post请求

http://localhost:9003/actuator/bus-refresh，各个客户端配置即可自动刷新 

在广播模式下实现了一次请求，处处更新，如果我只想定向更新呢?

 在发起刷新请求的时候http://localhost:9006/actuator/bus-refresh/lagou-service-resume:8081 

即为最后面跟上要定向刷新的实例的 服务名**:**端口号即可

### 第 **7** 节 **Spring Cloud Stream**消息驱动组件

Spring Cloud Stream 消息驱动组件帮助我们更快速，更方便，更友好的去构建消息驱动微服务的。 

当时定时任务和消息驱动的一个对比。(消息驱动:基于消息机制做一些事情) 

MQ:消息队列/消息中间件/消息代理，产品有很多，ActiveMQ RabbitMQ RocketMQ Kafka

#### 7.1 Stream解决的痛点问题

MQ消息中间件广泛应用在应用解耦合、异步消息处理、流量削峰等场景中。

不同的MQ消息中间件内部机制包括使用方式都会有所不同，比如RabbitMQ中有Exchange(交换机/交 换器)这一概念，kafka有Topic、Partition分区这些概念，MQ消息中间件的差异性不利于我们上层的 开发应用，当我们的系统希望从原有的RabbitMQ切换到Kafka时，我们会发现比较困难，很多要操作可 能重来(**因为应用程序和具体的某一款MQ消息中间件耦合在一起了**)。

Spring Cloud Stream进行了很好的上层抽象，可以让我们与具体消息中间件解耦合，屏蔽掉了底层具 体MQ消息中间件的细节差异，就像Hibernate屏蔽掉了具体数据库(Mysql/Oracle一样)。如此一 来，我们学习、开发、维护MQ都会变得轻松。目前Spring Cloud Stream支持RabbitMQ和Kafka。

**本质:屏蔽掉了底层不同MQ消息中间件之间的差异，统一了MQ的编程模型，降低了学习、开发、维 护MQ的成本**

#### **7.2 Stream**重要概念

Spring Cloud Stream 是一个构建消息驱动微服务的框架。应用程序通过inputs(相当于消息消费者 consumer)或者outputs(相当于消息生产者producer)来与Spring Cloud Stream中的binder对象交 互，而Binder对象是用来屏蔽底层MQ细节的，它负责与具体的消息中间件交互。

说白了:对于我们来说，只需要知道如何使用**Spring Cloud Stream**与**Binder**对象交互即可

![image-20200722134014172](https://tva1.sinaimg.cn/large/007S8ZIlly1ggzp3ypjfsj317g0tc193.jpg)

**Binder绑定器**

Binder绑定器是Spring Cloud Stream 中非常核心的概念，就是通过它来屏蔽底层不同MQ消息中间件 的细节差异，当需要更换为其他消息中间件时，我们需要做的就是更换对应的**Binder**绑定器而不需要修 改任何应用逻辑(Binder绑定器的实现是框架内置的，Spring Cloud Stream目前支持Rabbit、Kafka两 种消息队列)

#### **7.3** 传统**MQ**模型与**Stream**消息驱动模型

![image-20200722134839280](https://tva1.sinaimg.cn/large/007S8ZIlly1ggzpbf7sikj318i0g0tbb.jpg)

#### **7.4 Stream**消息通信方式及编程模型

##### **7.4.1 Stream**消息通信方式

Stream中的消息通信方式遵循了发布—订阅模式。

在Spring Cloud Stream中的消息通信方式遵循了发布-订阅模式，当一条消息被投递到消息中间件之 后，它会通过共享的 Topic 主题进行广播，消息消费者在订阅的主题中收到它并触发自身的业务逻辑处 理。这里所提到的 Topic 主题是Spring Cloud Stream中的一个抽象概念，用来代表发布共享消息给消 费者的地方。在不同的消息中间件中， Topic 可能对应着不同的概念，比如:在RabbitMQ中的它对应 了Exchange、在Kakfa中则对应了Kafka中的Topic。

##### **7.4.2 Stream**编程注解

**如下的注解无非在做一件事，把我们结构图中那些组成部分上下关联起来，打通通道(这样的话生产者 的message数据才能进入mq，mq中数据才能进入消费者工程)。**

| 注解                                                    | 描述                                                      |
| ------------------------------------------------------- | --------------------------------------------------------- |
| @Input(在消费者工程中使用)                              | 注解标识输入通道，通过该输入通道接收到的 消息进入应用程序 |
| @Output(在生产者工程中使用)                             | 注解标识输出通道，发布的消息将通过该通道 离开应用程序     |
| @StreamListener(在消费者工程中使用，监 听message的到来) | 监听队列，用于消费者的队列的消息的接收 (有消息监听.....)  |
| @EnableBinding                                          | 把Channel和Exchange(对于RabbitMQ)绑 定在一起              |

接下来，我们创建三个工程(我们基于RabbitMQ，RabbitMQ的安装和使用这里不再说明)

- lagou-cloud-stream-producer-9090， 作为生产者端发消息 
- lagou-cloud-stream-consumer-9091，作为消费者端接收消息 
- lagou-cloud-stream-consumer-9092，作为消费者端接收消息

##### **7.4.5 Stream**消息驱动之开发生产者端

1)在lagou_parent下新建子module:lagou-cloud-stream-producer-9090 

2)pom.xml中添加依赖

3)application.yml添加配置

4) 启动类

5)业务类开发(发送消息接口、接口实现类、Controller)

##### **7.4.6 Stream**消息驱动之开发消费者端

此处我们记录lagou-cloud-stream-consumer-9091编写过程，9092工程类似 

1)application.yml

2)消息消费者监听

#### **7.5 Stream**高级之自定义消息通道

Stream 内置了两种接口Source和Sink分别定义了 binding 为 “input” 的输入流和 “output” 的输出流， 我们也可以自定义各种输入输出流(通道)，但实际我们可以在我们的服务中使用多个binder、多个输 入通道和输出通道，然而默认就带了一个input的输入通道和一个output的输出通道，怎么办?

我们是可以自定义消息通道的，学着Source和Sink的样子，给你的通道定义个自己的名字，多个输入通 道和输出通道是可以写在一个类中的。

定义接口

```java
interface CustomChannel {
 String INPUT_LOG = "inputLog"; 
 String OUTPUT_LOG = "outputLog";

  @Input(INPUT_LOG) 
  SubscribableChannel inputLog();

  @Output(OUTPUT_LOG)
  MessageChannel outputLog();
}
```

如何使用?

1)在 @EnableBinding 注解中，绑定自定义的接口
 2)使用 @StreamListener 做监听的时候，需要指定 CustomChannel.INPUT_LOG

```yaml
bindings:
  inputLog:
    destination: lagouExchange
  outputLog:
    destination: eduExchange
```

#### **7.6 Stream**高级之消息分组

如上我们的情况，消费者端有两个(消费同一个MQ的同一个主题)，但是呢我们的业务场景中希望这 个主题的一个Message只能被一个消费者端消费处理，此时我们就可以使用消息分组。

**解决的问题:能解决消息重复消费问题**

我们仅仅需要在服务消费者端设置 spring.cloud.stream.bindings.input.group 属性，多个消费者实例 配置为同一个group名称(在同一个group中的多个消费者只有一个可以获取到消息并消费)。

![image-20200722143355703](https://tva1.sinaimg.cn/large/007S8ZIlly1ggzqmkt2xsj31980acwkc.jpg)

## 第五部分 常⻅问题及解决方案

本部分主要讲解 Eureka 服务发现慢的原因，Spring Cloud 超时设置问题。

如果你刚刚接触Eureka，对Eureka的设计和实现都不是很了解，可能就会遇到一些无法快速解决的问 题，这些问题包括:新服务上线后，服务消费者不能访问到刚上线的新服务，需要过一段时间后才能访 问?或是将服务下线后，服务还是会被调用到，一段时候后才彻底停止服务，访问前期会导致频繁报 错?这些问题还会让你对 Spring Cloud 产生严重的怀疑，这难道不是一个 Bug?

问题场景 上线一个新的服务实例，但是服务消费者无感知，过了一段时间才知道 某一个服务实例下线了，服务消费者无感知，仍然向这个服务实例在发起请求

这其实就是服务发现的一个问题，当我们需要调用服务实例时，信息是从注册中心Eureka获取的，然后 通过Ribbon选择一个服务实例发起调用，如果出现调用不到或者下线后还可以调用的问题，原因肯定是 服务实例的信息更新不及时导致的。

**Eureka 服务发现慢的原因**

Eureka 服务发现慢的原因主要有两个，一部分是因为服务缓存导致的，另一部分是因为客户端缓存导 致的。

![image-20200722144527285](https://tva1.sinaimg.cn/large/007S8ZIlly1ggzqyij8hrj31600pg0wz.jpg)

1)服务端缓存

服务注册到注册中心后，服务实例信息是存储在注册表中的，也就是内存中。但Eureka为了提高响应速 度，在内部做了优化，加入了两层的缓存结构，将Client需要的实例信息，直接缓存起来，获取的时候 直接从缓存中拿数据然后响应给 Client。 第一层缓存是readOnlyCacheMap，readOnlyCacheMap是 采用ConcurrentHashMap来存储数据的，主要负责定时与readWriteCacheMap进行数据同步，默认同 步时间为 30 秒一次。

第二层缓存是readWriteCacheMap，readWriteCacheMap采用Guava来实现缓存。缓存过期时间默认 为180秒，当服务下线、过期、注册、状态变更等操作都会清除此缓存中的数据。

Client获取服务实例数据时，会先从一级缓存中获取，如果一级缓存中不存在，再从二级缓存中获取， 如果二级缓存也不存在，会触发缓存的加载，从存储层拉取数据到缓存中，然后再返回给 Client。

Eureka 之所以设计二级缓存机制，也是为了提高 Eureka Server 的响应速度，缺点是缓存会导致 Client 获取不到最新的服务实例信息，然后导致无法快速发现新的服务和已下线的服务。

了解了服务端的实现后，想要解决这个问题就变得很简单了，我们可以缩短只读缓存的更新时间 (eureka.server.response-cache-update-interval-ms)让服务发现变得更加及时，或者直接将只读缓 存关闭(eureka.server.use-read-only-response-cache=false)，多级缓存也导致C层面(数据一致 性)很薄弱。

Eureka Server 中会有定时任务去检测失效的服务，将服务实例信息从注册表中移除，也可以将这个失 效检测的时间缩短，这样服务下线后就能够及时从注册表中清除。

2)客户端缓存 客户端缓存主要分为两块内容，一块是 Eureka Client 缓存，一块是 Ribbon 缓存。

**Eureka Client** 缓存

EurekaClient负责跟EurekaServer进行交互，在EurekaClient中的 com.netflix.discovery.DiscoveryClient.initScheduledTasks() 方法中，初始化了一个 CacheRefreshThread 定时任务专⻔用来拉取 Eureka Server 的实例信息到本地。

所以我们需要缩短这个定时拉取服务信息的时间间隔(eureka.client.registryFetchIntervalSeconds) 来快速发现新的服务。

**Ribbon** 缓存 Ribbon会从EurekaClient中获取服务信息，ServerListUpdater是Ribbon中负责服务实例 更新的组件，默认的实现是PollingServerListUpdater，通过线程定时去更新实例信息。定时刷新的时 间间隔默认是30秒，当服务停止或者上线后，这边最快也需要30秒才能将实例信息更新成最新的。我们 可以将这个时间调短一点，比如 3 秒。

刷新间隔的参数是通过 getRefreshIntervalMs 方法来获取的，方法中的逻辑也是从 Ribbon 的配置中进 行取值的。

将这些服务端缓存和客户端缓存的时间全部缩短后，跟默认的配置时间相比，快了很多。我们通过调整 参数的方式来尽量加快服务发现的速度，但是还是不能完全解决报错的问题，间隔时间设置为3秒，也 还是会有间隔。所以我们一般都会开启重试功能，当路由的服务出现问题时，可以重试到另一个服务来 保证这次请求的成功。

**Spring Cloud 各组件超时**

在SpringCloud中，应用的组件较多，只要涉及通信，就有可能会发生请求超时。那么如何设置超时时

间? 在 Spring Cloud 中，超时时间只需要重点关注 Ribbon 和 Hystrix 即可。

**Ribbon** 如果采用的是服务发现方式，就可以通过服务名去进行转发，需要配置Ribbon的超时。 Rbbon的超时可以配置全局的ribbon.ReadTimeout和ribbon.ConnectTimeout。也可以在前面指定服 务名，为每个服务单独配置，比如 user-service.ribbon.ReadTimeout。

其次是Hystrix的超时配置，Hystrix的超时时间要大于Ribbon的超时时间，因为Hystrix将请求包装了起 来，特别需要注意的是，如果Ribbon开启了重试机制，比如重试3 次，Ribbon 的超时为 1 秒，那么 Hystrix 的超时时间应该大于 3 秒，否则就会出现 Ribbon 还在重试中，而 Hystrix 已经超时的现象。

**Hystrix** Hystrix全局超时配置就可以用default来代替具体的command名称。 hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds=3000 如果想对具体的 command 进行配置，那么就需要知道 command 名称的生成规则，才能准确的配置。

如果我们使用 @HystrixCommand 的话，可以自定义 commandKey。如果使用FeignClient的话，可以 为FeignClient来指定超时时间: hystrix.command.UserRemoteClient.execution.isolation.thread.timeoutInMilliseconds = 3000

如果想对FeignClient中的某个接口设置单独的超时，可以在FeignClient名称后加上具体的方法: hystrix.command.UserRemoteClient#getUser(Long).execution.isolation.thread.timeoutInMilliseco nds = 3000

**Feign** Feign本身也有超时时间的设置，如果此时设置了Ribbon的时间就以Ribbon的时间为准，如果没 设置Ribbon的时间但配置了Feign的时间，就以Feign的时间为准。Feign的时间同样也配置了连接超时 时间(feign.client.config.服务名称.connectTimeout)和读取超时时间(feign.client.config.服务名 称.readTimeout)。

建议，我们配置Ribbon超时时间和Hystrix超时时间即可。