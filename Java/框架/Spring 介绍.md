[toc]

# Spring理解
这篇文章主要是想通过一些问题，加深大家对于 Spring 的理解，所以不会涉及太多的代码！这篇文章整理了挺长时间，下面的很多问题我自己在使用 Spring 的过程中也并没有注意，自己也是临时查阅了很多资料和书籍补上的。网上也有一些很多关于 Spring 常见问题/面试题整理的文章，我感觉大部分都是互相 copy，而且很多问题也不是很汗，有些回答也存在问题。所以，自己花了一周的业余时间整理了一下，希望对大家有帮助。

### Bean周期和作用范围
生命周期：

1. 实例化 Bean 对象。
1. 设置 Bean 属性。
1. 如果我们通过各种 Aware 接口声明了依赖关系，则会注入 Bean 对容器基础设施层面的依赖。具体包括 BeanNameAware、BeanFactoryAware 和 ApplicationContextAware，分别会注入 Bean ID、Bean Factory 或者 ApplicationContext。
1. 调用 BeanPostProcessor 的前置初始化方法 postProcessBeforeInitialization。
1. 如果实现了 InitializingBean 接口，则会调用 afterPropertiesSet 方法。
1. 调用 Bean 自身定义的 init 方法。
1. 调用 BeanPostProcessor 的后置初始化方法 postProcessAfterInitialization。
1. 创建过程完毕。
1. 调用自身和DispsableBean的destroy方法

Spring Bean 有五个作用域，其中最基础的有下面两种：
1. Singleton(单例) 在整个应用中,只创建bean的一个实例
2. Propotype(原型) 每次注入或者通过Spring应用上下文获取的时候,都会创建一个新
的bean实例。
3. Session(会话) 在Web应用中,为每个会话创建一个bean实例。
4. Request(请求) 在Web应用中,为每个请求创建一个bean实例。



他们是什么时候创建的:
1. 一个单例的bean,而且lazy-init属性为false(默认),在Application Context创建的时候构造
2. 一个单例的bean,lazy-init属性设置为true,那么,它在第一次需要的时候被构造.
3. 其他scope的bean,都是在第一次需要使用的时候创建

他们是什么时候销毁的:
1. 单例的bean始终 存在与application context中, 只有当 application 终结的时候,才会销毁
2. 和其他scope相比,Spring并没有管理prototype实例完整的生命周期,在实例化,配置,组装对象交给应用后,spring不再管理.只要bean本身不持有对另一个资源（如数据库连接或会话对象）的引用，只要删除了对该对象的所有引用或对象超出范围，就会立即收集垃圾.
3. Request: 每次客户端请求都会创建一个新的bean实例,一旦这个请求结束,实例就会离开scope,被垃圾回收.
4. Session: 如果用户结束了他的会话,那么这个bean实例会被GC.


### Spring 的基础机制

至少你需要理解下面两个基本方面。

1. 控制反转（Inversion of Control），或者也叫依赖注入（Dependency Injection），广泛应用于 Spring 框架之中，可以有效地改善了模块之间的紧耦合问题。

从 Bean 创建过程可以看到，它的依赖关系都是由容器负责注入，具体实现方式包括带参数的构造函数、setter 方法或者AutoWired方式实现。

2. AOP，我们已经在前面接触过这种切面编程机制，Spring 框架中的事务、安全、日志等功能都依赖于 AOP 技术，下面我会进一步介绍。

### Spring 到底是指什么？

我前面谈到的 Spring，其实是狭义的Spring Framework，其内部包含了依赖注入、事件机制等核心模块，也包括事务、O/R Mapping 等功能组成的数据访问模块，以及 Spring MVC 等 Web 框架和其他基础组件。

广义上的 Spring 已经成为了一个庞大的生态系统，例如：

- Spring Boot，通过整合通用实践，更加自动、智能的依赖管理等，Spring Boot 提供了各种典型应用领域的快速开发基础，所以它是以应用为中心的一个框架集合。
- Spring Cloud，可以看作是在 Spring Boot 基础上发展出的更加高层次的框架，它提供了构建分布式系统的通用模式，包含服务发现和服务注册、分布式配置管理、负载均衡、分布式诊断等各种子系统，可以简化微服务系统的构建。
- 当然，还有针对特定领域的 Spring Security、Spring Data 等。

上面的介绍比较笼统，针对这么多内容，如果将目标定得太过宽泛，可能就迷失在 Spring 生态之中，我建议还是深入你当前使用的模块，如 Spring MVC。并且，从整体上把握主要前沿框架（如 Spring Cloud）的应用范围和内部设计，至少要了解主要组件和具体用途，毕竟如何构建微服务等，已经逐渐成为 Java 应用开发面试的热点之一。

### Spring AOP 自身设计和实现的细节。

先问一下自己，我们为什么需要切面编程呢？

==切面编程落实到软件工程其实是为了更好地模块化，而不仅仅是为了减少重复代码。通过 AOP 等机制==，我们可以把横跨多个不同模块的代码抽离出来，让模块本身变得更加内聚，进而业务开发者可以更加专注于业务逻辑本身。从迭代能力上来看，我们可以通过切面的方式进行修改或者新增功能，这种能力不管是在问题诊断还是产品能力扩展中，都非常有用。

在之前的分析中，我们已经分析了 AOP Proxy 的实现原理，简单回顾一下，它底层是基于 JDK 动态代理或者 cglib 字节码操纵等技术，运行时动态生成被调用类型的子类等，并实例化代理对象，实际的方法调用会被代理给相应的代理对象。但是，这并没有解释具体在 AOP 设计层面，什么是切面，如何定义切入点和切面行为呢？

Spring AOP 引入了其他几个关键概念：

- Aspect，通常叫作方面，它是跨不同 Java 类层面的横切性逻辑。在实现形式上，既可以是 XML 文件中配置的普通类，也可以在类代码中用“@Aspect”注解去声明。在运行时，Spring 框架会创建类似Advisor来指代它，其内部会包括切入的时机（Pointcut）和切入的动作（Advice）。
- Join Point，它是 Aspect 可以切入的特定点，在 Spring 里面只有方法可以作为 Join Point。
- Advice，它定义了切面中能够采取的动作。如果你去看 Spring 源码，就会发现 Advice、Join Point 并没有定义在 Spring 自己的命名空间里，这是因为他们是源自AOP 联盟，可以看作是 Java 工程师在 AOP 层面沟通的通用规范。

Java 核心类库中同样存在类似代码，例如 Java 9 中引入的 Flow API 就是 Reactive Stream 规范的最小子集，通过这种方式，可以保证不同产品直接的无缝沟通，促进了良好实践的推广


## 什么是 Spring 框架?

Spring 是一种轻量级开发框架，旨在提高开发人员的开发效率以及系统的可维护性。Spring 官网：<https://spring.io/>。

我们一般说 Spring 框架指的都是 Spring Framework，它是很多模块的集合，使用这些模块可以很方便地协助我们进行开发。==这些模块是：核心容器、数据访问/集成,、Web、AOP（面向切面编程）、工具、消息和测试模块==。比如：Core Container 中的 Core 组件是Spring 所有组件的核心，Beans 组件和 Context 组件是实现IOC和依赖注入的基础，AOP组件用来实现面向切面编程。

Spring 官网列出的 Spring 的 6 个特征:

