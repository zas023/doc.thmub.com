Spring是一个开放源代码的设计层面框架，它解决的是业务逻辑层和其他各层的松耦合问题，因此它将面向接口的编程思想贯穿整个系统应用。

Spring是一个容器框架，它可以接管Web层，业务层，Dao层，持久层的各个组件，并且可以配置各种Bean， 并可以维护Bean之间的关系，当我们需要使用某个Bean的时候，我们可以直接调用getBean(id)即可。

- **ioc（inverse of control）控制反转：**所谓反转就是把创建对象和维护对象的关系的权利从程序转移到Spring的容器；
- **di（dependency injection）依赖注入：**实际上di和ioc是同一个概念，spring的设计者认为di更准确的表示spring的核心。



### 1. Spring体系结构

> Spring是模块化的，提供了约20个功能模块，可以根据应用程序的要求来使用。

#### 1.1 核心容器

- **spring-core**模块提供了框架的基本组成部分，包括 IoC 和依赖注入功能。
- **spring-beans** 模块提供 BeanFactory，工厂模式的微妙实现，它移除了编码式单例的需要，并且可以把配置和依赖从实际编码逻辑中解耦。
- **context**模块建立在由**core**和 **beans** 模块的基础上建立起来的，它以一种类似于JNDI注册的方式访问对象。Context模块继承自Bean模块，并且添加了国际化（比如使用资源束）、事件传播、资源加载和透明地创建上下文（比如通过Servelet容器）等功能。Context模块也支持Java EE的功能，比如EJB、JMX和远程调用等。**ApplicationContext**接口是Context模块的焦点。**spring-context-support**提供了对第三方库集成到Spring上下文的支持，比如缓存（EhCache, Guava, JCache）、邮件（JavaMail）、调度（CommonJ, Quartz）、模板引擎（FreeMarker, JasperReports, Velocity）等。
- **spring-expression**模块提供了强大的表达式语言，用于在运行时查询和操作对象图。它是JSP2.1规范中定义的统一表达式语言的扩展，支持set和get属性值、属性赋值、方法调用、访问数组集合及索引的内容、逻辑算术运算、命名变量、通过名字从Spring IoC容器检索对象，还支持列表的投影、选择以及聚合等。

#### 1.2 数据访问/集成

- **JDBC** 模块提供了JDBC抽象层，它消除了冗长的JDBC编码和对数据库供应商特定错误代码的解析。
- **ORM** 模块提供了对流行的对象关系映射API的集成，包括JPA、JDO和Hibernate等。通过此模块可以让这些ORM框架和spring的其它功能整合，比如前面提及的事务管理。
- **OXM** 模块提供了对OXM实现的支持，比如JAXB、Castor、XML Beans、JiBX、XStream等。
- **JMS** 模块包含生产（produce）和消费（consume）消息的功能。从Spring 4.1开始，集成了spring-messaging模块。。
- **事务**模块为实现特殊接口类及所有的 POJO 支持编程式和声明式事务管理。（注：编程式事务需要自己写beginTransaction()、commit()、rollback()等事务管理方法，声明式事务是通过注解或配置由spring自动处理，编程式事务粒度更细）

#### 1.3Web

- **Web** 模块提供面向web的基本功能和面向web的应用上下文，比如多部分（multipart）文件上传功能、使用Servlet监听器初始化IoC容器等。它还包括HTTP客户端以及Spring远程调用中与web相关的部分。。
- **Web-MVC** 模块为web应用提供了模型视图控制（MVC）和REST Web服务的实现。Spring的MVC框架可以使领域模型代码和web表单完全地分离，且可以与Spring框架的其它所有功能进行集成。
- **Web-Socket** 模块为 WebSocket-based 提供了支持，而且在 web 应用程序中提供了客户端和服务器端之间通信的两种方式。
- **Web-Portlet** 模块提供了用于Portlet环境的MVC实现，并反映了spring-webmvc模块的功能。

#### 1.4 其他

