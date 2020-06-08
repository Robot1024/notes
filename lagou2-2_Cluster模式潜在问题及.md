分布式集群架构场景化解方案

主要课程内容

第一部分：一致性Hash算法

第二部分：集群时钟同步问题

第三部分：分布式Id解决方案

第四部分：分布式调度问题

第五部分：Session 共享问题



# 第一部分 一致性Hash算法

Hash算法较多的应用在数据存储和查找领域，最经典的就是Hash表，它的查询效率非常之高，其中的 哈希算法如果设计的比较ok的话，那么Hash表的数据查询时间复杂度可以接近于O(1)，示例 

需求:提供一组数据 1,5,7,6,3,4,8，对这组数据进行存储，然后随便给定一个数n，请你判断n是否存在 于刚才的数据集中? 



ngx_http_upstream_consistent_hash 模块是一个负载均衡器，使用一个内部一致性hash算法来选择 

合适的后端节点。
 该模块可以根据配置参数采取不同的方式将请求均匀映射到后端机器，
 consistent_hash $remote_addr:可以根据客户端ip映射
 consistent_hash $request_uri:根据客户端请求的uri映射
 consistent_hash $args:根据客户端携带的参数进行映
 ngx_http_upstream_consistent_hash 模块是一个第三方模块，需要我们下载安装后使用 

![image-20200607135135144](https://tva1.sinaimg.cn/large/007S8ZIlly1gfjoineq4qj30ui0kydla.jpg)

![image-20200607141812276](https://tva1.sinaimg.cn/large/007S8ZIlly1gfjpaa5ce8j30xy0feq6w.jpg)



# 第四部分 分布式调度问题

定时任务形式:每隔一定时间/特定某一时刻执行 例如: 

```shell
  订单审核、出库
  订单超时自动取消、支付退款
  礼券同步、生成、发放作业
  物流信息推送、抓取作业、退换货处理作业
  数据积压监控、日志监控、服务可用性探测作业
  定时备份数据
  金融系统每天的定时结算
  数据归档、清理作业
  报表、离线数据分析作业
```



本质不同 定时任务作业是时间驱动，而MQ是事件驱动; 

  时间驱动是不可代替的，比如金融系统每日的利息结算，不是说利息来一条(利息到来事件)就算
  一下，而往往是通过定时任务批量计算;

所以，定时任务作业更倾向于批处理，MQ倾向于逐条处理; 



Elastic-Job是当当网开源的一个分布式调度解决方案，基于Quartz二次开发的，由两个相互独立的子项 目Elastic-Job-Lite和Elastic-Job-Cloud组成。我们要学习的是 Elastic-Job-Lite，它定位为轻量级无中心 化解决方案，使用Jar包的形式提供分布式任务的协调服务，而Elastic-Job-Cloud子项目需要结合Mesos 以及Docker在云环境下使用。 

Elastic-Job的github地址:https://github.com/elasticjob 



主要功能介绍 

1. 分布式调度协调 
2.   在分布式环境中，任务能够按指定的调度策略执行，并且能够避免同一任务多实例重复执行
3. 丰富的调度策略 基于成熟的定时任务作业框架Quartz cron表达式执行定时任务 
4. 弹性扩容缩容 当集群中增加某一个实例，它应当也能够被选举并执行任务;当集群减少一个实例 时，它所执行的任务能被转移到别的实例来执行。 
5. 失效转移 某实例在任务执行失败后，会被转移到其他实例执行 
6. 错过执行作业重触发 若因某种原因导致作业错过执行，自动记录错过执行的作业，并在上次作业 完成后自动触发。 
7. 支持并行调度 支持任务分片，任务分片是指将一个任务分为多个小任务项在多个实例同时执行。 
8. 作业分片一致性 当任务被分片后，保证同一分片在分布式环境中仅一个执行实例。 



**Elastic-Job-Lite轻量级去中心化的特点**

![image-20200607193455906](https://tva1.sinaimg.cn/large/007S8ZIlly1gfjyfv1o5bj31500ncjw6.jpg)



# 第五部分 Session共享问题

Session共享，Session集中存储(推荐)
Session的本质就是缓存，那Session数据为什么不交给专业的缓存中间件呢?比如Redis

![image-20200607212255520](https://tva1.sinaimg.cn/large/007S8ZIlly1gfk1k7un4qj314y0fsjtl.jpg)



优点: 

能适应各种负载均衡策略

 服务器重启或者宕机不会造成Session丢失

 扩展能力强
 适合大集群数量使用 

缺点: 

对应用有入侵，引入了和Redis的交互代码
 Spring Session使得基于Redis的Session共享应用起来非常之简单

 1)引入Jar 

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.session</groupId>
  <artifactId>spring-session-data-redis</artifactId>
</dependency>
```

2)配置redis

```xml
spring.redis.database=0
spring.redis.host=127.0.0.1
spring.redis.port=6379
```

3)添加注解

![image-20200607212625991](https://tva1.sinaimg.cn/large/007S8ZIlly1gfk1nv4qhtj314c06mmzg.jpg)

源码示意(了解)

![image-20200607212704016](https://tva1.sinaimg.cn/large/007S8ZIlly1gfk1ok0yxbj315y0f0grk.jpg)



该注解可以创建一个过滤器使得SpringSession替代HttpSession发挥作用，找到那个过滤器!








