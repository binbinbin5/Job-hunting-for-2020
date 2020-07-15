[toc]

# Socket
Socket编程进行的是端到端的通信，往往意识不到中间经过多少局域网，多少路由器，因而能够设置的参数，也只能是端到端协议之上网络层和传输层的。

我们平时说的最多的socket是什么呢，==实际上socket是对TCP/IP协议的封装，Socket本身并不是协议，而是一个调用接口（API），通过Socket，我们才能使用TCP/IP协议==。 实际上，Socket跟TCP/IP协议没有必然的联系。Socket编程接口在设计的时候，就希望也能适应其他的网络协议。所以说，Socket的出现 只是使得程序员更方便地使用TCP/IP协议栈而已，是对TCP/IP协议的抽象，从而形成了我们知道的一些最基本的函数接口，比如create、 listen、connect、accept、send、read和write等等。网络有一段关于socket和TCP/IP协议关系的说法比较容易理解：

“TCP/IP只是一个协议栈，就像操作系统的运行机制一样，必须要具体实现，同时还要提供对外的操作接口。这个就像操作系统会提供标准的编程接口，比如win32编程接口一样，TCP/IP也要提供可供程序员做网络开发所用的接口，这就是Socket编程接口。” 

实际上，传输层的TCP是基于网络层的IP协议的，而应用层的HTTP协议又是基于传输层的TCP协议的，而Socket本身不算是协议，就像上面所说，它只是提供了一个针对TCP或者UDP编程的接口。socket是对端口通信开发的工具,它要更底层一些.

在网络层，==Socket函数需要指定到底是IPv4还是IPv6==，分别对应设置为AF_INET和AF_INET6。另外，还
==要指定到底是TCP还是UDP==。还记得咱们前面讲过的，TCP协议是基于数据流的，所以设置
为SOCK_STREAM，而UDP是基于数据报的，因而设置为SOCK_DGRAM。


## 基于TCP协议的Socket程序函数调用过程
两端创建了Socket之后，TCP的服务端要先监听一个端口，一般是先调用bind函数，给这个Socket赋予一个IP地址和端口。为什么需要端口呢？要知道，你写的是一个应用程序，当一个网络包来的时候，内核要通过TCP头里面的这个端口，来找到你这个应用程序，把包给你。为什么要IP地址呢？有时候，一台机器会有多个网卡，也就会有多个IP地址，你可以选择监听所有的网卡，也可以选择监听一个网卡，这样，只有发给这个网卡的包，才会给你。

当服务端有了IP和端口号，就可以调用listen函数进行监听。在TCP的状态图里面，有一个listen状态，
当调用这个函数之后，服务端就进入了这个状态，这个时候客户端就可以发起连接了

在内核中，为每个Socket维护两个队列。==一个是已经建立了连接的队列，这时候连接三次握手已经完
毕，处于established状态；一个是还没有完全建立连接的队列，这个时候三次握手还没完成，处
于syn_rcvd的状态。==

接下来，服务端调用accept函数，拿出一个已经完成的连接进行处理。如果还没有完成，就要等着。

在服务端等待的时候，==客户端可以通过connect函数发起连接。先在参数中指明要连接的IP地址和端口
号，然后开始发起三次握手。内核会给客户端分配一个临时的端口。一旦握手成功，服务端的accept就
会返回另一个Socket。==

这是一个经常考的知识点，就是监听的Socket和真正用来传数据的Socket是两个，一个叫作监
听Socket，一个叫作已连接Socket.连接建立成功之后，双方开始通过read和write函数来读写数据，就像往一个文件流里面写东西一样.

