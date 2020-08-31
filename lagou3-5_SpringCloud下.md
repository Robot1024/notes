# 第六部分 **Spring Cloud**高级进阶

### 第 **1** 节 微服务监控之 **Turbine** 聚合监控

参考上文Hystrix部分

### 第 **2** 节 微服务监控之分布式链路追踪技术 **Sleuth + Zipkin**

#### **2.1** 分布式链路追踪技术适用场景(问题场景)

- 场景描述

  为了支撑日益增⻓的庞大业务量，我们会使用微服务架构设计我们的系统，使得我们的系统不仅能 够通过集群部署抵挡流量的冲击，又能根据业务进行灵活的扩展。

  那么，在微服务架构下，一次请求少则经过三四次服务调用完成，多则跨越几十个甚至是上百个服 务节点。那么问题接踵而来:

  1)如何动态展示服务的调用链路?(比如A服务调用了哪些其他的服务---依赖关系) 

  2)如何分析服务调用链路中的瓶颈节点并对其进行调优?(比如A—>B—>C，C服务处理时间特别

  ⻓)

   3)如何快速进行服务链路的故障发现? 这就是分布式链路追踪技术存在的目的和意义

- 分布式链路追踪技术

  如果我们在一个请求的调用处理过程中，在各个链路节点都能够记录下日志，并最终将日志进行集 中可视化展示，那么我们想监控调用链路中的一些指标就有希望了~~~比如，请求到达哪个服务实 例?请求被处理的状态怎样?处理耗时怎样?这些都能够分析出来了...

  分布式环境下基于这种想法实现的监控技术就是就是分布式链路追踪(全链路追踪)。

- 市场上的分布式链路追踪方案

  分布式链路追踪技术已然成熟，产品也不少，国内外都有，比如

  - **Spring Cloud Sleuth + Twitter Zipkin**

  - 阿里巴巴的“鹰眼”

  - 大众点评的“CAT”

  - 美团的“Mtrace”

  - 京东的“Hydra”

  - 新浪的“Watchman” 

  另外还有最近也被提到很多的**Apache Skywalking**。

#### **2.2** 分布式链路追踪技术核心思想

本质:记录日志，作为一个完整的技术，分布式链路追踪也有自己的理论和概念

微服务架构中，针对请求处理的调用链可以展现为一棵树，示意如下

