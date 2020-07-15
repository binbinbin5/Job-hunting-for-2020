
[TOC]


## 本地变量类型推断（Var）
字面量定义局部变量

```
private static void testVar() {
    var javastack = "javastack"; //等于String javastack = "javastack";
    System.out.println(javastack);
}
```


接收方法返回值定义局部变量

```
private static void testMethod() {
    var javastack = getJavastack();
    System.out.println(javastack);
}
public static String getJavastack() {
    return "javastack";
}
```


循环中定义局部变量

```
private static void testLoop() {
    for (var i = 0; i < 3; i++) {
        for (var m = 10; m < 15; m++) {
            System.out.println(i + m);
        }
    }
}
```


泛型结合局部变量

```
private static void testGeneric() {
    // 表达式1
    List<String> list1 = new ArrayList<>();
    list1.add("javastack");
    // 表达式2
    var list2 = new ArrayList<>();
    list2.add(2018);
    // 表达式3
    var list3 = new ArrayList<String>();
    list3.add("javastack");
}
    表达式1后面 < > 里面 jdk 1.7+开始是不用带具体类型的，在接口中指明就行了。
    表达式2中如果使用 var 的话， < > 里面默认会是 Object 的，所以可以添加任意类型。
    表达式3中在 < > 强制使用了 String 来指定泛型。
```



a.局部变量类型推断不能用在以下场景
类成员变量类型

```
// 编译报错
private var javastack = "Java技术栈";

方法返回类型
/**
 * 编译报错
 * @return
 */
public static var getJavastack(){
     return "Java技术栈";
}

Lambda 表达式
private static void testLambda() {
    Runnable runnable = () -> System.out.println("javastack");
    // 编译报错
    // var runnable = () -> System.out.println("javastack");
}
```



b.局部变量类型推断优缺点

```
优点：简化代码
CopyOnWriteArrayList list1 = new CopyOnWriteArrayList();
ConcurrentModificationException cme1 = new ConcurrentModificationException();
DefaultServiceUnavailableRetryStrategy strategy1 = new
        DefaultServiceUnavailableRetryStrategy();
        //======================
var list2 = new CopyOnWriteArrayList<>();
var cme2 = new ConcurrentModificationException();
var strategy2 = new DefaultServiceUnavailableRetryStrategy();
```

从以上代码可以看出，很长的定义类型会显得代码很冗长，使用 var 大大简化了代码编写，同时类型统一显得代码很对齐。
缺点：掩盖类型

```
var token = new JsonParserDelegate(parser).currentToken();
```


看以上代码，不进去看返回结果类型，谁知道返回的类型是什么？所以这种情况最好别使用 var，而使用具体的抽象类、接口或者实例类型。

c.var关键字原理

var其实就是 Java 10 增加的一种语法糖而已，在编译期间会自动推断实际类型，其编译后的字节码和实际类型一致。


## 字符串加强
Java 11 增加了一系列的字符串处理方法，如以下所示。

```
// 判断字符串是否为空白
" ".isBlank(); // true

// 去除首尾空格
" Javastack ".strip(); // "Javastack"

// 去除尾部空格 
" Javastack ".stripTrailing(); // " Javastack"

// 去除首部空格 
" Javastack ".stripLeading(); // "Javastack "

// 复制字符串
"Java".repeat(3); // "JavaJavaJava"

// 行数统计
"A\nB\nC".lines().count(); // 3
```



## 集合加强
自 Java 9 开始，Jdk 里面为集合（List/ Set/ Map）都添加了 of 和 copyOf 方法，它们两个都用来创建不可变的集合，来看下它们的使用和区别。
示例1：

```
var list = List.of("Java", "Python", "C");
var copy = List.copyOf(list);
System.out.println(list == copy);  // true
```



示例2：

```
var list = new ArrayList<String>();
var copy = List.copyOf(list);
System.out.println(list == copy);  // false
```



示例1和2代码差不多，为什么一个为true,一个为false?
来看下它们的源码：

