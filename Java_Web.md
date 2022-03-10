# Web

- web相关概念
- web服务器软件：Tomcat

## web相关概念

1. 软件架构
   - C/S：客户端/服务器端
   - B/S：浏览器/服务器端

2. 网页资源分类
   1. 静态资源：所有用户访问后，得到的结果都是一样的，称为静态资源。静态资源可以直接被浏览器解析；如：html、css、JavaScript...
   2. 动态资源：每个用户访问相同资源后，得到的结果可能是不一样的，称为动态资源。动态资源被访问后，需要先转换为静态资源，再返回给浏览器；如：servlet/jsp、jsp、asp...

3. 网络通信三要素
   1. IP：电子设备（计算机）在网络中的唯一标识；
   2. 端口：应用程序在计算机中的唯一标识（0~65536）；
   3. 传输协议：规定了数据传输的规则。
      - 基础协议：
        1. tcp：安全协议，三次握手。速度稍慢；
        2. udp：不安全协议。速度快。

## web服务器软件

1. 概念
   - 服务器：安装了服务器软件的计算机；
   - 服务器软件：接受用户的请求，处理请求、做出响应；
   - web服务器软件：接受用户的请求，处理请求、做出响应，在web服务器软件中，可以部署web项目，让用户通过浏览器来访问这些项目。

2. 常见的java相关的web服务器软件
   1. webLogic：属于oracle公司，大型的JavaEE服务器，支持所有的JavaEE规范，收费软件；
   2. webSphere：属于IBM公司，大型的JavaEE服务器，支持所有的JavaEE规范，收费软件；
   3. JBOSS：属于JBOSS公司，大型的JavaEE服务器，支持所有的JavaEE规范，收费软件；
   4. Tomcat：属于Apache基金组织，中小型JavaEE服务器，仅仅支持少量的JavaEE规范（servlet/jsp），开源、免费软件。

3. JavaEE：Java语言在企业级开发中使用的技术规范的总和，一共规定了13项大的规范。

# Tomcat

## Tomcat基本操作

1. 下载：http://tomcat.apache.org/；
2. 安装：解压即可；（注：安装目录不建议有中文、空格）
3. 卸载：删除目录即可；
4. 启动：
   - 访问bin/startup.bat，运行该文件即可启动Tomcat服务器；
   - 通过访问http://localhost:8080访问自己的Tomcat页面；

5. 注意事项：

   1. 黑窗口闪退：
      - 原因：没有正确配置JAVA_HOME环境变量，Tomcat依赖JAVA_HOME；
      - 解决：正确配置JAVA_HOME环境变量即可。

   2. 启动报错：
      - 原因：端口被占用；
      - 解决：
        1. 使用控制台命令行netstat -ano找到占用端口的进程PID，杀掉该进程；
        2. 修改自身服务器的端口号。

```xml
<!-- 通过修改conf/server.xml中以下配置，可以修改服务器端口 -->
<Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" />
```

6. 关闭：

   - 正常关闭：
     1. 通过访问/bin目录下的shutdown.bat，关闭Tomcat服务器；
     2. 在激活的Tomcat服务器控制台上，使用Ctrl+C关闭Tomcat服务器。

   - 强制关闭：
     - 直接关闭已激活的Tomcat服务器控制台。

## Tomcat项目部署

1. 直接将项目放置到webapps目录下即可；
   - /hello：项目的访问路径，同时也是虚拟目录；
   - 简化的部署方式：可以将项目打包成一个war包，再将war包放置到webapps目录下；war包将会自动将web项目解压缩；如果删除war包，将自动删除解压到webapps目录下的web项目。

2. 修改conf/server.xml文件，配置项目路径、虚拟目录
   - docBase：项目存放的硬盘路径
   - path：访问该项目的虚拟目录

```xml
<!-- 在<Host>标签体中配置以下信息 -->
<Context docBase="D:\hello" path="/hey" />
```

3. 通过在/conf/Catalina/localhost/下创建任意名称的xml文件，在文件中配置项目路径
   - docBase：项目存放的硬盘路径
   - xml文件名：访问该项目的虚拟目录

```xml
<!-- 在conf/Catalina/localhost中创建任意名称的xml -->
<!-- 如果没有该目录请自行创建！！！ -->
<Context docBase="D:\hello" />
```

4. Java动态项目的目录结构
   - -- Web项目根目录
     - -- WEB-INF目录
       - -- web.xml：Web项目的核心配置文件，用于配置各种Servlet、Filter、Listener
       - -- classes目录：用于放置字节码文件的目录，src目录下的所有配置文件都被打包在此目录下
       - -- lib目录：用于放置项目所依赖的jar包，必须存在，否则项目无法运行

# Servlet

- ServletRequest：请求
- ServletResponse：响应
- Cookie：客户端会话技术
- Session：服务器端会话技术

## Servlet(Server Applet)