![image-20200802163147875](https://tva1.sinaimg.cn/large/007S8ZIlly1ghcjulivfbj31240jy0uc.jpg)

上图描述了一个常⻅的调用场景，一个请求通过网关服务路由到下游的微服务-1，然后微服务-1调用微 服务-2，拿到结果后再调用微服务-3，最后组合微服务-2和微服务-3的结果，通过网关返回给用户

为了追踪整个调用链路，肯定需要记录日志，日志记录是基础，在此之上肯定有一些理论概念，当下主 流的的分布式链路追踪技术/系统所基于的理念都来自于Google的一篇论文《Dapper, a Large-Scale Distributed Systems Tracing Infrastructure》，这里面涉及到的核心理念是什么，我们来看下，还以 前面的服务调用来说

![image-20200802163311075](https://tva1.sinaimg.cn/large/007S8ZIlly1ghcjw2g4f9j314e0pmn0s.jpg)

上图标识一个请求链路，一条链路通过TraceId唯一标识，span标识发起的请求信息，各span通过 parrentId关联起来

**Trace**:服务追踪的追踪单元是从客户发起请求(request)抵达被追踪系统的边界开始，到被追踪系统 向客户返回响应(response)为止的过程

**Trace ID**:为了实现请求跟踪，当请求发送到分布式系统的入口端点时，只需要服务跟踪框架为该请求 创建一个唯一的跟踪标识Trace ID，同时在分布式系统内部流转的时候，框架始终保持该唯一标识，直 到返回给请求方

一个Trace由一个或者多个Span组成，每一个Span都有一个SpanId，Span中会记录TraceId，同时还有 一个叫做ParentId，指向了另外一个Span的SpanId，表明父子关系，其实本质表达了依赖关系

**Span ID**:为了统计各处理单元的时间延迟，当请求到达各个服务组件时，也是通过一个唯一标识Span ID来标记它的开始，具体过程以及结束。对每一个Span来说，它必须有开始和结束两个节点，通过记录 开始Span和结束Span的时间戳，就能统计出该Span的时间延迟，除了时间戳记录之外，它还可以包含 一些其他元数据，比如时间名称、请求信息等。

每一个Span都会有一个唯一跟踪标识 Span ID,若干个有序的 span 就组成了一个 trace。 Span可以认为是一个日志数据结构，在一些特殊的时机点会记录了一些日志信息，比如有时间戳、

spanId、TraceId，parentIde等，Span中也抽象出了另外一个概念，叫做事件，核心事件如下

- CS :client send/start 客户端/消费者发出一个请求，描述的是一个span开始

- SR: server received/start 服务端/生产者接收请求 SR-CS属于请求发送的网络延迟

- SS: server send/finish 服务端/生产者发送应答 SS-SR属于服务端消耗时间

- CR:client received/finished 客户端/消费者接收应答 CR-SS表示回复需要的时间(响应的网络延 迟)

------

Spring Cloud Sleuth (追踪服务框架)可以追踪服务之间的调用，Sleuth可以记录一个服务请求经过哪 些服务、服务处理时⻓等，根据这些，我们能够理清各微服务间的调用关系及进行问题追踪分析。

- 耗时分析:通过 Sleuth 了解采样请求的耗时，分析服务性能问题(哪些服务调用比较耗时) 
- 链路优化:发现频繁调用的服务，针对性优化等 Sleuth就是通过记录日志的方式来记录踪迹数据的

**注意:我们往往把Spring Cloud Sleuth 和 Zipkin 一起使用，把 Sleuth 的数据信息发送给 Zipkin 进 行聚合，利用 Zipkin 存储并展示数据。**  

![image-20200802164647098](https://tva1.sinaimg.cn/large/007S8ZIlly1ghcka5cjtij310g0o2wmv.jpg)

#### **2.3 Sleuth + Zipkin**

### 第 **3** 节 微服务统一认证方案 **Spring Cloud OAuth2 + JWT**

认证:验证用户的合法身份，比如输入用户名和密码，系统会在后台验证用户名和密码是否合法，

合法的前提下，才能够进行后续的操作，访问受保护的资源

#### **3.0** 微服务架构下统一认证场景

分布式系统的每个服务都会有认证需求，如果每个服务都实现一套认证逻辑会非常冗余，考虑分布 式系统共享性的特点，需要由独立的认证服务处理系统认证的请求。

![image-20200802173329872](https://tva1.sinaimg.cn/large/007S8ZIlly1ghclmrj7e1j30ks0vkgpb.jpg)

#### **3.1** 微服务架构下统一认证思路

- 基于Session的认证方式

  在分布式的环境下，基于session的认证会出现一个问题，每个应用服务都需要在session中存储用 户身份信息，通过负载均衡将本地的请求分配到另一个应用服务需要将session信息带过去，否则 会重新认证。我们可以使用Session共享、Session黏贴等方案。

  Session方案也有缺点，比如基于cookie，移动端不能有效使用等

- 基于token的认证方式

  基于token的认证方式，服务端不用存储认证数据，易维护扩展性强， 客户端可以把token 存在任 意地方，并且可以实现web和app统一认证机制。其缺点也很明显，token由于自包含信息，因此 一般数据量较大，而且每次请求 都需要传递，因此比较占带宽。另外，token的签名验签操作也会 给cpu带来额外的处理负担。

#### **3.2 OAuth2**开放授权协议**/**标准

##### **3.2.1 OAuth2**介绍

OAuth(开放授权)是一个开放协议/标准，允许用户授权第三方应用访问他们存储在另外的服务提供者 上的信息，而不需要将用户名和密码提供给第三方应用或分享他们数据的所有内容。

**允许用户授权第三方应用访问他们存储在另外的服务提供者上的信息，而不需要将用户名和密码提供给**
**第三方应用或分享他们数据的所有内容**

结合“使用QQ登录拉勾”这个场景拆分理解上述那句话

用户:我们自己

第三方应用:拉勾网

另外的服务提供者:QQ

OAuth2是OAuth协议的延续版本，但不向后兼容OAuth1即完全废止了OAuth1。

##### **3.3.2 OAuth2**协议⻆色和流程

拉勾网要开发使用QQ登录这个功能的话，那么拉勾网是需要提前到QQ平台进行登记的(否则QQ凭什 么陪着拉勾网玩授权登录这件事)

 1)拉勾网——登记——>QQ平台

 2)QQ 平台会颁发一些参数给拉勾网，后续上线进行授权登录的时候(刚才打开授权⻚面)需要携带

这些参数

 client_id :客户端id(QQ最终相当于一个认证授权服务器，拉勾网就相当于一个客户端了，所以会给
一个客户端id)，相当于账号 

secret:相当于密码

![image-20200802180209119](https://tva1.sinaimg.cn/large/007S8ZIlly1ghcmgkhba1j313q0m2amp.jpg)

- 资源所有者(Resource Owner):可以理解为用户自己 
- 客户端(Client):我们想登陆的网站或应用，比如拉勾网 
- 认证服务器(Authorization Server):可以理解为微信或者QQ 
- 资源服务器(Resource Server):可以理解为微信或者QQ

##### **3.2.3** 什么情况下需要使用**OAuth2**?

第三方授权登录的场景:比如，我们经常登录一些网站或者应用的时候，可以选择使用第三方授权登录 的方式，比如:微信授权登录、QQ授权登录、微博授权登录等，这是典型的 OAuth2 使用场景。

单点登录的场景:如果项目中有很多微服务或者公司内部有很多服务，可以专⻔做一个认证中心(充当 认证平台⻆色)，所有的服务都要到这个认证中心做认证，只做一次登录，就可以在多个授权范围内的 服务中自由串行。

##### **3.2.4 OAuth2**的颁发**Token**授权方式

**1)授权码(authorization-code)** 

**2)密码式(password)提供用户名+密码换取token令牌** 

3)隐藏式(implicit)

4)客户端凭证(client credentials) 

授权码模式使用到了回调地址，是最复杂的授权方式，微博、微信、QQ等第三方登录就是这种模式。
我们重点讲解接口对接中常使用的password密码模式(提供用户名+密码换取token)。

#### **3.3 Spring Cloud OAuth2 + JWT** 实现

##### **3.3.1 Spring Cloud OAuth2**介绍

