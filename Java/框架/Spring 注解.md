[toc]
## 注解



### 组件类注解

思考：Spring怎么知道应该把哪些Java类当成bean注册到容器中呢？

答案：使用配置文件或者注解的方式进行标识需要处理的java类!

**1、注解类介绍**

- @Component ：标注一个普通的spring Bean类。

- @Repository：标注一个DAO组件类。

- @Service：标注一个业务逻辑组件类。

- @Controller：标注一个控制器组件类。

这些都是注解在平时的开发过程中出镜率极高，@Component、@Repository、@Service、@Controller实质上属于同一类注解，用法相同，功能相同，区别在于标识组件的类型。@Component可以代替@Repository、@Service、@Controller，因为这三个注解是被@Component标注的。如下代码

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Controller {
    String value() default "";
}
```

**2、举例详解**
 （1）当一个组件代表数据访问层（DAO）的时候，我们使用@Repository进行注解，如下

```
@Repository
public class HappyDaoImpl implements HappyDao{
private final static Logger LOGGER = LoggerFactory.getLogger(HappyDaoImpl .class);
public void  club(){
        //do something ,like drinking and singing
    }
}
```

（2）当一个组件代表业务层时，我们使用@Service进行注解，如下

```
@Service(value="goodClubService")
//使用@Service注解不加value ,默认名称是clubService
public class ClubServiceImpl implements ClubService {
    @Autowired
    private ClubDao clubDao;

    public void doHappy(){
        //do some Happy
    }
 }
```

（3）当一个组件作为前端交互的控制层，使用@Controller进行注解，如下

```
@Controller
public class HappyController {
    @Autowired //下面进行讲解
    private ClubService clubService;

    // Control the people entering the Club
    // do something
}
/*Controller相关的注解下面进行详细讲解，这里简单引入@Controller*/
```

**3、总结注意点**

1.  被注解的java类当做Bean实例，Bean实例的名称默认是Bean类的首字母小写，其他部分不变。@Service也可以自定义Bean名称，但是必须是唯一的！

2. 尽量使用对应组件注解的类替换@Component注解，在spring未来的版本中，@Controller，@Service，@Repository会携带更多语义。并且便于开发和维护！

3. 指定了某些类可作为Spring Bean类使用后，最好还需要让spring搜索指定路径，在Spring配置文件加入如下配置：

```
<!-- 自动扫描指定包及其子包下的所有Bean类 -->
<context:component-scan base-package="org.springframework.*"/>
```

### 装配bean时常用的注解

1、注解介绍

- @Autowired：属于Spring 的org.springframework.beans.factory.annotation包下,可用于为类的属性、构造器、方法进行注值
- @Resource：不属于spring的注解，而是来自于JSR-250位于java.annotation包下，使用该annotation为目标bean指定协作者Bean
- @PostConstruct 和 @PreDestroy 方法 实现初始化和销毁bean之前进行的操作

2、举例说明
 （1）：@Autowired

这里`@Autowired`注解的意思就是，当Spring发现`@Autowired`注解时，将自动在代码上下文中找到与其匹配（默认是类型匹配）的Bean，并自动注入到相应的地方去。

```java
public class School{
        private Teacher teacher;
        private Student student;
        public void setTeacher(Teacher teacher){
            this.teacher = teacher;
        }
        public void setStudent(Student student){
            this.student = student;
        }
        public Teacher getTeacher(){
            return teacher;
        }
        public Student getStudent(){
            return student;
        }
        public String toString(){
            return teacher + "\n" + student;
        }
    }

public class School{
        @Autowired
        private Teacher teacher;
        @Autowired
        private Student student;
        public String toString(){
            return teacher + "\n" + student;
        }
    }
```

因为，`@Autowired`注解要去寻找的是一个Bean，Teacher和 Student的Bean定义都给去掉了，自然就不是一个Bean了，Spring容器找不到也很好理解。那么，如果属性找不到又不想让Spring容器抛出异常，而就是显示null，可以吗？可以，其实异常信息里面也给出了提示，就是将`@Autowired`注解的required属性设置为false 即可：

```java
public class School{
        @Autowired(required = false)
        private Teacher teacher;
        @Autowired(required = false)
        private Student student;
        public String toString(){
            return teacher + "\n" + student;
        }
    }
```

@Autowired接口注入----->@Qualifier

上面仅仅只是注入一个Java类，比较简单。那么如果有一个接口，有多个实现，Bean里引用的是接口名，又该怎么做呢？比如有一个Person接口：

```
public interface Person{
        public String personName();
    }