1. 概念：运行在服务器端的小程序

   - Servlet就是一个接口，定义了Java类被浏览器访问到（tomcat识别）的规则。

2. 快速入门：

   1. 创建JavaEE项目；
   2. 定义类，实现Servlet接口：
      - public class DemoServlet implements HttpServlet;

   3. 实现接口中的抽象方法；
   4. Servlet配置：
      - web.xml；
      - 注解配置。

- web.xml配置：

```xml
<!-- 主要的目的就是将servlet-class和url-pattern关联起来 -->
<servlet>
    <servlet-name>demo1</servlet-name>
    <servlet-class>cn.dylanphang.servlet.DemoServlet</servlet-class>
</servlet>

<servlet-mapping>
    <servlet-name>demo1</servlet-name>
    <url-pattern>/demoServlet</url-pattern>
</servlet-mapping>
```

- 注解配置：（Tomcat7之后才支持注解配置，因为使用的是Servlet 3.0）
- Servlet 3.0：
  - 支持注解配置，可以不需要创建web.xml文件。
  - 使用步骤：
    1. 创建JavaEE项目，选择Servlet的版本为3.0以上，可以不勾选创建web.xml文件；
    2. 定义一个类，实现Servlet接口；
    3. 复写接口中的方法；
    4. 在类上使用@WebServlet注解，进行配置。

```java
@WebServlet("/demoServlet")
public class DemoServlet implements HttpServlet {
	...
}
```

3. 执行原理
   1. 当服务器接收到来自客户端浏览器的请求后，会解析请求URL路径，获取访问的Servlet的资源路径；
   2. 程序会查找web.xml文件，找寻是否有对应的<url-pattern>标签体内容；
   3. 如果存在，则进一步找寻对应的<servlet-class>标签体内容中的全限定类名；
   4. Tomcat会将字节码文件加载进内存，并且创建其对象；
   5. 调用目标Servlet中的方法。

4. 生命周期

   0. Servlet什么时候被创建？

      - 默认情况下，在第一次被访问时，Servlet被创建；

      - 可以通过配置web.xml文件，控制Servlet的创建时机，在<servlet>标签内添加<load-on-startup>：

        1. 第一次被访问时，创建：
           - <load-on-startup>的值为负数；

        2. 在服务器启动时，创建：
           - <load-on-startup>的值为0或正整数。

   1. Servlet被创建：执行init方法，只执行一次；
      - Servlet的init方法，只执行一次，说明一个Servlet在内存中只存在一个对象，Servlet是单例的；
      - 因此多个用户同时访问时，可能存在线程安全的问题；
      - 解决方法：
        - 尽量不要再Servlet中定义成员变量（字段）。即使定义了成员变量，也不要对其值进行修改。

   2. Servlet提供服务：执行service方法，执行多次；
      - 每次访问Servlet时，service方法都会被调用一次。

   3. Servlet被销毁：执行destroy方法，只执行一次；
      - Servlet被销毁时执行。服务器关闭时，Servlet被销毁；
      - 只有服务器正常关闭时，才会执行destroy方法；
      - destroy方法在Servlet被销毁之前执行，一般用于释放资源。


## ServletRequest

### 获取请求消息数据

#### 获取请求行数据

```java
// 1.获取虚拟目录*（重点）
String getContextPath(); // /12-tomcat

// 2.获取请求URI*（重点）
String getRequestURI(); // /12-tomcat/ServletDemo7
StringBuffer getRequestURL(); // http://localhost:8080/12-tomcat/ServletDemo7

// 3.获取请求方式
String getMethod();

// 4.获取当前Servlet的配置路径
String getServletPath();  // /ServletDemo7

// 5.获取get方式请求参数
String getQueryString();

// 6.获取协议及版本
String getProtocol();

// 7.获取客户机的IP地址（IPv6地址）
String getRemoteAddr();
```

#### 获取请求头数据

```java
// 1.通过请求头的名称获取请求头的值*（重点）
String getHeader(String name);

// 2.获取所有的请求头名称
Enumeration<String> getHeaderNames();
```

#### 获取请求体数据

```java
/**
* 只有POST请求方式，才有请求体；请求体中封装了POST请求的请求参数。
*/

// 1.获取流对象
BufferReader getReader(); // 获取字符输入流，只能操作字符数据
ServletInputStream getInputStream(); // 获取字节输入流，可以操作所有类型数据

// 2.从流对象中取出数据
...
```

### 获取请求参数通用方式

- 无论是get请求还是post请求，都可以使用一些通用的方法来获得请求参数的数据。
- 为了防止Tomcat的乱码问题，使用以下方法时推荐设置request的字符集编码：
  - request.setCharacterEncoding("utf-8");

```java
// 1.根据参数名称获取参数值
String getParameter(String name);

// 2.根据参数名称获取参数值的数组
String[] getParameterValues(String name);

// 3.获取所有请求参数的名称
Enumeration<String> getParameterNames();

// 4.获取所有参数的map集合
Map<String, String[]> getParameterMap();
```

