# Spring MVC 应用

1. Spring MVC 应用

2. Spring MVC 高级技术（拦截器，异常处理器）

3. 手写MVC 框架

4. SpringMVC 源码深度剖析

5. SSM 整合



SpringMVC 三大件：

1.springMVC的视图解析器  org.springframework.web.servlet.view.InternalResourceViewResolver。

```xml
<!--
     自动注册最合适的处理器映射器，处理器适配器(调用handler方法)
 -->
<mvc:annotation-driven/>
```

2.处理器映射器。

3.处理器适配器。



SpringMVC 请求流程：

![image-20200513133748081](https://tva1.sinaimg.cn/large/007S8ZIlly1geqrmklobej31760o8gw5.jpg)



SpringMVC九大组件：

- HandlerMapping(处理器映射器) 

HandlerMapping 是用来查找 Handler 的，也就是处理器，具体的表现形式可以是类，也可以是 方法。比如，标注了@RequestMapping的每个方法都可以看成是一个Handler。Handler负责具 体实际的请求处理，在请求到达后，HandlerMapping 的作用便是找到请求相应的处理器 Handler 和 Interceptor. 

- HandlerAdapter(处理器适配器) 

HandlerAdapter 是一个适配器。因为 Spring MVC 中 Handler 可以是任意形式的，只要能处理请 求即可。但是把请求交给 Servlet 的时候，由于 Servlet 的方法结构都是 doService(HttpServletRequest req,HttpServletResponse resp)形式的，要让固定的 Servlet 处理 方法调用 Handler 来进行处理，便是 HandlerAdapter 的职责。 

- HandlerExceptionResolver

HandlerExceptionResolver 用于处理 Handler 产生的异常情况。它的作用是根据异常设置 

ModelAndView，之后交给渲染方法进行渲染，渲染方法会将 ModelAndView 渲染成⻚面。

-  ViewResolver 

ViewResolver即视图解析器，用于将String类型的视图名和Locale解析为View类型的视图，只有一 个resolveViewName()方法。从方法的定义可以看出，Controller层返回的String类型视图名 viewName 最终会在这里被解析成为View。View是用来渲染⻚面的，也就是说，它会将程序返回 的参数和数据填入模板中，生成html文件。ViewResolver 在这个过程主要完成两件事情: ViewResolver 找到渲染所用的模板(第一件大事)和所用的技术(第二件大事，其实也就是找到 视图的类型，如JSP)并填入参数。默认情况下，Spring MVC会自动为我们配置一个 InternalResourceViewResolver,是针对 JSP 类型视图的。 

- RequestToViewNameTranslator 


RequestToViewNameTranslator 组件的作用是从请求中获取 ViewName.因为 ViewResolver 根据 ViewName 查找 View，但有的 Handler 处理完成之后,没有设置 View，也没有设置 ViewName， 便要通过这个组件从请求中查找 ViewName。 

- LocaleResolver 


ViewResolver 组件的 resolveViewName 方法需要两个参数，一个是视图名，一个是 Locale。 LocaleResolver 用于从请求中解析出 Locale，比如中国 Locale 是 zh-CN，用来表示一个区域。这 个组件也是 i18n 的基础。 

- ThemeResolver 


ThemeResolver 组件是用来解析主题的。主题是样式、图片及它们所形成的显示效果的集合。 Spring MVC 中一套主题对应一个 properties文件，里面存放着与当前主题相关的所有资源，如图 片、CSS样式等。创建主题非常简单，只需准备好资源，然后新建一个“主题名.properties”并将资 源设置进去，放在classpath下，之后便可以在⻚面中使用了。SpringMVC中与主题相关的类有 ThemeResolver、ThemeSource和Theme。ThemeResolver负责从请求中解析出主题名， ThemeSource根据主题名找到具体的主题，其抽象也就是Theme，可以通过Theme来获取主题和 具体的资源。 

- MultipartResolver 


MultipartResolver 用于上传请求，通过将普通的请求包装成 MultipartHttpServletRequest 来实 现。MultipartHttpServletRequest 可以通过 getFile() 方法 直接获得文件。如果上传多个文件，还 可以调用 getFileMap()方法得到Map<FileName，File>这样的结构，MultipartResolver 的作用就 是封装普通的请求，使其拥有文件上传的功能。 

- FlashMapManager 


FlashMap 用于重定向时的参数传递，比如在处理用户订单时候，为了避免重复提交，可以处理完 post请求之后重定向到一个get请求，这个get请求可以用来显示订单详情之类的信息。这样做虽然 可以规避用户重新提交订单的问题，但是在这个⻚面上要显示订单的信息，这些数据从哪里来获得 呢?因为重定向时么有传递参数这一功能的，如果不想把参数写进URL(不推荐)，那么就可以通 过FlashMap来传递。只需要在重定向之前将要传递的数据写入请求(可以通过 

ServletRequestAttributes.getRequest()方法获得)的属性OUTPUT_FLASH_MAP_ATTRIBUTE 中，这样在重定向之后的Handler中Spring就会自动将其设置到Model中，在显示订单信息的⻚面 上就可以直接从Model中获取数据。FlashMapManager 就是用来管理 FalshMap 的。 



SpringMVC 的 servlet-mapping的配置：

```xml
<servlet-mapping>
  <servlet-name>springmvc</servlet-name>

  <!--
    方式一：带后缀，比如*.action  *.do *.aaa
           该种方式比较精确、方便，在以前和现在企业中都有很大的使用比例
    方式二：/ 不会拦截 .jsp，但是会拦截.html等静态资源（静态资源：除了servlet和jsp之外的js、css、png等）

          为什么配置为/ 会拦截静态资源？？？
              因为tomcat容器中有一个web.xml（父），你的项目中也有一个web.xml（子），是一个继承关系
                    父web.xml中有一个DefaultServlet,  url-pattern 是一个 /
                    此时我们自己的web.xml中也配置了一个 / ,覆写了父web.xml的配置
          为什么不拦截.jsp呢？
              因为父web.xml中有一个JspServlet，这个servlet拦截.jsp文件，而我们并没有覆写这个配置，
              所以springmvc此时不拦截jsp，jsp的处理交给了tomcat


          如何解决/拦截静态资源这件事？


    方式三：/* 拦截所有，包括.jsp
  -->
  <!--拦截匹配规则的url请求，进入springmvc框架处理-->
  <url-pattern>/</url-pattern>
</servlet-mapping>
```

静态资源的处理：

```xml
<!--静态资源配置，方案一-->
<!--
    原理：添加该标签配置之后，会在SpringMVC上下文中定义一个DefaultServletHttpRequestHandler对象
         这个对象如同一个检查人员，对进入DispatcherServlet的url请求进行过滤筛查，如果发现是一个静态资源请求
         那么会把请求转由web应用服务器（tomcat）默认的DefaultServlet来处理，如果不是静态资源请求，那么继续由
         SpringMVC框架处理
-->
<mvc:default-servlet-handler/>



<!--静态资源配置，方案二，SpringMVC框架自己处理静态资源
    mapping:约定的静态资源的url规则
    location：指定的静态资源的存放位置

-->
<mvc:resources location="classpath:/"  mapping="/resources/**"/>
<mvc:resources location="/WEB-INF/js/" mapping="/js/**"/>
```



SpringMVC 的数据输出机制

```java
/**
 * SpringMVC在handler方法上传入Map、Model和ModelMap参数，并向这些参数中保存数据（放入到请求域），都可以在页面获取到
 *
 * 它们之间是什么关系？
 * 运行时的具体类型都是BindingAwareModelMap，相当于给BindingAwareModelMap中保存的数据都会放在请求域中
 *
 *  Map(jdk中的接口)        Model（spring的接口）
 *
 *  ModelMap(class,实现Map接口)
 *
 *
 *
 *
 *              BindingAwareModelMap继承了ExtendedModelMap，ExtendedModelMap继承了ModelMap,实现了Model接口
 *
 */
```

自定义类型转换器：

定义转换器：

```java
/**
 * @author 应癫
 *  自定义类型转换器
 * S：source，源类型
 * T：target：目标类型
 */
public class DateConverter implements Converter<String, Date> {
    @Override
    public Date convert(String source) {
        // 完成字符串向日期的转换
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");

        try {
            Date parse = simpleDateFormat.parse(source);
            return parse;
        } catch (ParseException e) {
            e.printStackTrace();
        }

        return null;
    }
}
```

注册自定义类型转换器：

```xml
<!--
    自动注册最合适的处理器映射器，处理器适配器(调用handler方法)
-->
<mvc:annotation-driven conversion-service="conversionServiceBean"/>


<!--注册自定义类型转换器-->
<bean id="conversionServiceBean" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
    <property name="converters">
        <set>
            <bean class="com.lagou.edu.converter.DateConverter"></bean>
        </set>
    </property>
</bean>
```



用过滤器处理post乱码问题：

```xml
<!--springmvc提供的针对post请求的编码过滤器-->
<filter>
  <filter-name>encoding</filter-name>
  <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
  <init-param>
    <param-name>encoding</param-name>
    <param-value>UTF-8</param-value>
  </init-param>
</filter>

<filter-mapping>
  <filter-name>encoding</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```

用过滤器处理请求方式的转换：

```xml
<!--配置springmvc请求方式转换过滤器，会检查请求参数中是否有_method参数，如果有就按照指定的请求方式进行转换-->
<filter>
  <filter-name>hiddenHttpMethodFilter</filter-name>
  <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>

<filter-mapping>
  <filter-name>hiddenHttpMethodFilter</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```



SpringMVC 请求返回对JSON的支持：

前端注意设置媒体类型：contentType: 'application/json;charset=utf-8'

@RequstBody  @ResponseBody 

SpringMVC 六大组件：

1.过滤器，2.监听器，3. 拦截器，

**1.1 监听器、过滤器和拦截器对比**

-  Servlet:处理Request请求和Response响应 

- 过滤器(Filter):对Request请求起到过滤的作用，作用在Servlet之前，如果配置为/*可以对所 有的资源访问(servlet、js/css静态资源等)进行过滤处理 

- 监听器(Listener):实现了javax.servlet.ServletContextListener 接口的服务器端组件，它随 Web应用的启动而启动，只初始化一次，然后会一直运行监视，随Web应用的停止而销毁 

       作用一:做一些初始化工作，web应用中spring容器启动ContextLoaderListener 
        
       作用二:监听web中的特定事件，比如HttpSession,ServletRequest的创建和销毁;变量的创建、 销毁和修改      
        
       等。可以在某些动作前后增加处理，实现监控，比如统计在线人数，利用 HttpSessionLisener等。 

- 拦截器(Interceptor):是SpringMVC、Struts等表现层框架自己的，不会拦截 jsp/html/css/image的访问等，只会拦截访问的控制器方法(Handler)。 

       从配置的⻆度也能够总结发现:serlvet、filter、listener是配置在web.xml中的，而interceptor是 配置在表现    
        
       层框架自己的配置文件中的 
        
       **在Handler业务逻辑执行之前拦截一次** 
        
       **在Handler逻辑执行完毕但未跳转⻚面之前拦截一次** 
        
       **在跳转⻚面之后拦截一次** 

![image-20200513155018321](https://tva1.sinaimg.cn/large/007S8ZIlly1geqvgfarnaj31460saqbu.jpg)

SpringMVC 自定义拦截器，并在SpringMVC.xml中把拦截器进行注册。

```java
public class MyIntercepter01 implements HandlerInterceptor {


    /**
     * 会在handler方法业务逻辑执行之前执行
     * 往往在这里完成权限校验工作
     * @param request
     * @param response
     * @param handler
     * @return  返回值boolean代表是否放行，true代表放行，false代表中止
     * @throws Exception
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("MyIntercepter01 preHandle......");
        return true;
    }


    /**
     * 会在handler方法业务逻辑执行之后尚未跳转页面时执行
     * @param request
     * @param response
     * @param handler
     * @param modelAndView  封装了视图和数据，此时尚未跳转页面呢，你可以在这里针对返回的数据和视图信息进行修改
     * @throws Exception
     */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("MyIntercepter01 postHandle......");
    }

    /**
     * 页面已经跳转渲染完毕之后执行
     * @param request
     * @param response
     * @param handler
     * @param ex  可以在这里捕获异常
     * @throws Exception
     */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("MyIntercepter01 afterCompletion......");
    }
}
```

注册拦截器：

```xml
<mvc:interceptors>
    <!--拦截所有handler-->
    <!--<bean class="com.lagou.edu.interceptor.MyIntercepter01"/>-->
    
    <mvc:interceptor>
        <!--配置当前拦截器的url拦截规则，**代表当前目录下及其子目录下的所有url-->
        <mvc:mapping path="/**"/>
        <!--exclude-mapping可以在mapping的基础上排除一些url拦截-->
        <!--<mvc:exclude-mapping path="/demo/**"/>-->
        <bean class="com.lagou.edu.interceptor.MyIntercepter01"/>
    </mvc:interceptor>


    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <bean class="com.lagou.edu.interceptor.MyIntercepter02"/>
    </mvc:interceptor>
    