```

两个实现类Doctor和Police：

```
    @Service
    public class Doctor implements Person{
        public String docName(){
            return “Doctor person”;
        }
    }

    @Service
    public class Police implements Person{
        public String personName(){
            return “Police person”;
        }
    }
```

写一个PersonFactory，引用Person：

```
    @Service
    public class PersonFactory{
        @Autowired
        private Person person;
        public String toString(){
            return person.personName();
        }
    }
```

Person接口有两个实现类，Spring并不知道应当引用哪个实现类，代码报错。这种情况通常有两个解决办法：

1. 删除其中一个实现类，Spring会自动去base-package下寻找Person接口的实现类，发现Person接口只有一个实现类，便会直接引用这个实现类。
2. 实现类就是有多个该怎么办？此时可以使用@Qualifier注解：

```
@Service
public class PersonFactory{
    @Autowired
    @Qualifier("Doctor")
    private Person person;
    public String toString(){
        return person.personName();
    }
}
```

注意@Qualifier注解括号里面的必须是Person接口实现类的类名，不是bean的名字。写成"doctor"，结果会报错。



2）：@Resource

```java
@Service
public class School{
    @Resource(name = "teacher")
    private Teacher teacher;
    @Resource(type = Student.class)
    private Student student;
    public String toString(){
        return teacher + "\n" + student;
    }
}



```

这是详细一些的用法，说一下@Resource的装配顺序：

- @Resource后面没有任何内容，默认通过name属性去匹配bean，找不到再按type去匹配。
- 指定了name或者type则根据指定的类型去匹配bean。
- 指定了name和type则根据指定的name和type去匹配bean，任何一个不匹配都会报错。

 3、总结

（1）：相同点
 @Resource的作用相当于@Autowired，均可标注在字段或属性的setter方法上。
 （2）：不同点
 a：**提供方** @Autowired是Spring的注解，@Resource是javax.annotation注解，而是来自于JSR-250，J2EE提供，需要JDK1.6及以上。
 b ：**注入方式** @Autowired只按照Type 注入；@Resource默认按Name自动注入，也提供按照Type 注入；
 c：**属性**
 @Autowired注解可用于为类的属性、构造器、方法进行注值。默认情况下，其依赖的对象必须存在（bean可用），如果需要改变这种默认方式，可以设置其required属性为false。
 还有一个比较重要的点就是，@Autowired注解**默认按照类型装配**，如果容器中包含多个同一类型的Bean，那么启动容器时会报找不到指定类型bean的异常，解决办法是结合**@Qualified**注解进行限定，指定注入的bean名称。
 @Resource有两个中重要的属性：name和type。name属性指定byName，如果没有指定name属性，当注解标注在字段上，即默认取字段的名称作为bean名称寻找依赖对象，当注解标注在属性的setter方法上，即默认取属性名作为bean名称寻找依赖对象。
 需要注意的是，@Resource如果没有指定name属性，并且按照默认的名称仍然找不到依赖对象时， @Resource注解会回退到按类型装配。但一旦指定了name属性，就只能按名称装配了。  d：@Resource注解的使用性更为灵活，可指定名称，也可以指定类型 ；@Autowired注解进行装配容易抛出异常，特别是装配的bean类型有多个的时候，而解决的办法是需要在增加@Qualitied进行限定。

[Spring中 @Autowired注解与@Resource注解的区别](http://qiangmzsx.blog.51cto.com/2052549/1359952)

注意点：使用@Resource也要注意添加配置文件到Spring，如果没有配置component-scan

```
<context:component-scan base-package="com.zxt" />
```

则一定要配置 annotation-config

```
<context:annotation-config/>
```

### @Component vs @Configuration and @Bean

1、简单介绍

Spring的官方团队说@Component可以替代 @Configuration注解，事实上我们看源码也可以发现看到，如下

```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component  //看这里！！！
public @interface Configuration {
    String value() default "";
```

虽然说可以替代但是两个注解之间还是有区别的！

Bean注解主要用于方法上，有点类似于工厂方法，当使用了@Bean注解，我们可以连续使用多种定义bean时用到的注解，譬如用@Qualifier注解定义工厂方法的名称，用@Scope注解定义该bean的作用域范围，譬如是singleton还是prototype等。

Spring 中新的 Java 配置支持的核心就是@Configuration 注解的类。这些类主要包括 @Bean 注解的方法来为 Spring 的 IoC 容器管理的对象定义实例，配置和初始化逻辑。

使用@Configuration 来注解类表示类可以被 Spring 的 IoC 容器所使用，作为 bean 定义的资源。

```
@Configuration
public class AppConfig {
    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```

这和 Spring 的 XML 文件中的非常类似

```
<beans>
    <bean id="myService" class="com.acme.services.MyServiceImpl"/>
</beans>
```

@Bean 注解扮演了和元素相同的角色。

2、举例说明@Component 和 @Configuration

```
@Configuration
public static class Config {