- **AOP** 模块提供了面向方面的编程实现，允许你定义方法拦截器和切入点对代码进行干净地解耦，从而使实现功能的代码彻底的解耦出来。使用源码级的元数据，可以用类似于.Net属性的方式合并行为信息到代码中。
- **Aspects** 模块提供了与 **AspectJ** 的集成，这是一个功能强大且成熟的面向切面编程（AOP）框架。
- **Instrumentation** 模块在一定的应用服务器中提供了类 instrumentation 的支持和类加载器的实现。
- **Messaging** 模块为 STOMP 提供了支持作为在应用程序中 WebSocket 子协议的使用。它也支持一个注解编程模型，它是为了选路和处理来自 WebSocket 客户端的 STOMP 信息。
- **测试**模块支持对具有 JUnit 或 TestNG 框架的 Spring 组件的测试。



### 2. Spring IoC 容器

> Spring 容器是 Spring 框架的核心。容器将创建对象，把它们连接在一起，配置它们，并管理他们的整个生命周期从创建到销毁。

Spring 提供了以下两种不同类型的容器：

| 容器                    | 描述                                                         |
| ----------------------- | ------------------------------------------------------------ |
| BeanFactory 容器        | 它是最简单的容器，给 DI 提供了基本的支持，它用 org.springframework.beans.factory.BeanFactory 接口来定义。（但已废弃） |
| ApplicationContext 容器 | 该容器添加了更多的企业特定的功能，例如从一个属性文件中解析文本信息的能力，发布应用程序事件给感兴趣的事件监听器的能力。该容器是由 org.springframework.context.ApplicationContext 接口定义。 |

#### 2.1 ApplicationContext

> ApplicationContext 包含 BeanFactory 所有的功能，一般情况下，相对于 BeanFactory，ApplicationContext 会更加优秀。当然，BeanFactory 仍可以在轻量级应用中使用，比如移动设备或者基于 applet 的应用程序。

最常被使用的 ApplicationContext 接口实现：

- **FileSystemXmlApplicationContext**：该容器从 XML 文件中加载已被定义的 bean。在这里，你需要提供给构造器 XML 文件的完整路径。
- **ClassPathXmlApplicationContext**：该容器从 XML 文件中加载已被定义的 bean。在这里，你不需要提供 XML 文件的完整路径，只需正确配置 CLASSPATH 环境变量即可，因为，容器会从 CLASSPATH 中搜索 bean 配置文件。
- **WebXmlApplicationContext**：该容器会在一个 web 应用程序的范围内加载在 XML 文件中已被定义的 bean。

#### 2.2 Bean定义

| 属性                | 属性名          | 描述                                                         |
| ------------------- | --------------- | ------------------------------------------------------------ |
| calss               | calss           | 这个属性是强制性的，并且指定用来创建 bean 的 bean 类         |
| name                | name            | 这个属性指定唯一的 bean 标识符                               |
| scope               | scope           | 这个属性指定由特定的 bean 定义创建的对象的作用域             |
| constructor-arg     | constructor-arg | 它是用来注入依赖关系的                                       |
| properties          | property        | 它是用来注入依赖关系的                                       |
| autowiring          | autowire        | 它是用来注入依赖关系的                                       |
| lazy-initialization | lazy-init       | 延迟初始化的 bean 告诉 IoC 容器在它第一次被请求时，而不是在启动时去创建一个 bean 实 |
| initialization      | init-method     | 在 bean 的所有必需的属性被容器设置之后，调用回调方法         |
| destruction         | destroy-method  | 当包含该 bean 的容器被销毁时，使用回调方法                   |

#### 2.3 Bean作用域

| 参数           | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| singleton      | 在spring IoC容器仅存在一个Bean实例，Bean以单例方式存在，**默认值** |
| prototype      | 每次从容器中调用Bean时，都返回一个新的实例，即每次调用getBean()时，相当于执行newXxxBean() |
| request        | 每次HTTP请求都会创建一个新的Bean，该作用域仅适用于WebApplicationContext环境 |
| session        | 同一个HTTP Session共享一个Bean，不同Session使用不同的Bean，仅适用于WebApplicationContext环境 |
| global-session | 一般用于Portlet应用环境，该运用域仅适用于WebApplicationContext环境 |

#### 2.4 Bean生命周期

> 为了定义安装和拆卸一个 bean，我们只要声明带有 **init-method** 和/或 **destroy-method** 参数的 。

