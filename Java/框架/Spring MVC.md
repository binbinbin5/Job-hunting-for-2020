[toc]

# 先来看一下什么是 MVC 模式

web层的MVC框架：
- model 模型
- view  视图
- contorller  控制器

MVC 是一种设计模式：将责任进行拆分，不同组件负责不同的事情。

**MVC 的原理图如下：**

![](http://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-10-11/60679444.jpg)




优点：
- 结构清晰
- 更好维护

缺点：
- 复杂

简单介绍：

SpringMVC 框架是以请求为驱动，围绕 Servlet 设计，将请求发给控制器，然后通过模型对象，分派器来展示请求结果视图。其中核心类是 DispatcherServlet，它是一个 Servlet，顶层是实现的Servlet接口。

# SpringMVC 使用

需要在 web.xml 中配置 DispatcherServlet 。并且需要配置 Spring 监听器ContextLoaderListener。目的是为了让springmvc处理各种请求（映射配置）。

前端控制器是希望我们院里原生API，将操作进一步简化。本身还是基于serlvt设计的。

默认情况下，用dispatcherServlet的名字当做命名空间，[namespace]-servlet.xml之下寻找


```xml

<listener>
	<listener-class>org.springframework.web.context.ContextLoaderListener
	</listener-class>
</listener>

<!--注册一个前端控制器dispatcherSerlevt-->
<servlet>
<!--这里写的名字有讲究，如果不去修改spring配置文件的默认位置，
那么springMVC会去web-inf下面找一个叫springmnv-servel.xml文件-->
	<servlet-name>springmvc</servlet-name>
	<servlet-class>org.springframework.web.servlet.DispatcherServlet
	</servlet-class>
	<!-- 如果不设置init-param标签，则必须在/WEB-INF/下创建xxx-servlet.xml文件，其中xxx是servlet-name中配置的名称。
	
	这是上下文配置文件位置的制定，推荐使用下列配置路径
	在该类路径下面寻找springmvc-servlet.xml
	-->
	<init-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:spring/springmvc-servlet.xml</param-value>
	</init-param>
	<load-on-startup>1</load-on-startup>
</servlet>


<!--servelt映射，可以重新申明配置文件名字-->
<servlet-mapping>
	<servlet-name>springmvc</servlet-name>
	<url-pattern>/</url-pattern>
</servlet-mapping>

```


url-pattern写法问题

- /
- /*  (永远不要这么写)
- *.do

/*  (永远不要这么写原因)

请求/helloController过去的时候,它的视图名称是girl.jsp页面它将其当做一个叫做girl.JSP的请求,尝试去匹配controller,但是容器中根本找不到这样的controller,所以无法匹配导致404

*.do  一些团队习惯上将请求加一个小尾巴用来区分还有一些

​     *.action

```
<servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-                         class>
</servlet>
<servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>*.do</url-pattern>
</servlet-mapping>
<bean class="cn.edu.sict.controller.HelloController" name="/helloController.do"></bean>
http://localhost/helloController.do


/用法

处理所有的请求  他处理完成后不会再将其作为一个新的请求,而是将渲染的结果直接返回给浏览器
```




springmvc-servlet.xml 配置视图解析器
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
    http://www.springframework.org/schema/mvc
    http://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-3.2.xsd">
 
    <!-- 控制器注解 -->
    <mvc:annotation-driven />
    <!--配置一个扫描注解的包-->
    <context:component-scan base-package="comm.dhee" />
    
    <!-- 配置试图解析器 
    它支持各种视图技术：JSP,freemaker
    
    
    -->
    <bean id="jspViewResolver"
    class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!--前缀
            他是我们请求响应的资源的路径配置
        -->
        <property name="prefix" value="/WEB-INF/jsp/"></property>
        <!--后缀
            此时 前缀+视图名称（logicViewName）+后缀 就是目标文件
        -->
        <property name="suffix" value=".jsp"></property>
    </bean>
</beans>
```
视图逻辑 = 前缀+视图名称（logicViewName）+后缀
### 控制器的解释
是一种比较传统的实现一个接口的方式完成的，Controller。
如果要给接口只有一个方式，这种接口叫函数式接口。

```java
public interface Controller{
    @Nullable
    ModeAndView HanderRequest(HttpServetRequest request,HttpServetRespone response) throw Exception;
}
//sevlet的doget(),doPost()
//就处理一个请求 
```
这个已经被淘汰，使用注解

# 注解开发模式（**）
1. 记得配置扫描的包
1. 然后添加下面两个注解
    - @Controller 不需要集成和实现接口，标记为spring的一个组件，并且是控制器的组件，我们的handermapping会去扫描这个controller匹配，然后把处理工作交给他。
    - @RequestMapping(URL) 是一个匹配的规则，通过URL进行匹配 @RequestMapping 推荐写在类上,或者和方法一共使用
```
@Controller
@RequestMappering("/order")
public class ByeController{
    moder.addAttribute("model","moderllor");
    //modeANDciew
    @RequestMappering("/bye")
    return "bye";//这边去的是/jsp/bye.jsp页面
    //返回类型是String就是我们的视图名称
}
```
### 转发和重定向
- 重定向到页面：redirect:/jsp/目标.jsp (:后面是路径，不写/是相对路径，根据上下文决定)，两次请求，和视图解析规则无关
- 转发到页面：默认到选项
- 转发到另一个控制器：forward:path


### 关于访问WEB元素
- request
- session
- application
可以使用模拟的对象完成操作,也可以使用原生的servletAPI完成操作

```java
package cn.edu.sict.controller;


import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.context.request.WebRequest;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;

@Controller
@RequestMapping("/web")
public class WebElementController {

    //模拟请求
    @RequestMapping("/request")
    public String request(WebRequest request) {
        System.out.println(request.getParameter("girl"));
        return "forward";
    }

    @RequestMapping("/request2")
    public String request2(HttpServletRequest request) {
        System.out.println(request.getParameter("boy"));
        return "/forward";

    }
    @RequestMapping("/request3")
    public String request3(HttpSession session) {
        session.setAttribute("LOL","疾风剑豪");
        return "redirect:/jsp/redirect.jsp";

    }
    @RequestMapping("/request4")
    public String request4(HttpSession session) {
        session.setAttribute("LOL","疾风剑豪");
        session.getServletContext().setAttribute("DOTA2","无极剑圣");
        return "redirect:/jsp/redirect.jsp";

    }
}
```

### 注解详解
#### @RequestMapping参数

- value,path可以为其指定多个值

```java
@RequestMapping(value = {"/m1", "/m2"})
@RequestMapping(path = {"/m3", "/m4"})
```

- method控制请求类型POST,GET

```java
@RequestMapping(path = {"/m3", "/m4"}, method = RequestMethod.POST)
数组形式 method = {RequestMethod.GET, RequestMethod.POST}
//否则错误信息如下

HTTP Status 405 – Method Not Allowed

Type Status Report
Message Request method 'POST' not supported
Description The method received in the request-line is known by the origin server but not supported by the target resource.

Apache Tomcat/8.5.40
```

- params描述请求参数

```java
params = {"name","pass"}
控制参数值
params = {"name=arno", "pass=tiger"}
```

- headers:能够影响浏览器的行为
- consumers消费者媒体类型可以限定为application/json/;charset=UTF-8
- produces产生的响应类型

#### 关于请求路径的问题

Spring支持ant风格

- ?     任意字符斜杠问号除外
- `*`    任意个字符符斜杠除外
- ** 支持任意层的路径@RequestMapping(path = {"/m7/**"})

#### @GetMapping()

#### @PostMapping()

可以设定具体请求类型为GET或者POST

#### 对于非get、post的请求需要额外添加一个过滤器

1.在XML文件中创建一个支持所有HTTP请求类型的过滤器

```java
<filter>
    <filter-name>hiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>hiddenHttpMethodFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

2.在表单里面添加一个隐藏的参数

```java
<input type="hidden" name="_method" value="DELETE">
```

#### 关于静态资源访问

由于servlet设置了URL匹配方式为/所以,它将静态资源也当做一个后台请求比如http://localhost:8080/static/css/index.css

他尝试请求static/css/index.css的Controller里面的RequestMapping的组合,因为没有,所以404

解决方式让springMVC单独处理,将这些交给容器的默认的servlet处理,就不让DispatcherServlet来处理了

解决方式一

```XML
<!--默认servlet处理-->
    <mvc:default-servlet-handler/><!--只加这一个会全部处理掉-->
    <mvc:annotation-driven/><!--重新把注解模式激活Controller才可以使用-->
```

解决方式二

通过映射关系,一一编写规则

```XML
<mvc:resources mapping="/static/css/*" location="static/css/"/>
```

解决方式三

自行在web.xml定义映射规则

#### @PathVariable

restful风格

常规    http://localhost:8080/product/add?id=1&name=arnome=&price=22.2

改进后    http://localhost:8080/product/add/1/arno/22.2

```
@Controller
@RequestMapping("/product")
public class ProdectController {


    @GetMapping("/add/{id}/{name}/{price}")
    private String addProdect(@PathVariable("id") Integer id, @PathVariable("name") String name, @PathVariable("price") Double price) {
        System.out.println(id+name+price);
        return "forward";
    }
}
```

#### @ResponseBody

用于返回数据一般返回json格式

#### @resController
= responseBody+Controller

#### 关于post请求的编码问题

##### 1.接收乱码

SPringMVC提供了过滤器通过注册即可

```XML
<!--编码过滤器-->
<filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
        <param-name>forceRequestEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

##### 2.返回数据乱码

使用@RequestMapping注解的produces属性设置编码

```java
package cn.edu.sict.controller;


import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
@RequestMapping(value = "/user", produces="text/html;charset=UTF-8")
public class UserController {
    @PutMapping("/put")
    @ResponseBody
    public String put(String name) {
        System.out.println(name);
       /* Map<String, Object> map = new HashMap<>();
        map.put("msg", "ok");*/
        return  name;
    }
}
```

#### 关于form表单提交数据的方式

```Java
public String put(String name) {
    System.out.println(name);
   /* Map<String, Object> map = new HashMap<>();
    map.put("msg", "ok");*/
    return  name;
}
<form action="${ctx}/user/put" method="post">
    <input type="text" name="name"><br>
    <input type="hidden" name="_method" value="put">
    <input type="submit" value="提交">