### ServletRequest特性

1. **从request中获取中文，出现中文乱码的问题：**
   - get方式：Tomcat8之后，get方式不再会出现获取中文出现乱码的情况，但post会出现；
   - post方式：在获取参数之前，需设置request的编码：**request.setCharacterEncoding("utf-8");**

2. **请求转发（dispatch）：一种在服务器内部的资源跳转方式**
   - 通过request对象获取请求转发器对象：**RequestDispatcher getRequestDispatcher (String path);**
   - 使用RequestDispatcher对象方法进行转发：**forward(ServletRequest request, ServletResponse response);**
   - 一般使用链式编程的方式进行请求转发：**request.getRequestDispatcher(path).forward(request, response);**
- <u>**path：指的是Servlet的配置路径，而不是URI，即不需要添加虚拟目录。**</u>
  
3. **请求转发（dispatch）特点：**
   1. 浏览器地址栏路径不发生变化；

   2. 只能转发到当前服务器内部资源中；

   3. 转发是一次请求。
4. **数据共享：**
   1. 域对象：一个有作用范围的对象，可以在范围内共享数据<u>**（域对象有很多种类，request是其中一种）**</u>
   2. request域：代表一次请求的范围，一般用于请求转发的多个资源中的共享数据
   3. 在request域中存储、获取及删除数据的方法：

```java
// 1.存储数据
void setAttribute(String name, Object obj);

// 2.通过键获取对应的值
Object getAttribute(String name);

// 3.通过键移除对应的键值对
void removeAttribute(String name);
```

5. **（重要）获取ServletContext：**

```java
// 获取ServletContext（Servlet上下文对象）
// 主要用作文件的读取，能使用其getRealPath方法定位到web项目的/web目录下
ServletContext getServletContext();
```



## ServletResponse

### 设置响应消息数据

#### 设置响应行数据

```java
// 1.设置状态码
void setStatus(int sc);
```

#### 设置响应头数据

```java
// 1.设置响应头
void setHeader(String name, String value);
```

#### 设置响应体数据

```java
// 1.获取流对象
PrintWriter getWirter(); // 获取字符输出流
ServletOutputStream getOutputStream(); // 获取字节输出流

// 2.使用输出流将数据输出到客户端浏览器
...
```

### 请求转发与重定向区别

- **转发的特点：forward**
  1. 转发地址栏路径不变；
  2. 转发只能访问当前服务器下的资源；
  3. 转发是一次请求，可以使用request域来共享数据。
- **重定向的特点：redirect**
  1. 地址栏发生变化；
  2. 重定向可以访问其他站点（服务器）的资源；
  3. 重定向是两次请求，不能使用request对象来共享数据。

### ServletResponse特性

1. **请求重定向（redirect）：一种在服务器外部的资源跳转方式**
  - 通过response对象中的sendRedirect(String URI)方法，可以将请求重定向至目标URI上。
  - 通常配合request对象中的getContextPath使用：**response.sendRedirect(request.getContextPath() + "/ServletDemo")；**
2. **使用字符输出流产生乱码的问题：**
  - response.getWriter();获取的流默认编码为**ISO-8859-1**
  - 解决步骤：
    1. 设置该流的默认编码：**response.setCharacterEncoding("utf-8");**
    2. 告诉浏览器响应体的类型：**response.setHeader("content-type", "text/html");**
    3. <u>可以直接使用**response.setContentType("text/html;charset=utf-8");**完成编码集的设置</u>

## ServletContext

- **<u>ServletContext代表整个web应用，可以和程序的容器（服务器）进行通信</u>**

### ServletContext对象

```java
// 1.通过request对象获取ServletContext对象
request.getServletContext();

// 2.通过HttpServlet获取ServletContext对象
this.getServletContext();
```

### ServletContext功能

```java
// 1.获取MIME类型：在互联网通信过程中定义的一种文件数据类型
// 格式： 大类型/小类型  text/html  image/jpeg
// 传入文件名字符串即可获取到对应的MIME类型
String getMimeType(String file);

// 2.ServletContext域对象：共享数据
// ServletContext对象范围：所有用户所有请求的数据！！！
void setAttribute(String name, Object value);
void getAttribute(String name);
void removeAttrbute(String name);

// 3.获取文件的真实（服务器）路径，一般用于读取文件（InputStream）
// 该方法的根目录是web目录，可以获取指定文件的绝对路径
// 文件如果存放在/src目录下，对应的web项目目录为/WEB-INF/classes/
String getRealPath(String relativePath); 
```

## Cookie与Session概述

1. 会话：一次会话中包含多次请求和响应。
   - 一次会话：浏览器第一次给服务器资源发送请求，会话建立，直到有一方断开为止