    @Bean
    public SimpleBean simpleBean() {
        return new SimpleBean();
    }

    @Bean
    public SimpleBeanConsumer simpleBeanConsumer() {
        return new SimpleBeanConsumer(simpleBean());
    }
}
@Component
public static class Config {

    @Bean
    public SimpleBean simpleBean() {
        return new SimpleBean();
    }

    @Bean
    public SimpleBeanConsumer simpleBeanConsumer() {
        return new SimpleBeanConsumer(simpleBean());
    }
}
```

第一个代码正常工作，正如预期的那样，SimpleBeanConsumer将会得到一个单例SimpleBean的链接。第二个配置是完全错误的，因为Spring会创建一个SimpleBean的单例bean，但是SimpleBeanConsumer将获得另一个SimpleBean实例（也就是相当于直接调用new SimpleBean() ，这个bean是不归Spring管理的），既new SimpleBean() 实例是Spring上下文控件之外的。

3、原因总结

使用@ configuration，所有标记为@ bean的方法将被包装成一个CGLIB包装器，它的工作方式就好像是这个方法的第一个调用，那么原始方法的主体将被执行，最终的对象将在spring上下文中注册。所有进一步的调用只返回从上下文检索的bean。
 在上面的第二个代码块中，新的SimpleBeanConsumer(simpleBean())只调用一个纯java方法。为了纠正第二个代码块，我们可以这样做

```
@Component
public static class Config {
    @Autowired
    SimpleBean simpleBean;

    @Bean
    public SimpleBean simpleBean() {
        return new SimpleBean();
    }

    @Bean
    public SimpleBeanConsumer simpleBeanConsumer() {
        return new SimpleBeanConsumer(simpleBean);
    }
}
```

[Spring @Configuration vs @Component](http://dimafeng.com/2015/08/29/spring-configuration_vs_component/)
 [基本概念：@Configuration 和@Bean](https://wizardforcel.gitbooks.io/spring-doc-3x/content/28.html)

### spring MVC模块注解

1、web模块常用到的注解

- @Controller ：表明该类会作为与前端作交互的控制层组件，通过服务接口定义的提供访问应用程序的一种行为，解释用户的输入，将其转换成一个模型然后将试图呈献给用户。

```
@Controller
public class HappyController {
    //do something
...
}
```

Spring MVC 使用 @Controller 定义控制器，它还允许自动检测定义在类路径下的组件（配置文件中配置扫描路径）并自动注册。

@RequestMapping ： 这个注解用于将url映射到整个处理类或者特定的处理请求的方法。可以只用通配符！

```
@Controller
@RequestMapping("/happy")
public class HappyController  {

  @Autowired
  private HappyService happyService;

