## DI

### 依赖注入（Dependency Inversion）DI介绍

DI举例：设计行李箱

这里有一个依赖关系，行李箱以来箱体，箱体依赖于底盘，底盘依赖于轮子。这样的设计看起来可行，但是可维护性却很低。如果再后期，修改需求要求轮子的尺寸都改大一码，这样会很麻烦，轮子的尺寸一修改，底盘的尺寸就会修改。往上类推，箱体和行李箱都要改。

![](https://raw.githubusercontent.com/wfc1994/Figurebed/master/img/20190918142352.png)

这里就是系统实现部分，类似于这样的设计，几乎是不可维护的，当底层类需要修改时，就需要修改所有的类。

![](https://raw.githubusercontent.com/wfc1994/Figurebed/master/img/20190918143321.png)

因此我们要进行控制反转，要让下层控制上层，用依赖注入DI来实现

含义：把底层类作为参数传递给上层类，实现上层对下层的“控制”。

![](https://raw.githubusercontent.com/wfc1994/Figurebed/master/img/20190918184745.png)

很显然这种实现方式，只需要修改底层的Tire类就好，更容易维护。

![](https://raw.githubusercontent.com/wfc1994/Figurebed/master/img/20190918190229.png)

### DI和DL区别

IOC除了DI来实现，还有用DL（依赖查找）来实现，EJB就是用DL来实现的。相比于DI来说，他是更为主动的方法，在需要时通过调用框架提供的方法来获取对象，获取时需要提供相关的配置文件路径、key等信息来确定获取对象的状态，DL需要用户自己去使用API进行查找资源和组装对象，极有侵入性。DI是Spring和Google GUI使用的方式，负责组件的装配，是当今IOC的主流实现。

![](https://raw.githubusercontent.com/wfc1994/Figurebed/master/img/20190918190724.png)

### 依赖注入的方式

- Setter：实现特定属性的public setter方法，让IOC容器调用注入所依赖类型的对象。
- Interface：实现特定的接口，以供IOC容器注入所依赖类型的对象。
- Constructor：实现特定参数的构造函数，在创建对象时，让IOC容器注入所依赖类型的对象。
- Annotation：基于Java的注解机制，让IOC容器注入所依赖类型的对象。

## IOC

Inversion of Control:控制反转

IOC本身并非一种技术，而是一种思想。从繁琐的对象交互中解脱出来，继而专注于对象本身，更能突出面向对象。

要了解IOC就需要先了解依赖注入（Dependency Inversion）DI

## 依赖倒置原则、IOC、DI、IOC容器的关系

依赖倒置原则是一种思想，大致含义是高层模块不应该依赖于低层模块，两者都依赖于抽象。在这个原则的指导下，给IOC提供了思路。要实现IOC，则离不开DI方法的支撑。Spring这个框架是基于IOC才提出来IOC容器的概念，容器管理着Bean的生命周期，控制Bean的依赖注入。

上图中右边模块new的实例就是IOC容器所做的事。

### IOC容器的优势

- 避免在各处使用new来创建类，并且可以做到统一维护
- 创建实例的时候不需要了解其中的细节

在上面的例子中，我们在创建一个Luggage的实例时，是从底层往上层去new的，我们需要去了解每个类的构造函数是怎么定义的，才能一步步的去注入。

而IOC容器的工作则是反过来的，从一开始往下去查找依赖关系，到达最底层时，再一步步往上去new。

![](https://raw.githubusercontent.com/wfc1994/Figurebed/master/img/20190919082302.png)

通过IOC容器可以隐藏具体的创建实例的细节。我们只要像这个“工厂”请求一个Luggage实例，工厂就会帮我们按照config去创建一个Luggage实例，并返回给我们，不需要去管这个Luggage实例是如何一步步创建出来的。

假如在实际项目中，Server Class可能有几百个类的底层，假设我们新写了一个API，需要实例化这个Service，我们总不能去搞清楚这几百个类的构造函数，IOC容器就很完美的解决了这个问题。 只要在配置文件中注入Service就好了，提高了可维护性。

![](https://raw.githubusercontent.com/wfc1994/Figurebed/master/img/20190919082804.png)


### IOC容器应用流程

![](https://raw.githubusercontent.com/wfc1994/Figurebed/master/img/20190919090807.png)

支持的功能

- 依赖注入
- 依赖检查
- 自动装配
- 支持集合
- 指定初始化方法和销毁方法
- 支持回调方法


Spring IOC容器的核心接口

- BeanFactory
- ApplicationContext

### BeanFactory

它是Spring框架最核心的接口

- 提供IOC的配置机制
- 包含Bean的各种定义，便于实例化Bean
- 建立Bean之间的依赖关系
- Bean生命周期的控制

getBean方法的代码逻辑

- 转换beanName
- 从缓存加载实例
- 实例化Bean
- 检测parentBeanFactory
- 初始化依赖的Bean
- 创建Bean

### ApplicationContext

两者比较

- BeanFactory是Spring框架的基础设施，面向Spring
- ApplicationContext面向使用Spring框架的开发者

ApplicationContext功能

- BeanFactory：能够管理和装配Bean
- ResourcePatternResolver：能够加载资源文件
- MessageSource：能够实现国际化等功能
- ApplicationEventPublisher：能够注册监听器，实现监听机制


## 面试相关

### Spring Bean的作用域

- singleton：Spring的默认作用域，容器里拥有唯一的Bean实例
- prototype：针对每个getBean请求，容器都会创建一个Bean实例
- request：会为每个Http请求创建一个Bean实例
- session：会为每个session创建一个Bean实例
- globalSession：会为每个全局Http Session创建一个Bean实例

### Spring Bean的生命周期

Spring的Bean主要分为两个时期，创建和销毁

创建过程

![](https://raw.githubusercontent.com/wfc1994/Figurebed/master/img/20190919102854.png)

3、在Spring完成实例化之后，对实例化的Bean添加自定义的处理逻辑
4、做一些属性被设定后的自定义的事
5、完成Bean实例化后自定义处理的工作

销毁过程

- 若实现了DisposableBean接口，会调用destroy方法
- 若配置了destry-method属性，则会调用其配置的销毁方法