2. 功能：在一次会话的范围内的多次请求间，共享数据。
3. 方式：
   - 客户端会话技术：Cookie
   - 服务器端会话技术：Session

## Cookie

#### Cookie概念

- 客户端会话技术，将数据保存到客户端。

#### Cookie常用方法

```java
// 1.创建Cookie对象，绑定数据
Cookie cookie = new Cookie(String name, String value);

// 2.发送Cookie对象
response.addCookie(Cookie cookie);

// 3.获取Cookie，拿到数据
Cookie[] cookies = request.getCookies();
```

#### Cookie实现原理

- 基于响应头set-cookie和请求头cookie实现。

#### Cookie的细节

1. **一次可不可以发送多个cookie？**
   
   - 可以。
- 可以创建多个Cookie对象，使用response调用多次addCookie方法发送cookie即可。
  
2. **cookie在浏览器中保存多长时间？**

   1. 默认情况下，当浏览器关闭后，Cookie数据被销毁。
   2. 持久化存储：使用Cookie对象的方法setMaxAge(int seconds)
      1. 正数：将Cookie数据写到硬盘的文件中，持久化存储。并指定cookie存活时间，时间到后相应的cookie文件将自动失效；
      2. 负数：默认值；
      3. 零：删除cookie信息

3. **cookie能不能存储中文？**
   
    - 在tomcat 8 之前，cookie中不能直接存储中文数据，需要将中文数据转码——一般采用URL编码。
    - 在tomcat 8 之后，cookie支持中文数据。但对于**特殊字符**仍然不支持，建议仍然使用**URL编码存储、解析。**
      - 使用**URLEncoder.encode(String s, Charset charset)**对放入Cookie中的数据进行编译存储。
      - 使用**URLDecoder.decode(String s, Charset charset)**对从Cookie中取出来的数据进行解析。
4. **cookie共享问题？**

    1. **假设在一个tomcat服务器中，部署了多个web项目，那么在这些web项目中cookie能不能共享？**
        - 默认情况下，cookie不能共享。
        - setPath(String path)：设置cookie的获取范围。默认情况下，获取范围指的是当前的虚拟目录。
            - 如果要进行共享，则可以将path设置为"/"。

    2. **不同的tomcat服务器间cookie共享问题？**
        - 默认情况下，cookie不能共享。
        - setDomain(String path)：如果设置一级域名相同，那么多个服务器间cookie可以共享。
            - 例：setDomain(".baidu.com")：那么tieba.baidu.com和news.baidu.com中cookie可以共享。

5. **cookie的特点和作用？**

   - **cookie的特点：**
     - cookie存储的数据在客户端浏览器。
     - 浏览器对于单个cookie的大小是有限制的（4kb），以及对于同一个域名下的总cookie数量也是有限制的（20个）。

   - **cookie的作用：**

     - cookie一般用于存储少量的、不太敏感的数据；

     - 在不登录的情况下，完成服务器对客户端的身份识别。

## Session

#### Session概念

- 服务器端会话技术，在一次会话的多次请求间共享数据，将数据保存在服务器端的对象中

#### Session常用方法

```java
// 1.获取HttpSession对象
HttpSession session = request.getSession();

// 2.使用HttpSession对象
void setAttribute(String name, Object value);
Object getAttribute(String name);
void removeAttribute(String name);
```

#### Session实现原理

- Session的实现是依赖于Cookie的。
  - 服务器会自动调用response对象的addCookie(Cookie cookie)方法，传入一个键为JSESSIONID、值为session.getID()的cookie对象；
  - 客户端发送请求的时候，会自动携带此cookie当服务器端使用request.getSession()的时候，服务器会自动寻找cookies中是否有键为JSESSIONID的cookie；如果有，则获取此session作为返回对象；如果没有，则创建一个新的session对象。

#### Session的细节

1. **当客户端关闭后，服务器不关闭，两次获取Session是否为同一个？**
   - 默认情况下：不是。
   - 如果需要相同的session，则可以创建cookie，使键为JSESSIONID，设置最大的存活时间，让cookie持久化保存：
     - Cookie cookie = new Cookie("JSESSIONID", session.getID());
     - cookie.setMaxAge(60 * 60);
     - response.addCookie(cookie);

2. **当客户端不关闭，服务器关闭，两次获取的Session是同一个吗？**
- 不是同一个，但是要确保数据不丢失。tomcat中自动完成以下工作：
  
  - session的钝化：
       - 在服务器正常关闭之前，tomcat会将session对象序列化到硬盘上（work目录中）
  
  - session的活化：
       - 在服务器启动后，tomcat会将session对象反序列化到内存中
- 注：IDEA活化的时候，会先将work目录删除，从而导致无法活化的现象。但在实际操作中，一般会把项目直接部署到tomcat中的webapps目录下，tomcat将读取上级目录下的work目录中的数据，完成session的钝化与活化。

3. **Session什么时候被销毁？**
   1. 服务器关闭；

   2. session对象调用invalidate()；

   3. session默认失效时间为：30分钟；