</mvc:interceptors>
```

多个拦截器执行流程：

![image-20200513160427435](https://tva1.sinaimg.cn/large/007S8ZIlly1geqvv576xej30ut0u0q9r.jpg)



SpringMVC 文件上传。

SpringMVC 异常处理机制：

SpringMVC 重定向问题。

	return "forward:/hello"； 转发。
	
	return "redirect:/hello";    重定向。



# SpringMVC 源代码剖析

SpringMVC处理请求的流程即为 org.springframework.web.servlet.DispatcherServlet#doDispatch方法的执行过程，其中步骤 2、3、4、5是核心步骤 

**1)调用getHandler()获取到能够处理当前请求的执行链 HandlerExecutionChain(Handler+拦截器)** 

    但是如何gethandler的 ？ 后面进行分析
**2)调用getHandlerAdapter();获取能够执行1)中Handler的适配器**
    但是如何去getHandlerAdapter的?后面进行分析
**3)适配器调用Handler执行ha.handle(总会返回一个ModelAndView对象)**
**4)调用processDispatchResult()方法完成视图渲染跳转**



SpringMVC九大组件初始化 

1)在DispatcherServlet中定义了九个属性，每一个属性都对应一种组件 

```java
/** MultipartResolver used by this servlet. */ // 多部件解析器
 @Nullable
 private MultipartResolver multipartResolver; 