```
static <E> List<E> of(E... elements) {
    switch (elements.length) { // implicit null check of elements
        case 0:
            return ImmutableCollections.emptyList();
        case 1:
            return new ImmutableCollections.List12<>(elements[0]);
        case 2:
            return new ImmutableCollections.List12<>(elements[0], elements[1]);
        default:
            return new ImmutableCollections.ListN<>(elements);
    }
}

static <E> List<E> copyOf(Collection<? extends E> coll) {
    return ImmutableCollections.listCopy(coll);
}

static <E> List<E> listCopy(Collection<? extends E> coll) {
    if (coll instanceof AbstractImmutableList && coll.getClass() != SubList.class) {
        return (List<E>)coll;
    } else {
        return (List<E>)List.of(coll.toArray());
    }
}
```


可以看出 copyOf 方法会先判断来源集合是不是 AbstractImmutableList 类型的，如果是，就直接返回，如果不是，则调用 of 创建一个新的集合。
示例2因为用的 new 创建的集合，不属于不可变 AbstractImmutableList 类的子类，所以 copyOf 方法又创建了一个新的实例，所以为false.

注意：使用 of 和 copyOf 创建的集合为不可变集合，不能进行添加、删除、替换、排序等操作，不然会报 java.lang.UnsupportedOperationException 异常。

上面演示了 List 的 of 和 copyOf 方法，Set 和 Map 接口都有。

## Stream 加强

Stream 是 Java 8 中的新特性，Java 9 开始对 Stream 增加了以下 4 个新方法。

1) 增加单个参数构造方法，可为null


```
Stream.ofNullable(null).count(); // 0
```



2) 增加 takeWhile 和 dropWhile 方法


```
Stream.of(1, 2, 3, 2, 1)

    .takeWhile(n -> n < 3)

    .collect(Collectors.toList());  // [1, 2]
```



从开始计算，当 n < 3 时就截止。


```
Stream.of(1, 2, 3, 2, 1)

    .dropWhile(n -> n < 3)

    .collect(Collectors.toList());  // [3, 2, 1]
```



这个和上面的相反，一旦 n < 3 不成立就开始计算。

## iterate重载

这个 iterate 方法的新重载方法，可以让你提供一个 Predicate (判断条件)来指定什么时候结束迭代。

如果你对 JDK 8 中的 Stream 还不熟悉，可以看之前分享的这一系列教程。

## Optional 加强

Opthonal 也增加了几个非常酷的方法，现在可以很方便的将一个 Optional 转换成一个 Stream, 或者当一个空 Optional 时给它一个替代的。

```
Optional.of("javastack").orElseThrow();     // javastack

Optional.of("javastack").stream().count();  // 1

Optional.ofNullable(null)

    .or(() -> Optional.of("javastack"))

    .get();   // javastack
```



## InputStream 加强

InputStream 终于有了一个非常有用的方法：transferTo，可以用来将数据直接传输到 OutputStream，这是在处理原始数据流时非常常见的一种用法，如下示例。


```
var classLoader = ClassLoader.getSystemClassLoader();

var inputStream = classLoader.getResourceAsStream("javastack.txt");

var javastack = File.createTempFile("javastack2", "txt");

try (var outputStream = new FileOutputStream(javastack)) {

    inputStream.transferTo(outputStream);

}
```



## HTTP Client API

这是 Java 9 开始引入的一个处理 HTTP 请求的的孵化 HTTP Client API，该 API 支持同步和异步，而在 Java 11 中已经为正式可用状态，你可以在 java.net 包中找到这个 API。

来看一下 HTTP Client 的用法：


```
var request = HttpRequest.newBuilder()

    .uri(URI.create("https://javastack.cn"))

    .GET()

    .build();

var client = HttpClient.newHttpClient();

// 同步

HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());

System.out.println(response.body());

// 异步

client.sendAsync(request, HttpResponse.BodyHandlers.ofString())

    .thenApply(HttpResponse::body)

    .thenAccept(System.out::println);
```


上面的 .GET() 可以省略，默认请求方式为 Get！

更多使用示例可以看这个 API，后续有机会再做演示。

现在 Java 自带了这个 HTTP Client API，我们以后还有必要用 Apache 的 HttpClient 工具包吗？

## 化繁为简，一个命令编译运行源代码

看下面的代码。


```
// 编译

javac Javastack.java

// 运行

java Javastack
```



在我们的认知里面，要运行一个 Java 源代码必须先编译，再运行，两步执行动作。而在未来的 Java 11 版本中，通过一个 java 命令就直接搞定了，如以下所示。


```
java Javastack.java
```