```xml
<!-- 可以通过配置文件修改session默认失效时间 -->
<session-config>
    <session-timeout>30</session-timeout>
</session-config>
```

4. **Session的特点：**
   - session用于存储一次会话的多次请求的数据，数据是存储于服务器端的；
   - session可以存储任意类型，任意大小的数据。

## Cookie与Session差异

1. session存储数据在服务器端，cookie在客户端；

2. session没有数据大小限制，cookie有数据大小限制；

3. session数据安全，cookie相对于不安全。

# Filter

1. 概念和作用
   - web中的过滤器，当访问服务器的资源时， 过滤器可以将请求拦截下来，完成一些特殊的功能。
   - 一般用于完成通用的操作。如：登录验证、统一编码处理、敏感字符过滤等。

2. 使用步骤
   1. 定义一个类，实现接口Filter；
   2. 复写方法；
   3. 配置拦截路径；
      - web.xml；
      - 注解。

- web.xml配置：

```xml
<filter>
    <filter-name>demo1</filter-name>
    <filter-class>cn.dylanphang.filter.FilterDemo1</filter-class>
</filter>
<filter-mapping>
    <filter-name>demo1</filter-name>
    <!-- 拦截路径 -->
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

- 注解配置：

```java
@WebFilter("/*")
public class DemoFilter implements Filter {
	...
}
```

3. 执行流程
   1. 执行过滤器；
   2. 执行放行后的资源；
   3. 回来执行过滤器放行代码下边的代码。

4. 生命周期
   1. init：在服务器启动后，会创建Filter对象，然后调用init方法。只执行一次。用于加载资源；
   2. doFilter：每一次请求被拦截资源时，会执行。执行多次；
   3. destroy：在服务器关闭后，Filter对象被销毁。如果服务器正常关闭，则会执行destroy方法。只执行一次。用于释放资源。

5. 过滤器配置详解

   - 拦截路径：
     1. 具体资源路径：/index.jsp    只有访问index.jsp资源时，过滤器才会被执行
     2. 拦截目录：/user/*    访问/user下的所有资源，过滤器都会被执行
     3. 后缀名拦截：*.jsp    访问所有后缀名为jsp的资源，过滤器都会被执行
     4. 拦截所有资源：/*    访问所有资源，过滤器都会被执行

   - 拦截方式：**（该Filter在什么时候会被访问）**

     - 注解配置：设置dispatcherTypes属性
       1. REQUEST：默认值。浏览器直接请求资源时，会被Filter拦截；
       2. FORWARD：转发访问资源时，会被Filter拦截；
       3. INCLUDE：包含访问资源时，会被Filter拦截；
       4. ERROR：错误跳转资源时，会被Filter拦截；
       5. ASYNC：异步访问资源时，会被Filter拦截。

     - web.xml配置：
       - 设置<dispatcher></dispatcher>标签即可

- 注解配置：

```java
@WebFilter(value = "/*", dispatcherTypes = {DispatcherType.REQUEST, DispatcherType.FORWARD})
public class FilterDemo implements Filter {
    ...
}
```
- web.xml配置：

```xml
<filter>
    <filter-name>demo1</filter-name>
    <filter-class>cn.dylanphang.filter.FilterDemo</filter-class>
</filter>