</form>
```

##### 方式一：通过属性名字绑定

页面中的name值要和后台的形参保持一致多个值绑定多个形参（不利于多个值）

##### 方式二：@RequestParam注解

```Java
public String put(@RequestParam("name")String name,@RequestParam("password")String password)
```

##### 方式三：直接使用pojo方式传递

```Java
@PutMapping("/put")
@ResponseBody
public String put(User user) {
    System.out.println(user.getName()+"---"+user.getPassword());
   /* Map<String, Object> map = new HashMap<>();
    map.put("msg", "ok");*/
    return  user.getName()+"---"+user.getPassword();
}
```

user中的属性值要和页面中的type值value保持一致

#### Date类型如何提交

##### 方式一日期类型解析

```java
    //日期类型解析
    @InitBinder
    public void init(WebDataBinder webDataBinder) {
//        指定解析格式
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy/MM/dd");
        simpleDateFormat.setLenient(false);//不是严格解析
        webDataBinder.registerCustomEditor(Date.class, new CustomDateEditor(simpleDateFormat, false));//注册一个自定义编辑器设置日期型数据类型

    }
```

方式二注解pojo类中

```
@DateTimeFormat(pattern = "yyyy-mm-dd")
private Date date;
```

##### 时间+日期

```java
   @InitBinder
    public void init(WebDataBinder webDataBinder) {
//        指定解析格式
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        simpleDateFormat.setLenient(false);//不是严格解析
        webDataBinder.registerCustomEditor(Date.class, new CustomDateEditor(simpleDateFormat, false));//注册一个自定义编辑器设置日期型数据类型

    }
