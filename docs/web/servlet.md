### 1. 概述

> Servlet技术是一种基于Java语言，用于创建Web应用程序(处在服务器端并生成动态网页)。
>
> 狭义的Servlet是指Java语言的接口，广义上讲是指实现这个Servlet接口的类。

继承HttpServlet，并从写service()方法：

```java
public class MyServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        resp.getWriter().write("<h1>Hello Servlet !!!</h1>");
    }
}
```

配置`WEB-INF`目录下的`web.xml`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                             http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <!--配置servlet-->
    <servlet>
        <servlet-name>myservlet</servlet-name>
        <servlet-class>com.enosh.study.servlet.MyServlet</servlet-class>
    </servlet>
    <!--配置访问路径-->
    <servlet-mapping>
        <servlet-name>myservlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

#### 1.1 访问流程

URL组成：http://服务器地址:端口号/虚拟项目名/servlet别名

浏览器发送请求到服务器，服务器根据请求URL的信息在webapps目录下找到对于项目的文件夹，然后在webxml中检索对应servlet的class文件并调用执行。

同时，使用servlet的真实名称`/com.enosh.study.servlet.MyServlet `亦可访问该servlet。



#### 1.2 生命周期

servlet的生命周期通常是从第一次请求加载进内存至服务器关闭而销毁。

```java
//初始化方法，在第一次调用servlet时被调用
public void init() throws ServletException;

//处理请求的方法
protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException;

//销毁方法是在servlet对象被销毁时，即服务器关闭时调用
public void destroy();
```

若在`web.xml`文件中配置了`load-on-start`属性：

- 其值为非负时，则在服务器启动时便加载servlet至内存，并调用ini()方法。
- 且数值越大，优先级越高，启动时优先加载，当数值相同，则由容器自己选择加载顺序。
- 但其值为负，则不启动不加载。

项目任意类文件发生变化后，才会重新编译并输出到目标文件。



### 2. 常用方法

#### 2.1 doGet & doPost

- **service**：可以处理get/post方法请求，如果servlet中由service方法，会优先调用service方法对请求进行处理；
- **doGet**：处理get方法请求；
- **doPost**：处理post方法请求。

如果在覆写service的方法红调用了父类的service，即super.service()，则service执行之后会在此根据请求相应doGet或doPost方法。所以我们要避免在service中调用父类方法，避免405错误。



#### 2.2 Request

request对象由Tomcat容器创建，封装了当前请求的所有信息，并作为实参传递给处理请求的servlet对象中的service方法。

- 获取请求行数据

```java
//获取请求方法
req.getMethod();
//获取请求URL
req.getRequestURL();
//获取请求URI
req.getRequestURI();
//获取协议
req.getScheme();
```

- 获取请求头数据

请求头是HTTP的报文头。报文头包含若干个属性，格式为“属性名:属性值”，服务端据此获取客户端的信息，以及与缓存相关的规则信息，均包含在header中。

```java
//获取指定请求头信息
req.getHeader("User-Agent");
//获取所有请求头信息，返回所有请求体键名的枚举
Enumeration e = req.getHeaderNames();
```

- 获取请求体信息

是报文体，它将一个页面表单中的组件值通过param1=value1&param2=value2的键值对形式编码成一个格式化串，它承载多个请求参数的数据。

```java
//获取指定请求体信息
String str=req.getParameter("username");
//objects=1&objects=2&objects=3
String[] objects = req.getParameterValues("objects");
//获取所有请求体信息
Enumeration enumeration = req.getParameterNames();
```

如果获取的请求数据不存在，服务器不会报错，而是返回`null`。



#### 2.3 Response

- 设置响应头

```java
//设置响应头，同键覆盖
resp.setHeader("name","admin");
//添加响应头，同键不会覆盖
resp.addHeader("key","value");
```

- 设置响应编码设

```java
resp.setHeader("content-type","text/html;charset=utf-8");
resp.setContentType("text/html;charset=utf-8");
```

- 设置响应状态码

```java
resp.sendError(405,"This method is not supported");
```

- 设置响应实体

```java
resp.getWriter().write("This method is not supported");
```



#### 2.4 请求乱码

方法一：使用String进行重新编码

```java
String str=new String(str.getBytes("ios8859-1"),"utf-8");
```

方法二：

在获取请求数据之前设置：`req.setCharacterEncoding("utf-8")`；

配置Tomcat的server.xml文件：添加`useBodyEncodingForURI`表签，并设置为`true`。

#### 2.5 请求转发

实现多个servlet的联动操作来处理请求，避免代码冗余，可以明确各个servlet的职责。

```java
//跳转至新的servlet
req.getRequestDispatcher("/").forward(req,resp);
//请求转发后建议return结束
return;
```

特点：一次请求（**服务器内部跳转**），浏览器地址栏信息不会发生变化。

#### 2.6 重定向

用于解决表单重复提交的问题，以及当前请求servlet无法处理的问题。

```java
resp.sendRedirect("/");
```

特点：两次请求（**浏览器进行的二次请求**），两个request对象。