<filter-mapping>
    <filter-name>demo1</filter-name>
    <url-pattern>/*</url-pattern>
    <dispatcher>ASYNC</dispatcher>
    <dispatcher>INCLUDE</dispatcher>
</filter-mapping>
```



6. 过滤器链（配置多个过滤器）
   - 执行顺序：如果有两个过滤器——过滤器a和过滤器b
     1. 过滤器a；
     2. 过滤器b；
     3. 资源执行；
     4. 过滤器b；
     5. 过滤器a。
   - 过滤器先后顺序：
     - 注解配置：按照类名的字符串比较规则，值小的先执行；
       - 例：AFilter和BFilter，那么AFilter就限制性。
     - web.xml配置：<filter-mapping>谁定义在上边，谁就先执行。

- **LoginStatusFilter.java**

```java
package filter;

import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;

@WebFilter("/*")
public class LoginStatusFilter implements Filter {
    public void destroy() {
    }

    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws ServletException, IOException {
        /**
         * 所有的请求都先要经过Filter。
         */

        // 1.判断用户是否要登录
        HttpServletRequest request = (HttpServletRequest) req;
        String uri = request.getRequestURI();
        if (uri.contains("/index.jsp") || uri.contains("/LoginServlet") || uri.contains("/bootstrap/") || uri.contains("/css/") || uri.contains("/js/")) {

            // 2.用户请求的是登录资源，需要放行
            chain.doFilter(req, resp);
        } else {

            // 3.用户请求的不是登录资源，进一步判定是否为已登录状态
            HttpServletResponse response = (HttpServletResponse) resp;
            HttpSession session = request.getSession();
            if (session.getAttribute("LoginStatus") == null || !(boolean) session.getAttribute("LoginStatus")) {

                // 4.未登录状态则直接跳转至主页要求登录
                response.sendRedirect(request.getContextPath() + "/index.jsp");
            } else {

                // 5.已登录状态继续判断

                // 6.涉及访问add.jsp以及alter.jsp资源
                if (uri.contains("/add.jsp") || uri.contains("/alter.jsp")) {

                    // 7.如果来源不是home.jsp以及index.jsp，则要求重定向至home.jsp，否则放行
                    if (request.getHeader("referer") == null || !(request.getHeader("referer").contains("/home.jsp") || request.getHeader("referer").contains("/index.jsp"))) {
                        response.sendRedirect(request.getContextPath() + "/home.jsp");
                    } else {
                        chain.doFilter(req, resp);
                    }
                } else {
                    // 8.不涉及到add.jsp以及alter.jsp资源的，放行
                    chain.doFilter(req, resp);
                }
            }
        }
    }

    public void init(FilterConfig config) throws ServletException {

    }
}
```

7. 增强对象的功能
   - 设计模式：一些通用的解决固定问题的方式
     1. 装饰模式；
     2. 代理模式；

8. 代理模式

   1. 真实对象：被代理的对象；

   2. 代理对象；

   3. 代理模式：代理对象代理真实对象，达到增强真实对象功能的目的。

   4. 实现方式：

      1. 静态代理：有一个类文件描述代理模式；

      2. 动态代理：在内存中形成代理类；

         - 实现步骤：
           1. 代理对象和真实对象实现相同的接口；
           2. 代理对象 = Proxy.newProxyInstance()；
           3. 使用代理对象调用方法；
           4. 增强方法。

         - 增强方式：
           1. 增强参数列表；
           2. 增强返回值类型；
           3. 增强方法体执行逻辑。

- **SensitiveWordsFilter.java**

```java
package filter;

import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.lang.reflect.Proxy;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

/**
 * 用于过敏感词汇。
 */

