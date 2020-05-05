# Spring 框架介绍

1.spring 核心结构

![image-20200504203416589](https://tva1.sinaimg.cn/large/007S8ZIlly1gegp39ggq3j30wo0sg0ux.jpg)

# 手写Ioc 和 AOP

1. **ThreadLocal 使用场景**

   - 每个线程需要有自己单独的实例 （实例的生命周期与线程一致，此实例被每个线程单独持有）
   - 实例需要在多个方法中共享，但不希望被多线程共享（实例要再多个方法方法中共享，单不希望被多线程共享，这样也满足了线程安全。可以理解一个变量通过参数的形式，在线程方法调用栈中持续向下层传播，但是这样写代码不优雅，所有用一个与线程绑定的容器进行数据共享，这就是ThreadLocal的作用）

2. **ThreadLocal实现原理**

   首先 ThreadLocal 是一个泛型类，保证可以接受任何类型的对象。

   因为一个线程内可以存在多个 ThreadLocal 对象，所以其实是 ThreadLocal 内部维护了一个 Map ，这个 Map 不是直接使用的 HashMap ，而是 ThreadLocal 实现的一个叫做 ThreadLocalMap 的静态内部类。而我们使用的 get()、set() 方法其实都是调用了这个ThreadLocalMap类对应的 get()、set() 方法。例如下面的 set 方法：

```java
public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
```


```java
public T get() {   
        Thread t = Thread.currentThread();   
        ThreadLocalMap map = getMap(t);   
        if (map != null)   
            return (T)map.get(this);   

        // Maps are constructed lazily.  if the map for this thread   
        // doesn't exist, create it, with this ThreadLocal and its   
        // initial value as its only entry.   
        T value = initialValue();   
        createMap(t, value);   
        return value;   
    }
```

3. **内存泄漏问题**

实际上 ThreadLocalMap 中使用的 key 为 ThreadLocal 的弱引用，弱引用的特点是，如果这个对象只存在弱引用，那么在下一次垃圾回收的时候必然会被清理掉。

所以如果 ThreadLocal 没有被外部强引用的情况下，在垃圾回收的时候会被清理掉的，这样一来 ThreadLocalMap中使用这个 ThreadLocal 的 key 也会被清理掉。但是，value 是强引用，不会被清理，这样一来就会出现 key 为 null 的 value。

ThreadLocalMap实现中已经考虑了这种情况，在调用 set()、get()、remove() 方法的时候，会清理掉 key 为 null 的记录。如果说会出现内存泄漏，那只有在出现了 key 为 null 的记录后，没有手动调用 remove() 方法，并且之后也不再调用 get()、set()、remove() 方法的情况下。



**使用场景**

如上文所述，ThreadLocal 适用于如下两种场景

- 每个线程需要有自己单独的实例
- 实例需要在多个方法中共享，但不希望被多线程共享

对于第一点，每个线程拥有自己实例，实现它的方式很多。例如可以在线程内部构建一个单独的实例。ThreadLoca 可以以非常方便的形式满足该需求。

对于第二点，可以在满足第一点（每个线程有自己的实例）的条件下，通过方法间引用传递的形式实现。ThreadLocal 使得代码耦合度更低，且实现更优雅。

1）存储用户Session

一个简单的用ThreadLocal来存储Session的例子：

```java
    private static final ThreadLocal threadSession = new ThreadLocal();

    public static Session getSession() throws InfrastructureException {
        Session s = (Session) threadSession.get();
        try {
            if (s == null) {
                s = getSessionFactory().openSession();
                threadSession.set(s);
            }
        } catch (HibernateException ex) {
            throw new InfrastructureException(ex);
        }
        return s;
    }
```

2）解决线程安全的问题

比如Java7中的SimpleDateFormat不是线程安全的，可以用ThreadLocal来解决这个问题：

```java
public class DateUtil {
    private static ThreadLocal<SimpleDateFormat> format1 = new ThreadLocal<SimpleDateFormat>() {
        @Override
        protected SimpleDateFormat initialValue() {
            return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        }
    };

    public static String formatDate(Date date) {
        return format1.get().format(date);
    }
}
```

这里的DateUtil.formatDate()就是线程安全的了。(Java8里的 **java.time.format.DateTimeFormatter**`是线程安全的，Joda time里的DateTimeFormat也是线程安全的）。`

2. JDK 动态代理参数详解

   ```java
   Proxy.newProxyInstance(proxyFactory.getClass().getClassLoader(), obj.getClass().getInterfaces(),
           new InvocationHandler() {
               @Override
               public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                   Object result = null;
   
                   // 写增强逻辑
                   System.out.println("中介（代理）收取服务费3000元");
                   // 调用原有业务逻辑
                   result = method.invoke(obj,args);
   
                   System.out.println("客户信息卖了3毛钱");
   
                   return result;
               }
           });
   ```

```java
Proxy.newProxyInstance(obj.getClass().getClassLoader(),obj.getClass().getInterfaces(),
```

```java
Proxy.newProxyInstance(proxyFactory.getClass().getClassLoader(),obj.getClass().getInterfaces(),
```

第一个参数要的是一个classLoader , 开始以为任何classLoader 都是可以的，java 只有一个classLoader ，将对象改成其他自己创建的对象，代码也不会报错。

但是使用 Object.getClass().getClassLoader() 就会报错，String.getClass().getClassLoader 也会报错

因为拿不到这些对象的ClassLoader ，java 在运行的时候是有3个classLoader的。

第二参数是被代理对象的，实现的接口。这个没是好说的了这个参数可以是数组，一些列的接口。代理对象是通过接口方法来调动方法的，所以被代理实例里的非实现接口方法是不能被动态代理的。



```java
Object invoke(Object proxy, Method method, Object[] args)
```

第一个参数生产的代理对象，第二参数回调方法，第三个参数被代理对象的参数列表。



classLoader 相关知识：

> 为什么Java中有三种基本的类加载器？ 引言
>
> Java中有三种基础的类加载器
> BootStrap,
> Extension,
> System
>
> 他们都有一个职能，就是从不同的包中加载类。
>
> 但是一个类加载器完全可以加载所有的类，为什么要有3种基础的类型的类加载器呢？
>
> ##  最佳答案
>
> Java中有三种基础的类加载器主要为了安全。
>
> 1.2版本的JVM中，只有一个类加载器，就是现在的“Bootstrap”类加载器。
>
> 类加载器加载类的方式是，加载器先调用父加载器对类进行加载，如果父加载器找不到该类，此加载器才会去加载该类。
>
> 最关键的是， 除非是同一个类加载器加载的类 ，否则JVM不会保证包访问级别（如果不指明private/public或protected，则方法和属性具有包访问级别）。
>
> ![img](https://img-blog.csdnimg.cn/20181218160442727.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3c2MDUyODMwNzM=,size_16,color_FFFFFF,t_70)
>
> 因此，假如用户调用他编写的java.lang.MyClass类。理论上该类可以访问和改变java.lang包下其他类的默认访问修饰符的属性和方法的能力。Java语言本身并没有阻止这种行为。但是JVM则会阻止这种行为，因为java核心类库的java.lang包下的类是由bootstrap类加载器加载的。不是同一个类加载器加载的类等于不具有包级别的访问权限。
>
> 类加载器中的其他安全特性也会阻止这种类型侵入。
>
> 所以为什么有三种基础的类加载器？是因为他们代表三种不同的信任级别。最可信的级别是java核心API类。然后是安装的拓展类，最后才是在类路径中的类（属于你本机的类）。
>
>

> Cglib代理详解与性能测试
>
> **1、概述：**
>
>       常用Cglib代理但没详细研究过MethodInterceptor里method与MethodProxy对象，这两对象都可以调用自身方法那到底谁更快些以及使用区别？
>
> ```
> public Object buildProxy(Object target) {
>      Class<?> proxyClass = object.getClass();
>      if (proxyClass == null) {
>          Enhancer enhancer = new Enhancer();
>          if (targetClass.isInterface()) {
>           enhancer.setInterfaces(new Class[] { targetClass });
>          } else {
>           enhancer.setSuperclass(targetClass);
>          }
>          enhancer.setCallbackType(AsyncMethodInterceptor.class);
>          proxyClass = enhancer.createClass();
>          logger.debug("create proxy class:{}",targetClass);
>      }
>      Enhancer.registerCallbacks(proxyClass,new Callback[]{new AsyncMethodInterceptor(target)});
>      Object proxyObject = null;
>      try{
>          proxyObject = ReflectionHelper.newInstance(proxyClass);
>      }finally{
>          Enhancer.registerStaticCallbacks(proxyClass, null);
>      }
>      return proxyObject;
> }
> public class AsyncMethodInterceptor implements MethodInterceptor {
> 
>     private Object targetObject;     
> 
>     public AsyncMethodInterceptor(Object targetObject) {
>      this.targetObject = targetObject;
>     }
>     @Override
>     public Object intercept(Object target, Method method, Object[] args, MethodProxy 
>          methodProxy)   throws Throwable {
>        
> 
>        // return method.invoke(targetObject,args);    //方法一
>        
>         return methodProxy.invokeSuper(target, args);  //方法二
> 
>     }
> }
> ```
>
>  
>
> **二、两者区别讲解**
>
> 例如：
>
> ```
> @Service
> public class UserService implements IUserService{
>     
>     private final static Logger logger = LoggerFactory.getLogger(UserService.class); 
>     
>     @Async
>     public User test2(User user,long sellpTime){
> 	     logger.debug("测试1正在访问UserService.editUser");
> 	
> 	     return user;
>     } 
> 
>     @Async
>     public AsyncResult<User> test1(int a){
>         User u = test2();
> 	    return new AsyncResult<User>().setData(u);
>     }
>     
> }
> ```
>
>   在用Cglib代理这个UserService过程中发现，如果是使用method.invoke则当在调用test1方法中调用test2是不会被MethodIntercepter拦截的；如果使用MethodProxy.invokeSuper则在调用test1方法中调用test2是会被MethodIntercepter拦截；
>
> **附性能测试结果**：
>
> ![img](http://static.oschina.net/uploads/space/2016/0826/091642_Fhjg_559627.png)



```java
                //不会拦截，代理方法里调用的，同类下的其他方法。
                result = method.invoke(target,objects);
                //会拦截，代理方法里调用的，同类下的其他方法。
                result = methodProxy.invokeSuper(proxyTarget,objects);
                //不会拦截，代理方法里调用的，同类下的其他方法。
                result = methodProxy.invoke(target,objects);
```

测试得到结论，jdk 动态代理不能实现方法内调用的二次拦截。

# Spring Ioc基础知识和高级特性

1. spring 容器启动入口

   ![image-20200505030809505](https://tva1.sinaimg.cn/large/007S8ZIlly1geh0h04wsxj31he0lotip.jpg)

   2. BeanFactory 与 ApplicationContext 区别

      BeanFactory 是Spring 框架中Ioc容器的顶层接口，它只是用来定义一些基础功能，定义一些基础规范，而ApplicationContext是它的一个子接口，所以ApplicationContext是容器的高级接口，比BeanFactory要拥有更多的功能，比如说国际化支持和资源访问（xml，java）等等。

      **启动 IoC 容器的方式** 

      - Java环境下启动IoC容器 

        ClassPathXmlApplicationContext:从类的根路径下加载配置文件(推荐使用) 

        FileSystemXmlApplicationContext:从磁盘路径上加载配置文件 

        AnnotationConfigApplicationContext:纯注解模式下启动Spring容器 

             Web环境下启动IoC容器
             从xml启动容器



```xml
 
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >
<web-app>
  <display-name>Archetype Created Web Application</display-name>
<!--配置Spring ioc容器的配置文件--> 
    <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value> 
    </context-param>
<!--使用监听器启动Spring的IOC容器-->
<listener>
    <listener-class>
        org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
</web-app>
```

从配置类启动容器

```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >
<web-app>
  <display-name>Archetype Created Web Application</display-name>
    <!--告诉ContextloaderListener知道我们使用注解的方式启动ioc容器-->
<context-param> 
<param-name>contextClass</param-name>
    <param-value>
        org.springframework.web.context.support.AnnotationConfigWebApplicationContext
    </param-value>
</context-param>
    <!--配置启动类的全限定类名--> 
    <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>com.lagou.edu.SpringConfig</param-value> 
    </context-param>
 <!--使用监听器启动Spring的IOC容器-->
 <listener> 
   <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
</web-app>
```



> Spring配置文件beans.xml头部配置解释
>
> ```xml
> <?xml version="1.0" encoding="UTF-8"?>
> <beans xmlns="http://www.springframework.org/schema/beans"
>     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
>     xsi:schemaLocation="http://www.springframework.org/schema/beans
>                         http://www.springframework.org/schema/beans/spring-beans.xsd">
>   
>   
> </beans>
> ```
>
> **xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"**
>
>  这个是头文件的规范，也正是因为它，xmlns 和 xsi:schemaLocation 才能一对一对对应的上。
>
> **解释：**
>
> 1、【xmlns="http://www.springframework.org/schema/beans"】
>
> 声明xml文件默认的命名空间，表示未使用其他命名空间的所有标签的默认命名空间。
>
> 2、【xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"】
>
> 声明XML Schema实例，声明后就可以使用schemaLocation属性。
>
> 3、【xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd】
>
> 指定Schema的位置这个属性必须结合命名空间使用。这个属性有两个值，第一个值表示需要使用的命名空间。第二个值表示供命名空间使用的XML schema的位置。
>
> 上面配置的命名空间指定xsd规范文件，这样你在进行下面具体配置的时候就会根据这些xsd规范文件给出相应的提示，比如说每个标签是怎么写的，都有些什么属性是都可以智能提示的，在启动服务的时候也会根据xsd规范对配置进行校验。
>
> 配置技巧：对于属性值的写法是有规律的，中间使用空格隔开，后面的值是前面的补充，也就是说，前面的值是去除了xsd文件后得来的。



实例化Bean的三种方式：

```xml
<!--Spring ioc 实例化Bean的三种方式-->
<!--方式一：使用无参构造器（推荐）-->
<bean id="connectionUtils" class="com.lagou.edu.utils.ConnectionUtils"></bean>

<!--另外两种方式是为了我们自己new的对象加入到SpringIOC容器管理-->
<!--方式二：静态方法-->
<bean id="connectionUtils" class="com.lagou.edu.factory.CreateBeanFactory" factory-method="getInstanceStatic"/>
<!--方式三：实例化方法-->
<bean id="createBeanFactory" class="com.lagou.edu.factory.CreateBeanFactory"></bean>
<bean id="connectionUtils" factory-bean="createBeanFactory" factory-method="getInstance"/>
```



Bean的生命周期：

![image-20200505041221337](https://tva1.sinaimg.cn/large/007S8ZIlly1geh2bujxc2j31fo0u0wvv.jpg)

DI 依赖注入：

```xml
<!--set注入使用property标签，如果注入的是另外一个bean那么使用ref属性，如果注入的是普通值那么使用的是value属性-->
```

(具体实现略)

XML 和 注解相结合模式

自己开发的bean 定义使用xml注解，第三jar中的bean 定义xml 。



@Autowired(推荐使用)

如上代码所示，这样装配回去spring容器中找到类型为AccountDao的类，然后将其注入进来。这 样会产生一个问题，当一个类型有多个bean值的时候，会造成无法选择具体注入哪一个的情况， 这个时候我们需要配合着@Qualifier使用。 

@Qualifier告诉Spring具体去装配哪个对象。 autowired 安装类型注入，qualifier  通过id 去过滤。

```java
public class TransferServiceImpl {
    @Autowired
    @Qualifier(name="jdbcAccountDaoImpl")
    private AccountDao accountDao;
}
```

```
这个时候我们就可以通过类型和名称定位到我们想注入的对象。
```

@Autowired(推荐使用)   默认先安装类型进行装配，如果不能成功，再把成员变量的变量名当做id去装配，如何还不成功，可以使用@Qualifier  去指定bean ID 了。

```java
@Repository("jdbdAccountDao")
public class JdbcAccountDaoImpl implements AccountDao, Test{}
```

JdbcAccountDaoImpl 实现多接口也不影响自动装配。



```xml
<!--开启注解扫描，base-package指定扫描的包路径-->
<context:component-scan base-package="com.lagou.edu"/>

<!--引入外部资源文件-->
<context:property-placeholder location="classpath:jdbc.properties"/>
```



纯注解的方式：需要写一个启动类

```java
// @Configuration 注解表明当前类是一个配置类
@Configuration
@ComponentScan({"com.lagou.edu"})
@PropertySource({"classpath:jdbc.properties"})
/*@Import()*/
public class SpringConfig {

    @Value("${jdbc.driver}")
    private String driverClassName;
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.username}")
    private String username;
    @Value("${jdbc.password}")
    private String password;


    @Bean("dataSource")
    public DataSource createDataSource(){
        DruidDataSource druidDataSource = new DruidDataSource();
        druidDataSource.setDriverClassName(driverClassName);
        druidDataSource.setUrl(url);
        druidDataSource.setUsername(username);
        druidDataSource.setPassword(password);
        return  druidDataSource;
    }
}
```

javaSE 的启动方式:

```java
@Test
public void testIoC() throws Exception {

    // 通过读取classpath下的xml文件来启动容器（xml模式SE应用下推荐）
    ApplicationContext applicationContext = new AnnotationConfigApplicationContext(SpringConfig.class);

    AccountDao accountDao = (AccountDao) applicationContext.getBean("accountDao");

    System.out.println(accountDao);



}
```

web.xml 配置

```xml
<!--告诉ContextloaderListener知道我们使用注解的方式启动ioc容器-->
<context-param>
  <param-name>contextClass</param-name>
  <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
</context-param>
<!--配置启动类的全限定类名-->
<context-param>
  <param-name>contextConfigLocation</param-name>
  <param-value>com.lagou.edu.SpringConfig</param-value>
</context-param>
```

![image-20200505052426139](https://tva1.sinaimg.cn/large/007S8ZIlly1geh4evyxvwj31560ro431.jpg)

Spring IOC 高级特性：

1. lazy-init 延迟加载。

2. FactoryBean  和 BeanFactory 的区别。

   spring 中Bean有两种，一种是普通的Bean ,一种是工厂Bean (FactoryBean) , FactoryBean 可以生成某一个类型的Bean实例（返回给我们）,多用于生成复杂的Bean对象，**其实@Bean 也可以实现这个功能**。
```java
   // 可以让我们自定义Bean的创建过程(完成复杂Bean的定义) 
    public interface FactoryBean<T> { 

   @Nullable 
   // 返回FactoryBean创建的Bean实例，如果isSingleton返回true，则该实例会放到Spring容器 的单例对象缓存池中Map 

   T getObject() throws Exception;

   @Nullable 
   // 返回FactoryBean创建的Bean类型 Class<?> getObjectType(); 
    // 返回作用域是否单例
    default boolean isSingleton() { 

       return true;
     }
   

   } 
```

xml 中的配置
```xml
<bean id="companyBean" class="com.lagou.edu.factory.CompanyFactoryBean"> <property name="companyInfo" value="拉勾,中关村,500"/>
</bean>
```
3. 后置处理器。

Spring提供了两种后处理bean的扩展接口，分别为 BeanPostProcessor 和 

BeanFactoryPostProcessor，两者在使用上是有所区别的。 

工厂初始化(BeanFactory)—> Bean对象 

在BeanFactory初始化之后可以使用BeanFactoryPostProcessor进行后置处理做一些事情 

在Bean对象实例化(并不是Bean的整个生命周期完成)之后可以使用BeanPostProcessor进行后置处 理做一些事情 

注意:对象不一定是springbean，而springbean一定是个对象 SpringBean的生命周期

![image-20200505054718640](https://tva1.sinaimg.cn/large/007S8ZIlly1geh52nm2vjj31eo0p2187.jpg)

BeanDefinition  每个节点加载成xml ,BeanDefinition 抽象成xml 里的每一个bean。

# Spring Ioc源码分析

1. 好出：提高培养代码架构思维，深入理解框架
2. 原则：定焦原则，抓主线，宏观原则，站在上帝角度。关注源码结构和业务流程（淡化具体某行代码的编写细节）
3. 读源码的方法和技巧：断点(观察调用栈)，反调（Find Usages）,经验。

 

![3b3a810f9844885e31a06ad9b4001e4](https://tva1.sinaimg.cn/large/007S8ZIlly1gehgyiqc6lg30bl061mxh.gif)

 

 

编译⼯程（顺序：core-oxm-context-beans-aspects-aop） 

⼯程—>tasks—>compileTestJava

 

新建spring-model【在项目上右击新建model】

之后将给的spring-lagou的代码、xml、build.gradle 内容复制到新建的model spring-lagou工程，测试运行

还遇到各种问题：首先一定要换阿里厂库，要不会被虐哭。

碰到的问题按照word文档一步一步解决。

1. spring Ioc容器初始化主流程

   ```java
   // ApplicationContext是容器的高级接口，BeanFacotry（顶级容器/根容器，规范了/定义了容器的基础行为）
   // Spring应用上下文，官方称之为 IoC容器（错误的认识：容器就是map而已；准确来说，map是ioc容器的一个成员，
   // 叫做单例池, singletonObjects,容器是一组组件和过程的集合，包括BeanFactory、单例池、BeanPostProcessor等以及之间的协作流程）
   ```

![image-20200505170338398](https://tva1.sinaimg.cn/large/007S8ZIlly1gehomewtc7j31c40q010i.jpg)

2. 主流程初始化分析

   ```java
   /**
    * Ioc容器创建管理Bean对象的，Spring Bean是有生命周期的
    * 构造器执行、初始化方法执行、Bean后置处理器的before/after方法、：AbstractApplicationContext#refresh#finishBeanFactoryInitialization
    * Bean工厂后置处理器初始化、方法执行：AbstractApplicationContext#refresh#invokeBeanFactoryPostProcessors
    * Bean后置处理器初始化：AbstractApplicationContext#refresh#registerBeanPostProcessors
    */
   ```

3. refresh 方法分析

   创建工厂，用工厂实例化bean对象，初始化钩子方法，FactoryPostProcessors，BeanPostProcessors，

   在这些时间节点上，可以让用户重写方法，进行扩展。

4. BeanFactory 创建流程

    ![image-20200505173845628](https://tva1.sinaimg.cn/large/007S8ZIlly1gehpmt3umpj30uj0u0wix.jpg)

5. BeanDefinition加载解析注册的子流程
  ![image-20200505175739303](/var/folders/8p/526n2gy97_b00s_7jphrlml00000gn/T/abnerworks.Typora/image-20200505175739303.png)

6. Bean对象创建流程

7. lazy-init 延迟加载机制的原理

    初始化的时候不会创建，当第一次进行getBean时候才进行初始化并依赖注入

8. 循环依赖问题

     **单例 bean 构造器参数循环依赖(无法解决)**
     **prototype 原型 bean循环依赖(无法解决)**

![image-20200505182220850](https://tva1.sinaimg.cn/large/007S8ZIlly1gehqw737nlj31km0ogqcw.jpg)

# Spring AOP 基础知识和高级特性

1.AOP 相关术语

![image-20200505184533326](https://tva1.sinaimg.cn/large/007S8ZIlly1gehrkez7wdj30vu0sawqa.jpg)

2. 在Spring的AOP配置中，也和IoC配置一样，支持3类配置方式。

    第一类:使用XML配置 

    第二类:使用XML+注解组合配置 

    第三类:使用纯注解配置

3. ```xml
   <!--进行aop相关的xml配置,配置aop的过程其实就是把aop相关术语落地-->
   <!--横切逻辑bean-->
   <!--<bean id="logUtils" class="com.lagou.edu.utils.LogUtils"></bean>
   <!--使用config标签表明开始aop配置,在内部配置切面aspect-->
   
   <!--aspect   =    切入点（锁定方法） + 方位点（锁定方法中的特殊时机）+ 横切逻辑 -->
   <aop:config>
       <aop:aspect id="logAspect" ref="logUtils">
   
           <!--切入点锁定我们感兴趣的方法，使用aspectj语法表达式-->
           <!--<aop:pointcut id="pt1" expression="execution(* *..*.*(..))"/>-->
           <aop:pointcut id="pt1" expression="execution(* com.lagou.edu.service.impl.TransferServiceImpl.*(..))"/>
   
   
           <!--方位信息,pointcut-ref关联切入点-->
           <!--aop:before前置通知/增强-->
           <aop:before method="beforeMethod" pointcut-ref="pt1"/>
           <!--aop:after，最终通知，无论如何都执行-->
           <!--aop:after-returnning，正常执行通知-->
           <aop:after-returning method="successMethod" returning="retValue"/>
           <!--aop:after-throwing，异常通知-->
   
           <aop:around method="arroundMethod" pointcut-ref="pt1"/>
   
       </aop:aspect>
   </aop:config>-->
   ```

4. xml  和  注解相结合的方式

5. 事务特性并发问题隔离级别

   原子性，一致性，隔离性，持久性

   隔离级别解决事务并发的问题：脏读，不可重复读，虚读

   数据库定义四种隔离级别：

   Serializable(串行化):可避免脏读、不可重复读、虚读情况的发生。(串行化) 最高 

   Repeatable read(可重复读):可避免脏读、不可重复读情况的发生。(幻读有可能发生) 第二 

   **(可重复读)该机制下会对要update的行进行加锁** 

   Read committed(读已提交):可避免脏读情况发生。不可重复读和幻读一定会发生。 第三 

   Read uncommitted(读未提交):最低级别，以上情况均无法保证。(读未提交) 最低 

6. 事务的传播行为

   事务往往在service层进行控制，如果出现service层方法A调用了另外一个service层方法B，A和B方法本 身都已经被添加了事务控制，那么A调用B的时候，就需要进行事务的一些协商，这就叫做事务的传播行 为。 

   A调用B，我们站在B的⻆度来观察来定义事务的传播行为 

   | PROPAGATION_REQUIRED      | `如果当前没有事务，就新建一个事务，如果已经存在一个事务中， 加入到这个事务中。这是最常⻅的选择。 ` |
   | ------------------------- | ------------------------------------------------------------ |
   | PROPAGATION_SUPPORTS      | `支持当前事务，如果当前没有事务，就以非事务方式执行。 `      |
   | PROPAGATION_MANDATORY     | `使用当前的事务，如果当前没有事务，就抛出异常。 `            |
   | PROPAGATION_REQUIRES_NEW  | `新建事务，如果当前存在事务，把当前事务挂起。 `              |
   | PROPAGATION_NOT_SUPPORTED | `以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。 ` |
   | PROPAGATION_NEVER         | `以非事务方式执行，如果当前存在事务，则抛出异常。 `          |
   | PROPAGATION_NESTED        | 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则 执行与PROPAGATION_REQUIRED类似的操作。 |

   在mysql 的 Repeatable read(可重复读) 级别下，不会产生幻读。因为有间隙锁的存在。详解见「为知笔记」

7. 声明式事务的xml 配置

   ```xml
   <!-- <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <!--定制事务细节，传播行为、隔离级别等-->
        <tx:attributes>
            <!--一般性配置-->
            <tx:method name="*" read-only="false" propagation="REQUIRED" isolation="DEFAULT" timeout="-1"/>
            <!--针对查询的覆盖性配置-->
            <tx:method name="query*" read-only="true" propagation="SUPPORTS"/>
        </tx:attributes>
    </tx:advice>
   
    <aop:config>
        <!--advice-ref指向增强=横切逻辑+方位-->
        <aop:advisor advice-ref="txAdvice" pointcut="execution(* com.lagou.edu.service.impl.TransferServiceImpl.*(..))"/>
    </aop:config>-->
   ```

   read-only 详解：

   ![image-20200505201211838](https://tva1.sinaimg.cn/large/007S8ZIlly1gehu2gu1ujj30ne0csage.jpg)


# Spirng AOP 源码分析

1. 创建代理对象的后置处理器AbstractAutoProxyCreator#postProcessAfterInitialization

2. 晕了不知道写啥好了，就不写了。



