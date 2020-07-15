

一般都写好.java文件，用javac编译成.class文件后，通过类加载器进行加载，就可以成为运行时候使用的Java类了。

## 有哪些方法可以在运行时动态生成一个Java类
1、通常的开发过程是，开发者编写java代码，调用javac编译成class文件，然后通过类加载机制载入JVM，就成为应用运行时可以使用的java类了。

2、直接用ProcessBuilder之类启动javac进程，并指定上面生成的文件作为输入，进行编译。最后，再利用类加载器，在运行时加载即可。

3、可以使用java Compiler API，这是JDK提供的标准API，里面提供了与javac对等的编译器功能。

### 其他问题

>字节码和类加载到底是怎么无缝进行转换的？发生在整个类加载过程的哪一步？

    通过defineClass

>如何利用字节码操纵技术，实现基本的动态代理逻辑？

对于一个普通的 Java 动态代理，其实现过程可以简化成为：
    
- 提供一个基础的接口，作为被调用类型（com.mycorp.HelloImpl）和代理类之间的统一入口，如 com.mycorp.Hello。
- 实现InvocationHandler，对代理对象方法的调用，会被分派到其 invoke 方法来真正实现动作。
- 通过 Proxy 类，调用其 newProxyInstance 方法，生成一个实现了相应基础接口的代理类实例，可以看下面的方法签名。
    
```
    public static Object newProxyInstance(ClassLoader loader,
                                      	Class<?>[] interfaces,
                                      	InvocationHandler h)
```
    
    
我们分析一下，动态代码生成是具体发生在什么阶段呢？
    
不错，就是在 newProxyInstance 生成代理类实例的时候。我选取了 JDK 自己采用的 ASM 作为示例，一起来看看用 ASM 实现的简要过程，请参考下面的示例代码片段。
    
第一步，生成对应的类，其实和我们去写 Java 代码很类似，只不过改为用 ASM 方法和指定参数，代替了我们书写的源码。
    
    
```
    ClassWriter cw = new ClassWriter(ClassWriter.COMPUTE_FRAMES);
     
    cw.visit(V1_8,                      // 指定 Java 版本
        	ACC_PUBLIC,             	// 说明是 public 类型
            "com/mycorp/HelloProxy",	// 指定包和类的名称
        	null,                   	// 签名，null 表示不是泛型
        	"java/lang/Object",             	// 指定父类
        	new String[]{ "com/mycorp/Hello" }); // 指定需要实现的接口
```
更进一步，我们可以按照需要为代理对象实例，生成需要的方法和逻辑。
    
    ```
    MethodVisitor mv = cw.visitMethod(
        	ACC_PUBLIC,         	    // 声明公共方法
        	"sayHello",             	// 方法名称
        	"()Ljava/lang/Object;", 	// 描述符
        	null,                   	// 签名，null 表示不是泛型
        	null);                      // 可能抛出的异常，如果有，则指定字符串数组
     
    mv.visitCode();
    // 省略代码逻辑实现细节
    cw.visitEnd();                      // 结束类字节码生成
    ```
    
    
上面的代码虽然有些晦涩，但总体还是能多少理解其用意，不同的 visitX 方法提供了创建类型，创建各种方法等逻辑。ASM API，广泛的使用了Visitor模式，如果你熟悉这个模式，就会知道它所针对的场景是将算法和对象结构解耦，非常适合字节码操纵的场合，因为我们大部分情况都是依赖于特定结构修改或者添加新的方法、变量或者类型等。
    
按照前面的分析，字节码操作最后大都应该是生成 byte 数组，ClassWriter 提供了一个简便的方法。
    
```
cw.toByteArray();
```
然后，就可以进入我们熟知的类加载过程了。



>除了动态代理，字节码操纵技术还有那些应用场景？

IOC ORM框架等