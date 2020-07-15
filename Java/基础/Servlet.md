[toc]
## **Servlet 是什么？**

Java Servlet 是运行在 Web 服务器或应用服务器上的程序，它是作为来自 Web 浏览器或其他 HTTP 客户端的请求和 HTTP 服务器上的数据库或应用程序之间的中间层。

==Servlet是一个Java编写的程序，此程序是基于Http协议的，在服务器端运行的(如tomcat)，是按照Servlet规范编写的一个Java类。Servlet可以支持客户端和服务器之间的请求影响==

使用 Servlet，您可以收集来自网页表单的用户输入，呈现来自数据库或者其他源的记录，还可以动态创建网页。

Java Servlet 通常情况下与使用 CGI（Common Gateway Interface，公共网关接口）实现的程序可以达到异曲同工的效果。但是相比于 CGI，Servlet 有以下几点优势：

- 性能明显更好。
- Servlet 在 Web 服务器的地址空间内执行。这样它就没有必要再创建一个单独的进程来处理每个客户端请求。
- Servlet 是独立于平台的，因为它们是用 Java 编写的。
- 服务器上的 Java 安全管理器执行了一系列限制，以保护服务器计算机上的资源。因此，Servlet 是可信的。
- Java 类库的全部功能对 Servlet 来说都是可用的。它可以通过 sockets 和 RMI 机制与 applets、数据库或其他软件进行交互。

