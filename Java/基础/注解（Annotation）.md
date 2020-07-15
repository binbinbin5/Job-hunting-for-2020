[TOC]

## 简介

- 定义：一种标识 / 标签


```
注解属于 Java中的一种类型，同 类class 和 接口interface 一样 
在 Java SE 5.0开始引入
```


- 基础作用：标识 / 解释 Java 代码


```
//比如重写 
@overwrite
public void set(){}
```




## 应用场景

- 面向 编译器 / APT 使用

- APT（Annotation Processing Tool）：提取 & 处理 Annotation 的代码 
- 因为当开发者使用Annotation 修饰类、方法、方法 等成员后，这些 Annotation 不会自己生效，必须由开发者提供相应的代码来提取并处理 Annotation 信息。这些处理提取和处理 Annotation 的代码统称为 APT 

- 所以说，注解除了最基本的解释 & 说明代码，注解的应用场景还取决于你想如何利用它

比如：


```
场景1：测试代码
public class ExampleUnitTest {
    @Test
    public void Method() throws Exception {
          ...
    }
}
// @Test 标记了要进行测试的方法Method() 
```


```
场景2：简化使用 & 降低代码量
<-- Http网络请求库 Retrofit -->
// 采用 注解 描述 网络请求参数
public interface GetRequest_Interface {

 @GET("ajax.php?a=fy&f=auto&t=auto&w=hello%20world")
    Call<Translation> getCall();

 }

<-- IOC 框架ButterKnife -->
public class MainActivity extends AppCompatActivity {

// 通过注解设置资源
@BindView(R.id.test)
    TextView mTv;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ButterKnife.bind(this);
    }
}
```




## 注解的类型

注解的类型包括：元注解 、 Java内置注解、 自定义注解。

### 元注解


```
============
//元注解作用于注解 & 解释注解

// 元注解在注解定义时进行定义
@元注解
public @interface Carson_Annotation {

}
=============
//注解作用于Java代码 & 解释Java代码
@Carson_Annotation
public class Carson {
}

```

