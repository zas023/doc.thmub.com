### 1. 基本配置

**spring配置**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/cache"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/cache http://www.springframework.org/schema/cache/spring-cache.xsd">

    <!--组件扫描-->
    <context:component-scan base-package="com.enosh.study.springstudy" />

    <!--配置视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="WEB-INF/views/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

    <!--开启springMVC框架注解的支持-->
    <mvc:annotation-driven/>
</beans>
```

**springMVC配置**

```xml
<web-app>
    <display-name>Archetype Created Web Application</display-name>

    <!--Spring配置文件路径-->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:applicationContext.xml</param-value>
    </context-param>

    <!--配置解决中文乱码的过滤器-->
    <filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
   
    <!--Spring上下文监听器,读取其他 XML 配置文件-->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!--配置前端控制器-->
    <servlet>
        <servlet-name>SpringStudy</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:applicationContext.xml</param-value>
        </init-param>
        <!--当值小于0或者没有指定时，则表示容器在该servlet被选择时才会去加载。-->
        <!--正数的值越小，该servlet的优先级越高，应用启动时就越先加载。-->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>SpringStudy</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

</web-app>

```



### 2. 参数绑定

- 基本类型
- 实体类型
- 集合类型

**自定义类型转换器**

1. 实现`Coverter<S,T>`接口

2. 在spring配置中注册自定义转换器

   ```xml
    <bean id="conversion" class="org.springframework.context.support.ConversionServiceFactoryBean">
           <property name="converters">
               <set>
                   <bean class="com.enosh.study.springstudy.AppInitializer" />
               </set>
           </property>
       </bean>
   ```



### 3. 常用注解

**@RequestMapping**

- `value`：用于指定请求的url，和path属性的作用一致
- `method`：用于指定请求的方法
- `params`：用于指定限制请求参数的条件，要求key-value和配置的一模一样
- `headers`：用于指定限制请求消息头的条件

**@RequestParam**

把请求中指定名称的参数给控制器的形参赋值。

- `value`：请求中的参名称
- `required`：请求中是否必须含有此参数，默认值为true

**@RequestBody**

用于获取请求体内容，直接得到`key=value&key=value...`内容。（GET请求不适用）

- `required`：是否必须有请求体，默认true

**@PathVariable**

用于绑定url中的占位符。

- `value`：占位符名称
- `required`：是否必须提供z占位符

**HiddentHttpMethodFilter过滤器**

由于浏览器只能发送GET和POST请求，HiddentHttpMethodFilter可以将浏览器请求修改为指定的请求方式。

**@RequestHeader**

用于获取请求消息头。（不常用）

- `value`：消息头名称
- `required`：

**@CookieValue**

用于把指定cookie名称的值传入控制器方法参数。

- `value`：
- `required`：

**@ModelAttribute**

用于修饰方法时，表示当前方法会在控制器方法执行之前先执行。（方法可以返回值）；

用于修饰参数时，获取指定的数据给参数赋值。

- `value`：用于获取数据的key

**@SessionAttribute**

用于多次执行控制器方法间的共享。

- `value`：用于指定存入属性的名称
- `type`：用于指定存入属性的类型



### 4. 注解启动

#### 4.1 配置

**配置前端控制器**

```java
public class AppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer{
   
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{RootConfig.class};
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{WebConfig.class};
    }

    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }
}
```

**配置MVC**

```java
@Configuration
@EnableWebMvc
@ComponentScan(basePackages = {"com.enosh.study.mvc.controller"})
public class WebConfig implements WebMvcConfigurer {

    //  配置默认的defaultServlet处理
    // <mvc:default-servlet-handler/>
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        // 配置静态资源处理
        configurer.enable("default");
        //对静态资源的请求转发到容器缺省的servlet，而不使用DispatcherServlet
    }

    /**
     * 静态资源访问控制：假如defaultServlet 没有过滤到接收的静态资源是会报404的
     */
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/statics/**").addResourceLocations("/statics/");
    }

    /**
     * 配置jsp视图解析器
     */
    @Bean
    public ViewResolver viewResolver() {
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        resolver.setPrefix("/WEB-INF/views/");
        resolver.setSuffix(".jsp");
        return resolver;
    }
}

```

网上教程大多使用`WebMvcConfigurerAdapter`类，但已经过时。

**配置Spring**

```java
@Configuration
@ComponentScan(basePackages = "com.enosh.study.mvc",excludeFilters = {@ComponentScan.Filter(type = FilterType.ANNOTATION ,classes = {Controller.class})})
public class RootConfig {
}
```

#### 4.2 原理

在Servlet 3.0环境中，容器会在类路径中查找实现`javax.servlet.ServletContainerInitializer`接口的类，如果能发现的话，就会用它来配置Servlet容器。

Spring提供了这个接口的实现，名为SpringServletContainerInitializer，这个类反过来又会查找实现WebApplicationInitializer的类并将配置的任务交给它们来完成。Spring 3.2引入了一个WebApplicationInitializer基础实现，也就是AbstractAnnotationConfigDispatcherServletInitializer。