## Servlet处理流程
![](https://raw.github.com/binbinbin5/myPics/master/Servlet%E5%A4%84%E7%90%86%E6%B5%81%E7%A8%8B.png)

- 服务器创建对象，并且执行init方法，并且调用service方法
- 容器创建两个对象，httpservletresponse和httpservletrequest
- http请求为get那么doget，post那么dopost
- Servlet处理请求的方式是以线程的方式

>流程
1. 浏览器向服务器发出GET请求(请求服务器ServletA)
2. 服务器上的容器逻辑接收到该url,根据该url判断为Servlet请求，此时容器逻辑将产生两个对象：请求对象(HttpServletRequest)和响应对象(HttpServletResponce)
3. 容器逻辑根据url找到目标Servlet(本示例目标Servlet为ServletA),且创建一个线程A
4. 容器逻辑将刚才创建的请求对象和响应对象传递给线程A
5. 容器逻辑调用Servlet的service()方法
6. service()方法根据请求类型(本示例为GET请求)调用doGet()(本示例调用doGet())或doPost()方法
7. doGet()执行完后，将结果返回给容器逻辑
8. 线程A被销毁或被放在线程池中
## forward和redirect区别
- 服务器重定向，直接访问URL，服务器直接访问，客户端不知道，浏览器上没有重新跳转，定向过程中有一个request。
- redirect服务器重定向，完全跳转，浏览器会跳转到其他网址，（baidu.com变成www.baidu.com）。多用一次网络请求，

## **javax.servlet 包中包含了 7 个接口 ,3 个类和 2 个异常类** 


接口 :

```
RequestDispatcher, 
Servlet,ServletConfig,
ServletContext, ServletRequest, 
ServletResponse  
SingleThreadModel
```


类 :

```
GenericServlet, 
ServletInputStream,
ServletOutputStream
```

异常类 :

```
ServletException   
UnavailableException
```




### **一、UML**



下图为Servlet UML关系图。



![](https://raw.githubusercontent.com/binbinbin5/myPics/master/imgs/554dfb635f0e1f394e516ea8203a6e1e.jpg)



从图中，可以看出：




```
getParameter()是获取POST/GET传递的参数值； 
getInitParameter（）获取Tomcat的server.xml中设置Context的初始化参数 
getAttribute()是获取对象容器中的数据值； getRequestDispatcher（）是请求转发。
```




1. 抽象类HttpServlet继承抽象类GenericServlet，其有两个比较关键的方法，doGet()和doPost()
2. GenericServlet实现接口Servlet,ServletConfig,Serializable
3. MyServlet(用户自定义Servlet类)继承HttpServlet，重写抽象类HttpServlet的doGet()和doPost()方法

注：任何一个用户自定义Servlet，只需重写抽象类HttpServlet的doPost()和doGet()即可，如上图的MyServlet



### **二、Servlet在容器中的执行过程**

 

Servlet只有放在容器中，方可执行，且Servlet容器种类较多，如Tomcat,WebLogic等。






分析：

1. 浏览器向服务器发出GET请求(请求服务器ServletA)
2. 服务器上的容器逻辑接收到该url,根据该url判断为Servlet请求，此时容器逻辑将产生两个对象：请求对象(HttpServletRequest)和响应对象(HttpServletResponce)
3. 容器逻辑根据url找到目标Servlet(本示例目标Servlet为ServletA),且创建一个线程A
4. 容器逻辑将刚才创建的请求对象和响应对象传递给线程A
5. 容器逻辑调用Servlet的service()方法
6. service()方法根据请求类型(本示例为GET请求)调用doGet()(本示例调用doGet())或doPost()方法
7. doGet()执行完后，将结果返回给容器逻辑
8. 线程A被销毁或被放在线程池中



注意：

1. 在容器中的每个Servlet原则上只有一个实例
2. 每个请求对应一个线程
3. 多个线程可作用于同一个Servlet(这是造成Servlet线程不安全的根本原因)
4. 每个线程一旦执行完任务，就被销毁或放在线程池中等待回收


### **三、Servlet在JavaWeb中扮演的角色**



**下面的方法可用在 Servlet 程序中读取 HTTP 头**

这些方法通过 *HttpServletRequest* 对象可用：


```
**1）Cookie[] getCookies()**
//返回一个数组，包含客户端发送该请求的所有的 Cookie 对象。


**2）Object getAttribute(String name)**
//以对象形式返回已命名属性的值，如果没有给定名称的属性存在，则返回 null。


**3）String getHeader(String name)**
//以字符串形式返回指定的请求头的值。Cookie也是头的一种；


**4）String getParameter(String name)**
//以字符串形式返回请求参数的值，或者如果参数不存在则返回 null。
```



除此之外：Servlet在JavaWeb中，扮演两个角色**：页面角色和控制器角色**。

有了jsp等动态页面技术后，Servlet更侧重于控制器角色，jsp+servlert+model 形成基本的三层架构



- （一）页面Page角色

```
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
       request.setCharacterEncoding("UTF-8");
       response.setContentType("text/html;charset=utf-8");
       PrintWriter out=response.getWriter();
       out.println("Hello!Servlet.");
   }
```



- （二）控制器角色

jsp充当页面角色，Servlet扮演控制器角色，两者组合构建基本的MVC三层架构模式



 

### **四、Servlet在容器中的生命周期**

servlet 的生命周期

servlet的生命周期是由servlet的容器来控制的，主要分为初始化、运行、销毁3个阶段，Servlet容器加载servlet，实例化后调用init()方法进行初始化，当请求到达时运行service()方法，根据对应请求调用doget或dopost方法，当服务器决定将实例销毁时调用destroy()方法（释放servlet占用的资源:关闭数据库连接、关闭文件输入输出流），在整个生命周期中，servlet的初始化和销毁只会发生一次，而service方法执行的次数则取决于servlet被客户端访问的次数。



下图为Servlet生命周期简要概图


![](https://raw.githubusercontent.com/binbinbin5/myPics/master/imgs/20190730103858.png)



分析：

- 第一步：容器先==加载==Servlet类

- 第二步：容器==创建==实例化Servlet(Servlet无参构造函数执行)

- 第三步：执行==init()方法==（在Servlet生命周期中，只执行一次，且在service()方法执行前执行） 负责在装载Servlet时初始化Servlet对象

- 第四步：==执行service()方法，处理客户请求核心方法==，一般HttpServlet中会有get,post两种处理方式。在调用doGet和doPost方法时会构造servletRequest和servletResponse请求和响应对象作为参数。

- 第五步：==执行destroy()，销毁线程==，在停止并且卸载Servlet时执行，负责释放资源初始化阶段：Servlet启动，会读取配置文件中的信息，构造指定的Servlet对象，创建ServletConfig对象，将ServletConfig作为参数来调用init()方法



### **五、Servlet过滤器的配置包括两部分**

- 第一部分是过滤器在Web应用中的定义，由<filter>元素表示，包括<filter-name>和<filter-class>两个必需的子元素

- 第二部分是过滤器映射的定义，由<filter-mapping>元素表示,可以将一个过滤器映射到一个或者多个Servlet或JSP文件，也可以采用url-pattern将过滤器映射到任意特征的URL。