/** LocaleResolver used by this servlet. */ // 区域化 国际化解析器
 @Nullable
 private LocaleResolver localeResolver; 

/** ThemeResolver used by this servlet. */ // 主题解析器
 @Nullable
 private ThemeResolver themeResolver; 

/** List of HandlerMappings used by this servlet. */ // 处理器映射器组件
 @Nullable
 private List<HandlerMapping> handlerMappings; 

/** List of HandlerAdapters used by this servlet. */ // 处理器适配器组件
 @Nullable 

private List<HandlerAdapter> handlerAdapters;

/** List of HandlerExceptionResolvers used by this servlet. */ // 异常解析器组件
 @Nullable
 private List<HandlerExceptionResolver> handlerExceptionResolvers; 

/** RequestToViewNameTranslator used by this servlet. */ // 默认视图名转换器组件
 @Nullable
 private RequestToViewNameTranslator viewNameTranslator; 

/** FlashMapManager used by this servlet. */ // flash属性管理组件
 @Nullable
 private FlashMapManager flashMapManager; 

/** List of ViewResolvers used by this servlet. */ // 视图解析器
 @Nullable
private List<ViewResolver> viewResolvers;
```

# SSM 整合

# SpringData Jpa 框架

Spring Data Jpa 是应用于Dao层的一个框架，简化数据库开发的，作用和Mybatis框架一样，但是在使
用方式和底层机制是有所不同的。最明显的一个特点，Spring Data Jpa 开发Dao的时候，很多场景我们
连sql语句都不需要开发。由Spring出品。

![image-20200514113308385](https://tva1.sinaimg.cn/large/007S8ZIlly1gertn49mjsj314g0oiwl2.jpg)



JPA 是一套规范，内部是由接口和抽象类组成的，Hiberanate 是一套成熟的 ORM 框架，而且 Hiberanate 实现了 JPA 规范，所以可以称 Hiberanate 为 JPA 的一种实现方式，我们使用 JPA 的 API 编 程，意味着站在更高的⻆度去看待问题(面向接口编程)。 

Spring Data JPA 是 Spring 提供的一套对 JPA 操作更加高级的封装，是在 JPA 规范下的专⻔用来进行数 据持久化的解决方案。 

**SpringData jpa 具体工作还需要 hibernate 去实现。**



![image-20200514122332237](https://tva1.sinaimg.cn/large/007S8ZIlly1gerv3l3l64j30ni06ijt3.jpg)

![image-20200514122349123](https://tva1.sinaimg.cn/large/007S8ZIlly1gerv3vw7j0j30n80nq45g.jpg)

# SpringDataJpa 源码