Spring Cloud OAuth2 是 Spring Cloud 体系对OAuth2协议的实现，可以用来做多个微服务的统一认证 (验证身份合法性)授权(验证权限)。通过向OAuth2服务(统一认证授权服务)发送某个类型的 grant_type进行集中认证和授权，从而获得access_token(访问令牌)，而这个token是受其他微服务 信任的。

**注意:使用OAuth2解决问题的本质是，引入了一个认证授权层，认证授权层连接了资源的拥有者，在 授权层里面，资源的拥有者可以给第三方应用授权去访问我们的某些受保护资源。**

##### **3.3.2 Spring Cloud OAuth2**构建微服务统一认证服务思路

![image-20200802183347618](https://tva1.sinaimg.cn/large/007S8ZIlly1ghcndhrbckj314i0lqadz.jpg)

注意:在我们统一认证的场景中，**Resource Server**其实就是我们的各种受保护的微服务，微服务中的 各种**API**访问接口就是资源，发起**http**请求的浏览器就是**Client**客户端(对应为第三方应用)

##### **3.3.3** 搭建认证服务器(**Authorization Server**)

认证服务器(Authorization Server)，负责颁发token

- 新建项目lagou-cloud-oauth-server-9999

- pom.xml

- application.yml(构建认证服务器，配置文件无特别之处)

- 入口类无特殊之处

- 认证服务器配置类

  - 关于三个configure方法

    - **configure(ClientDetailsServiceConfigurer clients)**

      用来配置客户端详情服务(ClientDetailsService)，客户端详情信息在 这里进行初始化，你 能够把客户端详情信息写死在这里或者是通过数据库来存储调取详情信息

    - **configure(AuthorizationServerEndpointsConfigurer endpoints)**
      用来配置令牌(token)的访问端点和令牌服务(token services)
    - **configure(AuthorizationServerSecurityConfigurer oauthServer)**
      用来配置令牌端点的安全约束.
    
  - 关于 TokenStore
  
    - InMemoryTokenStore
      默认采用，它可以完美的工作在单服务器上(即访问并发量 压力不大的情况下，并且它 在失败的时候不会进行备份)，大多数的项目都可以使用这个版本的实现来进行 尝试， 你可以在开发的时候使用它来进行管理，因为不会被保存到磁盘中，所以更易于调试。
    - JdbcTokenStore
      这是一个基于JDBC的实现版本，令牌会被保存进关系型数据库。使用这个版本的实现 时， 你可以在不同的服务器之间共享令牌信息，使用这个版本的时候请注意把"spring- jdbc"这个依赖加入到你的 classpath当中。
    - JwtTokenStore 这个版本的全称是 JSON Web Token(JWT)，它可以把令牌相关的数 据进行编码(因此对于后端服务来说，它不需要进行存储，这将是一个重大优势)，缺 点就是这个令牌占用的空间会比较大，如果你加入了比较多用户凭证信息， JwtTokenStore 不会保存任何数据。

- 测试

  获取token:http://localhost:9999/oauth/token?client_secret=abcxyz&grant_type=password&

  username=admin&password=123456&client_id=client_lagou

  

  - endpoint:/oauth/token 获取token携带的参数
  - 获取token携带的参数
    - client_id:客户端id
    - client_secret:客户单密码 
    - grant_type:指定使用哪种颁发类型，password 
    - username:用户名
    - password:密码

  

  校验token:http://localhost:9999/oauth/check_token?token=a9979518-838c-49ff-b14a-ebdb7fde7d08

  刷新token:http://localhost:9999/oauth/token?grant_type=refresh_token&client_id=client_lagou&client_secret=abcxyz&refresh_token=8b640340-30a3-4307-93d4-ed60cc54fbc8

- 资源服务器(希望访问被认证的微服务)Resource Server配置
  
  - 资源服务配置类

思考:当我们第一次登陆之后，认证服务器颁发token并将其存储在认证服务器中，后期我们 访问资源服务器时会携带token，资源服务器会请求认证服务器验证token有效性，如果资源 服务器有很多，那么认证服务器压力会很大.......

另外，资源服务器向认证服务器check_token，获取的也是用户信息UserInfo，能否把用户信 息存储到令牌中，让客户端一直持有这个令牌，令牌的验证也在资源服务器进行，这样避免 和认证服务器频繁的交互......

我们可以考虑使用 JWT 进行改造，使用JWT机制之后资源服务器不需要访问认证服务器......

##### **3.3.4 JWT**改造统一认证授权中心的令牌存储机制

**JWT**令牌介绍

通过上边的测试我们发现，当资源服务和授权服务不在一起时资源服务使用RemoteTokenServices 远

程请求授权 服务验证token，如果访问量较大将会影响系统的性能。

解决上边问题: 令牌采用JWT格式即可解决上边的问题，用户认证通过会得到一个JWT令牌，JWT令牌 中已经包括了用户相关的信 息，客户端只需要携带JWT访问资源服务，资源服务根据事先约定的算法自 行完成令牌校验，无需每次都请求认证 服务完成授权。

1)什么是JWT?

JSON Web Token(JWT)是一个开放的行业标准(RFC 7519)，它定义了一种简介的、自包含的协议 格式，用于 在通信双方传递json对象，传递的信息经过数字签名可以被验证和信任。JWT可以使用 HMAC算法或使用RSA的公 钥/私钥对来签名，防止被篡改。

