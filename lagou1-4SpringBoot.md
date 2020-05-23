#  1.SpringBoot基础回顾

1. 约定优于配置（核心思想）

2. spring 核心一：起步依赖

   起步依赖本质是一个Maven项目对象模型（Project Object Mode,POM）,定义了对其他库的传递依赖，这些东西加在一起即支持某项功能。

3. spring 核心二：自动配置

   springboot的自动配置，指的是springboot，会自动将一些配置类的bean注册进ioc容器，我们可以需要的地方使用@autowired 或者 @resource 等注解来使用它。

   ”自动“的表现形式就是我们只要引我们想用功能的包，相关的配置我们无安全不用管，springBoot会自动注入这些配置Bean，我们直接使用这些bean即可

   springboot：简单、快捷、方便地搭建项目；对主流开发框架的无配置集成；极大提高了开发、部署效率。

4. 使用Spring Initializr 方式构建Spring boot 项目

![image-20200522092151726](https://tva1.sinaimg.cn/large/007S8ZIlly1gf0yt0u23uj31d80u0gte.jpg)

5. springboot 单元测试与热部署

6. spirngboot application.properties 全局配置文件。

   可以修改spring框架默认的配置属性值，也可把自定义配置的属性注入到pojo对象中。

7. springboot application.yaml 全局配置文件

8. spirngboot 配置文件的注入类型回顾

   @ConfigurationProperties 批量注入

   ```java
   @Component
   @ConfigurationProperties(prefix = "person")
   public class Person {
   
   private int id;
    //    setXX()  
    public void setId(int id) { 
   
           this.id = id;
       }
   
   } 
   ```

   @Vule 单个属性注入

   ```java
   @Component 
   
   public class Student {
   
   @Value("${person.id}") 
   private int id; 
   
   @Value("${person.name}") private String name; 
   //   
   
   //  toString } 
   ```

   注意：@Value注解对于包含Map集合、对象以及YAML文件格式的行内写法的配置文件的属性注入都不支持，如果赋值会出现错误。

   9. SpringBoot自定义配置文件和配置类

      @PropertySource 和 @Configuration

      ```java
      @Component
      @PropertySource("classpath:testa.properties")  //配置自定义配置文件的名称及位置
      @ConfigurationProperties(prefix = "test")
      public class MyProperties {
      
      
          private int id;
          private String name;
      
          @Override
          public String toString() {
              return "MyProperties{" +
                      "id=" + id +
                      ", name='" + name + '\'' +
                      '}';
          }
      
          public int getId() {
              return id;
          }
      
          public void setId(int id) {
              this.id = id;
          }
      
          public String getName() {
              return name;
          }
      
          public void setName(String name) {
              this.name = name;
          }
      }
      ```

   ```java
   @Configuration //标明该类为配置类
   public class MyConfig {
   
       @Bean(name = "iservice")   //将返回值对象作为组件添加到spring容器中，标识id默认是方法名
       public MyService myService() {
   
           return new MyService();
       }
   
   
   }
   ```

9. SpringBoot 随机值设置及参数引用

   ```properties
   propertiestom.age=${random.int[10,20]}
   ```

   ```properties
   app.name=MyApp
   
   app.description=${app.name}  is a Spring Boot application
   ```

   # 2. SpringBoot 原理深入及源码剖析

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
   ```

   其实是依赖继承和依赖传递。

   spring-boot-starter-web 是一个启动器。springboot 还提供了其他类型的启动器。

   ![image-20200523202848345](https://tva1.sinaimg.cn/large/007S8ZIlly1gf2npa2mdoj31re0u0127.jpg)

   其他的依赖管理启动器。

@AutoConfiguration ,@Import 

![image-20200523215609183](https://tva1.sinaimg.cn/large/007S8ZIlly1gf2q8639xpj31660ncth9.jpg)



@ComponetScan 包扫描器。

11. 自定义start :

      新建maven jar 工程，工程名为zdy-spring-boot-starter  导入依赖

      编写javaBean

      编写配置类MyAutoConfiguration    

      resources   下创建/META-INF/spring.factories  



12. 源码分析

      ![image-20200524001103784](https://tva1.sinaimg.cn/large/007S8ZIlly1gf2u4j3bw9j315u072juj.jpg)

    1.SpringApplication 实例化的初始化创建

    ![image-20200524001235189](https://tva1.sinaimg.cn/large/007S8ZIlly1gf2u64upmdj31780p813g.jpg)



    2.项目初始化启动

    ![image-20200524001345935](https://tva1.sinaimg.cn/large/007S8ZIlly1gf2u7daqqzj30ut0u0qf7.jpg)

    ![image-20200524001426138](https://tva1.sinaimg.cn/large/007S8ZIlly1gf2u81m4mvj310s0sa12k.jpg)

    ![image-20200524001444240](https://tva1.sinaimg.cn/large/007S8ZIlly1gf2u8cyjpwj30yo0u0e2v.jpg)























                   