![](https://raw.githubusercontent.com/binbinbin5/myPics/master/imgs20190601231458.png)

简单来讲：服务器端，首先是服务器初始化Socket，然后是与端口进行绑定(blind()),端口创建ServerSocket进行监听(listen()),然后调用阻塞(accept()),等待客户端连接。与客户端发生连接后，会进行相关的读写操作(read(),write())，最后调用close()关闭连接。

TCP的Socket就是一个文件流，是非常准确的。因为，Socket在Linux中就是以文件的形式存在的。除
此之外，还存在文件描述符。写入和读出，也是通过文件描述符。

在内核中，Socket是一个文件，那对应就有文件描述符。每一个进程都有一个数据结构task_struct，里
面指向一个文件描述符数组，来列出这个进程打开的所有文件的文件描述符。文件描述符是一个整数，
是这个数组的下标。

这个数组中的内容是一个指针，指向内核中所有打开的文件的列表。既然是一个文件，就会有一
个inode，只不过Socket对应的inode不像真正的文件系统一样，保存在硬盘上的，而是在内存中的。在
这个inode中，指向了Socket在内核中的Socket结构。

在这个结构里面，主要的是两个队列，一个是发送队列，一个是接收队列。在这两个队列里面保存的是
一个缓存sk_buff。这个缓存里面能够看到完整的包的结构.

![](https://raw.githubusercontent.com/binbinbin5/myPics/master/imgs20190601231602.png)

### TCP SOCKET代码
1. TCP协议是面向连接的、可靠的、有序的、以字节流的方式发送数据，通过三次握手方式建立连接，形成传输数据的通道，在连接中进行大量数据的传输，效率会稍低
2. Java中基于TCP协议实现网络通信的类
     - 客户端的Socket类
     - 服务器端的ServerSocket类
3. Socket通信的步骤
    - ① 创建ServerSocket和Socket
    - ② 打开连接到Socket的输入/输出流
    - ③ 按照协议对Socket进行读/写操作
    - ④ 关闭输入输出流、关闭Socket

4. 服务器端：
    - ① 创建ServerSocket对象，绑定监听端口
    - ② 通过accept()方法监听客户端请求
    - ③ 连接建立后，通过输入流读取客户端发送的请求信息
    - ④ 通过输出流向客户端发送响应信息
    - ⑤ 关闭相关资源
    

```
 /**
   * 基于TCP协议的Socket通信，实现用户登录，服务端
 */
  //1、创建一个服务器端Socket，即ServerSocket，指定绑定的端口，并监听此端口
 ServerSocket serverSocket =new ServerSocket(10086);//1024-65535的某个端口
  //2、调用accept()方法开始监听，等待客户端的连接
  Socket socket = serverSocket.accept();
  //3、获取输入流，并读取客户端信息
  InputStream is = socket.getInputStream();
 InputStreamReader isr =newInputStreamReader(is);
 BufferedReader br =newBufferedReader(isr);
 String info =null;
 while((info=br.readLine())!=null){
     System.out.println("我是服务器，客户端说："+info)；
 }
 socket.shutdownInput();//关闭输入流
 //4、获取输出流，响应客户端的请求
 OutputStream os = socket.getOutputStream();
 PrintWriter pw = new PrintWriter(os);
 pw.write("欢迎您！");
 pw.flush();

 
 //5、关闭资源
 pw.close();
 os.close();
 br.close();
 isr.close();
 is.close();
 socket.close();
 serverSocket.close();
```
5. 客户端：
    - ① 创建Socket对象，指明需要连接的服务器的地址和端口号
    - ② 连接建立后，通过输出流向服务器端发送请求信息
    - ③ 通过输入流获取服务器响应的信息
    - ④ 关闭响应资源 

```
//客户端
//1、创建客户端Socket，指定服务器地址和端口
Socket socket =newSocket("localhost",10086);
//2、获取输出流，向服务器端发送信息
OutputStream os = socket.getOutputStream();//字节输出流
PrintWriter pw =newPrintWriter(os);//将输出流包装成打印流
pw.write("用户名：admin；密码：123");
pw.flush();
socket.shutdownOutput();
//3、获取输入流，并读取服务器端的响应信息
InputStream is = socket.getInputStream();
BufferedReader br = new BufferedReader(new InputStreamReader(is));
String info = null;
while((info=br.readLine())!null){
System.out.println("我是客户端，服务器说："+info);
}

//4、关闭资源
br.close();
is.close();
pw.close();
os.close();
socket.close();
```



6. 应用多线程实现服务器与多客户端之间的通信
    - ① 服务器端创建ServerSocket，循环调用accept()等待客户端连接
    - ② 客户端创建一个socket并请求和服务器端连接
    - ③ 服务器端接受客户端请求，创建socket与该客户建立专线连接
    - ④ 建立连接的两个socket在一个单独的线程上对话
    - ⑤ 服务器端继续等待新的连接 

```java
//服务端代码：
public class multithreadingServer {
  public static void main(String[] args) {
	  try {
		ServerSocket serversocket=new ServerSocket(8888);
		Socket socket=null;
		int n=0;
		while(true){
			socket=serversocket.accept();//再来一个客户端就新建一个socket
			ServerThread serverthread=new ServerThread(socket);
			serverthread.start();
			n++;
			System.out.println("已有"+n+"台客户端连接");
			InetAddress address=socket.getInetAddress();//获取客户端的inetaddress对象
			    System.out.println("当前主机ip:"+address.getHostAddress());//获取客户端的ip
			}
		} catch (IOException e) {
			e.printStackTrace();
		}
    }
}

//服务器线程处理类：
public class ServerThread extends Thread{
     Socket socket=null;
     InputStream is=null;
     BufferedReader br=null;
     OutputStream os=null;
     PrintWriter pw=null;
      
     public ServerThread(Socket socket){
    	 this.socket=socket;
     }
    
    public void run(){
	  try {
		    is=socket.getInputStream();//获得字节流
		    br=new BufferedReader(new InputStreamReader(is));//字节流转化为字符流并添加缓冲
		    String info=null;
		    while((info=br.readLine())!=null){
			   System.out.println("我是服务端,客户端说:"+info);
			}
			socket.shutdownInput();//必须要及时关闭，因为readline这个方法是以\r\n作为界定符的，由于发送消息的那一端用的是PrintWriter的write()方法，这个方法并没加上\r\n,所以会一直等待
			//回复客户端
		   os=socket.getOutputStream();
		   pw=new PrintWriter(os);
		   pw.write("欢迎您!");
		   pw.flush();
			
    	} catch (IOException e) {
    		e.printStackTrace();
    	}finally{
    		if (pw!=null)
    		pw.close();
    		try {
    			if (os!=null)
    			os.close();
    			if (br!=null)
    			br.close();
    			if (is!=null)
    			is.close();//关闭返回的 InputStream 将关闭关联套接字。 
    			socket.close();
    		} catch (IOException e) {
    			e.printStackTrace();
    		}
	    }
    }
}


//客户端代码：       
public class client {
	public static void main(String[] args) {
		try {
			Socket socket=new Socket("127.0.0.1",8888);//创建客户端socket,因为是本地,ip地址就是127.0.0.1
			OutputStream os=socket.getOutputStream();
			PrintWriter pw=new PrintWriter(os);
			pw.write("用户名:admin;密码:123");
			pw.flush();
			socket.shutdownOutput();//必须加上这句话，表示输出流的结束，注意此时不能关闭os,因为关闭os会关闭绑定的socket

			//在客户端获取回应信息
			InputStream is=socket.getInputStream();
			BufferedReader br=new BufferedReader(new InputStreamReader(is));
			String info=null;
			while((info=br.readLine())!=null){
				System.out.println("我是客户端,服务端说:"+info);
			}
			br.close();
			is.close();
			pw.close();
			os.close();
			socket.close();
		} catch (UnknownHostException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}
	} 
}
```


![](https://raw.githubusercontent.com/binbinbin5/myPics/master/imgs/socket%E9%80%9A%E4%BF%A1.jpeg)


## 基于UDP协议的Socket程序函数调用过程

对于UDP来讲，过程有些不一样。UDP是没有连接的，所以不需要三次握手，也就不需要调
用listen和connect，但是，UDP的的交互仍然需要IP和端口号，因而也需要bind。UDP是没有维护连接状态的，因而不需要每对连接建立一组Socket，而是只要有一个Socket，就能够和多个客户端通信。也
正是因为没有连接状态，每次通信的时候，都调用sendto和recvfrom，都可以传入IP地址和端口。

![](https://raw.githubusercontent.com/binbinbin5/myPics/master/imgs20190601231641.png)

UDP协议（用户数据报协议）是无连接的、不可靠的、无序的,速度快
- 进行数据传输时，首先将要传输的数据定义成数据报（Datagram），大小限制在64k，在数据报中指明数据所要达到的Socket（主机地址和端口号），然后再将数据报发送出去
- DatagramPacket类:表示数据报包
- DatagramSocket类：进行端到端通信的类
       
1. 服务器端实现步骤
   - ① 创建DatagramSocket，指定端口号
   - ② 创建DatagramPacket
   - ③ 接受客户端发送的数据信息
   - ④ 读取数据


```
//服务器端，实现基于UDP的用户登录
 //1、创建服务器端DatagramSocket，指定端口
  DatagramSocket socket =new datagramSocket(10010);
  //2、创建数据报，用于接受客户端发送的数据
  byte[] data =new byte[1024];//
 DatagramPacket packet =new DatagramPacket(data,data.length);
  //3、接受客户端发送的数据
 socket.receive(packet);//此方法在接受数据报之前会一致阻塞
  //4、读取数据
 String info =new String(data,0,data.length);
 System.out.println("我是服务器，客户端告诉我"+info);

 //=========================================================
 //向客户端响应数据
 //1、定义客户端的地址、端口号、数据
 InetAddress address = packet.getAddress();
 int port = packet.getPort();
 byte[] data2 = "欢迎您！".geyBytes();
 //2、创建数据报，包含响应的数据信息
 DatagramPacket packet2 = new DatagramPacket(data2,data2.length,address,port);
 //3、响应客户端
socket.send(packet2);
 //4、关闭资源
 socket.close();
```



2. 客户端实现步骤
    - ① 定义发送信息
    - ② 创建DatagramPacket，包含将要发送的信息
    - ③ 创建DatagramSocket
    - ④ 发送数据

```
//客户端
//1、定义服务器的地址、端口号、数据
InetAddress address =InetAddress.getByName("localhost");
int port =10010;
byte[] data ="用户名：admin;密码：123".getBytes();
//2、创建数据报，包含发送的数据信息
DatagramPacket packet = new DatagramPacket(data,data.length,address,port);
//3、创建DatagramSocket对象
DatagramSocket socket =new DatagramSocket();
//4、向服务器发送数据
socket.send(packet);


//接受服务器端响应数据
//======================================
//1、创建数据报，用于接受服务器端响应数据
byte[] data2 = new byte[1024];
DatagramPacket packet2 = new DatagramPacket(data2,data2.length);
//2、接受服务器响应的数据
socket.receive(packet2);
String raply = new String(data2,0,packet2.getLenth());
System.out.println("我是客户端，服务器说："+reply);
//4、关闭资源
socket.close();
```
## 注意点
注意问题
- 1、多线程的优先级问题： 根据实际的经验，适当的降低优先级，否则可能会有程序运行效率低的情况
- 2、是否关闭输出流和输入流：
对于同一个socket，如果关闭了输出流，则与该输出流关联的socket也会被关闭，所以一般不用关闭流，直接关闭socket即可
- 3、使用TCP通信传输对象，IO中序列化部分
- 4、socket编程传递文件，IO流部分

## 服务器如何接更多的项目？

会了这几个基本的Socket函数之后，你就可以轻松地写一个网络交互的程序了。就像上面的过程一样，
在建立连接后，进行一个while循环。客户端发了收，服务端收了发。

当然这只是万里长征的第一步，因为如果使用这种方法，基本上只能一对一沟通。如果你是一个服务
器，同时只能服务一个客户，肯定是不行的。这就相当于老板成立一个公司，只有自己一个人，自己亲
自上来服务客户，只能干完了一家再干下一家，这样赚不来多少钱。

那作为老板你就要想了，我最多能接多少项目呢？当然是越多越好。

我们先来算一下理论值，也就是最大连接数，系统会用一个四元组来标识一个TCP连接。

```
{本机IP, 本机端口, 对端IP, 对端端口}
```

服务器通常固定在某个本地端口上监听，等待客户端的连接请求。因此，服务端端TCP连接四元组中只
有对端IP, 也就是客户端的IP和对端的端口，也即客户端的端口是可变的，因此，++最大TCP连接数=客户
端IP数×客户端端口数++。对IPv4，客户端的IP数最多为2的32次方，客户端的端口数最多为2的16次方，
也就是服务端单机最大TCP连接数，约为2的48次方。

当然，服务端最大并发TCP连接数远不能达到理论上限。首先主要是文件描述符限制，按照上面的原
理，Socket都是文件，所以首先要通过ulimit配置文件描述符的数目；另一个限制是内存，按上面的数
据结构，每个TCP连接都要占用一定内存，操作系统是有限的。

所以，作为老板，在资源有限的情况下，要想接更多的项目，就需要降低每个项目消耗的资源数目.

### 方式一：将项目外包给其他公司（多进程方式）
这就相当于你是一个代理，在那里监听来的请求。一旦建立了一个连接，就会有一个已连接Socket，这
时候你可以创建一个子进程，然后将基于已连接Socket的交互交给这个新的子进程来做。就像来了一个
新的项目，但是项目不一定是你自己做，可以再注册一家子公司，招点人，然后把项目转包给这家子公
司做，以后对接就交给这家子公司了，你又可以去接新的项目了。

这里有一个问题是，如何创建子公司，并如何将项目移交给子公司呢？

在Linux下，创建子进程使用fork函数。通过名字可以看出，这是在父进程的基础上完全拷贝一个子进
程。在Linux内核中，会复制文件描述符的列表，也会复制内存空间，还会复制一条记录当前执行到了
哪一行程序的进程。显然，复制的时候在调用fork，复制完毕之后，父进程和子进程都会记录当前刚刚
执行完fork。这两个进程刚复制完的时候，几乎一模一样，只是根据fork的返回值来区分到底是父进
程，还是子进程。如果返回值是0，则是子进程；如果返回值是其他的整数，就是父进程。

![](https://raw.githubusercontent.com/binbinbin5/myPics/master/imgs20190601231833.png)

因为复制了文件描述符列表，而文件描述符都是指向整个内核统一的打开文件列表的，因而父进程刚才因为accept创建的已连接Socket也是一个文件描述符，同样也会被子进程获得。

接下来，子进程就可以通过这个已连接Socket和客户端进行互通了，当通信完毕之后，就可以退出进
程，那父进程如何知道子进程干完了项目，要退出呢？还记得fork返回的时候，如果是整数就是父进程吗？这个整数就是子进程的ID，父进程可以通过这个ID查看子进程是否完成项目，是否需要退出

### 方式二：将项目转包给独立的项目组（多线程方式）

上面这种方式你应该也能发现问题，如果每次接一个项目，都申请一个新公司，然后干完了，就注销掉
这个公司，实在是太麻烦了。毕竟一个新公司要有新公司的资产，有新的办公家具，每次都买了再卖，
不划算。

于是你应该想到了，我们可以使用线程。相比于进程来讲，这样要轻量级的多。如果创建进程相当于成
立新公司，购买新办公家具，而创建线程，就相当于在同一个公司成立项目组。一个项目做完了，那这
个项目组就可以解散，组成另外的项目组，办公家具可以共用。

在Linux下，通过pthread_create创建一个线程，也是调用do_fork。不同的是，虽然新的线程在task列
表会新创建一项，但是很多资源，例如文件描述符列表、进程空间，还是共享的，只不过多了一个引用而已。

![](https://raw.githubusercontent.com/binbinbin5/myPics/master/imgs20190601231924.png)

新的线程也可以通过已连接Socket处理请求，从而达到并发处理的目的。

上面基于进程或者线程模型的，其实还是有问题的。新到来一个TCP连接，就需要分配一个进程或者线
程。一台机器无法创建很多进程或者线程。有个C10K，它的意思是一台机器要维护1万个连接，就要创
建1万个进程或者线程，那么操作系统是无法承受的。如果维持1亿用户在线需要10万台服务器，成本也
太高了。

其实C10K问题就是，你接项目接的太多了，如果每个项目都成立单独的项目组，就要招聘10万人，你肯
定养不起，那怎么办呢？

### 方式三：一个项目组支撑多个项目（IO多路复用，一个线程维护多个Socket）

当然，一个项目组可以看多个项目了。这个时候，每个项目组都应该有个项目进度墙，将自己组看的项
目列在那里，然后每天通过项目墙看每个项目的进度，一旦某个项目有了进展，就派人去盯一下。

由于Socket是文件描述符，因而某个线程盯的所有的Socket，都放在一个文件描述符集合f_set中，这
就是项目进度墙，然后调用select函数来监听文件描述符集合是否有变化。一旦有变化，就会依次查看
每个文件描述符。那些发生变化的文件描述符在fd_set对应的位都设为1，表示Socket可读或者可写，从
而可以进行读写操作然后再调用select接着盯着下一轮的变化。。

### 一个项目组支撑多个项目（IO多路复用，从“派人盯着”到“有事通知”）

上面select函数还是有问题的，因为每次Socket所在的文件描述符集合中有Socket发生变化的时候，都需要通过轮询的方式，也就是需要将全部项目都过一遍的方式来查看进度，这大大影响了一个项目组能够支撑的最大的项目数量。因而使用select能够同时盯的项目数量由FDSETSIZE限制.

如果改成事件通知的方式，情况就会好很多，项目组不需要通过轮询挨个盯着这些项目，而是当项目进
度发生变化的时候，主动通知项目组，然后项目组再根据项目进展情况做相应的操作。

能完成这件事情的函数叫epoll，它在内核中的实现不是通过轮询的方式，而是通过注册callback函数的
方式，当某个文件描述符发送变化的时候，就会主动通知。

![](https://raw.githubusercontent.com/binbinbin5/myPics/master/imgs20190601232105.png)

如图所示，假设进程打开了Socket m, n, x等多个文件描述符，现在需要通过epoll来监听是否这
些Socket都有事件发生。其中epoll_create创建一个epoll对象，也是一个文件，也对应一个文件描述
符，同样也对应着打开文件列表中的一项。在这项里面有一个红黑树，在红黑树里，要保存这
个epoll要监听的所有Socket。

当epoll_ctl添加一个Socket的时候，其实是加入这个红黑树，同时红黑树里面的节点指向一个结构，将
这个结构挂在被监听的Socket的事件列表中。当一个Socket来了一个事件的时候，可以从这个列表中得
到epoll对象，并调用call back通知它。

这种通知方式使得监听的Socket数据增加的时候，效率不会大幅度降低，能够同时监听的Socket的数目
也非常的多了。上限就为系统定义的、进程打开的最大文件描述符个数。因而，epoll被称为解
决C10K问题的利器。



# I/O 模型

一个输入操作通常包括两个阶段：

- 等待数据准备好
- 从内核向进程复制数据

对于一个套接字上的输入操作，第一步通常涉及等待数据从网络中到达。当所等待数据到达时，它被复制到内核中的某个缓冲区。第二步就是把数据从内核缓冲区复制到应用进程缓冲区。

Unix 有五种 I/O 模型：

- 阻塞式 I/O
- 非阻塞式 I/O
- I/O 复用（select 和 poll）
- 信号驱动式 I/O（SIGIO）
- 异步 I/O（AIO）

## 阻塞式 I/O

应用进程被阻塞，直到数据从内核缓冲区复制到应用进程缓冲区中才返回。

应该注意到，在阻塞的过程中，其它应用进程还可以执行，因此阻塞不意味着整个操作系统都被阻塞。因为其它应用进程还可以执行，所以不消耗 CPU 时间，这种模型的 CPU 利用率会比较高。

下图中，recvfrom() 用于接收 Socket 传来的数据，并复制到应用进程的缓冲区 buf 中。这里把 recvfrom() 当成系统调用。

```c
ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags, struct sockaddr *src_addr, socklen_t *addrlen);
```

<div align="center"> <img src="pics/1492928416812_4.png"/> </div><br>

## 非阻塞式 I/O

应用进程执行系统调用之后，内核返回一个错误码。应用进程可以继续执行，但是需要不断的执行系统调用来获知 I/O 是否完成，这种方式称为轮询（polling）。

由于 CPU 要处理更多的系统调用，因此这种模型的 CPU 利用率比较低。

<div align="center"> <img src="pics/1492929000361_5.png"/> </div><br>

## I/O 复用

使用 select 或者 poll 等待数据，并且可以等待多个套接字中的任何一个变为可读。这一过程会被阻塞，当某一个套接字可读时返回，之后再使用 recvfrom 把数据从内核复制到进程中。

它可以让单个进程具有处理多个 I/O 事件的能力。又被称为 Event Driven I/O，即事件驱动 I/O。

如果一个 Web 服务器没有 I/O 复用，那么每一个 Socket 连接都需要创建一个线程去处理。如果同时有几万个连接，那么就需要创建相同数量的线程。相比于多进程和多线程技术，I/O 复用不需要进程线程创建和切换的开销，系统开销更小。

<div align="center"> <img src="pics/1492929444818_6.png"/> </div><br>

## 信号驱动 I/O

应用进程使用 sigaction 系统调用，内核立即返回，应用进程可以继续执行，也就是说等待数据阶段应用进程是非阻塞的。内核在数据到达时向应用进程发送 SIGIO 信号，应用进程收到之后在信号处理程序中调用 recvfrom 将数据从内核复制到应用进程中。

相比于非阻塞式 I/O 的轮询方式，信号驱动 I/O 的 CPU 利用率更高。

<div align="center"> <img src="pics/1492929553651_7.png"/> </div><br>

## 异步 I/O

应用进程执行 aio_read 系统调用会立即返回，应用进程可以继续执行，不会被阻塞，内核会在所有操作完成之后向应用进程发送信号。

异步 I/O 与信号驱动 I/O 的区别在于，异步 I/O 的信号是通知应用进程 I/O 完成，而信号驱动 I/O 的信号是通知应用进程可以开始 I/O。

<div align="center"> <img src="pics/1492930243286_8.png"/> </div><br>

## 五大 I/O 模型比较

- 同步 I/O：将数据从内核缓冲区复制到应用进程缓冲区的阶段，应用进程会阻塞。
- 异步 I/O：不会阻塞。

阻塞式 I/O、非阻塞式 I/O、I/O 复用和信号驱动 I/O 都是同步 I/O，它们的主要区别在第一个阶段。

非阻塞式 I/O 、信号驱动 I/O 和异步 I/O 在第一阶段不会阻塞。

<div align="center"> <img src="pics/1492928105791_3.png"/> </div><br>

# I/O 复用

select/poll/epoll 都是 I/O 多路复用的具体实现，select 出现的最早，之后是 poll，再是 epoll。

## select

```c
int select(int n, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```

有三种类型的描述符类型：readset、writeset、exceptset，分别对应读、写、异常条件的描述符集合。fd_set 使用数组实现，数组大小使用 FD_SETSIZE 定义。

timeout 为超时参数，调用 select 会一直阻塞直到有描述符的事件到达或者等待的时间超过 timeout。

成功调用返回结果大于 0，出错返回结果为 -1，超时返回结果为 0。

```c
fd_set fd_in, fd_out;
struct timeval tv;

// Reset the sets
FD_ZERO( &fd_in );
FD_ZERO( &fd_out );

// Monitor sock1 for input events
FD_SET( sock1, &fd_in );

// Monitor sock2 for output events
FD_SET( sock2, &fd_out );

// Find out which socket has the largest numeric value as select requires it
int largest_sock = sock1 > sock2 ? sock1 : sock2;

// Wait up to 10 seconds
tv.tv_sec = 10;
tv.tv_usec = 0;

// Call the select
int ret = select( largest_sock + 1, &fd_in, &fd_out, NULL, &tv );

// Check if select actually succeed
if ( ret == -1 )
    // report error and abort
else if ( ret == 0 )
    // timeout; no event detected
else
{
    if ( FD_ISSET( sock1, &fd_in ) )
        // input event on sock1

    if ( FD_ISSET( sock2, &fd_out ) )
        // output event on sock2
}
```

## poll

```c
int poll(struct pollfd *fds, unsigned int nfds, int timeout);
```

pollfd 使用链表实现。

```c
// The structure for two events
struct pollfd fds[2];

// Monitor sock1 for input
fds[0].fd = sock1;
fds[0].events = POLLIN;

// Monitor sock2 for output
fds[1].fd = sock2;
fds[1].events = POLLOUT;

// Wait 10 seconds
int ret = poll( &fds, 2, 10000 );
// Check if poll actually succeed
if ( ret == -1 )
    // report error and abort
else if ( ret == 0 )
    // timeout; no event detected
else
{
    // If we detect the event, zero it out so we can reuse the structure
    if ( fds[0].revents & POLLIN )
        fds[0].revents = 0;
        // input event on sock1

    if ( fds[1].revents & POLLOUT )
        fds[1].revents = 0;
        // output event on sock2
}
```

## 比较

### 1. 功能

select 和 poll 的功能基本相同，不过在一些实现细节上有所不同。

- select 会修改描述符，而 poll 不会；
- select 的描述符类型使用数组实现，FD_SETSIZE 大小默认为 1024，因此默认只能监听 1024 个描述符。如果要监听更多描述符的话，需要修改 FD_SETSIZE 之后重新编译；而 poll 的描述符类型使用链表实现，没有描述符数量的限制；
- poll 提供了更多的事件类型，并且对描述符的重复利用上比 select 高。
- 如果一个线程对某个描述符调用了 select 或者 poll，另一个线程关闭了该描述符，会导致调用结果不确定。

### 2. 速度

select 和 poll 速度都比较慢。

- select 和 poll 每次调用都需要将全部描述符从应用进程缓冲区复制到内核缓冲区。
- select 和 poll 的返回结果中没有声明哪些描述符已经准备好，所以如果返回值大于 0 时，应用进程都需要使用轮询的方式来找到 I/O 完成的描述符。

### 3. 可移植性

几乎所有的系统都支持 select，但是只有比较新的系统支持 poll。

## epoll

```c
int epoll_create(int size);
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)；
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
```

epoll_ctl() 用于向内核注册新的描述符或者是改变某个文件描述符的状态。已注册的描述符在内核中会被维护在一棵红黑树上，通过回调函数内核会将 I/O 准备好的描述符加入到一个链表中管理，进程调用 epoll_wait() 便可以得到事件完成的描述符。

从上面的描述可以看出，epoll 只需要将描述符从进程缓冲区向内核缓冲区拷贝一次，并且进程不需要通过轮询来获得事件完成的描述符。

epoll 仅适用于 Linux OS。

epoll 比 select 和 poll 更加灵活而且没有描述符数量限制。

epoll 对多线程编程更有友好，一个线程调用了 epoll_wait() 另一个线程关闭了同一个描述符也不会产生像 select 和 poll 的不确定情况。

```c
// Create the epoll descriptor. Only one is needed per app, and is used to monitor all sockets.
// The function argument is ignored (it was not before, but now it is), so put your favorite number here
int pollingfd = epoll_create( 0xCAFE );

if ( pollingfd < 0 )
 // report error

// Initialize the epoll structure in case more members are added in future
struct epoll_event ev = { 0 };

// Associate the connection class instance with the event. You can associate anything
// you want, epoll does not use this information. We store a connection class pointer, pConnection1
ev.data.ptr = pConnection1;

// Monitor for input, and do not automatically rearm the descriptor after the event
ev.events = EPOLLIN | EPOLLONESHOT;
// Add the descriptor into the monitoring list. We can do it even if another thread is
// waiting in epoll_wait - the descriptor will be properly added
if ( epoll_ctl( epollfd, EPOLL_CTL_ADD, pConnection1->getSocket(), &ev ) != 0 )
    // report error

// Wait for up to 20 events (assuming we have added maybe 200 sockets before that it may happen)
struct epoll_event pevents[ 20 ];

// Wait for 10 seconds, and retrieve less than 20 epoll_event and store them into epoll_event array
int ready = epoll_wait( pollingfd, pevents, 20, 10000 );
// Check if epoll actually succeed
if ( ret == -1 )
    // report error and abort
else if ( ret == 0 )
    // timeout; no event detected
else
{
    // Check if any events detected
    for ( int i = 0; i < ret; i++ )
    {
        if ( pevents[i].events & EPOLLIN )
        {
            // Get back our connection pointer
            Connection * c = (Connection*) pevents[i].data.ptr;
            c->handleReadEvent();
         }
    }
}
```


## 工作模式

epoll 的描述符事件有两种触发模式：LT（level trigger）和 ET（edge trigger）。

### 1. LT 模式

当 epoll_wait() 检测到描述符事件到达时，将此事件通知进程，进程可以不立即处理该事件，下次调用 epoll_wait() 会再次通知进程。是默认的一种模式，并且同时支持 Blocking 和 No-Blocking。

### 2. ET 模式

和 LT 模式不同的是，通知之后进程必须立即处理事件，下次再调用 epoll_wait() 时不会再得到事件到达的通知。

很大程度上减少了 epoll 事件被重复触发的次数，因此效率要比 LT 模式高。只支持 No-Blocking，以避免由于一个文件句柄的阻塞读/阻塞写操作把处理多个文件描述符的任务饿死。

## 应用场景

很容易产生一种错觉认为只要用 epoll 就可以了，select 和 poll 都已经过时了，其实它们都有各自的使用场景。

### 1. select 应用场景

select 的 timeout 参数精度为 1ns，而 poll 和 epoll 为 1ms，因此 select 更加适用于实时性要求比较高的场景，比如核反应堆的控制。

select 可移植性更好，几乎被所有主流平台所支持。

### 2. poll 应用场景

poll 没有最大描述符数量的限制，如果平台支持并且对实时性要求不高，应该使用 poll 而不是 select。

### 3. epoll 应用场景

只需要运行在 Linux 平台上，有大量的描述符需要同时轮询，并且这些连接最好是长连接。

需要同时监控小于 1000 个描述符，就没有必要使用 epoll，因为这个应用场景下并不能体现 epoll 的优势。

需要监控的描述符状态变化多，而且都是非常短暂的，也没有必要使用 epoll。因为 epoll 中的所有描述符都存储在内核中，造成每次需要对描述符的状态改变都需要通过 epoll_ctl() 进行系统调用，频繁系统调用降低效率。并且 epoll 的描述符存储在内核，不容易调试。