- init-method 属性指定一个方法，在实例化 bean 时，立即调用该方法。同样，destroy-method 指定一个方法，只有从容器中移除 bean 之后，才能调用该方法。
- Bean的生命周期可以表达为：Bean的定义--->Bean的初始化--->Bean的使用--->Bean的销毁
- 当有太多具有相同名称的初始化或者销毁方法的 Bean，无需在每一个 bean 上声明初始化方法和销毁方法。框架使用元素中的 **default-init-method** 和 **default-destroy-method** 属性。

#### 2.5 Bean后置处理器

- Bean 后置处理器允许在调用初始化方法前后对 Bean 进行额外的处理。
- **BeanPostProcessor** 接口定义回调方法，你可以实现该方法来提供自己的实例化逻辑，依赖解析逻辑等。
- 我们可以在 Spring 容器通过插入一个或多个 BeanPostProcessor 的实现来完成实例化，配置和初始化一个bean之后实现一些自定义逻辑回调方法。
- 同时通过设置 BeanPostProcessor 实现的 Ordered 接口提供的 order 属性来控制这些 BeanPostProcessor 接口的执行顺序。
- **ApplicationContext** 会自动检测由 BeanPostProcessor 接口的实现定义的 bean，注册这些 bean 为后置处理器，然后通过在容器中创建 bean，在适当的时候调用它。

#### 2.6 Bean定义继承

- 继承

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.springframework.org/schema/beans
      http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
 
     <bean id="helloWorld" class="com.tutorialspoint.HelloWorld">
        <property name="message1" value="Hello World!"/>
        <property name="message2" value="Hello Second World!"/>
     </bean>
 
     <bean id="helloIndia" class="com.tutorialspoint.HelloIndia" parent="helloWorld">
        <property name="message1" value="Hello India!"/>
        <property name="message3" value="Namaste India!"/>
     </bean>
 
  </beans>
  ```

- 模板

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.springframework.org/schema/beans
      http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
 
     <bean id="beanTeamplate" abstract="true">
        <property name="message1" value="Hello World!"/>
        <property name="message2" value="Hello Second World!"/>
        <property name="message3" value="Namaste India!"/>
     </bean>
 
     <bean id="helloIndia" class="com.tutorialspoint.HelloIndia" parent="beanTeamplate">
        <property name="message1" value="Hello India!"/>
        <property name="message3" value="Namaste India!"/>
     </bean>
 
  </beans>
  ```

  父 bean 自身不能被实例化，因为它是不完整的，而且它也被明确地标记为抽象的。当一个定义是抽象的，它仅仅作为一个纯粹的模板 bean 定义来使用的，充当子定义的父定义使用。

 

### 3. Spring 依赖注入

> Spring框架的核心功能之一就是通过依赖注入的方式来管理Bean之间的依赖关系。

#### 3.1 基于构造函数的依赖注入

假设你有一个包含文本编辑器组件的应用程序，并且你想要提供拼写检查：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <!-- Definition for textEditor bean -->
   <bean id="textEditor" class="com.tutorialspoint.TextEditor">
      <constructor-arg ref="spellChecker"/>
   </bean>

   <!-- Definition for spellChecker bean -->
   <bean id="spellChecker" class="com.tutorialspoint.SpellChecker">
   </bean>

</beans>
```

如果存在不止一个参数时，当把参数传递给构造函数时，可能会存在歧义。要解决这个问题，那么构造函数的参数在 bean 定义中的顺序就是把这些参数提供给适当的构造函数的顺序就可以了：

```java
package x.y;
public class Foo {
   public Foo(Bar bar, Baz baz) {
      // ...
   }
}
```

```xml
<beans>
   <bean id="foo" class="x.y.Foo">
      <constructor-arg ref="bar"/>
      <constructor-arg ref="baz"/>
   </bean>

   <bean id="bar" class="x.y.Bar"/>
   <bean id="baz" class="x.y.Baz"/>