2)JWT令牌结构

JWT令牌由三部分组成，每部分中间使用点(.)分隔，比如:xxxxx.yyyyy.zzzzz

- Header

  头部包括令牌的类型(即JWT)及使用的哈希算法(如HMAC SHA256或RSA)，例如

  ```json
  {
   "alg": "HS256", "typ": "JWT"
  }
  ```

将上边的内容使用Base64Url编码，得到一个字符串就是JWT令牌的第一部分。

- Payload

  第二部分是负载，内容也是一个json对象，它是存放有效信息的地方，它可以存放jwt提供的现成 字段，比 如:iss(签发者),exp(过期时间戳), sub(面向的用户)等，也可自定义字段。 此部 分不建议存放敏感信息，因为此部分可以解码还原原始内容。 最后将第二部分负载使用Base64Url 编码，得到一个字符串就是JWT令牌的第二部分。 一个例子:

  ```json
  {
   "sub": "1234567890", "name": "John Doe", "iat": 1516239022
  }
  ```

- Signature

  第三部分是签名，此部分用于防止jwt内容被篡改。 这个部分使用base64url将前两部分进行编

  码，编码后使用点(.)连接组成字符串，最后使用header中声明 签名算法进行签名。

  ```java
  HMACSHA256( 
  	base64UrlEncode(header) + "." + 
  	base64UrlEncode(payload), 
  	secret)
  ```

base64UrlEncode(header):jwt令牌的第一部分。 

base64UrlEncode(payload):jwt令牌的第二部分。 

secret:签名所使用的密钥。

认证服务器端**JWT**改造(改造主配置类)

- 修改 JWT 令牌服务方法

- 资源服务器校验**JWT**令牌 

- 不需要和远程认证服务器交互，添加本地tokenStore

  

## 第七部分 第二代 **Spring Cloud** 核心组件(**SCA**)

第一代 Spring Cloud (主要是 SCN)很多组件已经进入停更维护模式。

Spring Cloud:Netflix，Spring官方，SCA(被Spring官方认可)

注意:市场上主要使用的还是SCN，SCA一套框架的集合

Alibaba 更进一步，搞出了Spring Cloud Alibaba(SCA)，SCA 是由一些阿里巴巴的开源组件和云产品 组成的，2018年，Spring Cloud Alibaba 正式入住了 Spring Cloud 官方孵化器。

Nacos(服务注册中心、配置中心)

Sentinel哨兵(服务的熔断、限流等) 

Dubbo RPC/LB 

Seata分布式事务解决方案

#### **7.1 SCA Nacos** 服务注册和配置中心

**7.1.1 Nacos** 介绍

Nacos (Dynamic Naming and Configuration Service)是阿里巴巴开源的一个针对微服务架构中服务 发现、配置管理和服务管理平台。

Nacos就是注册中心+配置中心的组合(Nacos=Eureka+Config+Bus)

官网:https://nacos.io 下载地址:https://github.com/alibaba/Nacos

**Nacos**功能特性

- 服务发现与健康检查
- 动态配置管理
- 动态DNS服务 
- 服务和元数据管理(管理平台的⻆度，nacos也有一个ui⻚面，可以看到注册的服务及其实例信息 (元数据信息)等)，动态的服务权重调整、动态服务优雅下线，都可以去做

**7.1.2 Nacos** 单例服务部署

- 下载解压安装包，执行命令启动(我们使用最近比较稳定的版本 nacos-server-1.2.0.tar.gz)

  ```
  linux/mac:sh startup.sh -m standalone
  windows:cmd startup.cmd
  ```

- 访问nacos管理界面:http://127.0.0.1:8848/nacos/#/login(默认端口8848，账号和密码 nacos/nacos)

