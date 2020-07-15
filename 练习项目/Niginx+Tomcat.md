[toc]

# 高并发目标
![](https://raw.githubusercontent.com/binbinbin5/myPics/master/file/redis.png)



## 优点
- 提高服务性能，并发能力，高可用性（一台挂掉后不会影响）
- 提供项目加工的横向扩展能力（天猫双十一，只要加tomcat节点就行了，根据实际情况多加节点，进行热部署）

一般一台机器部署一个tomcat，一台机器部署一个有共享瓶颈：磁盘IO，cpu等。

## 实现原理
通过Niginx负载均衡对多个tomcat进行请求转发。

## 集群问题
- Session登录信息存储以及读取的问题（1. Nginx IP hash policy,直接实现横向扩展（负载不平衡用得少，IP变化环境下无法服务） 2.增加一个分布式redis Session server）

![](https://raw.githubusercontent.com/binbinbin5/myPics/master/file/7ZQIR4978U%7B%5D5WYO1DXVM3N.jpg)

- 服务器定时任务并发的问题
- ...


## 部署应用
### 单机部署多应用 MAC
1. 修改/etc/profilr配置文件 增加tomcat环境变量
```
export CATALINA_BASE= 第一个路径
export CATALINA_HOME=
export TOMCAT_HOME=

export CATALINA_2_BASE= 第二个tomcat路径
export CATALINA_2_HOME=
export TOMCAT_2_HOME=

...
```
2. 保存退出wq
3. 执行配置文件source /etc/profile
4. 打开第二个tomcat目录下bin的catalina.sh
5. 找到# OS specific support. $var _must_ be set to either true or false 下面新增

```
# OS specific support. $var _must_ be set to either true or false 
export  CATALINA_BASE=$CATALINA_2_BASE
export  CATALINA_HOME=$CATALINA_2_HOME
```

6. 打开第二个tomcat的conf目录下的server.xml修改三个端口

```
server port = 9005 (统一递增1000，第一个为8005的话，第三个10005,也可以其他但是不重复)

Connector port = 9005 (访问端口 ，第一个8005)

Connector port = 9009 protocal = "AJP/1,3"(第一个8005)
```
7. 分别进入两个tomcat的bin目录，启动tomcat，执行startup.sh
8. 检查日志，命令行输出 Using CATALINA_HOME....
9. 访问localhost:8080，9080，都可以打开网页

>windos的话，新增环境变量和上面1一样，继续修改第二个Tomcat下 catlina.bat和startup.bat，打开两个文件，替换CATALINA_BASE编程CATALINA_2_BASE,CATALINA_HOME变成CATALINA_2_HOME,继续打开第二个conf目录下的server.xml，和上面6一样的修改8090，9005..。最后执行startup.bat并且检查日志如上7/8/9.

### 多机部署多应用
1. 每个机器安装一个tomcat
2. 保证每个机器网互通，Nginx装在任意一台服务器，也可以单独nginx独立出来，保证连接各个服务器。



## Niginx负载均衡配置、策略、场景
1. 轮询：优点：实现简单，缺点：不考虑每台服务器处理能力
1. 权重：优点：考虑了服务器处理能力的不同
1. 地址散列：优点：能实现同一个用户访问同一个服务器
1. 最少连接：优点：使集群中各个服务器负载更加均匀
1. 加权最少连接：在最少连接的基础上，为每台服务器加上权值。算法为(活动连接数*256+非活动连接数)/权重，计算出来的值小的服务器优先被选择。



- 轮询(默认)：实现简单，但是不考虑每台服务器的处理能力

```
upstream xx.com{
    server xx.com:8080
    server xx.com:8090
}
//配置host或者是否拥有域名的情况
```
- 权重：一台服务器功能好，一台差，希望更多请求到功能好的里面，就需要配置权重。考虑了每台服务器处理能力不同，

```
upstream xx.com{
    server xx.com:8080 weight = 15;
    server xx.com:8090 weight = 10;
}
//权重15和10代表访问的分配概率，8080是8090的1.5倍，而不是绝对的25次，15次在8080

//如果采用权重配置负载均衡未设置，默认为1
```

- IP hash：IP地址进行分配，同一个用户访问同一个服务器，但缺点是IP hash不一定平均

```
upstream xx.com{
    ip_hash;
    server xx.com:8080 weight = 15;
    server xx.com:8090 weight = 10;
}
```

- url hash ：第三方插件，安装Nginx插件,实现同一个服务访问同一个服务器（第一个开大的url和第二个打开的url访问相同服务器），缺点是分配请求不平均，请求频繁的url到同一个服务器。

```
upstream xx.com{
   
    server xx.com:8080 weight = 15;
    server xx.com:8090 weight = 10;
    hash $request_uri
}
```


- fair：第三方插件,按后端服务器的响应时间来分配请求。


```
upstream backserver{
    ip_hash;
    server 127.0.0.1:9090 down;//down表示当前server不参与负载
    server 127.0.0.1:8080 weight=2；//权重越大负载越高
    server 127.0.0.1:6060 //默认权重1
    server 127.0.0.1:7070 backup;//其他非backup机器down或者忙的时候，请求backup机器
}
```

https://blog.csdn.net/itcats_cn/article/details/82454657  参考介绍


## tomcat+nginx 集群
TOMCAT:
1. 单机多部署，修改本机host，etc/host  增加 127.0.0.1 xxx.com
2. 验证host，终端 ping xxx.com ，ip为127.0.0.1，如果浏览器没有转移，关闭清除浏览器缓存
3. localhost：8080,9080 打开一样

NIginx:
1. 启动NGINX,默认为80端口
2. 访问localhost或者xxx.com一样
3. 配置Nginx，conf/conf.com配置文件，增加include vhast/*.conf（增加Tomcat负载均衡配置，并且把域名的配置分开，方便管理）
4. /conf下创建vhost文件夹，增加xxx.com.conf配置文件 
```
upstream xxx.com{
    server xxx.com :8080; weight=1;
    # server xxx.com:8080
    //....
}

server{
    listen:80;
    autoindex on;
    server_name xxx.com
    access_log (Nginx下logs/access.log) combined;
    index index.html index.htm index.jsp index.php;
    
    location/{
        proxy_pass https://xxx.com
        add_header Access-Control-Allow-Origin *;
    }
}
```
5. 重新加载Nginx配置
6. 验证：xxx.com，多次刷新，观察是否在同一个页面


### tomcat挂了怎么办
首先我们nginx会访问tomcat，如果访问对应的tomcat挂掉，则会采用服务器宕机轮询配置规则(上面提到了)，即一定时间内，tomcat无响应则自动轮询下一台服务器，其次，还要使用keepalived重启宕机的tomcat，只需要修改tomcat重启脚本即可。

服务器宕机轮询配置规则

```
server {
        listen       80;
        server_name  www.nginx.cn;
        location / {
	            proxy_pass  http://backserver;
	            index  index.html index.htm;
 
 
                 proxy_connect_timeout 1;
                    proxy_send_timeout 1;
                    proxy_read_timeout 1;
 
 
        }
		
		}
```

加入三行代码，在访问某服务器1秒内没有响应的话，自动轮询下一台服务器。

实现tomcat服务器高可用：
- 1、当有tomcat节点挂掉时候，使用keepalived脚本重启服务器     
- 2、配置nginx服务器宕机轮询规则，超时未访问自动轮询下一台服务器    
- 3、若多次重启不成功，则通过邮件通知

### 如果在项目发布新版本时候，怎么访问服务器？

假设有主备两台服务器，主机发布版本期间无法访问主机，那么通过配置了nginx服务器宕机容错机制，nginx则会转发给备机，当主机发布版本完成后，再更新备机项目。

### tomcat发布版本的时候，session失效怎么解决?

由于session存放在jvm内存中，重启session失效，解决办法：把session存放在redis中。


### Tomcat OOM问题

两个解决方案：

（1）添加参数：永久保存区域溢出

-XX:MaxPermSize=256m 设定最大内存的永久保存区域
然后尝试重新启动


（2）修改参数：堆溢出

```
-Xms：初始堆大小
-Xmx：最大堆大小
```

-Xms1024m -Xmx1024m JVM的初始大小，最小和最大内存
修改后尝试启动。

### tomcat请求流程
https://www.jianshu.com/p/6e2b744074bb

请求发送到Tomcat之后，首先经过Service然后会交给我们的Connector，Connector用于接收请求并将接收的请求封装为Request和Response来具体处理，Request和Response封装完之后再交由Container进行处理，Container处理完请求之后再返回给Connector，最后在由Connector通过Socket将处理的结果返回给客户端，这样整个请求的就处理完了！

Connector最底层使用的是Socket来进行连接的，Request和Response是按照HTTP协议来封装的，所以Connector同时需要实现TCP/IP协议和HTTP协议！

具体：
-  1) 请求被发送到本机端口8080，被在那里侦听的Coyote HTTP/1.1 Connector获得
-                 2) Connector把该请求交给它所在的Service的Engine来处理，并等待来自Engine的回应 
-                 3) Engine获得请求localhost/项目/页面.jsp，匹配它所拥有的所有虚拟主机Host 
-                 4) Engine匹配到名为localhost的Host（即使匹配不到也把请求交给该Host处理，因为该Host被定义为该Engine的默认主机） 
-                 5) localhost Host获得请求/项目/页面.jsp，匹配它所拥有的所有Context 
-                 6) Host匹配到路径为/项目的Context（如果匹配不到就把该请求交给路径名为””的Context去处理） 
-                 7) path=”/项目”的Context获得请求/页面.jsp，在它的mapping table中寻找对应的servlet 
-                 8) Context匹配到URL PATTERN为*.jsp的servlet，对应于JspServlet类 
-                 9) 构造HttpServletRequest对象和HttpServletResponse对象，作为参数调用JspServlet的doGet或doPost方法 
-                 10)Context把执行完了之后的HttpServletResponse对象返回给Host 
-                 11)Host把HttpServletResponse对象返回给Engine 
-                 12)Engine把HttpServletResponse对象返回给Connector 
-                 13)Connector把HttpServletResponse对象返回给客户browser 