![](https://raw.githubusercontent.com/binbinbin5/myPics/master/imgs/YUANZHUJIE.png)

@Retention

- 定义：保留注解
- 作用：解释 / 说明了注解的生命周期
- 具体使用


```
// 元注解@Retention(RetentionPolicy.RUNTIME)的作用：说明 注解Carson_Annotation的生命周期 = 保留到程序运行时 & 被加载到 JVM 中
@Retention(RetentionPolicy.RUNTIME)
public @interface Carson_Annotation {
}

<-- 元注解@Retention参数说明 -->
// RetentionPolicy.RUNTIME：注解保留到程序运行时 & 会被加载进入到 JVM 中，所以在程序运行时可以获取到它们
// RetentionPolicy.CLASS：注解只被保留到编译进行时 & 不会被加载到 JVM 
// RetentionPolicy.SOURCE：注解只在源码阶段保留 & 在编译器进行编译时将被丢弃忽视。

```


@Documented

- 定义：Java文档注解
- 作用：将注解中的元素包含到 Javadoc文档中
- 具体使用


```
// 元注解@Documented作用：说明 注解Carson_Annotation的元素包含到 Javadoc 文档中
@Documented
public @interface Carson_Annotation {
}
```


@Target

- 定义：目标注解
- 作用：限定了注解作用的目标范围，包括类、方法等等
- 具体使用


```
// 元注解@Target作用：限定了注解Carson_Annotation作用的目标范围 = 方法
// 即注解Carson_Annotation只能用于解释说明 某个方法
@Target(ElementType.METHOD)
public @interface Carson_Annotation {
}
<-- @Target取值参数说明 -->
// ElementType.PACKAGE：可以给一个包进行注解
// ElementType.ANNOTATION_TYPE：可以给一个注解进行注解
// ElementType.TYPE：可以给一个类型进行注解，如类、接口、枚举
// ElementType.CONSTRUCTOR：可以给构造方法进行注解
// ElementType.METHOD：可以给方法进行注解
// ElementType.PARAMETER 可以给一个方法内的参数进行注解
// ElementType.FIELD：可以给属性进行注解
// ElementType.LOCAL_VARIABLE：可以给局部变量进行注解
```


@Inherited

- 定义：继承注解
- 作用：使得一个 被@Inherited注解的注解 作用的类的子类可以继承该类的注解

1. 前提：子类没有被任何注解应用
2. 如，注解@Carson_Annotation（被元注解@Inherited 注解）作用于A类，那么A类的子类则继承了A类的注解

- 具体使用


```
// 元注解@Inherited 作用于 注解Carson_Annotation
@Inherited
public @interface Carson_Annotation {
}
// 注解Carson_Annotation 作用于A类
@Carson_Annotation
public class A {
  }
// B类继承了A类，即B类 = A类的子类，且B类没被任何注解应用
// 那么B类继承了A类的注解 Carson_Annotation
public class B extends A {}
```


@Repeatable

- 定义：可重复注解

- Java 1.8后引进

- 作用：使得作用的注解可以取多个值
- 具体使用


```
// 1. 定义 容器注解 @ 职业
public @interface Job {
    Person[]  value();
}
<-- 容器注解介绍 -->
// 定义：本身也是一个注解
// 作用：存放其它注解
// 具体使用：必须有一个 value 属性；类型 = 被 @Repeatable 注解的注解数组
// 如本例中，被 @Repeatable 作用 = @Person ，所以value属性 = Person []数组

// 2. 定义@Person 
// 3. 使用@Repeatable 注解 @Person
// 注：@Repeatable 括号中的类 = 容器注解
@Repeatable（Job.class）
public @interface Person{
    String role default "";
}

// 在使用@Person（被@Repeatable 注解 ）时，可以取多个值来解释Java代码
// 下面注解表示：Carson类即是产品经理，又是程序猿
@Person(role="coder")
@Person(role="PM")
public class Carson{

}


```


注：一个注解可以被多个元注解作用


```
@Inherited
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Carson_Annotation {
}
```




###  Java内置的注解

- 定义：即Java内部已经实现好的注解
- 类型：Java中 内置的注解有5类，具体包括：

![](https://raw.githubusercontent.com/binbinbin5/myPics/master/imgs/YUZHIZHUJIE.png)


![](https://raw.githubusercontent.com/binbinbin5/myPics/master/imgs/YUZHIZHUJIE2.png)



## 自定义注解（import）

- 定义：开发者自定义注解

### 注解的定义


```
// 通过 @interface 关键字进行定义
// 形式类似于接口，区别在于多了一个 @ 符号

public @interface Carson_Annotation {

}

// 上面的代码创建了一个名为 Carson_Annotaion 的注解
```


###  注解的属性


```
<-- 1. 定义 注解的属性 -->
// 注解的属性 在定义该注解本身时 进行定义
public @interface Carson_Annotation {
// 注解的属性 = 成员变量
// 注解只有成员变量，没有方法

    // 注解@Carson_Annotation中有2个属性：id 和 msg  
    int id();
    String msg() default "Hi" ;

    // 说明：
      // 注解的属性以 “无形参的方法” 形式来声明
      // 方法名 = 属性名
      // 方法返回值 = 属性类型 = 8 种基本数据类型 + 类、接口、注解及对应数组类型
      // 用 default 关键值指定 属性的默认值，如上面的msg的默认值 = ”Hi“
}

<-- 2. 赋值 注解的属性 -->
// 注解的属性在使用时进行赋值
// 注解属性的赋值方式 = 注解括号内以 “value=”xx” “ 形式；用 ”，“隔开多个属性

// 注解Carson_Annotation 作用于A类
// 在作用 / 使用时对注解属性进行赋值
@Carson_Annotation(id=1,msg="hello,i am Carson")
public class A {
  }

// 特别说明：若注解只有一个属性，则赋值时”value“可以省略

// 假设注解@Carson_Annotation只有1个msg属性
// 赋值时直接赋值即可
@Carson_Annotation("hello,i am Carson")
public class A {
  }

```


### 注解的应用


```
// 在类 / 成员变量 / 方法 定义前 加上 “@注解名” 就可以使用该注解
@Carson_Annotation
public class Carson {
    @Carson_Annotation
    int a;
    @Carson_Annotation
    public void bMethod(){}
}
// 即注解@Carson_Annotation 用于标识解释 Carson类 / a变量 / b方法
```

### 获取注解

- 定义：即获取某个对象上的所有注解。
- 实现手段：Java 反射技术

>由于反射需要较高时间成本，所以注解使用时要谨慎考虑时间成本 

- 具体使用


```
<-- 步骤1：判断该类是否应用了某个注解 -->
// 手段：采用 Class.isAnnotationPresent() 
public boolean isAnnotationPresent(Class<? extends Annotation> annotationClass) {}

<-- 步骤2：获取 注解对象（Annotation）-->
  // 手段1：采用getAnnotation()  ；返回指定类型的注解
public <A extends Annotation> A getAnnotation(Class<A> annotationClass) {}
  // 手段2：采用getAnnotations() ；返回该元素上的所有注解
public Annotation[] getAnnotations() {}
```