- **核心技术** ：依赖注入(DI)，AOP，事件(events)，资源，i18n，验证，数据绑定，类型转换，SpEL。
- **测试** ：模拟对象，TestContext框架，Spring MVC 测试，WebTestClient。
- **数据访问** ：事务，DAO支持，JDBC，ORM，编组XML。
- **Web支持** : Spring MVC和Spring WebFlux Web框架。
- **集成** ：远程处理，JMS，JCA，JMX，电子邮件，任务，调度，缓存。
- **语言** ：Kotlin，Groovy，动态语言。

用过 Spring 框架的都知道 Spring 能流行是因为它的两把利器：IOC 和 AOP，IOC 可以帮助我们管理对象的依赖关系，极大减少对象的耦合性，而 AOP 的切面编程功能可以更方面的使用动态代理来实现各种动态方法功能（如事务、缓存、日志等）。

而要集成 Spring 框架，必须要用到 XML 配置文件，或者注解式的 Java 代码配置。无论是使用 XML 或者代码配置方式，都需要对相关组件的配置有足够的了解，然后再编写大量冗长的配置代码。
## 列举一些重要的Spring模块？

下图对应的是 Spring4.x 版本。目前最新的5.x版本中 Web 模块的 Portlet 组件已经被废弃掉，同时增加了用于异步响应式处理的 WebFlux 组件。

![Spring主要模块](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/Spring主要模块.png)

- **Spring Core：** 基础,可以说 Spring 其他所有的功能都需要依赖于该类库。主要提供 IOC 依赖注入功能。
- **Spring  Aspects** ： 该模块为与AspectJ的集成提供支持。
- **Spring AOP** ：提供了面向方面的编程实现。
- **Spring JDBC** : Java数据库连接。
- **Spring JMS** ：Java消息服务。
- **Spring ORM** : 用于支持Hibernate等ORM工具。
- **Spring Web** : 为创建Web应用程序提供支持。
- **Spring Test** : 提供了对 JUnit 和 TestNG 测试的支持。

## 谈谈自己对于 Spring IoC 和 AOP 的理解


## IOC
比如说我们要创建一个绿茶，手动进行new，那么当一段时间之后，我们需要改变需求，把绿茶改成红茶，那么就需要在整个程序中进行修改比较繁琐，那么我们可以委托给Spring的bean工厂进行创建，我们只需要提出要求，那么它自动会通过工厂内的方法创建不同的对象，这边我们绿茶改成红茶，那么工厂内调用getRedTea()就行了。只需要在一个地方进行修改就行了。


控制反转也被成为依赖注入，是一种降低耦合关系的设计：一般分层体系来说，上层依赖下层，当上层需要修改的时候，下层也会全部要改修，通过IOC使得下层依赖上层，完成控制反转，然后通过注入一个实例化对象的方式来进行完成。

IoC（Inverse of Control:控制反转）是一种**设计思想**，就是 **将原本在程序中手动创建对象的控制权，交由Spring框架来管理。**  IoC 在其他语言中也有应用，并非 Spirng 特有。 **IoC 容器是 Spring 用来实现 IoC 的载体，  IoC 容器实际上就是个Map（key，value）,Map 中存放的是各种对象。**

将对象之间的相互依赖关系交给 IOC 容器来管理，并由 IOC 容器完成对象的注入。这样可以很大程度上简化应用的开发，把应用从复杂的依赖关系中解放出来。  **IOC 容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的。** 在实际项目中一个 Service 类可能有几百甚至上千个类作为它的底层，假如我们需要实例化这个 Service，你可能要每次都要搞清这个 Service 所有底层类的构造函数，这可能会把人逼疯。如果利用 IOC 的话，你只需要配置好，然后在需要的地方引用就行了，这大大增加了项目的可维护性且降低了开发难度。

就是由spring来负责控制对象的生命周期和对象间的关系: 所有的类的创建、销毁都由 spring来控制，也就是说控制对象生存周期的不再是引用它的对象，而是spring。对于某个具体的对象而言，以前是它控制其他对象，现在是所有对象都被spring控制，所以这叫控制反转。
1. Spring IOC是一种设计模式，使对象不用显示的创建依赖对象，而是将对象创建的过程交给Spring的IOC容器去管理，通过依赖注入的方式，来实现IOC

2. 控制权的转移:应用程序本身不负责依赖对象的创建和维护，而是由外部容器负责创建和维护。
获得依赖对象的过程被反转了。
IOC相当于房屋中介：找中介=找IOC容器，中介介绍房子=容器返回对象，入住=使用对象。

>Sping Core最核心的部分。

IOC支持的功能
- 依赖注入
- 依赖检查
- 自动配装
- 支持集合
- 指定初始化方法和销毁方法
- 支持回调（谨慎）



Spring 时代我们一般通过 XML 文件来配置 Bean，后来开发人员觉得 XML 文件来配置不太好，于是 SpringBoot 注解配置就慢慢开始流行起来。

推荐阅读：https://www.zhihu.com/question/23277575/answer/169698662