- 请求转发可以共享请求参数 ，重定向之后，就获取不了共享参数；
- 请求转发不能跨域（不能访问其他服务器链接），重定向可以 ；
- 请求转发能转到WEB-INF目录下的文件，而重定向不能 。

### 3. 数据存储

#### 3.1 Cookie

浏览器端的数据存储技术，解决不同请求间数据共享的问题。

```java
//创建Cookie对象
Cookie cookie=new Cookie("study","servletStudy");
//设置Cookie的有效期
cookie.setMaxAge(3*24*3600);
//设置路径
cookie.setPath("/");
//响应Cookie信息给浏览器
resp.addCookie(cookie);
```

一个Cookie对象存储一条数据，多条数据可以创建多个Cookie对象进行存储。

- 临时存储：存储在浏览器端的运行内存中，浏览器关闭即失效；
- 定时存储：设置Cookie的有效期，存储在客户端的硬盘中，在有效期内符合路径要求的请求都会附带该信息。

#### 3.2 Session

解决一个用户不同请求处理的数据共享问题。

使用Session技术，用户第一次访问时服务器会创建一个session对象给此用户，并将该对象的JESSIONID使用Cookie技术存储在浏览器中，保证用户的其它请求能够获取到同一个session对象，也保证了不同请求能够获取到共享数据。

```java
//创建或获取对象
HttpSession hs=req.getSession();
```

如果请求中拥有session的标识符也就是JSEESIONID，则返回对于的session对象；若没有标识符或session失效，就会重新创建一个session对象。

```java
//设置session存储时间（默认30分钟）
hs.setMaxInactiveInterval(int seconds);
//设置session强制失效
hs.invalidate();
```

还可以在tomcat或项目web.xml配置session：

```xml
<session-config>
    <session-timeout>30</session-timeout>
</session-config>
```

在指定时间内session对象没有使用则被销毁，若使用，则会重新计算。

```java
//存储数据
hs.setAttribute("key","value");
//获取数据
hs.getAtrribute("key");
```

注意：JSEESIONID存储在Cookie的临时存储空间中，即关闭浏览器失效。



**ServletContext**

解决不同用户间数据共享的问题，由服务器创建，在整个项目内的所有用户间共享。

```java
//获取ServletContext
ServletContext sc= getServletContext();
ServletContext sc1= getServletConfig().getServletContext();
ServletContext sc2= req.getSession().getServletContext();
```

- 存储公共对象

```java
//存储数据
sc.setAttribute("key","value");
//获取数据
sc.getAtrribute("key");
```

- 获取web.xml的公共配置

作用：将静态数据与代码进行解耦。

```java
String str = sc.getInitParameter("key");
```

```xml
<!-- 一组标签之只能存放一组键值对 -->
<context-param>
    <param-name>key</param-name>
    <param-value>value</param-value>
</context-param>
```

- 获取项目根目录路径资源

```java
//Str path = "D:\\Apache-Tomcat-8.5.43\\webapps\\example\\docs\\1.txt";
//获取项目根目录下的资源绝对路径
String path =sc.getRealPath("/docs/1.txt");

//获取根目录资源流对象
InputStream is=sc.getResourceAsStream("/docs/1.txt");
```



**ServletConfig**

ServletConfig对象是servlet的专属配置对象，每个servlet都有一个单独的servletConfig对象，用来获取web.xml中的配置信息。

```java
//获取servletConfig对象
ServletConfig sc=getServletConfig();
//获取web.xml中的配置
String value= sc.getInitParameter("key");
```

xml中的配置：

```xml
<servlet>
    <servlet-name>myservlet</servlet-name>
    <servlet-class>com.enosh.study.servlet.MyServlet</servlet-class>
    <init-param>
        <param-name>key</param-name>
        <param-value>value</param-value>
    </init-param>
</servlet>
```

### 4. 文件配置

#### 4.1 web.xml

每个Web项目中和Tomcat容器conf目录下都会有一个`web.xml`文件。用于存储向相关的配置信息，包含Servlet，以及解耦一些数据对程序的依赖。

其中，Web项目的web.xml文件为局部配置，针对该项目，而Tomcat目录下的文件时全局配置，配置公有信息。

核心组件：全局上下文配置，Servlet配置，过滤器配置，监听器配置。

加载顺序：ServletContext --> context-param --> listener --> filter -->servlet，即这些元素标签的位置可以任意。

#### 4.2 server.xml

其中每一个元素都对应了Tomcat中的每一个组件，因此通过对xml文件中元素的配置可以实现对Tomcat中各个组件的控制。

```xml
<Server>
    <Service>
        <Connector>
        </Connector>
        <Engine>
            <Host>
                <Context />
            </Host>
        </Engine>
    </Service>
</Server>
```

Server在最顶层，代表整个Tomcat容器，一个Server元素中可以有一个或多个Service元素。

Service将Connector和Engine组装起来，对外提供服务。一个Service可以包含多个Connector，但是只能包含一个Engine。

Connector接收请求，Engine处理请求。

Engine、Host和Context都是容器。每个Host组件代表Engine中的一个虚拟主机，每个Context组件代表在特定Host上运行的一个Web应用。

**热部署**：

```xml
<Context path"/" reloadable="false" docBase="D:/webapps" />
```