```



#### @ModelAttribute

##### 使用方式一

```Java
@ModelAttribute//会在controller里面的任意一个具体的方法之前都会执行
public User init(User user) {
    System.out.println("init()执行了....");
    return user;
}

@RequestMapping("/login")
public String login(Model model) {
    System.out.println(model.containsAttribute("users"));
    System.out.println(model.containsAttribute("user"));
    System.out.println(model.containsAttribute("sdgsdg"));
    return "msg";
}
```

##### 使用方式二

```Java
@ModelAttribute("user")//会在controller里面的任意一个具体的方法之前都会执行
public void init2(Model model) {
    User user = new User();
    user.setName("哈哈哈");
    model.addAttribute("user", user);

}
```

##### 使用方式三

```Java
@RequestMapping("/login")//会在controller里面的任意一个具体的方法之前都会执行
   public String init2(@ModelAttribute User user) {
       System.out.println(user.getName() + "---" + user.getPassword() + "---" + user.getDate());
       return "msg";
```

#### @ResponseBody

JSON数据,不是通过from表单传递

ajax({

data:

})

#### @SessionAttributes("user")

用在类上面会将模型自动填充到回话上面去

#### @SessionAttribute

检查当前访问的会话里面要有某个对象

```java
@RequestMapping("/login")
public String login(@SessionAttribute Dog dog) {
    return "redirect:/jsp/logDog.jsp";
}
```

应用可以在注册时使用SessionAttributes将登陆的用户保存到session登陆时使用SessionAttribute检查验证登陆

#### @CookieValue

获取cookie

```java
@Controller
@RequestMapping(value = "/cookie", produces = "text/html;charset=UTF-8")
public class CookieController {

    @RequestMapping("/c1")
    @ResponseBody
    public String c1(@CookieValue("JSESSIONID") String JSESSIONID) {
        System.out.println(JSESSIONID);
        return "JSESSIONID=" + JSESSIONID;
    }
}
```

#### @PathVariable 

可以将 URL 中占位符参数绑定到控制器处理方法的入参中

```java
@RequestMapping("/pathVariable/{name}")
	public String pathVariable(@PathVariable("name")String name){
		System.out.println("hello "+name);
		return "helloworld";
	}
```

# JSON数据交互

额外依赖

```XML
<!--json依赖-->
<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.3</version>
</dependency>

<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-core -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.9.3</version>
</dependency>
<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-annotations -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
    <version>2.9.3</version>
</dependency>
<!-- https://mvnrepository.com/artifact/net.sf.json-lib/json-lib -->
<dependency>
    <groupId>net.sf.json-lib</groupId>
    <artifactId>json-lib</artifactId>
    <version>2.4</version>
</dependency>
<!--添加处理json的Javabean-->
<!-- https://mvnrepository.com/artifact/org.codehaus.jackson/jackson-core-asl -->
<dependency>
    <groupId>org.codehaus.jackson</groupId>
    <artifactId>jackson-core-asl</artifactId>
    <version>1.9.2</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.codehaus.jackson/jackson-mapper-asl -->
<dependency>
    <groupId>org.codehaus.jackson</groupId>
    <artifactId>jackson-mapper-asl</artifactId>
    <version>1.9.2</version>
</dependency>
<!--添加处理json的Javabean  end-->
```

### json向前台返回数据

##### 后台

```java
package cn.edu.sict.controller;


import cn.edu.sict.pojo.User;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

//@RestController= @RequestMapping+ @ResponseBody
@RestController
@RequestMapping("/json")
public class JSONController {


    @RequestMapping("/m1")
    public User m1() {
        User user = new User();
        user.setName("许晴");
        user.setPwd("12321");
        return user;
    }


    @RequestMapping("/m2")
    public Map<String, Object> m2() {
        Map<String, Object> map = new HashMap();
        map.put("name", "徐青青");
        map.put("age", 28);
        return map;
    }

    @RequestMapping("/m3")
    public User[] m3() {
        User user = new User();
        user.setName("开玩笑");
        user.setPwd("12321");
        User user2 = new User();
        user2.setName("不开玩笑");
        user2.setPwd("12321");
        return new User[]{user, user2};
    }

    @RequestMapping("/m4")
    public List<User> m4() {
        List<User> users = new ArrayList<User>();
        User user = new User();
        user.setName("鬼子来了");
        user.setPwd("12321");
        User user2 = new User();
        user2.setName("鬼子没来");
        user2.setPwd("12321");
        users.add(user);
        users.add(user2);
        return users;
    }

    @RequestMapping("/m5")
    public List<Map<String, User>> m5() {
        List<Map<String, User>> list = new ArrayList();

        User user1 = new User();
        user1.setName("鬼子来了");
        user1.setPwd("12321");
        User user2 = new User();
        user2.setName("鬼子没来");
        user2.setPwd("12321");

        Map<String, User> map1 = new HashMap<String, User>();
        map1.put("u1", user1);
        map1.put("u2", user2);

        User user3 = new User();
        user3.setName("鬼子跑来了");
        user3.setPwd("12321");
        User user4 = new User();
        user4.setName("鬼子没跑来");
        user4.setPwd("12321");

        Map<String, User> map2 = new HashMap<String, User>();
        map2.put("u1", user3);
        map2.put("u2", user4);

        list.add(map1);
        list.add(map2);
        return list;
    }
}
```

##### 前台

```javascript
$(function () {
    //   1.解析返回User
    $("button:eq(0)").click(function () {
        $.ajax({
            url:"${ctx}/json/m1",
            type:"post",
            success:function (data) {
                alert("name:"+data.name+"---pwd:"+data.pwd);
            }
        })
    })
    //   2.解析返回map
    $("button:eq(1)").click(function () {
        $.ajax({
            url:"${ctx}/json/m2",
            type:"post",
            success:function (data) {
                alert("name:"+data.name+"---age:"+data.age);
            }
        })
    })
    //   3.解析返回数组
    $("button:eq(2)").click(function () {
        $.ajax({
            url:"${ctx}/json/m3",
            type:"post",
            success:function (data) {
                for (var i=0;i<data.length;i++){
                    alert("name:"+data[i].name+"---pwd:"+data[i].pwd);
                }

            }
        })
    })
    //   4.解析返回list
    $("button:eq(3)").click(function () {
        $.ajax({
            url:"${ctx}/json/m4",
            type:"post",
            success:function (data) {
                for (var i=0;i<data.length;i++){
                    alert("name:"+data[i].name+"---pwd:"+data[i].pwd);
                }

            }
        })
    })
    //   5.解析返回List<Map<String, User>>
    $("button:eq(4)").click(function () {
        $.ajax({
            url:"${ctx}/json/m5",
            type:"post",
            success:function (data) {
                for (var i=0;i<data.length;i++){
                    alert("name:"+data[i].u1.name+"---pwd:"+data[i].u1.pwd);
                    alert("name:"+data[i].u2.name+"---pwd:"+data[i].u2.pwd);
                }
            }
        })
    })
})
```

### json向后台传送数据

##### 前台

```html
<button>发送一个User对象到后台,已Ajax形式发送</button>
<script>
    $("button:eq(0)").click(function () {
        var obj = {
            "name": "叶问",
            "pwd": "怕老婆"
        }
        $.ajax({
            url: "${ctx}/json2/add",
            type: "post",
            data: JSON.stringify(obj),
            contentType: "application/json;charset=utf-8",
            success: function (date) {

            }
        })
    })
</script>
```

##### 后台

```java
  //前台如何提交一个User对象过来
    @RequestMapping("/add")
//    user入参只能处理表单提交的数据
    public String add(@RequestBody User user) {
        System.out.println(user.getName()+user.getPwd());
        return "msg";
    }
```

### json向后台传送多组数据

##### 前台

```javascript
$("button:eq(1)").click(function () {
    var obj = {
        "name": "叶问",
        "pwd": "怕老婆"
    };
    var obj2 = {
        "name": "李小龙",
        "pwd": "怕老婆"
    }
    var arr = new Array();
    arr.push(obj);
    arr.push(obj2);
    $.ajax({
        url: "${ctx}/json2/addList",
        type: "post",
        data: JSON.stringify(arr),
        contentType: "application/json;charset=utf-8",
        success: function (date) {
            if (date.code == "2000") {
                alert("发送数据成功!")
            }
        }
    })
```

##### 后台

```java 
    //前台如何提交一个User对象过来
    @ResponseBody
    @RequestMapping("/addList")
//    user入参只能处理表单提交的数据
    public Map<String, Object> addList(@RequestBody List<User> user) {
        Map<String, Object> map = new HashMap<String, Object>();
        System.out.println(user);
        map.put("code", "2000");
        return map;
    }
```

## 关于form提交数据和ajax自定义方式的区别

form提交方式截图:

![](https://raw.githubusercontent.com/binbinbin5/myPics/master/file/formAndAjax.png)


通过ajax组装的json发送数据方式分析

![](https://raw.githubusercontent.com/binbinbin5/myPics/master/file/formAndAjax2.png)


两者发送的区域存在不同所以处理方式也不会相同

对于form提交数据   ContentType: application/x-www-form-urlencoded**

对于ajas发送json则是  contentType: "application/json"

# xml数据交互

添加依赖后处理方式将会以XML方式进行原有json将会不可用

```XML
<!--添加XML处理-->
<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.dataformat/jackson-dataformat-xml -->
<dependency>
  <groupId>com.fasterxml.jackson.dataformat</groupId>
  <artifactId>jackson-dataformat-xml</artifactId>
  <version>2.9.3</version>
</dependency>
```

```java
package cn.edu.sict.controller;


import cn.edu.sict.pojo.User;
import org.springframework.http.MediaType;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
@RequestMapping("/xml")

public class XMLController {

    @ResponseBody
    //对返回类型描述
//    @RequestMapping(value = "/m1",produces = {"application/xml"})
    @RequestMapping(value = "/m1", produces = {MediaType.APPLICATION_XML_VALUE})
    public User m1() {
        //将数据转换为XML形式
        User user = new User();
        user.setName("霍元甲");
        user.setPwd("213132");
        return user;
    }
}
```

# 上传与下载

### Apache上传组件方案

servlet3.0新特性

spring配置文件中

```XML
<bean id="multipartResolver" class="org.springframework.web.multipart.support.StandardServletMultipartResolver"></bean>
```

xml中

```xml
<servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>namespace</param-name>
        <param-value>springmvc_servlet</param-value>
    </init-param>
    <!--指定上下文路径-->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:springmvc_servlet.xml</param-value>
    </init-param>
    <!--servlet3.0配置文件上传-->
    <multipart-config>
        <max-file-size>20000000</max-file-size>
    </multipart-config>

</servlet>
<servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

### 单文件上传

##### 1.添加依赖

```XML
<!-- https://mvnrepository.com/artifact/commons-fileupload/commons-fileupload -->
<!--apache文件上传组件-->
<dependency>
  <groupId>commons-fileupload</groupId>
  <artifactId>commons-fileupload</artifactId>
  <version>1.3.3</version>
</dependency>
```

##### 2.在springMVC中注册一个文件视图解析器

```xml
<!--文件上传解析器
id必须为multipartResolver,源代码已写死
-->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <!--定义总的文件上传大小单位bytes-->
    <property name="maxUploadSize" value="1024*1024"></property>
    <!--指定上传编码-->
    <property name="defaultEncoding" value="UTF-8"></property>
    <!--单个文件最大上传大小-->
    <property name="maxUploadSizePerFile" value="2000000"></property>
</bean>
```

##### 3.创建上传页面

```HTML
<form action="${ctx}/file/upload" method="post" enctype="multipart/form-data">
    文件:<input type="file" name="file"><input type="submit" value="提交">
</form>
```



##### 4.后台接受

MultipartFile  类一般是用来接受前台传过来的文件

```Java
@Controller
@RequestMapping("/file")
public class FileController {

    private static String uploadPth = "D:/WorkFile/project/SSMQ/UploadDownload/src/main/uploadPath";

    //入参就可以代表上传的文件
    @RequestMapping("/upload")
    public String upload(@RequestParam("file") MultipartFile multipartFile, Model model) {
        //1.传到哪里去2.传什么3.细节

        //1.判断
        if (multipartFile != null && !multipartFile.isEmpty()) {
            //判断是否为空
            //2.获取原始文件名字
            String originalFilename = multipartFile.getOriginalFilename();
            //3.截取源文件文件名字前缀,不带后缀
            String fileNamePrefix = originalFilename.substring(0, originalFilename.lastIndexOf("."));
            //4.加工处理文件名,将源文件名+时间戳
            String newFileNamePrefix = fileNamePrefix + new Date().getTime();
            //5.得到新文件名字
            String newFileName = newFileNamePrefix + originalFilename.substring(originalFilename.lastIndexOf("."));
            //6.构建文件对象
            File file = new File(uploadPth, newFileName);
            //7.执行上传操作
            try {
                multipartFile.transferTo(file);
                model.addAttribute("FileName", newFileName);
            } catch (IOException e) {
                e.printStackTrace();
            }
            System.out.println(newFileName);
        }
        return "upload";
    }
}
```

### 多文件上传

```Java
//入参就可以代表上传的文件
@RequestMapping("/uploadMany")
public String uploadMany(@RequestParam("file") MultipartFile[] multipartFiles, Model model) {

    List<String> fileNames = new ArrayList<String>();
    if (multipartFiles != null && multipartFiles.length > 0) {
        //遍历
        for (MultipartFile multipartFile : multipartFiles) {
            //1.判断
            if (multipartFile != null && !multipartFile.isEmpty()) {
                //判断是否为空
                //2.获取原始文件名字
                String originalFilename = multipartFile.getOriginalFilename();
                //3.截取源文件文件名字前缀,不带后缀
                String fileNamePrefix = originalFilename.substring(0, originalFilename.lastIndexOf("."));
                //4.加工处理文件名,将源文件名+时间戳
                String newFileNamePrefix = fileNamePrefix + new Date().getTime();
                //5.得到新文件名字
                String newFileName = newFileNamePrefix + originalFilename.substring(originalFilename.lastIndexOf("."));
                //6.构建文件对象
                File file = new File(uploadPth, newFileName);
                //7.执行上传操作
                try {
                    multipartFile.transferTo(file);
                    fileNames.add(newFileName);
                } catch (IOException e) {
                    e.printStackTrace();
                }
                String type = multipartFile.getContentType();
                Long size = multipartFile.getSize();


            }
        }

    }
    model.addAttribute("fileNames",fileNames);

    System.out.println(fileNames);
    return "uploadMany";
}
```

```HTML
<form action="${ctx}/file/uploadMany" method="post" enctype="multipart/form-data">
    文件:<input type="file" name="file">
    文件:<input type="file" name="file">
    文件:<input type="file" name="file">
    <input type="submit" value="提交">
</form>
```

```jsp
上传成功,文件:
<c:forEach items="${fileNames}" var="obj">
    <h3>${obj}</h3>
</c:forEach>
```

### 文件下载

```java
@Controller
@RequestMapping("/download")
public class DownloadConroller {


    //定义一个文件下载目录
    private static String filePath = "D:/User/Pictures";

    @RequestMapping("/down")
    private String down(HttpServletResponse response) {

        //通过输出流写出
        String fileName = "哈哈哈.mp4";
         //1.获取文件下载名字
        //2.构建一个path对象,NIO新的io流
        Path path = Paths.get(filePath, fileName);
        //3.判断是否存在
        if (Files.exists(path)) {
            //存在下载
            //通过response设定响应类型
            //4.获取文件后缀
            String fileSufix = fileName.substring(fileName.lastIndexOf(".") + 1);
            //5.添加头部信息设置cotenttype

            //6.设置contentType 指定下载行为
            response.setContentType("application/" + fileSufix);
            //默认Iso8859-1
            try {
                response.setHeader("Content-Disposition", "attachment; filename=" + java.net.URLEncoder.encode(fileName, "UTF-8"));
            } catch (UnsupportedEncodingException e) {
                e.printStackTrace();
            }
          /*  try {
                //已UTF-8的形式打散为字节数据在将其重构new String 再将编码指定为ISO8859-1
                response.addHeader("Content-Disposition", "attachment:filename=" + new String(fileName.getBytes("UTF-8"),"ISO8859-1"));
            } catch (UnsupportedEncodingException e) {
                e.printStackTrace();
            }*/
            //7.通过path写出
            try {
                Files.copy(path, response.getOutputStream());
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return "msg";
    }
}
```

# 拦截器

SpringMVC提供了拦截器,类似于过滤器,它将在我们的请求距离之间先做检查,有权决定,接下来是否继续,对我们的请求进行加工.

通过实现HandlerInterceptor接口

定义了三个重要方法

- 前置通知
- 后置通知
- 完成通知

## 案例一方法耗时拦截器

拦截器实现方法耗时统计与警告

 MethodTimerInterceptor .java

```java
package cn.edu.sict.interceptors;


import org.apache.log4j.Logger;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;


/**
 * 方法耗时统计拦截器
 */
public class MethodTimerInterceptor implements HandlerInterceptor {
    private static final Logger LOGGER = Logger.getLogger(MethodTimerInterceptor.class);

    //前置功能
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        //1.定义开始时间
        long start = System.currentTimeMillis();
        //2.将其存到请求域中
        request.setAttribute("start", start);
        //返回true,才会找下一个拦截器,如果没有下一个拦截器,则去Controller
        //记录请求日志
        LOGGER.info(request.getRequestURL() + ",请求到达...");
        return true;//false表示拦截，不向下执行；true表示放行
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        //1.取出start
        long start = (Long) request.getAttribute("start");
        //2.得到end
        long end = System.currentTimeMillis();
        //3.记录耗时
        long spendTime = end - start;


        if (spendTime >= 1000) {
            LOGGER.warn("方法耗时严重,耗时:" + spendTime + "毫秒,请及时处理!");
        } else {
            LOGGER.info("方法耗时" + spendTime + "毫秒,速度正常..");
        }

    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }
}
```

.xml

```xml 
<!--定义一个Interceptor，将拦截所有的请求 -->
<mvc:interceptors>
    <mvc:interceptor>
        <!--/*  只能拦截一层/name
           /** 拦截任意层任意方法-->
        <mvc:mapping path="/**/*"/>
        <!-- 定义在mvc:interceptor下面的表示是对特定的请求才进行拦截的 -->
        <bean class="cn.edu.sict.interceptors.MethodTimerInterceptor"/>
    </mvc:interceptor>
    <!--会话拦截
    只想拦截/user/**/*
    -->
    <mvc:interceptor>
        <mvc:mapping path="/user/**/"/>
        <!--排除登录的URL-->
        <mvc:exclude-mapping path="/user/login"/>
        <bean class="cn.edu.sict.interceptors.SessionInterceptor"></bean>
    </mvc:interceptor>
</mvc:interceptors>
```

## 案例二会话拦截器

SessionInterceptor.java

```java
package cn.edu.sict.interceptors;


import cn.edu.sict.pojo.User;
import org.apache.log4j.Logger;
import org.springframework.web.servlet.HandlerInterceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;


/**
 * 会话拦截器
 */
public class SessionInterceptor implements HandlerInterceptor {
    private static final Logger LOGGER = Logger.getLogger(SessionInterceptor.class);

    //检查当前会话是否有User
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        Object user = request.getSession().getAttribute("SESSION_USER");
        if (user == null) {
            LOGGER.warn("您不具备权限请先登录");
            return false;
        }
        if (user instanceof User) {//指出对象是否是特定类的一个实例
            //再去数据库查询检身份
            //身份正确密码置空 防止丢失
            User u = (User) user;
            u.setPassword(null);
            request.getSession().getAttribute("SESSION_USER");
            LOGGER.info(u.getUsername() + "处于登录状态，可以执行操作。。。");
            return true;
        } else {
            LOGGER.warn("请不要搞事情 ");
            return false;
        }
    }
}
```

XML

```XML
<!--定义一个Interceptor，将拦截所有的请求 -->
<mvc:interceptors>
    <mvc:interceptor>
        <!--/*  只能拦截一层/name
           /** 拦截任意层任意方法-->
        <mvc:mapping path="/**/*"/>
        <!-- 定义在mvc:interceptor下面的表示是对特定的请求才进行拦截的 -->
        <bean class="cn.edu.sict.interceptors.MethodTimerInterceptor"/>
    </mvc:interceptor>
    <!--会话拦截
    只想拦截/user/**/*
    -->
    <mvc:interceptor>
        <mvc:mapping path="/user/**/"/>
        <!--排除登录的URL-->
        <mvc:exclude-mapping path="/user/login"/>
        <bean class="cn.edu.sict.interceptors.SessionInterceptor"></bean>
    </mvc:interceptor>
</mvc:interceptors>
```

## 需要使用log4j打印

```properties
log4j.rootCategory=INFO, stdout

log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d{ABSOLUTE} %5p %t %c{2}:%L - %m%n

log4j.category.org.springframework.beans.factory=DEBUG
```

## 排除登录的URL

```xml
<mvc:exclude-mapping path="/user/login"/>
```



# 过滤器

## 拦截器与过滤器比较

### 相似点

都有优先处理请求的能力,可以决定是否将请求转移到实际处理的控制器处,都可以对请求或者会话中的数据进行加工.

### 不同

- 拦截器可以做  前置后置以及完成后的处理,控制更加精细,而过滤器只负责前面的过滤行为和后置过滤而已.
- 过滤器比拦截器优先执行
- 过滤器是servlet的规范组件
- 拦截器是额外的框架组件

### 拦截器过滤器和监听器
1、过滤器：

- 依赖于servlet容器；
- 在实现上基于函数回调，可以对几乎所有请求进行过滤；
- 缺点是一个过滤器实例只能在容器初始化时调用一次；
- 使用过滤器的目的是用来做一些过滤操作，获取我们想要获取的数据，比如：在过滤器中修改字符编码；在过滤器中修改HttpServletRequest的一些参数，包括：过滤低俗文字、危险字符等。

2、拦截器：

- 依赖于web框架，在SpringMVC中就是依赖于SpringMVC框架；
- 在实现上基于Java的反射机制，属于面向切面编程（AOP）的一种运用；
- 缺点是只能对controller请求进行拦截，对其他的一些比如直接访问静态资源的请求则没办法进行拦截处理；
- 由于拦截器是基于web框架的调用，因此可以使用Spring的依赖注入（DI）进行一些业务操作，同时一个拦截器实例在一个controller生命周期之内可以多次调用。
- 
3、监听器

- 实现了javax.servlet.ServletContextListener 接口的服务器端程序；
- 随web应用的启动而启动；
- 只初始化一次；
- 随web应用的停止而销毁；
- 主要作用是： 做一些初始化的内容添加工作、设置一些基本的内容、比如一些参数或者是一些固定的对象等等。如SpringMVC的监听器org.springframework.web.context.ContextLoaderListener，实现了SpringMVC容器的加载、Bean对象创建、DispatchServlet初始化等。




# SpringMVC 工作原理（重要）

**简单来说：**

客户端发送请求-> 前端控制器 DispatcherServlet 接受客户端请求 -> 找到处理器映射 HandlerMapping 解析请求对应的 Handler-> HandlerAdapter 会根据 Handler 来调用真正的处理器开处理请求，并处理相应的业务逻辑 -> 处理器返回一个模型视图 ModelAndView -> 视图解析器进行解析 -> 返回一个视图对象->前端控制器 DispatcherServlet 渲染数据（Moder）->将得到视图对象返回给用户



**如下图所示：**
![SpringMVC运行原理](http://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-10-11/49790288.jpg)

上图的一个笔误的小问题：Spring MVC 的入口函数也就是前端控制器 DispatcherServlet 的作用是接收请求，响应结果。


1. springmvc先将请求发送给DispatcherServlet。
2. DispatcherServlet查询一个或多个HandlerMapping，找到处理请求的Controller。
3. DispatcherServlet再把请求提交到对应的Controller。
4. Controller进行业务逻辑处理后，会返回一个ModelAndView。
5. Dispathcher查询一个或多个ViewResolver视图解析器，找到ModelAndView 对象指定的视图对象。
6. 视图对象负责渲染返回给客户端。

**流程说明（重要）：**

（1）客户端（浏览器）发送请求，直接请求到 DispatcherServlet。

（2）DispatcherServlet 根据请求信息调用 HandlerMapping，解析请求对应的 Handler。

（3）解析到对应的 Handler（也就是我们平常说的 Controller 控制器）后，开始由 HandlerAdapter 适配器处理。

（4）HandlerAdapter 会根据 Handler 来调用真正的处理器开处理请求，并处理相应的业务逻辑。

（5）处理器处理完业务后，会返回一个 ModelAndView 对象，Model 是返回的数据对象，View 是个逻辑上的 View。

（6）ViewResolver 会根据逻辑 View 查找实际的 View。

（7）DispaterServlet 把返回的 Model 传给 View（视图渲染）。

（8）把 View 返回给请求者（浏览器）



# SpringMVC 重要组件说明


**1、前端控制器DispatcherServlet（不需要工程师开发）,由框架提供（重要）**

作用：**Spring MVC 的入口函数。接收请求，响应结果，相当于转发器，中央处理器。有了 DispatcherServlet 减少了其它组件之间的耦合度。用户请求到达前端控制器，它就相当于mvc模式中的c，DispatcherServlet是整个流程控制的中心，由它调用其它组件处理用户的请求，DispatcherServlet的存在降低了组件之间的耦合性。**

**2、处理器映射器HandlerMapping(不需要工程师开发),由框架提供**

作用：根据请求的url查找Handler。HandlerMapping负责根据用户请求找到Handler即处理器（Controller），SpringMVC提供了不同的映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等。

**3、处理器适配器HandlerAdapter**

作用：按照特定规则（HandlerAdapter要求的规则）去执行Handler
通过HandlerAdapter对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。

**4、处理器Handler(需要工程师开发)**

注意：编写Handler时按照HandlerAdapter的要求去做，这样适配器才可以去正确执行Handler
Handler 是继DispatcherServlet前端控制器的后端控制器，在DispatcherServlet的控制下Handler对具体的用户请求进行处理。
由于Handler涉及到具体的用户业务请求，所以一般情况需要工程师根据业务需求开发Handler。

**5、视图解析器View resolver(不需要工程师开发),由框架提供**

作用：进行视图解析，根据逻辑视图名解析成真正的视图（view）
View Resolver负责将处理结果生成View视图，View Resolver首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成View视图对象，最后对View进行渲染将处理结果通过页面展示给用户。 springmvc框架提供了很多的View视图类型，包括：jstlView、freemarkerView、pdfView等。
一般情况下需要通过页面标签或页面模版技术将模型数据通过页面展示给用户，需要由工程师根据业务需求开发具体的页面。

**6、视图View(需要工程师开发)**

View是一个接口，实现类支持不同的View类型（jsp、freemarker、pdf...）

**注意：处理器Handler（也就是我们平常说的Controller控制器）以及视图层view都是需要我们自己手动开发的。其他的一些组件比如：前端控制器DispatcherServlet、处理器映射器HandlerMapping、处理器适配器HandlerAdapter等等都是框架提供给我们的，不需要自己手动开发。**

# DispatcherServlet 源码详细解析

首先看下源码：

```java
package org.springframework.web.servlet;
 
@SuppressWarnings("serial")
public class DispatcherServlet extends FrameworkServlet {
 
	public static final String MULTIPART_RESOLVER_BEAN_NAME = "multipartResolver";
	public static final String LOCALE_RESOLVER_BEAN_NAME = "localeResolver";
	public static final String THEME_RESOLVER_BEAN_NAME = "themeResolver";
	public static final String HANDLER_MAPPING_BEAN_NAME = "handlerMapping";
	public static final String HANDLER_ADAPTER_BEAN_NAME = "handlerAdapter";
	public static final String HANDLER_EXCEPTION_RESOLVER_BEAN_NAME = "handlerExceptionResolver";
	public static final String REQUEST_TO_VIEW_NAME_TRANSLATOR_BEAN_NAME = "viewNameTranslator";
	public static final String VIEW_RESOLVER_BEAN_NAME = "viewResolver";
	public static final String FLASH_MAP_MANAGER_BEAN_NAME = "flashMapManager";
	public static final String WEB_APPLICATION_CONTEXT_ATTRIBUTE = DispatcherServlet.class.getName() + ".CONTEXT";
	public static final String LOCALE_RESOLVER_ATTRIBUTE = DispatcherServlet.class.getName() + ".LOCALE_RESOLVER";
	public static final String THEME_RESOLVER_ATTRIBUTE = DispatcherServlet.class.getName() + ".THEME_RESOLVER";
	public static final String THEME_SOURCE_ATTRIBUTE = DispatcherServlet.class.getName() + ".THEME_SOURCE";
	public static final String INPUT_FLASH_MAP_ATTRIBUTE = DispatcherServlet.class.getName() + ".INPUT_FLASH_MAP";
	public static final String OUTPUT_FLASH_MAP_ATTRIBUTE = DispatcherServlet.class.getName() + ".OUTPUT_FLASH_MAP";
	public static final String FLASH_MAP_MANAGER_ATTRIBUTE = DispatcherServlet.class.getName() + ".FLASH_MAP_MANAGER";
	public static final String EXCEPTION_ATTRIBUTE = DispatcherServlet.class.getName() + ".EXCEPTION";
	public static final String PAGE_NOT_FOUND_LOG_CATEGORY = "org.springframework.web.servlet.PageNotFound";
	private static final String DEFAULT_STRATEGIES_PATH = "DispatcherServlet.properties";
	protected static final Log pageNotFoundLogger = LogFactory.getLog(PAGE_NOT_FOUND_LOG_CATEGORY);
	private static final Properties defaultStrategies;
	static {
		try {
			ClassPathResource resource = new ClassPathResource(DEFAULT_STRATEGIES_PATH, DispatcherServlet.class);
			defaultStrategies = PropertiesLoaderUtils.loadProperties(resource);
		}
		catch (IOException ex) {
			throw new IllegalStateException("Could not load 'DispatcherServlet.properties': " + ex.getMessage());
		}
	}
 
	/** Detect all HandlerMappings or just expect "handlerMapping" bean? */
	private boolean detectAllHandlerMappings = true;
 
	/** Detect all HandlerAdapters or just expect "handlerAdapter" bean? */
	private boolean detectAllHandlerAdapters = true;
 
	/** Detect all HandlerExceptionResolvers or just expect "handlerExceptionResolver" bean? */
	private boolean detectAllHandlerExceptionResolvers = true;
 
	/** Detect all ViewResolvers or just expect "viewResolver" bean? */
	private boolean detectAllViewResolvers = true;
 
	/** Throw a NoHandlerFoundException if no Handler was found to process this request? **/
	private boolean throwExceptionIfNoHandlerFound = false;
 
	/** Perform cleanup of request attributes after include request? */
	private boolean cleanupAfterInclude = true;
 
	/** MultipartResolver used by this servlet */
	private MultipartResolver multipartResolver;
 
	/** LocaleResolver used by this servlet */
	private LocaleResolver localeResolver;
 
	/** ThemeResolver used by this servlet */
	private ThemeResolver themeResolver;
 
	/** List of HandlerMappings used by this servlet */
	private List<HandlerMapping> handlerMappings;
 
	/** List of HandlerAdapters used by this servlet */
	private List<HandlerAdapter> handlerAdapters;
 
	/** List of HandlerExceptionResolvers used by this servlet */
	private List<HandlerExceptionResolver> handlerExceptionResolvers;
 
	/** RequestToViewNameTranslator used by this servlet */
	private RequestToViewNameTranslator viewNameTranslator;
 
	private FlashMapManager flashMapManager;
 
	/** List of ViewResolvers used by this servlet */
	private List<ViewResolver> viewResolvers;
 
	public DispatcherServlet() {
		super();
	}
 
	public DispatcherServlet(WebApplicationContext webApplicationContext) {
		super(webApplicationContext);
	}
	@Override
	protected void onRefresh(ApplicationContext context) {
		initStrategies(context);
	}
 
	protected void initStrategies(ApplicationContext context) {
		initMultipartResolver(context);
		initLocaleResolver(context);
		initThemeResolver(context);
		initHandlerMappings(context);
		initHandlerAdapters(context);
		initHandlerExceptionResolvers(context);
		initRequestToViewNameTranslator(context);
		initViewResolvers(context);
		initFlashMapManager(context);
	}
}

```

DispatcherServlet类中的属性beans：

- HandlerMapping：用于handlers映射请求和一系列的对于拦截器的前处理和后处理，大部分用@Controller注解。
- HandlerAdapter：帮助DispatcherServlet处理映射请求处理程序的适配器，而不用考虑实际调用的是 哪个处理程序。- - - 
- ViewResolver：根据实际配置解析实际的View类型。
- ThemeResolver：解决Web应用程序可以使用的主题，例如提供个性化布局。
- MultipartResolver：解析多部分请求，以支持从HTML表单上传文件。- 
- FlashMapManager：存储并检索可用于将一个请求属性传递到另一个请求的input和output的FlashMap，通常用于重定向。

在Web MVC框架中，每个DispatcherServlet都拥自己的WebApplicationContext，它继承了ApplicationContext。WebApplicationContext包含了其上下文和Servlet实例之间共享的所有的基础框架beans。

**HandlerMapping**

![HandlerMapping](http://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-10-11/96666164.jpg)

HandlerMapping接口处理请求的映射HandlerMapping接口的实现类：

- SimpleUrlHandlerMapping类通过配置文件把URL映射到Controller类。
- DefaultAnnotationHandlerMapping类通过注解把URL映射到Controller类。

**HandlerAdapter**


![HandlerAdapter](http://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-10-11/91433100.jpg)

HandlerAdapter接口-处理请求映射

AnnotationMethodHandlerAdapter：通过注解，把请求URL映射到Controller类的方法上。

**HandlerExceptionResolver**


![HandlerExceptionResolver](http://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-10-11/50343885.jpg)

HandlerExceptionResolver接口-异常处理接口

- SimpleMappingExceptionResolver通过配置文件进行异常处理。
- AnnotationMethodHandlerExceptionResolver：通过注解进行异常处理。

**ViewResolver**

![ViewResolver](http://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-10-11/49497279.jpg)

ViewResolver接口解析View视图。

UrlBasedViewResolver类 通过配置文件，把一个视图名交给到一个View来处理。