**Spring IOC的初始化过程：** 
![Spring IOC的初始化过程](https://user-gold-cdn.xitu.io/2018/9/22/165fea36b569d4f4?w=709&h=56&f=png&s=4673)

IOC源码阅读

- https://javadoop.com/post/spring-ioc







 



### 依赖注入（DI）

理解DI的关键是：“谁依赖谁，为什么需要依赖，谁注入谁，注入了什么”，那我们来深入分析一下：

- 　　谁依赖于谁：当然是应用程序依赖于IoC容器；
- 　　为什么需要依赖：应用程序需要IoC容器来提供对象需要的外部资源；
- 　　谁注入谁：很明显是IoC容器注入应用程序某个对象，应用程序依赖的对象；
- 　　注入了什么：就是注入某个对象所需要的外部资源（包括对象、资源、常量数据）。

　　IoC和DI由什么关系呢？其实它们是同一个概念的不同角度描述，由于控制反转概念比较含糊（可能只是理解为容器控制对象这一个层面，很难让人想到谁来维护对象关系），所以2004年大师级人物Martin Fowler又给出了一个新的名字：“依赖注入”，相对IoC 而言，“依赖注入”明确描述了“被注入对象依赖IoC容器配置依赖对象”。



控制反转的一种方式，目的是为了创建对象并且组装对象之间的关系。IOC容器运行期间，动态的将某种依赖注入到对象之中。
```
A-> B ->C ->D
//如果A依赖B，B依赖C，C依赖D，那么当D修改的时候，ABC都会要进行修改
```
![](https://raw.githubusercontent.com/binbinbin5/myPics/master/imgs/20190630141512.png)

```
//<- 为注入,
A<- B <- C <-D
```
![](https://raw.githubusercontent.com/binbinbin5/myPics/master/imgs/20190630141447.png)

==我们要使用上层控制下层，而不是下层控制上层，这叫控制反转。用依赖注入来实现控制反转：把底层类作为参数传递给上层类，实现上层对下层的“控制”。==

多个类互不受影响。
![](https://raw.githubusercontent.com/binbinbin5/myPics/master/imgs/20190630141926.png)


> DL 依赖查找
根据框架提供的方法来获取对象，但是现在已经被抛弃了（需要用户自己使用API查找资源 具有侵入性）。

### 依赖倒置原则
高层模块不应该依赖底层模块。
![](https://raw.githubusercontent.com/binbinbin5/myPics/master/imgs/20190630144511.png)

### IOC容器
- 避免再各处使用new创建类，并且可以做到统一维护，自动进行初始化
- 创建实例的时候不需要了解其中细节

![](https://raw.githubusercontent.com/binbinbin5/myPics/master/imgs/20190630144719.png)




## AOP

AOP(Aspect-Oriented Programming:面向切面编程)能够将那些与业务无关，**却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来**，便于**减少系统的重复代码**，**降低模块间的耦合度**，并**有利于未来的可拓展性和可维护性**。

**Spring AOP就是基于动态代理的**，如果要代理的对象，实现了某个接口，那么Spring AOP会使用**JDK Proxy**，去创建代理对象，而对于没有实现接口的对象，就无法使用 JDK Proxy 去进行代理了，这时候Spring AOP会使用**Cglib** ，这时候Spring AOP会使用 **Cglib** 生成一个被代理对象的子类来作为代理，如下图所示：我们把过滤器的重复代码抽取出来。


![](https://raw.githubusercontent.com/binbinbin5/myPics/master/file/50a2b97086e8a20a54b844be2b8c5e6.png)


在日常的软件开发中，拿日志来说，一个系统软件的开发都是必须进行日志记录的，不然万一系统出现什么bug，你都不知道是哪里出了问题。举个小栗子，当你开发一个登陆功能，你可能需要在用户登陆前后进行权限校验并将校验信息（用户名,密码,请求登陆时间，ip地址等）记录在日志文件中，当用户登录进来之后，当他访问某个其他功能时，也需要进行合法性校验。想想看，当系统非常地庞大，系统中专门进行权限验证的代码是非常多的，而且非常地散乱，我们就想能不能将这些权限校验、日志记录等非业务逻辑功能的部分独立拆分开，并且在系统运行时需要的地方（连接点）进行动态插入运行，不需要的时候就不理，因此AOP是能够解决这种状况的思想吧！


当然你也可以使用 AspectJ ,Spring AOP 已经集成了AspectJ  ，AspectJ  应该算的上是 Java 生态系统中最完整的 AOP 框架了。

使用 AOP 之后我们可以把一些通用功能抽象出来，在需要用到的地方直接使用即可，这样大大简化了代码量。我们需要增加新功能时也方便，这样也提高了系统扩展性。日志功能、事务管理等等场景都用到了 AOP 。

### 接口和面向接口编程
接口是用于沟通的中介物的抽象化，实体是提供给外界的抽象化说明，使其能被修改内部而不影响外界。

结构设计中，分清层次以及调用，每层只向外提供一组功能接口，各层仅仅依赖接口而非实现类。
1. 接口实现的变得不影响各层的调用
2. 面向接口编程中的接口适用于隐藏具体实现和实现多态性的组件

```
public interface Inter{//接口
    String hello(String str);
}

public class ShiXianLei implements Inter{//实现类
    String hello(String str){
        System.out.println(str);
    }
}

public class Test{//使用
    public static void main(){
        Inter in = new ShiXianlei();//父类指向子类
        in.hello("nihao");
    }
}
```

## Spring AOP 和 AspectJ AOP 有什么区别？

**Spring AOP 属于运行时增强，而 AspectJ 是编译时增强。** Spring AOP 基于代理(Proxying)，而 AspectJ 基于字节码操作(Bytecode Manipulation)。

 Spring AOP 已经集成了 AspectJ  ，AspectJ  应该算的上是 Java 生态系统中最完整的 AOP 框架了。AspectJ  相比于 Spring AOP 功能更加强大，但是 Spring AOP 相对来说更简单，

如果我们的切面比较少，那么两者性能差异不大。但是，当切面太多的话，最好选择 AspectJ ，它比Spring AOP 快很多。


## Bean配置

在IOC容器中将所有的控制对象称作bean，Spring对于bean的使用有两种方式：基于spring-ioc.xml的配置和注解。
注意xml中关于bean的配置程序段

```
<bean id="oneInterface（自定义）" class="配置的实现类"></bean>
```

使用示例：

```
public void test（）{
OneInterface interface=super.getBean("oneInterface");//获取bean，不需要new对象
interface.hello();//调用函数
}
```






### 常用注入方式
Spring注入概念:指在启动Spring容器加载bean配置的时候，完成对变量的复制行为。（扫描xml文件中的bean标签时，实例化对象的同时，完成成员变量的赋值）




Spring常用的注入方式：
1. 设值注入：即通过XML中配置bean的依赖类，通过层级property属性，来配置依赖关系，然后Spring通过setter方法，来实现依赖类的注入；
2. 构造器注入：方法同设值注入，不过具体实现的方法是通过显示的创造一个构造方法，构造方法的参数名称要与XML中配置的name名称一致，XML配置的标签为constructor-arg
```
设值注入：通过一个成员变量的Set方式进行注入

<bean id="injectionService" class="com.imooc.ioc.injection.service.InjectionServiceImpl">

<property name="injectionDAO" ref="injectionDAO"/>

</bean>

<bean id="injectionDAO" class="com.imooc.ioc.injection.dao.InjectionDAOImpl">

</bean>
```



```
构造注入

<bean id="injectionService" class="com.imooc.ioc.injection.service.InjectionServiceImpl">

<constructor-arg name="injectionDAO" ref="injectionDAO"/>

</bean>

<bean id="injectionDAO" class="com.imooc.ioc.injection.dao.InjectionDAOImpl">

</bean>
```

### Spring 中的 bean 的作用域有哪些?


- BeanDefinition：描述Baen定义
- BeanDefinitionRegistry：提供向IOC容器注册BeanDefinition对象的方法

- singleton : 唯一 bean 实例，Spring 中的 bean 默认都是单例的。
- prototype : 每次请求都会创建一个新的 bean 实例。
- request : 每一次HTTP请求都会产生一个新的bean，该bean仅在当前HTTP request内有效。
- session : 每一次HTTP请求都会产生一个新的 bean，该bean仅在当前 HTTP session 内有效。
- global-session： 全局session作用域，仅仅在基于portlet的web应用中才有意义，Spring5已经没有了。Portlet是能够生成语义代码(例如：HTML)片段的小型Java Web插件。它们基于portlet容器，可以像servlet一样处理HTTP请求。但是，与 servlet 不同，每个 portlet 都有不同的会话

### Spring 中的单例 bean 的线程安全问题了解吗？

大部分时候我们并没有在系统中使用多线程，所以很少有人会关注这个问题。单例 bean 存在线程问题，主要是因为当多个线程操作同一个对象的时候，对这个对象的非静态成员变量的写操作会存在线程安全问题。

常见的有两种解决办法：

1. 在Bean对象中尽量避免定义可变的成员变量（不太现实）。

2. 在类中定义一个ThreadLocal成员变量，将需要的可变成员变量保存在 ThreadLocal 中（推荐的一种方式）。

### Spring 中的 bean 生命周期?
简单来讲
- 通过构造器或工厂方法创建Bean实例
- 为Bean的属性设置值和对其它Bean的引用
- 调用Bean的初始化方法
- Bean可以使用了
- 当容器关闭时，调用Bean的销毁方法


这部分网上有很多文章都讲到了，下面的内容整理自：<https://yemengying.com/2016/07/14/spring-bean-life-cycle/> ，除了这篇文章，再推荐一篇很不错的文章 ：<https://www.cnblogs.com/zrtqsk/p/3735273.html> 。

- Bean 容器找到配置文件中 Spring Bean 的定义。
- Bean 容器利用 Java Reflection API 创建一个Bean的实例。
- 如果涉及到一些属性值 利用 `set()`方法设置一些属性值。
- 如果 Bean 实现了 `BeanNameAware` 接口，调用 `setBeanName()`方法，传入Bean的名字。
- 如果 Bean 实现了 `BeanClassLoaderAware` 接口，调用 `setBeanClassLoader()`方法，传入 `ClassLoader`对象的实例。
- 如果Bean实现了 `BeanFactoryAware` 接口，调用 `setBeanClassLoader()`方法，传入 `ClassLoade` r对象的实例。
- 与上面的类似，如果实现了其他 `*.Aware`接口，就调用相应的方法。
- 如果有和加载这个 Bean 的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessBeforeInitialization()` 方法
- 如果Bean实现了`InitializingBean`接口，执行`afterPropertiesSet()`方法。
- 如果 Bean 在配置文件中的定义包含  init-method 属性，执行指定的方法。
- 如果有和加载这个 Bean的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessAfterInitialization()` 方法
- 当要销毁 Bean 的时候，如果 Bean 实现了 `DisposableBean` 接口，执行 `destroy()` 方法。
- 当要销毁 Bean 的时候，如果 Bean 在配置文件中的定义包含 destroy-method 属性，执行指定的方法。

图示：

![Spring Bean 生命周期](http://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-9-17/48376272.jpg)

与之比较类似的中文版本:

![Spring Bean 生命周期](http://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-9-17/5496407.jpg)

## 说说自己对于 Spring MVC 了解?

谈到这个问题，我们不得不提提之前 Model1 和 Model2 这两个没有 Spring MVC 的时代。

- **Model1 时代** : 很多学 Java 后端比较晚的朋友可能并没有接触过  Model1 模式下的 JavaWeb 应用开发。在 Model1 模式下，整个 Web 应用几乎全部用 JSP 页面组成，只用少量的 JavaBean 来处理数据库连接、访问等操作。这个模式下 JSP 即是控制层又是表现层。显而易见，这种模式存在很多问题。比如①将控制逻辑和表现逻辑混杂在一起，导致代码重用率极低；②前端和后端相互依赖，难以进行测试并且开发效率极低；
- **Model2 时代** ：学过 Servlet 并做过相关 Demo 的朋友应该了解“Java Bean(Model)+ JSP（View,）+Servlet（Controller）  ”这种开发模式,这就是早期的 JavaWeb MVC 开发模式。Model:系统涉及的数据，也就是 dao 和 bean。View：展示模型中的数据，只是用来展示。Controller：处理用户请求都发送给 ，返回数据给 JSP 并展示给用户。

Model2 模式下还存在很多问题，Model2的抽象和封装程度还远远不够，使用Model2进行开发时不可避免地会重复造轮子，这就大大降低了程序的可维护性和复用性。于是很多JavaWeb开发相关的 MVC 框架营运而生比如Struts2，但是 Struts2 比较笨重。随着 Spring 轻量级开发框架的流行，Spring 生态圈出现了 Spring MVC 框架， Spring MVC 是当前最优秀的 MVC 框架。相比于 Struts2 ， Spring MVC 使用更加简单和方便，开发效率更高，并且 Spring MVC 运行速度更快。

MVC 是一种设计模式,Spring MVC 是一款很优秀的 MVC 框架。Spring MVC 可以帮助我们进行更简洁的Web层的开发，并且它天生与 Spring 框架集成。Spring MVC 下我们一般把后端项目分为 Service层（处理业务）、Dao层（数据库操作）、Entity层（实体类）、Controller层(控制层，返回数据给前台页面)。

**Spring MVC 的简单原理图如下：**

![](https://raw.githubusercontent.com/binbinbin5/myPics/master/file/20190901101601.png)
## SpringMVC 工作原理了解吗?

**原理如下图所示：**
![SpringMVC运行原理](http://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-10-11/49790288.jpg)

上图的一个笔误的小问题：Spring MVC 的入口函数也就是前端控制器 `DispatcherServlet` 的作用是接收请求，响应结果。

**流程说明（重要）：**

1. 客户端（浏览器）发送请求，直接请求到 `DispatcherServlet`。
2. `DispatcherServlet` 根据请求信息调用 `HandlerMapping`，解析请求对应的 `Handler`。
3. 解析到对应的 `Handler`（也就是我们平常说的 `Controller` 控制器）后，开始由 `HandlerAdapter` 适配器处理。
4. `HandlerAdapter` 会根据 `Handler `来调用真正的处理器开处理请求，并处理相应的业务逻辑。
5. 处理器处理完业务后，会返回一个 `ModelAndView` 对象，`Model` 是返回的数据对象，`View` 是个逻辑上的 `View`。
6. `ViewResolver` 会根据逻辑 `View` 查找实际的 `View`。
7. `DispaterServlet` 把返回的 `Model` 传给 `View`（视图渲染）。
8. 把 `View` 返回给请求者（浏览器）

## Spring 框架中用到了哪些设计模式？

关于下面一些设计模式的详细介绍，可以看笔主前段时间的原创文章[《面试官:“谈谈Spring中都用到了那些设计模式?”。》](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485303&idx=1&sn=9e4626a1e3f001f9b0d84a6fa0cff04a&chksm=cea248bcf9d5c1aaf48b67cc52bac74eb29d6037848d6cf213b0e5466f2d1fda970db700ba41&token=255050878&lang=zh_CN#rd) 。

- **工厂设计模式** : Spring使用工厂模式通过 `BeanFactory`、`ApplicationContext` 创建 bean 对象。
- **代理设计模式** : Spring AOP 功能的实现。
- **单例设计模式** : Spring 中的 Bean 默认都是单例的。
- **模板方法模式** : Spring 中 `jdbcTemplate`、`hibernateTemplate` 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。
- **包装器设计模式** : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。
- **观察者模式:** Spring 事件驱动模型就是观察者模式很经典的一个应用。
- **适配器模式** :Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配`Controller`。
- ......

## @Component 和 @Bean 的区别是什么？

1. 作用对象不同: `@Component` 注解作用于类，而`@Bean`注解作用于方法。
2. `@Component`通常是通过类路径扫描来自动侦测以及自动装配到Spring容器中（我们可以使用 `@ComponentScan` 注解定义要扫描的路径从中找出标识了需要装配的类自动装配到 Spring 的 bean 容器中）。`@Bean` 注解通常是我们在标有该注解的方法中定义产生这个 bean,`@Bean`告诉了Spring这是某个类的示例，当我需要用它的时候还给我。
3. `@Bean` 注解比 `Component` 注解的自定义性更强，而且很多地方我们只能通过 `@Bean` 注解来注册bean。比如当我们引用第三方库中的类需要装配到 `Spring`容器时，则只能通过 `@Bean`来实现。

`@Bean`注解使用示例：

```java
@Configuration
public class AppConfig {
    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }

}
```

 上面的代码相当于下面的 xml 配置

```xml
<beans>
    <bean id="transferService" class="com.acme.TransferServiceImpl"/>
</beans>
```

下面这个例子是通过 `@Component` 无法实现的。

```java
@Bean
public OneService getService(status) {
    case (status)  {
        when 1:
                return new serviceImpl1();
        when 2:
                return new serviceImpl2();
        when 3:
                return new serviceImpl3();
    }
}
```

## 将一个类声明为Spring的 bean 的注解有哪些?

我们一般使用 `@Autowired` 注解自动装配 bean，要想把类标识成可用于 `@Autowired` 注解自动装配的 bean 的类,采用以下注解可实现：

- `@Component` ：通用的注解，可标注任意类为 `Spring` 组件。如果一个Bean不知道属于拿个层，可以使用`@Component` 注解标注。
- `@Repository` : 对应持久层即 Dao 层，主要用于数据库相关操作。
- `@Service` : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao层。
- `@Controller` : 对应 Spring MVC 控制层，主要用户接受用户请求并调用 Service 层返回数据给前端页面。

## Spring 管理事务的方式有几种？

1. 编程式事务，在代码中硬编码。(不推荐使用)
2. 声明式事务，在配置文件中配置（推荐使用）

**声明式事务又分为两种：**

1. 基于XML的声明式事务
2. 基于注解的声明式事务

## Spring 事务中的隔离级别有哪几种?

**TransactionDefinition 接口中定义了五个表示隔离级别的常量：**

- **TransactionDefinition.ISOLATION_DEFAULT:**  使用后端数据库默认的隔离级别，Mysql 默认采用的 REPEATABLE_READ隔离级别 Oracle 默认采用的 READ_COMMITTED隔离级别.
- **TransactionDefinition.ISOLATION_READ_UNCOMMITTED:** 最低的隔离级别，允许读取尚未提交的数据变更，**可能会导致脏读、幻读或不可重复读**
- **TransactionDefinition.ISOLATION_READ_COMMITTED:**   允许读取并发事务已经提交的数据，**可以阻止脏读，但是幻读或不可重复读仍有可能发生**
- **TransactionDefinition.ISOLATION_REPEATABLE_READ:**  对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读，但幻读仍有可能发生。**
- **TransactionDefinition.ISOLATION_SERIALIZABLE:**   最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，**该级别可以防止脏读、不可重复读以及幻读**。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

## Spring 事务中哪几种事务传播行为?

**支持当前事务的情况：**

- **TransactionDefinition.PROPAGATION_REQUIRED：** 如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
- **TransactionDefinition.PROPAGATION_SUPPORTS：** 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- **TransactionDefinition.PROPAGATION_MANDATORY：** 如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。（mandatory：强制性）

**不支持当前事务的情况：**

- **TransactionDefinition.PROPAGATION_REQUIRES_NEW：** 创建一个新的事务，如果当前存在事务，则把当前事务挂起。
- **TransactionDefinition.PROPAGATION_NOT_SUPPORTED：** 以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- **TransactionDefinition.PROPAGATION_NEVER：** 以非事务方式运行，如果当前存在事务，则抛出异常。

**其他情况：**

- **TransactionDefinition.PROPAGATION_NESTED：** 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。






# Spring代码演示


Spring作用将对象的创建交给Spring容器在applicationContext.xml配置文件中用bean声明要什么对象

```xml
<!--配置文件 applicationContext.xml中
//class:java类的全限定类名,他是通过全类名使用反射的技术进行创建
//id:给这个对象在整个应用程序上下文当中取一个唯一名字方便区分-->

<!--bean是注入-->
<!--bean2.xml-->
<bean id="girl" class="cn.edu.sict.pojo.girl">
    <property name="name" value="范冰冰"></property>
    <property name="age" value="23"></property>
</bean>
```
```xml
<!--bean1.xml-->
<bean id="yourgirl" class="cn.edu.sict.pojo.girl">
    <property name="name" value="李冰冰"></property>
    <property name="age" value="23"></property>
</bean>
```
```
//获取该bean

 
    @Test
    public void t1() {
    //获取上下文对象Spring里面声明对象都需要通过上下文对象获取
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
        
         /*读取多个Spring配置文件*/
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext(new String[]{"applicationContext.xml", "bean1.xml", "bean2.xml"});
        
        //指定了class，不需要强转
        girl girl = applicationContext.getBean("girl"，girl.class);
        System.out.println(girl);
        girl girl2 = applicationContext.getBean("yourgirl"，girl.class);
        System.out.println(girl2);
    }
```

对象产生在配置文件中产生的，而不需要在类中修改。




### 控制反转IOC

控制反转IOC也被称为依赖注入DI,控制既创建对象彼此关系的权利.

开始控制权在开发人员程序代码中掌控(new 的方式)

==Spring夺取控制权反转给Spring容器==，程序员只需要：
- 声明要什么
- 然后Spring容器进行具体控制

容器
![](https://raw.githubusercontent.com/binbinbin5/myPics/master/imgs/20190718130644.png)
  - pojos:自己定义的类
  - metadata:在Spring的配置文件里面写的这些就是元数据
  - 实例化容器：classpath... 将配置文件传入，实例化完毕

##### 值得注入（DI）

###### setter注入(最常用)
  -  但是属性必须有对应的setter方法才可以完成
  -  通过property 值注入

```
​    <property name="name" value="王菲"></property>
```


######  构造注入
默认通过无参构造器完成对象的创建的

如果没有无参构造器会报错：

  ```java
  //提供构造方法
    public Car(String name, Double price, Double speed) {
          this.name = name;
          this.price = price;
          this.speed = speed;
      }
      public Car(String name, Double price) {
          this.name = name;
          this.price = price;
      }
      public Car(Double price, Double speed) {
          this.price = price;
          this.speed = speed;
      }
      
```
```xml
  //构造方式一   name
  <constructor-arg name="name" value="凯迪拉克"></constructor-arg>
  <constructor-arg name="price" value="33333333"></constructor-arg>
  <constructor-arg name="speed" value="200"></constructor-arg>
  
  //构造方式二   index  会优先使用后面的构造器（不推荐）
  <bean id="car2" class="cn.edu.sict.pojo.Car">
          <constructor-arg index="0" value="2343"></constructor-arg>
          <constructor-arg index="1" value="33333333"></constructor-arg>
  </bean>
  //构造方式三  type 按照构造函数入参类型自动选择
  <bean id="car3" class="cn.edu.sict.pojo.Car">
       <constructor-arg type="java.lang.String" value="2343"></constructor-arg>
       <constructor-arg type="java.lang.Double" value="33333333"></constructor-arg>
  </bean>
  ```

##### bean元素探讨

属性探讨

- abstract=true  bean将无法实例化，用于被继承
- parent指明父bean,将会继承父bean的所有内容，通过ID进行指引 如下：

  ```
  <bean class="cn.edu.sict.pojo.Girl" id="yourgirl"></bean>
  <bean class="cn.edu.sict.pojo.Girl" id="hisgirl" parent="yourgirl"></bean>
  ```

- destroy-method:指定给bean销毁的时候执行的方法,适合做一下清理型工作,触发条件bean确实销毁，比如：
    - close
    - refreah
    - destory过时的方法

  ```java
   ((ClassPathXmlApplicationContext) applicationContext).close();  close方法销毁容器
  ((ClassPathXmlApplicationContext) applicationContext).refresh();refresh方法刷新容器
   ((ClassPathXmlApplicationContext) applicationContext).destroy();destroy销毁已过时
  ```

- init-method:初始化方法，优先执行方法，适合准备性工作。（比如数据库连接什么的），同一个bean默认单例模式，多个也只初始化一个method。

- name : 别名，可以通过他一样获取bean  (多个别名name="g1,g2 g3")支持逗号空格多种分隔符

- scope:指定范围

  -  单例模式singleton(默认) ==Spring上下文只有一个实例==
  -  原型模式：多例模式prototype  ==可以不断获取新的==多个对象，

- lazy-init:
    -   ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");这句话完成就是上下文完毕，执行构造方法，初始化完毕，bean马上注入。
    - true: 延迟初始化  初始化完成容器bean不会初始化,获取该bean初始化，getBean的时候初始化才注入获取。
    - 直接初始化程序启动慢一点，内存消耗大一点，好处是使用bean快，延迟初始化则相反启动快，内存少，使用bean慢。

- depends-on 指明一个依赖的bean，如果某一个BEAN严重依赖另一个BEAN就会配置depend on

- ref  指明一个bean

  ```java
  <bean id="girls" class="cn.edu.sict.pojo.Girl" lazy-init="true" depends-on="dog">
      <!--非字面值可以描述的属性使用ref指向bean的ID-->
      <property name="dog" ref="dog"></property>
  </bean>
  <bean id="dog" class="cn.edu.sict.pojo.Dog">
      <property name="name" value="哮天犬"></property>
  </bean>
  ```

- alias标签指定bean的别名

```java
<alias name="dog" alias="gdog"></alias>
 <bean id="dog" class="cn.edu.sict.pojo.Dog">
        <property name="name" value="哮天犬"></property>
 </bean>
  /*bean的别名*/
    @Test
    public void t7() {
        ApplicationContext applicationContext =
                new ClassPathXmlApplicationContext("bean3.xml");
        //直接通过类类型获取只有一个才可以
        Dog Girl1 = applicationContext.getBean( Dog.class);
    }
```

spring 多个配置文件中的bean是可以互相引用的（被上下文扫描到）

#### Spring中各种值的注入

简单值 直接写， 复杂就用内部BEAN。

```
public class People{
    private String name;
    private int age;
    private String[]  firend;
    private List<numbers> nums;
    private List<Cat> nums;
    private Set<Pig> pigs;
    private Map<String,User> user;
    
    ...setter,getter...
}

public class Cat{
    //名字，类型，...
}
```

##### String[]

```
//第一种可以使用逗号分隔数组元素
<bean id="people" class="cn.edu.sict.pojo.People">
    <property name="name" value="阿发"></property>
    <property name="age" value="66"></property>
    <property name="firends" value="郭富城,刘德华"></property>
</bean>


//第二种通过array标签描述数组
<bean id="people" class="cn.edu.sict.pojo.People">
    <property name="name" value="阿发"></property>
    <property name="age" value="66"></property>
    <property name="firends">
        <array>
            <value>刘德华</value>
            <value>郭富城</value>
        </array>
    </property>
</bean>
```

##### List < Integer>集合

```
<property name="nums">
    <list>
        <value>8</value>
        <value>7</value>
    </list>
</property>
```

##### List< Cat>  

                Cat为一个pojo类

```
<property name="name" value="阿发"></property>
<property name="age" value="66"></property>
<property name="cats">
    <list><!--内部bean无法引用-->
        <bean class="cn.edu.sict.pojo.Cat">
            <property name="leg" value="2"></property>
            <property name="skin" value="蓝色"></property>
        </bean>
        
        <bean class="cn.edu.sict.pojo.Cat">
            <property name="leg" value="4"></property>
            <property name="skin" value="青色"></property>
        </bean>
    </list>
</property>
```

##### Set< Pig>

```
<property name="pig">
    <set>
        <bean class="cn.edu.sict.pojo.Pig">
            <property name="name" value="小花"></property>
            <property name="sleep" value="88"></property>
            <property name="kw" value="香辣"></property>
        </bean>
        <bean class="cn.edu.sict.pojo.Pig">
            <property name="name" value="小宝"></property>
            <property name="sleep" value="88"></property>
            <property name="kw" value="酱香"></property>
        </bean>
    </set>
</property>
```

##### Map< String,User>

```
<property name="users">
    <map>
        <entry key="user1">
            <bean class="cn.edu.sict.pojo.User">
                <property name="name" value="韩雪"></property>
                <property name="address" value="梧桐村"></property>
            </bean>

        </entry>
        <entry key="user2">
            <bean class="cn.edu.sict.pojo.User">
                <property name="name" value="林青霞"></property>
                <property name="address" value="台湾"></property>
            </bean>
        </entry>
    </map>
</property>
```

### 自动驻入

```
public class user{
    priavte String name;
    priavte String address;
    priavte Pig pig;
}
```

 autowire

- byType  根据数据类型注入属性，在上下文中搜寻bean，有且只有一个注入，多个抛出异常，没有不注入。
- byName 按照bean对应pojo里面的属性名字进行匹配   private Pig pig（这边bean中 name  = "pig"）;
- constructor   
  -    优先按照类型匹配,匹配到一个直接注入,不止一个按照名字注入,名字找不到,注入失败
- default  默认值
- none  不会自动驻入

```
//byType根据类型进行注入它的属性,此时在上下文当中搜寻Pig这个bean,找到有且仅有一个的情况
//下,注入成功,一个没有,不会注入,不止一个,抛出异常.
<bean id="user" class="cn.edu.sict.pojo.User" autowire="byType">
    <property name="name" value="陈慧琳"/>
    <property name="address" value="香港"/>

</bean>
<bean class="cn.edu.sict.pojo.Pig">
    <property name="name" value="大宝"></property>
</bean>

//当有多个bean时  使用primary定义一个主次
<bean class="cn.edu.sict.pojo.Pig" primary="true">
        <property name="name" value="大宝"></property>
    </bean>
<bean class="cn.edu.sict.pojo.Pig" primary="false">
        <property name="name" value="巨大宝"></property>
</bean>
```


### Spring注解
从一个文件引入多个配置文件
```xml
<bean >
    <!--引入所有的spring-开头的xml文件-->
    <import resource = "classpath:spring/spring-*.xml">
</bean>
```



```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
    
   xmlns:task="http://www.springframework.org/schema/task"      
    xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
        http://www.springframework.org/schema/beans/spring-beans-3.2.xsd 
        http://www.springframework.org/schema/context 
        http://www.springframework.org/schema/context/spring-context-3.2.xsd 
      http://www.springframework.org/schema/tx
      http://www.springframework.org/schema/tx/spring-tx-3.2.xsd
      http://www.springframework.org/schema/aop
      http://www.springframework.org/schema/aop/spring-aop-3.2.xsd
        http://www.springframework.org/schema/mvc
      http://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd
      http://www.springframework.org/schema/task 
      http://www.springframework.org/schema/task/spring-task-3.2.xsd">
    <!-- 自动扫描的包名，如果有多个包，请使用逗号隔开 -->
    <context:component-scan base-package="com.jieyou.*" /> 

    <!-- 默认的注解映射的支持 -->
    <mvc:annotation-driven />
       <!-- 启动对aspectj的支持 -->
       <aop:aspectj-autoproxy/>
    
    
    <!-- 自动搜索指定包及其子包下的所有Bean类 -->
    <context:component-scan base-package="com.jieyou.*">
    <!-- 排除子包不扫描org.springframework.stereotype.Repository-->
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Repository">
    </context>
    
</bean>
```


- @Configuration定义配置类，可替换xml配置文件，使用AnnotationConfigApplicationContext扫描
- @ComponentScan定义**扫描的路径**从中找出标识了**需要装配**的类自动装配到spring的bean容器中
- @Component 标明一个类为Spring的一个组件,可以被Spring容器管理它是普通组件的语义
- @Service 同上语义上属于服务层
- @Repository同上语义上属于DAO层
- ＠Controller同上语义上属于控制层
- ＠ComponentScan:组件扫描，目标在哪个路径
- ＠Bean在Spring容器中注册一个bean
- ＠Autowired自动驻入组件

### AOP

applicationContext.xml
```xml
1.
<!--aop基于自动代理，启动激活-->
<aop:aspectj-autoproxy/>

2. 注册切面
<bean></bean>

3. 配置切入点信息
<aop.config></aop.config>
```


#### 简介

面向切面编程:

-    面向过程编程由起点到终点解决问题方式很直接,但是期间多次解决类似问题的过程时,期间可能会有几段的重复问题.

-    传统方式书写代码是从上到下,而AOP的重点将关注的重点将纵向变为了横向

#### 需要添加的jar包

```java
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-aop -->
        <dependency>
                  <groupId>org.springframework</groupId>
                  <artifactId>spring-aop</artifactId>
                  <version>5.1.5.RELEASE</version>
        </dependency>
    <!-- https://mvnrepository.com/artifact/org.aspectj/aspectjrt -->
        <dependency>
                  <groupId>org.aspectj</groupId>
                  <artifactId>aspectjrt</artifactId>
                  <version>1.9.1</version>
        </dependency>
    <!-- https://mvnrepository.com/artifact/org.aspectj/aspectjweaver -->
        <dependency>
                  <groupId>org.aspectj</groupId>
                  <artifactId>aspectjweaver</artifactId>
                  <version>1.9.2</version>
        </dependency>
```

#### 属性

```
import org.aspectj.lang.JoinPoint;
JointPoint可以通过该类获取AOP中的一些信息方法名等的一些参数信息
```

Aspect(切面): 通常是一个类

JointPoint(连接点):程序执行过程中明确的点，一般是方法的调用

Advice(通知):  AOP在特定的切入点上执行的增强处理:

Pointcut(切入点):   带有通知的连接点，在程序中主要体现为书写切入点表达式

匹配连接点的谓词。建议与切入点表达式相关联，并在切入点匹配的任何连接点处运行（例如，执行具有特定名称的方法）。由切入点表达式匹配的连接点的概念是AOP的核心，Spring默认使用AspectJ切入点表达式语言。

*简介*：代表类型声明其他方法或字段。Spring AOP允许您向任何建议的对象引入新接口（以及相应的实现）。例如，您可以使用简介使bean实现` IsModified` 接口，以简化缓存。（介绍被称为AspectJ社区中的类型间声明。）

*目标对象*：由一个或多个方面建议的对象。也称为 *建议*对象。由于Spring AOP是使用运行时代理实现的，因此该对象始终是 *代理*对象。

*AOP代理*：由AOP框架创建的对象，用于实现方面契约（建议方法执行等）。在Spring Framework中，AOP代理将是JDK动态代理(必须实现某个接口在接口情况下)或CGLIB代理。

织入(*Weaving*):最终以何种行为使之生效,该行为就称之为织入.

#### 增强类型

- before:  标识一个前置增强方法
- after:   标识一个 后置增强方法
- after-returning  返回结果之后增强
- after-throwing:  异常抛出增强
- around: 环绕增强，添加后需要手动让目标方法执行

#### AOP代理

##### XML方式

```xml
//执行任何公共方法：
//表示任意的类下的任意的方法的任意的参数
//execution(public * *(..))
  <!--1.aop基于代理完成,先要激活代理-->
    <aop:aspectj-autoproxy/>
    <!--2.注册一个切面具体要使用的类-->
    <bean id="beforeAdvice" class="cn.edu.sict.advice.BeforeAdvice"></bean>
    <!--3.配置切入点等等信息-->
    <aop:config>
        <aop:aspect id="beforeAspect" ref="beforeAdvice">
            <!--aop:before表明是前置通知
            method具体使用什么方法来切
            pointcut切入点-->
            <!--<aop:before method="methodBefore" pointcut="execution(* cn.edu.sict.*.*(..))"></aop:before>-->
            
            <!--
                execution(public  * *(..)) 切无参
                execution(public  * *(java.lang.String)) 切单个参数
                execution(public  * *(java.lang.String，int)) 多个（基本数据类型和包装类严格区分）
            -->
              <aop:before method="methodBefore" pointcut="execution(public  * *(..))"></aop:before>
        </aop:aspect>
    </aop:config>
 ```   
```java    
     <!--4.测试类是否增强-->
         @Test
    public void t1() {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("appliectionContext.xml");
        ProviderService providerService = ctx.getBean("providerService", ProviderService.class);
        providerService.add();
    }
```

```xml
<!--返回结果后增强，比after先执行-->
<aop:aspect ref="afterReturningAdvice">
    <aop:after-returning method="afterReturning" returning="returning" pointcut="execution(public  * *(..))"/>
</aop:aspect>


package cn.edu.sict.advice;
public class AfterReturningAdvice {
    public void afterReturning(String returning){
        System.out.println("在返回结果之后执行...");
        System.out.println("返回值:"+returning);
    }

}


package cn.edu.sict.service;
public class AfterReturningService {
    public String  msg(){
        System.out.println("执行AfterReturningService.msg()");
        return "王祖贤";
    }
}
<!--
对于增强的AfterReturningService.msg()方法可以通过returning="returning" 属性将
增强方法AfterReturningAdvice.afterReturning()中的returning和
要增强的方法AfterReturningService.msg()中return值绑定起来
returning="绑定值"要和AfterReturningAdvice.afterReturning()中的值名称一致-->
```

## 注解方式

```xml
<!-- 在applicationContext.xml配置-->
<!--开启代理-->
<aop:aspectj-autoproxy/>
<!--配置基础扫描包-->
<context:component-scan base-package="cn.edu.sict"></context:component-scan>
```

##### 多个想同类型增强类之间的执行顺序

可以再类前加入注解@Order(X)指明顺序

BeforeAdvice1

```java
@Order(1)//用于排序,不过推荐使用两个类的@Order()如下2，3
@Aspect  //标记为一个切面
@Component("BeforeAdvice1")//标记当前类为Spring的组件,相当于在XML注册一个bean
public class BeforeAdvice1 {
    /**
    @Order(2)//这样也可以一个类中两个前置切面
    @Before("execution(public * *(..))")
    public void before1() {
        System.out.println("前置通知2...");
    }*/
    @Order(1)
    @Before("execution(public * *(..))")
    public void before() {
        System.out.println("前置通知1...");
    }
    

```

BeforeAdvice2


```
@Order(2)//before和after顺序不一样
@Aspect  //标记为一个切面
@Component("BeforeAdvice2")//标记当前类为Spring的组件,相当于在XML注册一个bean
public class BeforeAdvice2 {
    
    @Before("execution(public * *(..))")
    public void before() {
        System.out.println("前置通知2...");
    }
}
```

BeforeAdvice3

```java
@Order(3)
@Aspect  //标记为一个切面
@Component("BeforeAdvice3")//标记当前类为Spring的组件,相当于在XML注册一个bean
public class BeforeAdvice3 {

    @Before("execution(public * *(..))")
    public void before() {
        System.out.println("前置通知3...");
    }
```

##### 异常抛出增强注解

```
@Aspect
@Component
public class AfterThrowingAdvice {

    @AfterThrowing(value = "execution(public * *(..))",throwing = "throwing")
    public void exe(JoinPoint joinPoint,Throwable throwing) {

        System.out.println("抛出异常执行...");
        System.out.println("异常之前"+joinPoint.getSignature());
        System.out.println("异常之后"+throwing);
    }
}
//异常增强类
```

##### 环绕增强手动让目标方法执行以及处理异常注意事项

```java
@Aspect
@Component
public class AroundAdvice {

    public void AroundAdvices() {
        System.out.println("环绕增强...");
    }
    @Around("execution(public * *(..))")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("环绕增强...");
        //作用是让目标方法执行
        Object proceed = pjp.proceed();
        return proceed;
       }
    }
}
//在处理异常时除了上述继承Throwable异常类还有  try/catch方式
public Object around(ProceedingJoinPoint pjp) {
        System.out.println("环绕增强...");
        //作用是让目标方法执行
        Object proceed = null;
        try {
            proceed = pjp.proceed();
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
        return proceed;
    }
//第一种方式遇到异常仍然会抛出
//第二种遇到异常会进入catch内不会抛出异常
//由于没有跑出异常会造成AOP中的异常增强的内容不会奏效
```


##### executeion表达式
先写访问修饰符 包的限定 类名 方法名 参数类型 + 组合条件，同事符合两个条件，多个条件和一个都可以。

```
public com.sz..*.*(java.lang.String,...)
修饰符为public的并且是在sz包下面的或者子包下面的任意类的热议的方法的一个参数为String的方法（后面可以多个参数的）就可以切到
```
# Spring相关教程/资料

### 官网相关

- [Spring官网](https://spring.io/)
- [Spring系列主要项目](https://spring.io/projects)
-  [Spring官网指南](https://spring.io/guides)
- [Spring Framework 4.3.17.RELEASE API](https://docs.spring.io/spring/docs/4.3.17.RELEASE/javadoc-api/)

## 系统学习教程

### 文档

-  [极客学院Spring Wiki](http://wiki.jikexueyuan.com/project/spring/transaction-management.html)
- [Spring W3Cschool教程 ](https://www.w3cschool.cn/wkspring/f6pk1ic8.html)

### 视频

- [网易云课堂——58集精通java教程Spring框架开发](http://study.163.com/course/courseMain.htm?courseId=1004475015#/courseDetail?tab=1&35)

- **黑马视频和尚硅谷视频（非常推荐）：


## 面试必备知识点

### SpringAOP,IOC实现原理

AOP实现原理、动态代理和静态代理、Spring IOC的初始化过程、IOC原理、自己实现怎么实现一个IOC容器？这些东西都是经常会被问到的。

推荐阅读：

- [自己动手实现的 Spring IOC 和 AOP - 上篇](http://www.coolblog.xyz/2018/01/18/自己动手实现的-Spring-IOC-和-AOP-上篇/)

- [自己动手实现的 Spring IOC 和 AOP - 下篇](http://www.coolblog.xyz/2018/01/18/自己动手实现的-Spring-IOC-和-AOP-下篇/)

### AOP

AOP思想的实现一般都是基于 **代理模式** ，在JAVA中一般采用JDK动态代理模式，但是我们都知道，**JDK动态代理模式只能代理接口而不能代理类**。因此，Spring AOP 会这样子来进行切换，因为Spring AOP 同时支持 CGLIB、ASPECTJ、JDK动态代理。

- 如果目标对象的实现类实现了接口，Spring AOP 将会采用 JDK 动态代理来生成 AOP 代理类；
- 如果目标对象的实现类没有实现接口，Spring AOP 将会采用 CGLIB 来生成 AOP 代理类——不过这个选择过程对开发者完全透明、开发者也无需关心。

推荐阅读：

- [静态代理、JDK动态代理、CGLIB动态代理讲解](http://www.cnblogs.com/puyangsky/p/6218925.html) ：我们知道AOP思想的实现一般都是基于 **代理模式** ，所以在看下面的文章之前建议先了解一下静态代理以及JDK动态代理、CGLIB动态代理的实现方式。
- [Spring AOP 入门](https://juejin.im/post/5aa7818af265da23844040c6) ：带你入门的一篇文章。这篇文章主要介绍了AOP中的基本概念：5种类型的通知（Before，After，After-returning，After-throwing，Around）；Spring中对AOP的支持：AOP思想的实现一般都是基于代理模式，在Java中一般采用JDK动态代理模式，Spring AOP 同时支持 CGLIB、ASPECTJ、JDK动态代理，
- [Spring AOP 基于AspectJ注解如何实现AOP](https://juejin.im/post/5a55af9e518825734d14813f) ： **AspectJ是一个AOP框架，它能够对java代码进行AOP编译（一般在编译期进行），让java代码具有AspectJ的AOP功能（当然需要特殊的编译器）**，可以这样说AspectJ是目前实现AOP框架中最成熟，功能最丰富的语言，更幸运的是，AspectJ与java程序完全兼容，几乎是无缝关联，因此对于有java编程基础的工程师，上手和使用都非常容易。Spring注意到AspectJ在AOP的实现方式上依赖于特殊编译器(ajc编译器)，因此Spring很机智回避了这点，转向采用动态代理技术的实现原理来构建Spring AOP的内部机制（动态织入），这是与AspectJ（静态织入）最根本的区别。**Spring 只是使用了与 AspectJ 5 一样的注解，但仍然没有使用 AspectJ 的编译器，底层依是动态代理技术的实现，因此并不依赖于 AspectJ 的编译器**。 Spring AOP虽然是使用了那一套注解，其实实现AOP的底层是使用了动态代理(JDK或者CGLib)来动态植入。至于AspectJ的静态植入，不是本文重点，所以只提一提。
- [探秘Spring AOP（慕课网视频，很不错）](https://www.imooc.com/learn/869):慕课网视频，讲解的很不错，详细且深入
- [spring源码剖析（六）AOP实现原理剖析](https://blog.csdn.net/fighterandknight/article/details/51209822) :通过源码分析Spring AOP的原理

### IOC

Spring IOC的初始化过程：
![Spring IOC的初始化过程](https://user-gold-cdn.xitu.io/2018/5/22/16387903ee72c831?w=709&h=56&f=png&s=4673)

- [[Spring框架]Spring IOC的原理及详解。](https://www.cnblogs.com/wang-meng/p/5597490.html)

- [Spring IOC核心源码学习](https://yikun.github.io/2015/05/29/Spring-IOC核心源码学习/) :比较简短，推荐阅读。
- [Spring IOC 容器源码分析](https://javadoop.com/post/spring-ioc) :强烈推荐，内容详尽，而且便于阅读。

## Spring事务管理

- [可能是最漂亮的Spring事务管理详解](https://juejin.im/post/5b00c52ef265da0b95276091)
- [Spring编程式和声明式事务实例讲解](https://juejin.im/post/5b010f27518825426539ba38)

### Spring单例与线程安全

- [Spring框架中的单例模式（源码解读）](http://www.cnblogs.com/chengxuyuanzhilu/p/6404991.html):单例模式是一种常用的软件设计模式。通过单例模式可以保证系统中一个类只有一个实例。spring依赖注入时，使用了 多重判断加锁 的单例模式。

### Spring源码阅读

阅读源码不仅可以加深我们对Spring设计思想的理解，提高自己的编码水品，还可以让自己在面试中如鱼得水。下面的是Github上的一个开源的Spring源码阅读，大家有时间可以看一下，当然你如果有时间也可以自己慢慢研究源码。

 - [spring-core](https://github.com/seaswalker/Spring/blob/master/note/Spring.md)
- [spring-aop](https://github.com/seaswalker/Spring/blob/master/note/spring-aop.md)
- [spring-context](https://github.com/seaswalker/Spring/blob/master/note/spring-context.md)
- [spring-task](https://github.com/seaswalker/Spring/blob/master/note/spring-task.md)
- [spring-transaction](https://github.com/seaswalker/Spring/blob/master/note/spring-transaction.md)
- [spring-mvc](https://github.com/seaswalker/Spring/blob/master/note/spring-mvc.md)
- [guava-cache](https://github.com/seaswalker/Spring/blob/master/note/guava-cache.md)