![image-20200802231521768](https://tva1.sinaimg.cn/large/007S8ZIlly1ghcvigv5y8j31340f640r.jpg)

##### **7.1.3 Nacos** 服务注册中心

###### **7.1.3.1** 服务提供者注册到**Nacos(**改造简历微服务**)**

- 在父pom中引入SCA依赖

- 在服务提供者工程中引入nacos客户端依赖(注释eureka客户端)

- application.yml修改，添加nacos配置信息

- 启动简历微服务，观察nacos控制台
  ![image-20200802232628687](https://tva1.sinaimg.cn/large/007S8ZIlly1ghcvu1boucj313w0t0k1a.jpg)

  保护阈值:可以设置为0-1之间的浮点数，它其实是一个比例值(当前服务健康实例数/当前服务总实例 数)

  场景:

  一般流程下，nacos是服务注册中心，服务消费者要从nacos获取某一个服务的可用实例信息，对于服 务实例有健康/不健康状态之分，nacos在返回给消费者实例信息的时候，会返回健康实例。这个时候在 一些高并发、大流量场景下会存在一定的问题

  如果服务A有100个实例，98个实例都不健康了，只有2个实例是健康的，如果nacos只返回这两个健康 实例的信息的话，那么后续消费者的请求将全部被分配到这两个实例，流量洪峰到来，2个健康的实例 也扛不住了，整个服务A 就扛不住，上游的微服务也会导致崩溃，，，产生雪崩效应。

  保护阈值的意义在于
   当服务A健康实例数/总实例数 < 保护阈值 的时候，说明健康实例真的不多了，这个时候保护阈值会被触

  发(状态true) nacos将会把该服务所有的实例信息(健康的+不健康的)全部提供给消费者，消费者可能访问到不健康

  的实例，请求失败，但这样也比造成雪崩要好，牺牲了一些请求，保证了整个系统的一个可用。 注意:阿里内部在使用nacos的时候，也经常调整这个保护阈值参数。


###### 7.1.3.2 服务消费者从**Nacos**获取服务提供者**(**改造自动投递微服务**)**

- 同服务提供者
- 测试

![image-20200802235137847](https://tva1.sinaimg.cn/large/007S8ZIlly1ghcwk79d9lj315i0dy0uy.jpg)

###### **7.1.3.3** 负载均衡

Nacos客户端引入的时候，会关联引入Ribbon的依赖包，我们使用OpenFiegn的时候也会引入Ribbon

的依赖，Ribbon包括Hystrix都按原来方式进行配置即可

此处，我们将简历微服务，又启动了一个8083端口，注册到Nacos上，便于测试负载均衡，我们通过后

台也可以看出。

###### **7.1.3.4 Nacos** 数据模型(领域模型)

Namespace命名空间、Group分组、集群这些都是为了进行归类管理，把服务和配置文件进行归类，

归类之后就可以实现一定的效果，比如隔离 

比如，对于服务来说，不同命名空间中的服务不能够互相访问调用

![image-20200803001957453](https://tva1.sinaimg.cn/large/007S8ZIlly1ghcxdo2nq1j319i0mgte0.jpg)

Namespace:命名空间，对不同的环境进行隔离，比如隔离开发环境、测试环境和生产环境 

Group:分组，将若干个服务或者若干个配置集归为一组，通常习惯一个系统归为一个组 

Service:某一个服务，比如简历微服务

DataId:配置集或者可以认为是一个配置文件

**Namespace + Group + Service** 如同 **Maven** 中的**GAV**坐标，**GAV**坐标是为了锁定**Jar**，二这里是为了 锁定服务

**Namespace + Group + DataId** 如同 **Maven** 中的**GAV**坐标，**GAV**坐标是为了锁定**Jar**，二这里是为了 锁定配置文件

**最佳实践**

Nacos抽象出了Namespace、Group、Service、DataId等概念，具体代表什么取决于怎么用(非常灵 活)，推荐用法如下

| 概念      | 描述                                              |
| --------- | ------------------------------------------------- |
| Namespace | 代表不同的环境，如开发dev、测试test、生产环境prod |
| Group     | 代表某项目，比如拉勾云项目                        |
| Service   | 某个项目中具体xxx服务                             |
| DataId    | 某个项目中具体的xxx配置文件                       |

- Nacos服务的分级模型

![image-20200803002321066](https://tva1.sinaimg.cn/large/007S8ZIlly1ghcxh7gwl7j316k0kite3.jpg)

###### **7.1.3.5 Nacos Server** 数据持久化

Nacos 默认使用嵌入式数据库进行数据存储，它支持改为外部Mysql存储

- 新建数据库 nacos_config，数据库初始化脚本文件 ${nacoshome}/conf/nacos-mysql.sql 
- 修改${nacoshome}/conf/application.properties，增加Mysql数据源配置

###### **7.1.3.6 Nacos Server** 集群

- 安装3个或3个以上的Nacos 

  复制解压后的nacos文件夹，分别命名为nacos-01、nacos-02、nacos-03

- 修改配置文件

  - 同一台机器模拟，将上述三个文件夹中application.properties中的server.port分别改为 8848、8849、8850
  - 同时给当前实例节点绑定ip，因为服务器可能绑定多个ip
  - 复制一份conf/cluster.conf.example文件，命名为cluster.conf
  - 在配置文件中设置集群中每一个节点的信息

- 分别启动每一个实例(可以批处理脚本完成)

##### **7.1.4 Nacos** 配置中心

之前:Spring Cloud Config + Bus

1) Github 上添加配置文件

2)创建Config Server 配置中心—>从Github上去下载配置信息 

3)具体的微服务(最终使用配置信息的)中配置Config Client—> ConfigServer获取配置信息

有Nacos之后，分布式配置就简单很多

Github不需要了(配置信息直接配置在Nacos server中)，Bus也不需要了(依然可以完成动态刷新) 

接下来
 1、去Nacos server中添加配置信息
 2、改造具体的微服务，使其成为Nacos Config Client，能够从Nacos Server中获取到配置信息

**Nacos server 添加配置集**

![image-20200803004814660](https://tva1.sinaimg.cn/large/007S8ZIlly1ghcy73ppgdj316g0icq5a.jpg)

Nacos 服务端已经搭建完毕，那么我们可以在我们的微服务中开启 Nacos 配置管理

1)添加依赖

2)微服务中如何锁定 Nacos Server 中的配置文件(dataId)

通过 Namespace + Group + dataId 来锁定配置文件，Namespace不指定就默认public，Group不指定

就默认 DEFAULT_GROUP 

**dataId 的完整格式如下**

```yaml
${prefix}-${spring.profile.active}.${file-extension}
```

- prefix 默认为 spring.application.name 的值，也可以通过配置项

  spring.cloud.nacos.config.prefix 来配置。

- spring.profile.active 即为当前环境对应的 profile。 注意:当 **spring.profile.active** 为空时，对应的连接符 **-** 也将不存在，**dataId** 的拼接格式变成 **${prefix}.${file- extension}**

- file-exetension 为配置内容的数据格式，可以通过配置项

  spring.cloud.nacos.config.file-extension 来配置。目前只支持 properties 和 yaml 类型。

3)通过 Spring Cloud 原生注解 @RefreshScope 实现配置自动更新