</beans>
```

#### 3.2 基于设值函数的依赖注入

#### 3.3 注入内部 Beans

#### 3.4 注入集合



### 4. Spring  自动装配

> Spring 容器可以在不使用`<constructor-arg>`和`<property>` 元素的情况下**自动装配**相互协作的 bean 之间的关系，这有助于减少编写一个大的基于 Spring 的应用程序的 XML 配置的数量。

#### 4.1 自动装配模式

| 模式        | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| no          | 这是**默认**的设置，它意味着没有自动装配，你应该使用显式的bean引用来连线。 |
| byName      | 由**属性名**自动装配。Spring 容器看到在 XML 配置文件中 bean 的自动装配的属性设置为 byName。然后尝试匹配，并且将它的属性与在配置文件中被定义为相同名称的 beans 的属性进行连接。 |
| byType      | 由**属性数据类型**自动装配。Spring 容器看到在 XML 配置文件中 bean 的自动装配的属性设置为 byType。然后如果它的类型匹配配置文件中的一个确切的 bean 名称，它将尝试匹配和连接属性的类型。如果存在不止一个这样的 bean，则一个致命的异常将会被抛出。 |
| constructor | 类似于 byType，但该类型适用于构造函数参数类型。如果在容器中没有一个构造函数参数类型的 bean，则一个致命错误将会发生。 |
| autodetect  | 首先尝试通过 constructor 使用自动装配来连接，如果它不执行，再尝试通过 byType 来自动装配。 |

#### 4.2 自动装配的局限性

- 从写的可能性：使用总是重写自动装配的 <constructor-arg>和 <property> 设置来指定依赖关系；
- 原始数据类型：不能自动装配所谓的简单类型包括基本类型，字符串和类；
- 混乱的本质：自动装配不如显式装配精确，所以如果可能的话尽可能使用显式装配。



### 5. 基于注解的配置

> 从 Spring 2.5 开始就可以使用注解来配置依赖注入，注解连线在默认情况下在 Spring 容器中不打开，因此，在可以使用基于注解的连线之前我们将需要在Spring 配置文件中启用它。

```xml
<beans xmlns="http://www.springframework.org/schema/beans">

   <context:annotation-config/>
   <!-- bean definitions go here -->