@WebFilter("/*")
public class SensitiveWordsFilter implements Filter {
    private List<String> list = new ArrayList<>();

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        InputStream is = null;
        BufferedReader bf = null;
        try {
            /**
             * InputStreamReader一般是以UTF-8为默认字符集的,但在web中默认使用的字符集为"GBK",可能与Tomcat中的配置有关
             * getResourceAsStream以文档的编码为字符编码，目标文档使用的是utf-8，因此这里要指定字符集为utf-8，不然会出现乱码
             */
            is = SensitiveWordsFilter.class.getClassLoader().getResourceAsStream("敏感词汇.txt");
            bf = new BufferedReader(new InputStreamReader(is, "utf-8"));

            String sensitiveWord = null;
            while ((sensitiveWord = bf.readLine()) != null) {
                list.add(sensitiveWord);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (bf != null) {
                try {
                    bf.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (is != null) {
                try {
                    is.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    /**
     * 动态代理增强getParameter的返回值
     *
     * @param servletRequest
     * @param servletResponse
     * @param filterChain
     * @throws IOException
     * @throws ServletException
     */
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        // 1.获取代理对象
        ServletRequest proxy_req = (ServletRequest) Proxy.newProxyInstance(servletRequest.getClass().getClassLoader(),
                servletRequest.getClass().getInterfaces(),
                ((proxy, method, args) -> {
            if ("getParameter".equals(method.getName())) {
                String str = (String) method.invoke(servletRequest, args);

                // *.过滤敏感词汇
                for (String s : list) {
                    if (str.contains(s)) {
                        str = str.replace(s, String.join("", Collections.nCopies(s.length(), "*")));
                    }
                }
                return str;
            }
            return method.invoke(servletRequest, args);
        }));

        // 2.放行
        filterChain.doFilter(proxy_req, servletResponse);
    }

    @Override
    public void destroy() {

    }
}

```

# Listener

1. 概念和作用

   - 事件监听机制：
     1. 事件：一个事件；
     2. 事件源：事件发生的地方；
     3. 监听器：一个对象；
     4. 注册监听：将事件、事件源、监听器绑定在一起。当事件源上发生某个事件后，执行监听器代码。

   - ServletContextListener：监听ServletContext对象的创建和销毁。
     - void contextDestroyed(ServletContextEvent sce)：ServletContext对象被销毁之前会调用该方法；
     - void ContextInitialized(ServletContextEvent sce)：ServletContext对象创建后会调用该方法。

   - 使用步骤：
     1. 定义一个类，实现ServletContextListener接口；
     2. 复写方法；
     3. 配置方式：
        - web.xml；
        - 注解配置。

- web.xml配置：

```xml
<listener>
	<listener-class>cn.dylanphang.listener.ContextLoaderListener</listener-class>
</listener>
```

- 注解配置：

```java
@WebListener("/*")
public class DemoListener implements ServletContextListener {
	...
}
```

# JSP/MVC/EL/JSTL

## JavaServer Pages

1. 概念

   - Java服务器端页面，可以在其中同时书写Java与Html代码，简化书写。
```xml
<!-- 在maven项目中使用jsp需要添加以下依赖 -->
<dependency>
    <groupId>javax.servlet.jsp</groupId>
    <artifactId>javax.servlet.jsp-api</artifactId>
    <version>2.3.3</version>
    <scope>provided</scope>
</dependency>
```

2. 原理

   - JSP本质上就是一个Servlet。

3. 格式

   - <%    代码%>：其中定义的java代码位于service方法中。service方法中可以定义的，该脚本中亦可以；
   - <%！代码%>：其中定义的java代码位于jsp转换为java类后的成员字段位置；
   - <%=  代码%>：其中定义的java代码会输出到页面上。输出语句中可以定义的，该脚本中亦可以。

4. jsp配置指令

   1. 作用：用于配置jsp页面，位于<html>标签之前，用于导入资源文件

   2. 格式：<% **指令名称** 属性名=属性值 ... %>

   3. **指令名称**：
- page：用于配置jsp页面的基本信息。需要了解以下**属性**：
      
  1. contentType：等同于response.setContentType()；
           - 设置响应体的mime类型以及字符集
           - 设置当前jsp页面的编码（注：仅支持高级的IDE，如果使用的是较为低级的工具，请使用pageEncoding属性设置当前页面的字符集）
      
  2. import：用于导包；
        3. errorPage：当前页面发送异常后，会自动跳转到指定的错误页面；
        4. isErrorPage：标识当前页是否是错误页面；
           - ture：是，可以使用内置对象exception
           - false：默认值。否，不可以使用内置对象exception
  
- include：页面包含的、导入页面的资源文件。
      
  - 例：<%@include file="top.jsp" %>
  
- taglib：导入资源。**（*最重要的使用方式是导入JSTL资源）**
      
  - 例：<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
  
5. jsp注释
   1. 支持html注释：但只能够注释html代码片段
   2. jsp注释：推荐使用，可以注释jsp页面中的所有内容
      - 格式：<%-- --%>

6. 内置对象
   - 概念：在jsp页面中不需要创建、可以直接使用的对象
   - 一共有9个jsp的内置对象：

|   变量名    |      真实类型       |                         作用                         |
| :---------: | :-----------------: | :--------------------------------------------------: |
| pageContext |     pageContext     | 当前页面共享数据，同时可以使用它获取其他八个内置对象 |
|   request   | HttpServletRequest  |            一次请求获取的多个资源（转发）            |
|  response   | HttpServletResponse |                       响应对象                       |
|   session   |     HttpSession     |                 一次会话的多个请求间                 |
| application |   ServletContext    |                  所有用户间共享数据                  |
|    page     |       Object        |           当前页面（Servlet）的对象，this            |
|     out     |      JspWriter      |              输出对象，数据输出到页面上              |
|   config    |    ServletConfig    |                   Servlet配置对象                    |
|  exception  |      Throwable      |                       异常对象                       |

- 注1：内置变量out，和response.getWriter()类似，但存在区别。
  - response.getWriter().write()和out.write()的区别：
    - 在tomcat服务器真正给客户端做出响应之前，会先找到response缓冲区数据，再找out缓冲区数据；
    - response.getWriter().write()数据的输出永远在out.write()之前。

- 注2：**<u>内置变量pageContext同时也是EL表达式的隐式变量之一，常用的用法是通过它获取request对象并获得虚拟目录的路径：${pageContext.request.contextPath}。</u>**

## Model View Controller

1. jsp演变历史
   - 早期的web只有servlet，只能使用response输出标签数据，十分麻烦；
   - 后来出现了jsp，大大简化了servlet的开发。但如果过度使用jsp，会使得源码中充斥大量的Java与Html代码，是的维护困难，难以分工协作；
   - 最后，java的web开发借鉴MVC开发模式，使得程序的设计更加具有合理性。

2. 什么是MVC？

   - M：Model，模型。（JavaBean）
     - 完成具体的业务操作，如：查询数据库，封装对象；

   - V：View，视图。（JSP）
     - 展示数据；

   - C：Controller，控制器。（Servlet）
     - 获取用户的输入；
     - 调用模型；
     - 将数据交给视图进行展示。

3. MVC的优缺点

   1. 优点：
      - 耦合性低，方便维护，利于分工协作；
      - 重用性高。

   2. 缺点：
      - 使得项目架构变得复杂，对开发人员能力的要求高。

## Expression Language

1. 概念与作用
   - Expression Language，表达式语言。
   - 用于替换和简化jsp页面中java代码的编写。

2. 语法：${表达式}
3. 注意事项
   - jsp默认支持el表达式，如果有需要忽略el表达式，可以做以下设置：
     - 在jsp的page指令中，设置键值对：isELIgnored=“true”，表示当前jsp页面会忽略所有的el表达式；
     - \\${表达式}：忽略当前的el表达式。


4. 运算符

|    类型    |           形式            |
| :--------: | :-----------------------: |
| 算数运算符 |  +  -  *  /(div)  %(mod)  |
| 比较运算符 |   \>  <  >=  <=  ==  !=   |
| 逻辑运算符 | &&(and)  \|\|(or)  !(not) |
|  空运算符  |           empty           |

- 注：空运算符empty

  - 功能：用于判断字符串、集合、数组对象是否为null或者是否长度为0。
    - 例：${not empty list}用于判断字符串、集合、数组对象是否不为null或者是否长度大于0。

5. 获取值语法：**el表达式只能从域对象中获取值；**
   1. ${域名称.键名}：从指定域中获取指定键的对应值。
      - 例：在request域中存储了键值对name=“张三”，则可以通过${requestScope.name}获取键的对应值“张三”。
      - el表达式中有4个域名城，分别对应着jsp中的4个内置对象；

|  EL表达式域名称  |     JSP内置对象（域对象）     |
| :--------------: | :---------------------------: |
|    pageScope     |          pageContext          |
|   requestScope   |            request            |
|   sessionScope   |            session            |
| applicationScope | application（ServletContext） |

- 注：域的范围依次从上到下增大。

  2. ${键名}：直接在表达式中书写键名，程序将依次从最小的域中查找是否有该键对应的值，直到找到为止。

  3. 获取对象、List集合、Map集合中的值：

     - 对象：${域名称.键名.属性名}
       - 其本质上会调用对象中响应的getter方法。

     - List集合：${域名称.键名[索引]}
     - Map集合：
       1. ${域名称.键名.key名称}
       2. ${域名称.键名["key名称"]}

6. 隐式对象
   - el表达式中共有11个隐式对象；
   - **pageContext：获取jsp其他八个内置对象。**
     - 例：**${pageContext.request.contextPath}，动态获取虚拟目录。**

## JSP Standarded Tag Library

1. 概念与作用
   - JSP标准标签库，是由Apache组织提供的开源的免费jsp标签。
   - 用于简化和替换jsp页面上的java代码。

```xml
<!-- 在maven项目中使用jstl需要添加以下依赖 -->
<dependency>
    <groupId>jstl</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2</version>
</dependency>
```

2. 使用步骤
   1. 导入jstl相关的jar包；
   2. 引入标签库：通过taglib指令引入标签库；
      - **格式：<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>**
   3. 在html中使用标签。

3. 常用JSTL标签
   1. if：相当于java中的if语句；
   2. choose：相当于java中的switch语句；
   3. foreach：相当于java中的for语句。

- if标签：

```jsp
<%
    // 1.用于测试<if>标签
    boolean display_content = true;
    request.setAttribute("display_content", display_content);
%>
<h3>if标签</h3>
<ul>
    <li>
        > display_content == true
        <c:if test="${display_content}">
            <p>I'm the content inside tag-if.</p>
        </c:if>
    </li>
    <li>
        > display_content == false
        <% request.setAttribute("display_content", false); %>
        <c:if test="${display_content}">
            <p>I'm the content inside tag-if.</p>
        </c:if>
    </li>
</ul>
```

- choose标签：

```jsp
<%
    // 2.用于测试<choose>标签
    request.setAttribute("number", 6);
%>
<h3>choose标签</h3>
<ul>
    <li>
        <c:choose>
            <c:when test="${number == 1}">星期一</c:when>
            <c:when test="${number == 2}">星期二</c:when>
            <c:when test="${number == 3}">星期三</c:when>
            <c:when test="${number == 4}">星期四</c:when>
            <c:when test="${number == 5}">星期五</c:when>
            <c:when test="${number == 6}">星期六</c:when>
            <c:when test="${number == 7}">星期日</c:when>

            <c:otherwise>日期设置不正确！！！</c:otherwise>
        </c:choose>
    </li>
</ul>
```

- foreach标签：

```jsp
<%
    // 3.用于测试<foreach>标签
    ArrayList<String> list = new ArrayList<>();
    list.add("张三");
    list.add("李四");
    list.add("王五");

    request.setAttribute("list", list);
%>
<h3>foreach标签简单应用</h3>
<ul>
    <c:forEach begin="0" end="2" var="i" step="1" varStatus="status">
        <li>数字：${i} 逻辑索引：${status.index} 当前循环次数：${status.count}</li>
    </c:forEach>
</ul>

<h3>foreach标签普通应用</h3>
<ul>
    <c:forEach items="${list}" var="name" step="1" varStatus="status">
        <li>姓名：${name} 数组列表索引：${status.index} 当前循环次数：${status.count}</li>
    </c:forEach>
</ul>
```