思考:一个微服务希望从配置中心Nacos server中获取多个dataId的配置信息，可以的，扩展多个 dataId

优先级:根据规则生成的dataId > 扩展的dataId(对于扩展的dataId，[n] n越大优先级越高)

#### **7.2 SCA Sentinel** 分布式系统的流量防卫兵

##### **7.2.1 Sentinel** 介绍

Sentinel是一个面向云原生微服务的流量控制、熔断降级组件。

替代Hystrix，针对问题:服务雪崩、服务降级、服务熔断、服务限流

Hystrix:

服务消费者(自动投递微服务)—>调用服务提供者(简历微服务)

在调用方引入Hystrix—> 单独搞了一个Dashboard项目—>Turbine

1)自己搭建监控平台 dashboard 

2)没有提供UI界面进行服务熔断、服务降级等配置(而是写代码，入侵了我们源程序环境)

Sentinel:

1)独立可部署Dashboard/控制台组件 

2)减少代码开发，通过UI界面配置即可完成细粒度控制(自动投递微服务)

![image-20200803015916471](https://tva1.sinaimg.cn/large/007S8ZIlly1ghd090bb65j316q0mawmt.jpg)

Sentinel 分为两个部分:

- 核心库:(Java 客户端)不依赖任何框架/库，能够运行于所有 Java 运行时环境，同时对 Dubbo / Spring Cloud 等框架也有较好的支持。

- 控制台:(Dashboard)基于 Spring Boot 开发，打包后可以直接运行，不需要额外的 Tomcat 等 应用容器。

Sentinel 具有以下特征:

- **丰富的应用场景**:Sentinel 承接了阿里巴巴近 10 年的双十一大促流量的核心场景，例如秒杀(即 突发流量控制在系统容量可以承受的范围)、消息削峰填谷、集群流量控制、实时熔断下游不可用 应用等。
-  **完备的实时监控**:Sentinel 同时提供实时的监控功能。您可以在控制台中看到接入应用的单台机器 秒级数据，甚至 500 台以下规模的集群的汇总运行情况。
- **广泛的开源生态**:Sentinel 提供开箱即用的与其它开源框架/库的整合模块，例如与 Spring Cloud、Dubbo的整合。您只需要引入相应的依赖并进行简单的配置即可快速地接入 Sentinel。 
- **完善的 SPI 扩展点**:Sentinel 提供简单易用、完善的 SPI 扩展接口。您可以通过实现扩展接口来快 速地定制逻辑。例如定制规则管理、适配动态数据源等。

Sentinel 的主要特性:

![image-20200803020144157](https://tva1.sinaimg.cn/large/007S8ZIlly1ghd0bkgy6ij315a0jcqa5.jpg)

来自官网
 Sentinel 的开源生态:

![image-20200803020338603](https://tva1.sinaimg.cn/large/007S8ZIlly1ghd0dk7r5kj317q0lidtc.jpg)

##### **7.2.2 Sentinel** 部署

下载地址:https://github.com/alibaba/Sentinel/releases 我们使用v1.7.1

启动:java -jar sentinel-dashboard-1.7.1.jar & 

用户名/密码:sentinel/sentinel

##### **7.2.3** 服务改造

在我们已有的业务场景中，“自动投递微服务”调用了“简历微服务”，我们在自动投递微服务进行的熔断

降级等控制，那么接下来我们改造自动投递微服务，引入Sentinel核心包。 为了不污染之前的代码，复制一个自动投递微服务 lagou-service-autodeliver-8098-sentinel

- pom.xml引入依赖
- application.yml修改(配置sentinel dashboard，暴露断点依然要有，删除原有hystrix配置，删 除原有OpenFeign的降级配置)
- 上述配置之后，启动自动投递微服务，使用 Sentinel 监控自动投递微服务 此时我们发现控制台没有任何变化，因为懒加载，我们只需要发起一次请求触发即可
![image-20200803020548131](/Users/leipeng/Library/Application Support/typora-user-images/image-20200803020548131.png)

##### **7.2.4 Sentinel** 关键概念

| 概 念 名 称 | 概念描述                                                     |
| ----------- | ------------------------------------------------------------ |
| 资 源       | 它可以是 Java 应用程序中的任何内容，例如，由应用程序提供的服务，或由应用程序调用 的其它应用提供的服务，甚至可以是一段代码。我们请求的**API**接口就是资源 |
| 规 则       | 围绕资源的实时状态设定的规则，可以包括流量控制规则、熔断降级规则以及系统保护规 则。所有规则可以动态实时调整。 |

##### **7.2.5 Sentinel** 流量规则模块

系统并发能力有限，比如系统A的QPS支持1个，如果太多请求过来，那么A就应该进行流量控制了，比 如其他请求直接拒绝

![image-20200803020851243](https://tva1.sinaimg.cn/large/007S8ZIlly1ghd0izd7z3j315g0o842a.jpg)

资源名:默认请求路径 

针对来源:Sentinel可以针对调用者进行限流，填写微服务名称，默认default(不区分来源)

阈值类型/单机阈值

QPS:(每秒钟请求数量)当调用该资源的QPS达到阈值时进行限流

线程数:当调用该资源的线程数达到阈值的时候进行限流(线程处理请求的时候，如果说业务逻辑执行 时间很⻓，流量洪峰来临时，会耗费很多线程资源，这些线程资源会堆积，最终可能造成服务不可用， 进一步上游服务不可用，最终可能服务雪崩)

是否集群:是否集群限流

**流控模式:** 

直接:资源调用达到限流条件时，直接限流 

关联:关联的资源调用达到阈值时候限流自己 

链路:只记录指定链路上的流量

**流控效果:** 

快速失败:直接失败，抛出异常

Warm Up:根据冷加载因子(默认3)的值，从阈值/冷加载因子，经过预热时⻓，才达到设置的QPS阈 值

排队等待:匀速排队，让请求匀速通过，阈值类型必须设置为QPS，否则无效



流控模式之关联限流：

关联的资源调用达到阈值时候限流自己，比如用户注册接口，需要调用身份证校验接口(往往身份证校 验接口)，如果身份证校验接口请求达到阈值，使用关联，可以对用户注册接口进行限流。

![image-20200803021147102](https://tva1.sinaimg.cn/large/007S8ZIlly1ghd0m0wx6xj31500mutf9.jpg)

模拟密集式请求/user/validateID验证接口，我们会发现/user/register接口也被限流了

**流控模式之链路限流**

链路指的是请求链路(调用链)

链路模式下会控制该资源所在的调用链路入口的流量。需要在规则中配置入口资源，即该调用链路入口 的上下文名称。

一棵典型的调用树如下图所示:(阿里云提供)

![image-20200803021253813](https://tva1.sinaimg.cn/large/007S8ZIlly1ghd0n6llx2j311g0cedju.jpg)

上图中来自入口 Entrance1 和 Entrance2 的请求都调用到了资源 NodeA ，Sentinel 允许只根据某 个调用入口的统计信息对资源限流。比如链路模式下设置入口资源为 Entrance1 来表示只有从入口Entrance1 的调用才会记录到 NodeA 的限流统计当中，而不关心经 Entrance2 到来的调用。

![image-20200803021330744](https://tva1.sinaimg.cn/large/007S8ZIlly1ghd0ntkoa2j313d0u0q9x.jpg)

**流控效果之Warm up**

当系统⻓期处于空闲的情况下，当流量突然增加时，直接把系统拉升到高水位可能瞬间把系统压垮，比

如电商网站的秒杀模块。

通过 Warm Up 模式(预热模式)，让通过的流量缓慢增加，经过设置的预热时间以后，到达系统处理 请求速率的设定值。

Warm Up 模式默认会从设置的 QPS 阈值的 1/3 开始慢慢往上增加至 QPS 设置值。

![image-20200803021412003](https://tva1.sinaimg.cn/large/007S8ZIlly1ghd0ojawlzj31180s20yo.jpg)

**流控效果之排队等待**

排队等待模式下会严格控制请求通过的间隔时间，即请求会匀速通过，允许部分请求排队等待，通常用 于消息队列削峰填谷等场景。需设置具体的超时时间，当计算的等待时间超过超时时间时请求就会被拒 绝。

很多流量过来了，并不是直接拒绝请求，而是请求进行排队，一个一个匀速通过(处理)，请求能等就 等着被处理，不能等(等待时间>超时时间)就会被拒绝

例如，QPS 配置为 5，则代表请求每 200 ms 才能通过一个，多出的请求将排队等待通过。超时时间代 表最大排队时间，超出最大排队时间的请求将会直接被拒绝。排队等待模式下，QPS 设置值不要超过 1000(请求间隔 1 ms)。

##### **7.2.6 Sentinel** 降级规则模块

流控是对外部来的大流量进行控制，熔断降级的视⻆是对内部问题进行处理。

Sentinel 降级会在调用链路中某个资源出现不稳定状态时(例如调用超时或异常比例升高)，对这个资 源的调用进行限制，让请求快速失败，避免影响到其它的资源而导致级联错误。当资源被降级后，在接 下来的降级时间窗口之内，对该资源的调用都自动熔断.

**=======>>>> 这里的降级其实是Hystrix中的熔断 还记得当时Hystrix的工作流程么**

![image-20200803125324556](https://tva1.sinaimg.cn/large/007S8ZIlly1ghdj5mw9wbj312k0msn1u.jpg)

策略

Sentinel不会像Hystrix那样放过一个请求尝试自我修复，就是明明确确按照时间窗口来，熔断触发后， 时间窗口内拒绝请求，时间窗口后就恢复。

- RT(平均响应时间 )

  当 1s 内持续进入 >=5 个请求，平均响应时间超过阈值(以 ms 为单位)，那么在接下的时间窗口 (以 s 为单位)之内，对这个方法的调用都会自动地熔断(抛出 DegradeException)。注意 Sentinel 默认统计的 RT 上限是 4900 ms，超出此阈值的都会算作 4900 ms，若需要变更此上限 可以通过启动配置项 -Dcsp.sentinel.statistic.max.rt=xxx 来配置。
  
  ![image-20200803125454559](https://tva1.sinaimg.cn/large/007S8ZIlly1ghdj77ch9kj311q078mye.jpg)

- 异常数

  当资源近 1 分钟的异常数目超过阈值之后会进行熔断。注意由于统计时间窗口是分钟级别的，若 timeWindow 小于 60s，则结束熔断状态后仍可能再进入熔断状态。

  时间窗口 >= 60s

  ![image-20200803125613782](https://tva1.sinaimg.cn/large/007S8ZIlly1ghdj8k9cx9j314s08qjsm.jpg)

- 异常比例

  当资源的每秒请求量 >= 5，并且每秒异常总数占通过量的比值超过阈值之后，资源进入降级状 态，即在接下的时间窗口(以 s 为单位)之内，对这个方法的调用都会自动地返回。异常比率的阈 值范围是 [0.0, 1.0] ，代表 0% - 100%。

  ![image-20200803125653836](https://tva1.sinaimg.cn/large/007S8ZIlly1ghdj99karsj313k07adh3.jpg)

##### **7.2.8 Sentinel** 自定义兜底逻辑

@SentinelResource注解类似于Hystrix中的@HystrixCommand注解

@SentinelResource注解中有两个属性需要我们进行区分，blockHandler属性用来指定不满足Sentinel 规则的降级兜底方法，fallback属性用于指定Java运行时异常兜底方法

- 在API接口资源处配置

- 自定义兜底逻辑类

  注意:兜底类中的方法为static静态方法

##### **7.2.9** 基于 **Nacos** 实现 **Sentinel** 规则持久化

目前，Sentinel Dashboard中添加的规则数据存储在内存，微服务停掉规则数据就消失，在生产环境下

不合适。我们可以将Sentinel规则数据持久化到Nacos配置中心，让微服务从Nacos获取规则数据。

![image-20200803131240791](https://tva1.sinaimg.cn/large/007S8ZIlly1ghdjpok4ygj316s0mc0vu.jpg)

- 自动投递微服务的pom.xml中添加依赖

- 自动投递微服务的application.yml中配置Nacos数据源

- Nacos Server中添加对应规则配置集(public命名空间—>DEFAULT_GROUP中添加) 

- 流控规则配置集 lagou-service-autodeliver-flow-rules

  所有属性来自源码**FlowRule**类

  - resource:资源名称
  - limitApp:来源应用
  -  grade:阈值类型 0 线程数 1 QPS
  -  count:单机阈值
  - strategy:流控模式，0 直接 1 关联 2 链路 
  - controlBehavior:流控效果，0 快速失败 1 Warm Up 2 排队等待 
  - clusterMode:true/false 是否集群

  降级规则配置集 lagou-service-autodeliver-degrade-rules

  **所有属性来自源码DegradeRule类**

  - resource:资源名称
  - grade:降级策略 0 RT 1 异常比例 2 异常数
  - count:阈值
  - timeWindow:时间窗

  **Rule** 源码体系结构

- 注意

  1)一个资源可以同时有多个限流规则和降级规则，所以配置集中是一个json数组

  2)Sentinel控制台中修改规则，仅是内存中生效，不会修改Nacos中的配置值，重启后恢复原来 的值; Nacos控制台中修改规则，不仅内存中生效，Nacos中持久化规则也生效，重启后规则依然 保持

#### **7.3 Nacos + Sentinel + Dubbo** 三剑合璧

改造“自动投递微服务”和“简历微服务”，删除OpenFeign 和 Ribbon，使用Dubbo RPC 和 Dubbo LB 首先，需要删除或者注释掉父工程中的热部署依赖

##### **7.3.1** 服务提供者工程改造

- 提取dubbo服务接口工程，lagou-service-dubbo-api
- 改造提供者工程(简历微服务)
  - pom文件添加spring cloud + dubbo整合的依赖，同时添加dubbo服务接口工程依赖
  - 删除原有ResumeService接口，引入dubbo服务接口工程中的ResumeService接口，适当调 整代码，在service的实现类上添加dubbo的@Service注解
  - application.yml或者bootstrap.yml配置文件中添加dubbo配置
  - 另外增加一项配置
  - 运行发布之后，会发现Nacos控制台已经有了服务注册信息,从元数据中可以看出,是dubbo注 册上来的

##### **7.3.2** 服务消费者工程改造

接下来改造服务消费者工程—>自动投递微服务

- pom.xml中删除OpenFeign相关内容

- application.yml配置文件中删除和Feign、Ribbon相关的内容;

- 代码中删除Feign客户端内容; pom.xml添加内容和服务提供者一样

- application.yml配置文件中添加dubbo相关内容

  同样，也配置下spring.main.allow-bean-definition-overriding=true

- Controller代码改造，其他不变

运行发布之后，同样会发现Nacos控制台已经有了服务注册信息

#### **7.4 SCA** 小结

1)因为内容重叠，SCA 中的分布式事务解决方案 Seata 会在紧接着的Mysql课程中讲解。 

2)SCA实际上发展了三条线

- 第一条线:开源出来一些组件

-  第二条线:阿里内部维护了一个分支，自己业务线使用 

- 第三条线:阿里云平台部署一套，付费使用 

  从战略上来说，SCA更是为了贴合阿里云。

  目前来看，开源出来的这些组件，推广及普及率不高，社区活跃度不高，稳定性和体验度上仍需进 一步提升，根据实际使用来看Sentinel的稳定性和体验度要好于Nacos。