</beans>
```

| 注解                | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| @Required           | 应用于 bean 属性的 setter 方法                               |
| @Autowired          | 应用到 bean 属性的 setter 方法，非 setter 方法，构造函数和属性 |
| @Qualifier          | 通过指定确切的将被连线的 bean，@Autowired 和 @Qualifier 注解可用于删除混乱 |
| JSR-250 Annotations | Spring 支持 JSR-250 的基础的注解，其中包括了 @Resource，@PostConstruct 和 @PreDestroy 注解 |

#### 5.1 @Required

- **@Required** 注释应用于 bean 属性的 setter 方法，它表明受影响的 bean 属性在配置时必须放在 XML 配置文件中，否则容器就会抛出一个 BeanInitializationException 异常。

#### 5.2 @Autowired

- 当 Spring遇到一个在 setter 方法中使用的 **@Autowired** 注释，它会在方法中视图执行 byType 自动连接。
- 当在属性中使用 **@Autowired** 注释时候，Spring 会将这些传递过来的值或者引用自动分配给那些属性。
- 一个构造函数 **@Autowired** 说明当创建 bean 时，即使在 XML 文件中没有使用 元素配置 bean ，构造函数也会被自动连接。
- 默认情况下，**@Autowired** 注释意味着依赖是必须的，它类似于 @Required 注释，然而，你可以使用 @Autowired 的 **（required=false）** 选项关闭默认行为。

#### 5.3 @Qualifier

- 可能会有这样一种情况，当你创建多个具有相同类型的 bean 时，并且想要用一个属性只为它们其中的一个进行装配，在这种情况下，你可以使用 **@Qualifier**注释和 **@Autowired** 注释通过指定哪一个真正的 bean 将会被装配来消除混乱。

#### 5.4 @Cacheable

@Cacheable 的作用 主要针对方法配置，能够根据方法的请求参数对其结果进行缓存。

@Cacheable(value=”accountCache”)，这个注释的意思是，当调用这个方法的时候，会从一个名叫 accountCache 的缓存中查询，如果没有，则执行实际的方法（一般是查询数据库），并将执行的结果存入缓存中，否则返回缓存中的对象。

#### 5.5 JSR-250 注释

- 使用 **@PostConstruct** 注释作为初始化回调函数的一个替代，**@PreDestroy** 注释作为销毁回调函数的一个替代；
- 在字段中或者 setter 方法中使用 **@Resource** 注释，它和在 Java EE 5 中的运作是一样的。

#### 5.6 基于Java 的配置

**@Componen 和 @Bean**

- @Component注解表明一个类会作为组件类，并告知Spring要为这个类创建bean。
- @Bean注解告诉Spring这个方法将会返回一个对象，这个对象要注册为Spring应用上下文中的bean。通常方法体中包含了最终产生bean实例的逻辑。

两者的目的是一样的，都是注册bean到Spring容器中。

#### 5.7 Spring 中的事件

| 内置事件              | 描述                                                         |
| --------------------- | ------------------------------------------------------------ |
| ContextRefreshedEvent | ApplicationContext 被初始化或刷新时，该事件被发布。这也可以在 ConfigurableApplicationContext 接口中使用 refresh() 方法来发生。 |
| ContextStartedEvent   | 当使用 ConfigurableApplicationContext 接口中的 start() 方法启动 ApplicationContext 时，该事件被发布。你可以调查你的数据库，或者你可以在接受到这个事件后重启任何停止的应用程序。 |
| ContextStoppedEvent   | 当使用 ConfigurableApplicationContext 接口中的 stop() 方法停止 ApplicationContext 时，发布这个事件。你可以在接受到这个事件后做必要的清理的工作。 |
| ContextClosedEvent    | 当使用 ConfigurableApplicationContext 接口中的 close() 方法关闭 ApplicationContext 时，该事件被发布。一个已关闭的上下文到达生命周期末端；它不能被刷新或重启。 |
| RequestHandledEvent   | 这是一个 web-specific 事件，告诉所有 bean HTTP 请求已经被服务。 |

- 由于 Spring 的事件处理是单线程的，所以如果一个事件被发布，直至并且除非所有的接收者得到的该消息，该进程被阻塞并且流程将不会继续。因此，如果事件处理被使用，在设计应用程序时应注意。



### 6. AOP

> 面向切面编程—>oop

**通知（Advice）**

通知是切面的工作，定义了切面需要何时完成何事。

Spring切面可以应用5种类型的通知：

1. 前置通知（Before）:在目标方法调用之前调用通知功能
2. 后置通知（After）:在目标方法调用之后调用通知，此时不会关心方法输出什么内容
3. 返回通知（After-returning）：在目标方法成功执行后调用通知
4. 异常通知（After-throwing）：在目标方法抛出异常后调用通知
5. 环绕通知（）：通知包裹了被通知的方法，被通知的方法调用前后执行自定义的行为

**连接点（Join Point）**

连接点是应用执行过程中能够插入切面的一个点，这个点可以是调用方法时、抛出异常时、甚至时修改一个字段的时候。

**切点（Pointcut）**

切点定义切面的位置，会匹配通知所要织入的一个或多个连接点。

**切面（Aspect）**

切面是通知和切点的结合。通知和切点共同定义了切面的全部内容——在何时何地完成何功能。

**引入（Introduction）**

引入允许我们向现有的类添加新的方法或属性。

**织入（Weaving）**

织入是把切面应用到目标对象并创建新的代理对象的过程。



### 7. 事务



### 8. Spring Security

Spring Security 是基于Spring应用程序提供声明式安全保护的安全性框架，能够在web请求级别和方法调用级别处理身份认证和授权。

**安全模块**

| 模块          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| ACL           | 通过访问控制列表（Access Control List）对域对象提供安全性    |
| Aspects(切面) | 当Security使用注解时，会使用介于Aspects的切面，而非AOP       |
| CAS客户端     | 提供与Jasig的中心认证服务（Central Authentication Service ）进行集成 |
| Configuration | 通过XML和Java配置Spring Security的功能                       |
| Core          | 基本库                                                       |
| Crytography   | 提供加密和密码编码的功能                                     |
| LDAP          | 支持基于LDAP进行认证                                         |
| OpenID        | 支持使用OpenID进行集中式认证                                 |
| Remoting      | 提供对Spring Remoting的支持                                  |
| Tag Library   | JSP标签库                                                    |
| Web           | 基于Filter的Web安全性支持                                    |

- 应用程序类路径下至少包含Core和Configuration两个模块，一般还需Web和JSP的模块