  @RequestMapping(/hello/*)
  public void sayHello(){
        //请求为 /happy/hello/* 都会进入这个方法！
        //例如：/happy/hello/123   /happy/hello/adb
        //可以通过get/post 请求
  }
  @RequestMapping(value="/haha",method=RequestMethod.GET)
  public void sayHaHa(){
  //只能通过get请求
  }
...
}
```

@RequestMapping 既可以作用在类级别，也可以作用在方法级别。当它定义在类级别时，标明该控制器处理所有的请求都被映射到 /favsoft 路径下。@RequestMapping中可以使用 method 属性标记其所接受的方法类型，如果不指定方法类型的话，可以使用 HTTP GET/POST 方法请求数据，但是一旦指定方法类型，就只能使用该类型获取数据。

@RequestParam ：将请求的参数绑定到方法中的参数上，有required参数，默认情况下，required=true，也就是改参数必须要传。如果改参数可以传可不传，可以配置required=false。

```
@RequestMapping("/happy")
  public String sayHappy(
  @RequestParam(value = "name", required = false) String name,
  @RequestParam(value = "age", required = true) String age) {
  //age参数必须传 ，name可传可不传
  ...
  }
```

@PathVariable ： 该注解用于方法修饰方法参数，会将修饰的方法参数变为可供使用的uri变量（可用于动态绑定）。

```
@RequestMapping(value="/happy/{dayid}",method=RequestMethod.GET)
public String findPet(@PathVariable String dayid, Model mode) {
//使用@PathVariable注解绑定 {dayid} 到String dayid
}
```

@PathVariable中的参数可以是任意的简单类型，如int, long, Date等等。Spring会自动将其转换成合适的类型或者抛出 TypeMismatchException异常。当然，我们也可以注册支持额外的数据类型。
 @PathVariable支持使用正则表达式，这就决定了它的超强大属性，它能在路径模板中使用占位符，可以设定特定的前缀匹配，后缀匹配等自定义格式。

@RequestBody ： @RequestBody是指方法参数应该被绑定到HTTP请求Body上。

```
@RequestMapping(value = "/something", method = RequestMethod.PUT)
public void handle(@RequestBody String body，@RequestBody User user){
   //可以绑定自定义的对象类型
}
```

@ResponseBody ： @ResponseBody与@RequestBody类似，它的作用是将返回类型直接输入到HTTP response body中。
 @ResponseBody在输出JSON格式的数据时，会经常用到。

```
@RequestMapping(value = "/happy", method =RequestMethod.POST)
@ResponseBody
public String helloWorld() {    
return "Hello World";//返回String类型
}
```

- @RestController ：控制器实现了REST的API，只为服务于JSON，XML或其它自定义的类型内容，@RestController用来创建REST类型的控制器，与@Controller类型。@RestController就是这样一种类型，它避免了你重复的写@RequestMapping与@ResponseBody。

- @ModelAttribute ：@ModelAttribute可以作用在方法或方法参数上，当它作用在方法上时，标明该方法的目的是添加一个或多个模型属性（model attributes）。
   该方法支持与@RequestMapping一样的参数类型，但并不能直接映射成请求。控制器中的@ModelAttribute方法会在@RequestMapping方法调用之前而调用。

  @ModelAttribute方法有两种风格：一种是添加隐形属性并返回它。另一种是该方法接受一个模型并添加任意数量的模型属性。用户可以根据自己的需要选择对应的风格。

### Spring事务模块注解
 1、常用到的注解

在处理dao层或service层的事务操作时，譬如删除失败时的回滚操作。使用**@Transactional** 作为注解，但是需要在配置文件激活

```
<!-- 开启注解方式声明事务 -->
    <tx:annotation-driven transaction-manager="transactionManager" />
```

2、举例

```
@Service
public class CompanyServiceImpl implements CompanyService {
  @Autowired
  private CompanyDAO companyDAO;

  @Transactional(propagation = Propagation.REQUIRED, readOnly = false, rollbackFor = Exception.class)
  public int deleteByName(String name) {

    int result = companyDAO.deleteByName(name);
    return result;
  }
  ...
}
```

 3、总结

事务的传播机制和隔离机制比较重要！

| 事务传播行为类型          | 说明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| PROPAGATION_REQUIRED      | 如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。这是最常见的选择。 |
| PROPAGATION_SUPPORTS      | 支持当前事务，如果当前没有事务，就以非事务方式执行。         |
| PROPAGATION_MANDATORY     | 使用当前的事务，如果当前没有事务，就抛出异常。               |
| PROPAGATION_REQUIRES_NEW  | 新建事务，如果当前存在事务，把当前事务挂起。                 |
| PROPAGATION_NOT_SUPPORTED | 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。   |
| PROPAGATION_NEVER         | 以非事务方式执行，如果当前存在事务，则抛出异常               |
| PROPAGATION_NESTED        | 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类 似的操作 |

------

[一图学习 Spring事务传播性](http://ju.outofmemory.cn/entry/113944)

------

readOnly : 事务的读写属性，取true或者false，true为只读、默认为false
 rollbackFor : 回滚策略，当遇到指定异常时回滚。譬如上例遇到异常就回滚
 timeout （补充的） : 设置超时时间，单位为秒
 isolation : 设置事务隔离级别，枚举类型，一共五种

| 类型                | 说明                                               |
| ------------------- | -------------------------------------------------- |
| DEFAULT             | 采用数据库默认隔离级别                             |
| READ_UNCOMMITTED    | 读未提交的数据（会出现脏读取）                     |
| READ_COMMITTED      | 读已提交的数据（会出现幻读，即前后两次读的不一样） |
| REPEATABLE_READ     | 可重复读，会出现幻读                               |
| SERIALIZABLE 串行化 | （对资源消耗较大，一般不使用）                     |

