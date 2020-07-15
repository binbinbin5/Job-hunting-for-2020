[toc]

## 介绍


**1. 引言**



在 Java 8 以前，若我们想要把某些功能传递给某些方法，总要去写匿名类。以前注册事件监听器的写法与下面的示例代码就很像：




```
manager.addScheduleListener(new ScheduleListener() {

​    @Override
​    public void **onSchedule**(ScheduleEvent e) {        
​        // Event listener implementation goes here...
​    }

});
```



这里我们添加了一些自定义代码到 Schedule 监听器中，需要先定义匿名内部类，然后传递一些功能到 onSchedule 方法中。



正是 Java 在作为参数传递普通方法或功能的限制，Java 8 增加了一个全新语言级别的功能，称为 **Lambda 表达式**。



**2. 为什么 Java 需要 Lambda 表达式**



Java 是**面向对象**语言，除了原始数据类型之处，Java 中的所有内容都是一个对象。而在**函数式**语言中，我们只需要给函数分配变量，并将这个函数作为参数传递给其它函数就可实现特定的功能。JavaScript 就是功能编程语言的典范（闭包）。



Lambda 表达式的加入，使得 Java 拥有了函数式编程的能力。在其它语言中，Lambda 表达式的类型是一个函数；但在 Java 中，Lambda 表达式被表示为对象，因此它们必须绑定到被称为**功能接口**的特定对象类型。



**3. Lambda 表达式简介**

Lambda 表达式是一个匿名函数（对于 Java 而言并不很准确，但这里我们不纠结这个问题）。简单来说，这是一种没有声明的方法，即没有访问修饰符，返回值声明和名称。

在仅使用一次方法的地方特别有用，方法定义很短。它为我们节省了，如包含类声明和编写单独方法的工作。



Java 中的 Lambda 表达式通常使用语法是 (argument) -> (body)，比如：


```
(arg1, arg2...) -> { body }

(type1 arg1, type2 arg2...) -> { body }

以下是 Lambda 表达式的一些示例：

(int a, int b) -> {  return a + b; }
() -> System.out.println("Hello World");
(String s) -> { System.out.println(s); }
() -> 42
() -> { return 3.1415 };
```



**3.1 Lambda 表达式的结构**



Lambda 表达式的结构：

- Lambda 表达式可以具有零个，一个或多个参数。
- 可以显式声明参数的类型，也可以由编译器自动从上下文推断参数的类型。例如 (int a) 与刚才相同 (a)。
- 参数用小括号括起来，用逗号分隔。例如 (a, b) 或 (int a, int b) 或 (String a, int b, float c)。
- 空括号用于表示一组空的参数。例如 () -> 42。
- 当有且仅有一个参数时，如果不显式指明类型，则不必使用小括号。例如 a -> return a*a。
- Lambda 表达式的正文可以包含零条，一条或多条语句。
- 如果 Lambda 表达式的正文只有一条语句，则大括号可不用写，且表达式的返回值类型要与匿名函数的返回类型相同。
- 如果 Lambda 表达式的正文有一条以上的语句必须包含在大括号（代码块）中，且表达式的返回值类型要与匿名函数的返回类型相同。

**4. 方法引用**



**4.1 从 Lambda 表达式到双冒号操作符**



使用 Lambda 表达式，我们已经看到代码可以变得非常简洁。



例如，要创建一个比较器，以下语法就足够了




```
Comparator c = (Person p1, Person p2) -> p1.getAge().compareTo(p2.getAge());
```




然后，使用类型推断：




```
Comparator c = (p1, p2) -> p1.getAge().compareTo(p2.getAge());
```




但是，我们可以使上面的代码更具表现力和可读性吗？我们来看一下：




```
Comparator c = Comparator.comparing(Person::getAge);
```




使用 :: 运算符作为 Lambda 调用特定方法的缩写，并且拥有更好的可读性。



**4.2 使用方式**



双冒号（::）操作符是 Java 中的**方法引用**。当们使用一个方法的引用时，目标引用放在 :: 之前，目标引用提供的方法名称放在 :: 之后，即 目标引用::方法。比如：




```
Person::getAge;
```




在 Person 类中定义的方法 getAge 的方法引用。


然后我们可以使用 Function 对象进行操作：




```
// 获取 getAge 方法的 Function 对象
Function<Person, Integer> getAge = Person::getAge;

// 传参数调用 getAge 方法
Integer age = getAge.apply(p);
```



我们引用 getAge，然后将其应用于正确的参数。



目标引用的参数类型是 Function<T,R>，T 表示传入类型，R 表示返回类型。比如，表达式 person -> person.getAge();，传入参数是 person，返回值是 person.getAge()，那么方法引用 Person::getAge 就对应着 Function<Person,Integer> 类型。



**5. 什么是功能接口（Functional interface）**



在 Java 中，功能接口（Functional interface）指**只有一个**抽象方法的接口。



java.lang.Runnable 是一个功能接口，在 Runnable 中只有一个方法的声明 void run()。我们使用匿名内部类实例化功能接口的对象，而使用 Lambda 表达式，可以简化写法。



每个 Lambda 表达式都可以隐式地分配给功能接口。例如，我们可以从 Lambda 表达式创建 Runnable 接口的引用，如下所示：




```
Runnable r = () -> System.out.println("hello world");
```




当我们不指定功能接口时，这种类型的转换会被编译器自动处理。例如：




```
new Thread(
​    () -> System.out.println("hello world")
).start();
```



在上面的代码中，编译器会自动推断，Lambda 表达式可以从 Thread 类的构造函数签名（public Thread(Runnable r) { }）转换为 Runnable 接口。



@FunctionalInterface 是在 Java 8 中添加的一个新注解，用于指示接口类型，声明接口为 Java 语言规范定义的功能接口。Java 8 还声明了 Lambda 表达式可以使用的功能接口的数量。当您注释的接口不是有效的功能接口时， @FunctionalInterface 会产生编译器级错误。



以下是自定义功能接口的示例：




```
package com.wuxianjiezh.demo.lambda;


@FunctionalInterface
public interface WorkerInterface {
​    public void doSomeWork();
}
```




正如其定义所述，功能接口只能有一个抽象方法。如果我们尝试在其中添加一个抽象方法，则会抛出编译时错误。例如：




```
package com.wuxianjiezh.demo.lambda;

@FunctionalInterface
public interface WorkerInterface {
​    public void doWork();
​    public void doMoreWork();

}
```



错误：



Error:(3, 1) java: 意外的 @FunctionalInterface 注释



  com.wuxianjiezh.demo.lambda.WorkerInterface 不是函数接口



​    在 接口 com.wuxianjiezh.demo.lambda.WorkerInterface 中找到多个非覆盖抽象方法



一旦定义了功能接口，我们就可以利用 Lambda 表达式调用。例如：




```
package com.wuxianjiezh.demo.lambda;



@FunctionalInterface
public interface WorkerInterface {
​    public void doWork();



}



class WorkTest {

​    public static void **main**(String[] args) {
​        // 通过匿名内部类调用
​        WorkerInterface work = new WorkerInterface() {
​            @Override
​            public void doWork() {
​                System.out.println("通过匿名内部类调用");
​            }
​        };



​        work.doWork();
​        // 通过 Lambda 表达式调用
​        // Lambda 表达式实际上是一个对象。
         // 我们可以将 Lambda 表达式赋值给一个变量，就可像其它对象一样调用。
​        work = ()-> System.out.println("通过 Lambda 表达式调用");
​        work.doWork();
​    }

}
```




运行结果：



通过匿名内部类调用



通过 Lambda 表达式调用



**6. Lambda 表达式的例子**



**6.1 线程初始化**




```
线程可以初始化如下：



// Old way



new Thread(new Runnable() {

​    @Override
​    public void run() {
​        System.out.println("Hello world");
​    }



}).start();

// New way
new Thread(
​    () -> System.out.println("Hello world")
).start();
```



我们在使用IDEA的时候，如果写出Old way的代码，IDEA会提示我们将其转换为Lambda表达式的形式，为IDEA点赞！

![img](C:/Users/binbin/AppData/Local/YNote/data/bhph7392@163.com/a0a29de9ea8d46978dfae4476d1cf4c0/3e58b702b32040dba2b77179e68bc24a.png)

IDEA自动检测并提示转换为Lambda表达式形式



我们将光标移动到灰色代码区域(new Runnable这里)，使用快捷键alt+Enter就可以实现自动转换了。

![img](C:/Users/binbin/AppData/Local/YNote/data/bhph7392@163.com/bee85b96aa6d4dc092c5254a3f75e51a/2af1ef66cc7a4dcba3427722ddaa84f1.png)

自动转换为Lambda表达式



**6.2 事件处理**



事件处理可以用 Java 8 使用 Lambda 表达式来完成。以下代码显示了将 ActionListener 添加到 UI 组件的新旧方式：




```
// Old way



button.addActionListener(new ActionListener() {



​    @Override
    public void **actionPerformed**(ActionEvent e) {
​        System.out.println("Hello world");
​    }
});



// New way
button.addActionListener( (e) -> {
​        System.out.println("Hello world");
});
```




**6.3 遍例输出（方法引用）**



输出给定数组的所有元素的简单代码。请注意，还有一种使用 Lambda 表达式的方式。



```
// old way
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7);



for (Integer n : list) {
    System.out.println(n);
}



// 使用 -> 的 Lambda 表达式
list.forEach(n -> System.out.println(n));


// 使用 :: 的 Lambda 表达式
list.forEach(System.out::println);
```




这里顺便补充一下Arrays.asList()方法。Arrays.asList()将数组转换为集合后,底层其实还是数组，《阿里巴巴》Java 开发使用手册对于这个方法有如下描述：

![img](C:/Users/binbin/AppData/Local/YNote/data/bhph7392@163.com/6176a0509c074990a960b9602b22b98b/a1175ebab0f24bf2a62a74e621cfc948.jpeg)

阿里巴巴Java开发手-Arrays.asList()方法



如何正确的将数组转换为ArrayList？可以像下面这样(参见：stackoverflow- https://dwz.cn/vcBkTiTW)




```
List list = new ArrayList<>(Arrays.asList("a", "b", "c"))
```



**6.4 逻辑操作**



输出通过逻辑判断的数据。




```
package com.wuxianjiezh.demo.lambda;
import java.util.Arrays;
import java.util.List;
import java.util.function.Predicate;

public class Main {

​    public static void main(String[] args) {

​        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7);
​        System.out.print("输出所有数字：");
​        evaluate(list, (n) -> true);
​        System.out.print("不输出：");
​        evaluate(list, (n) -> false);
​        System.out.print("输出偶数：");
​        evaluate(list, (n) -> n % 2 == 0);
​        System.out.print("输出奇数：");
         evaluate(list, (n) -> n % 2 == 1);
​        System.out.print("输出大于 5 的数字：");
​        evaluate(list, (n) -> n > 5);

​    }



​    public static void evaluate(List<Integer> list, Predicate<Integer> predicate) {


​        for (Integer n : list) {
​            if (predicate.test(n)) {
​                System.out.print(n + " ");
​            }
​        }
​        System.out.println();
   }
}
```




运行结果：



输出所有数字：1 2 3 4 5 6 7



不输出：



输出偶数：2 4 6



输出奇数：1 3 5 7



输出大于 5 的数字：6 7



**6.4 Stream API 示例**



java.util.stream.Stream接口 和 Lambda 表达式一样，都是 Java 8 新引入的。所有 Stream 的操作必须以 Lambda 表达式为参数。Stream 接口中带有大量有用的方法，比如 map() 的作用就是将 input Stream 的每个元素，映射成output Stream 的另外一个元素。



下面的例子，我们将 Lambda 表达式 x -> x*x 传递给 map() 方法，将其应用于流的所有元素。之后，我们使用 forEach打印列表的所有元素。




```
// old way



List<Integer> list = Arrays.asList(1,2,3,4,5,6,7);

for(Integer n : list) {
​    int x = n * n;
​    System.out.println(x);
}

// new way
List<Integer> list = Arrays.asList(1,2,3,4,5,6,7);
list.stream().map((x) -> x*x).forEach(System.out::println);
```




下面的示例中，我们给定一个列表，然后求列表中每个元素的平方和。这个例子中，我们使用了 reduce() 方法，这个方法的主要作用是把 Stream 元素组合起来。




```
// old way
List<Integer> list = Arrays.asList(1,2,3,4,5,6,7);
int sum = 0;
for(Integer n : list) {
​    int x = n * n;
​    sum = sum + x;
}

System.out.println(sum);
// new way
List<Integer> list = Arrays.asList(1,2,3,4,5,6,7);
int sum = list.stream().map(x -> x*x).reduce((x,y) -> x + y).get();
System.out.println(sum);
```




**7. Lambda 表达式和匿名类之间的区别**

- this 关键字。对于匿名类 this 关键字解析为匿名类，而对于 Lambda 表达式，this 关键字解析为包含写入 Lambda 的类。
- 编译方式。Java 编译器编译 Lambda 表达式时，会将其转换为类的私有方法，再进行动态绑定。




## 一、使用范例以及例子

使用匿名内部类:
```java
Comparator<Integer>com = new Comparator<Integer>() {
    @Override
    public int compare(Integer o1, Integer o2) {  //降序排列
        return Integer.compare(o2,o1);
    }
};
```

使用`Lambda`表达式: 
```java
 Comparator<Integer> com = (x, y) -> Integer.compare(y, x);
```

**下面给出一个例子来引入`Lambda`表达式**。

给出一个`Employee`类，有`name、age、salary`三个属性：

```java
public class Employee {
    private String name;
    private int age;
    private double salary;

    public Employee() {
    }
    public Employee(String name, int age, double salary) {
        this.name = name;
        this.age = age;
        this.salary = salary;
    }
    public String getName() {
        return name;
    }
    public int getAge() {
        return age;
    }
    public double getSalary() {
        return salary;
    }

    @Override
    public String toString() {
        return "name='" + name + '\'' +
                ", age=" + age +
                ", salary=" + salary;
    }
}

```

然后我们需要通过限制查询数据: 

 - 比如查询年龄`>25`岁的所有员工的信息；
 - 再如查询工资`>4000`的员工信息；


首先给出一个`List`集合类模拟数据库表: 

```java
//将数组转换成集合的
List<Employee> employees = Arrays.asList(
    new Employee("张三",23,3333.33),
    new Employee("李四",24,4444.44),
    new Employee("王五",25,5555.55),
    new Employee("赵六",26,6666.66),
    new Employee("田七",27,7777.77)
);
```
### 1、原始方法
然后我们写分别查询出<font color =red>年龄大于`25`岁的员工信息和工资大于`4000`</font>的员工信息，发现`findEmployeesByAge`和`findEmployeesBySalary`两个方法代码非常的相似，<font color =red>只有查询条件不同，所以这个方法是不太可取的</font>。
```java
public void test3(){
    //年龄
    List<Employee> list = findEmployeesByAge(employees);
    for(Employee emp : list){
        System.out.println(emp);
    }
    //工资
    System.out.println("---------------------");
    List<Employee> list2 = findEmployeesBySalary(employees);
    for(Employee emp : list2){
        System.out.println(emp);
    }
}

//原始方法 : 查询出年龄大于25岁的(这个是最原始的方法)
public List<Employee> findEmployeesByAge(List<Employee>list){
    List<Employee>emps = new ArrayList<>();
    for(Employee emp : list){
        if(emp.getAge() > 25){
            emps.add(emp);
        }
    }
    return emps;
}

//原始方法 : 查询出工资大于4000的(这个是最原始的方法)
//和上面的方法唯一的差别只有年龄和工资的改动，代码冗余
public List<Employee> findEmployeesBySalary(List<Employee>list){
    List<Employee>emps = new ArrayList<>();
    for(Employee emp : list){
        if(emp.getSalary() > 4000){
            emps.add(emp);
        }
    }
    return emps;
}

```
### 2、优化方式一-使用策略模式来优化
策略模式需要行为算法族，于是我们创建查询行为的接口`MyPredicate<T>`: 

```java
public interface MyPredicate <T>{
    public boolean test(T t);
}
```
并创建相关的实现类代表不同的算法行为: (分别是年龄` > 25`和工资` > 4000 `的 ): 

```java
public class FilterEmployeeByAge implements MyPredicate<Employee> {
    @Override
    public boolean test(Employee employee) {
        return  employee.getAge() > 25;
    }
}
```

```java
public class FilterEmployeeBySalary implements MyPredicate<Employee>{
    @Override
    public boolean test(Employee employee) {
        return employee.getSalary()  >= 4000;
    }
}
```
<font color = red>这时我们可以只需要创建通用的方法: 具体的调用只需要传入具体的实现类(接口作为参数)</font>

```java
public List<Employee> filterEmployees(List<Employee>list,MyPredicate<Employee>mp){
    List<Employee>emps = new ArrayList<>();
    for(Employee emp : list){
        if(mp.test(emp)){  //调用相应的过滤器
            emps.add(emp);
        }
    }
    return emps;
}
```
<font color =red>测试的时候就传入两个不同的类，来指定查询的行为</font>

```java
//优化方式一 :  使用策略设计模式进行优化  下面的方法只要写一个
public void test4(){
    List<Employee> list = filterEmployees(this.employees, new FilterEmployeeByAge());
    for(Employee emp : list){
        System.out.println(emp);
    }
    System.out.println("------------------------");
    List<Employee> list2 = filterEmployees(this.employees, new FilterEmployeeBySalary());
    for(Employee emp : list2){
        System.out.println(emp);
    }
}
```
### 3、优化方式二-使用匿名内部类优化
<font color =red>这样的好处在于不需要创建接口的具体的实现类</font>，(但是还是需要`MyPredicate`接口和`filterEmployees()`方法): 

```java
//优化方式二 ： 使用匿名内部类  这样的好处是不要创建一个额外的 策略类
public void test5(){
    List<Employee> list = filterEmployees(this.employees, new MyPredicate<Employee>() {
        @Override
        public boolean test(Employee employee) {
            return employee.getSalary() > 4000;
        }
    });
    for (Employee emp:list) {
        System.out.println(emp);
    }
}
```
### 4、优化方式三-使用Lambda表达式
<font color = red>省去匿名内部类的没用的代码，增强可读性:(注意还是需要那个`filterEmployees()`方法和`MyPredicate`接口)</font>

```java
public void test6(){
    List<Employee> list = filterEmployees(this.employees, (e) -> e.getSalary() > 4000);
    list.forEach(System.out::println);
}
```
### 5、优化方式四-使用Stream-API
<font color = red>使用`StreamAPI`完全不需要其他的代码，包括不需要`filterEmployees()`方法，代码很简洁: </font>

```java
public void test7(){
    employees.stream().filter( (e) -> e.getSalary() < 4000 ).limit(2).forEach(System.out::println);
    System.out.println("------------------");
    employees.stream().map(Employee::getName).forEach(System.out::println); //打印所有的名字
}
```
***
## 二、Lambda表达式基础语法
**关于箭头操作符:** 

 - `Java8`中引入了一个新的操作符，`"->"`，该操作符称为箭头操作符或者`Lambda`操作符，箭头操作符将`Lambda`表达式拆分成两部分；
 - 左侧:  `Lambda`表达式的<font color = blue>参数列表</font>，对应的是<font color = red>接口中抽象方法的参数列表</font>；
 - 右侧:  `Lambda`表达式中所需要执行的功能(<font color =blue>`Lambda`体</font>)，对应的是<font color = red>对抽象方法的实现；(函数式接口(只能有一个抽象方法))</font>
 - `Lambda`表达式的实质是　<font color =red>对接口的实现</font>；

**语法格式:**

(一)、接口中的抽象方法 : 无参数，无返回值；

例如: `Runnable`接口中的`run`方法: 

```java
public void test1(){
    /*final */int num = 2; //jdk1.7之前必须定义为final的下面的匿名内部类中才能访问

    Runnable r = new Runnable() {
        @Override
        public void run() {
            System.out.println("Hello world!" + num); //本质还是不能对num操作(只是jdk自己为我们设置成了final的)
        }
    };
    r.run();

    System.out.println("----------使用Lambda输出-----------");

    Runnable r1 = () -> System.out.println("Hello world!" + num);
    r1.run();
}
```

<font color =red>(二)、接口中的抽象方法 : 一个参数且无返回值；  (若只有一个参数，那么小括号可以省略不写)</font>

```java
public void test2(){
//  Consumer<String>con = (x) -> System.out.println(x);
    Consumer<String>con = x -> System.out.println(x);
    con.accept("Lambda牛逼!");
}
```

<font color =red>(三)、两个参数，有返回值，并且有多条语句 ：　**要用大括号括起来，而且要写上`return`**</font>

```java
public void test3(){
     Comparator<Integer>com = (x,y) -> {
         System.out.println("函数式接口");
         return Integer.compare(y,x); //降序
     };
     
     Integer[] nums = {4,2,8,1,5};
     Arrays.sort(nums,com);
     System.out.println(Arrays.toString(nums));
}
```
输出: 

```java
函数式接口
函数式接口
函数式接口
函数式接口
函数式接口
函数式接口
函数式接口
函数式接口
函数式接口
[8, 5, 4, 2, 1]
```

<font color =red>(四)、两个参数，有返回值，但是只有一条语句:　**大括号省略，`return`省略**</font>

 

```java
public void test4(){
     Comparator<Integer>com = (x,y) -> Integer.compare(x,y);//升序
     Integer[] nums = {4,2,8,1,5};
     Arrays.sort(nums,com);
     System.out.println(Arrays.toString(nums));
 }
```
输出: 

```java
[1, 2, 4, 5, 8]
```

 <font color =red>(五)、 `Lambda`表达式的参数列表的数据类型 可以省略不写，因为JVM编译器通过上下文推断出数据类型，即"类型推断"， `(Integer x,Integer y ) -> Integer.compare(x,y)`可以简写成`(x,y) -> Integer.compare(x,y)`；</font>

```java
上联: 左右遇一括号省
下联: 左侧推断类型省
横批: 能省则省
```
***
## 三、函数式接口

 - <font color =red>若接口中只有一个抽象方法的接口称为函数式接口；
 - <font color =red>可以使用注解`@FunctionlInterface`来标识，可以检查是否是函数式接口；

 例子: 对一个进行`+-*/`的运算：　

函数式接口: 
```java
@FunctionalInterface //函数式接口
public interface MyFunction {
    public Integer getValue(Integer num);
}
```
通用函数: 
```java
public Integer operation(Integer num,MyFunction mf){
    return mf.getValue(num);
}
```
测试: 

```java
public void test5(){
     Integer res = operation(200, (x) -> x * x);
     System.out.println(res);
 }
```
***
## 四、Lambda练习
### 1、练习一-`Employee`类中先按年龄比，年龄相同按照姓名比-都是升序
先给出集合，模拟数据库表: 

```java
List<Employee> employees = Arrays.asList(
        new Employee("田七",27,7777.77),
        new Employee("王五",24,5555.55),
        new Employee("张三",23,3333.33),
        new Employee("李四",24,4444.44),
        new Employee("赵六",26,6666.66)
);
```

```java
public void test1(){
   Collections.sort(employees,(x,y) ->{
       if(x.getAge() == y.getAge()){
           return x.getName().compareTo(y.getName());
       }else{
           return Integer.compare(x.getAge(),y.getAge());
       }
   });

   for (Employee emp: employees) {
       System.out.println(emp);
   }
}
```
输出: 

```java
name='张三', age=23, salary=3333.33
name='李四', age=24, salary=4444.44
name='王五', age=24, salary=5555.55
name='赵六', age=26, salary=6666.66
name='田七', age=27, salary=7777.77
```
### 2、练习二-声明一个带两个泛型的接口，并且对两个`Long`型数值计算

```java
@FunctionalInterface
public interface MyCalFunction<T,R> {
    public R getValue(T t1,T t2); 
}
```
对应函数和测试: 
```java
 public void test3(){
     op(200L,200L,(x,y) -> x + y);
     op(200L,200L,(x,y) -> x * y);
 }
 public void op(Long l1,Long l2,MyCalFunction<Long,Long>mc){//需求: 对于两个long型运算进行处理
     System.out.println(mc.getValue(l1, l2));
 }
```
更多的例子: (取自`<<`Java8`实战>>`)

![](https://raw.githubusercontent.com/binbinbin5/myPics/master/file/20190725181209.png)

> 根据上述语法规则，以下哪个不是有效的Lambda表达式？
> (1) ` () -> {}`
> (2) ` () -> "Raoul"`
> (3) `() -> {return "Mario";}`
> (4) ` (Integer i) -> return "Alan" + i;`
> (5)  `(String s) -> {"IronMan";}`
> 答案：只有4和 5是无效的Lambda。
>
> (1) 这个Lambda没有参数，并返回void。 它类似于主体为空的方法：public void run() {}。
> (2) 这个Lambda没有参数，并返回String作为表达式。
> (3) 这个Lambda没有参数，并返回String（利用显式返回语句）。
>
> (4) return是一个控制流语句。要使此Lambda有效，需要使花括号，如下所示：`(Integer i) -> {return "Alan" + i;}`。
>
> (5)“Iron Man”是一个表达式，不是一个语句。要使此Lambda有效，你可以去除花括号和分号，如下所示：`(String s) -> "Iron Man"`。或者如果你喜欢，可以使用显式返回语句，如下所示：`(String s)->{return "IronMan";}`。

(注意类型可以省略(类型推导)。

下面是一些使用示例:

![](https://raw.githubusercontent.com/binbinbin5/myPics/master/file/20190725181230.png)


上图的`Apple`类: 

```java
public class Apple {
    public String color;
    public int weight;

    public Apple() {
    }
    public Apple(String color, int weight) {
        this.color = color;
        this.weight = weight;
    }

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }

    public int getWeight() {
        return weight;
    }

    public void setWeight(int weight) {
        this.weight = weight;
    }
}
```

***
## 五、Java8四大内置函数式接口
我们发现，如果使用`Lambda`还要自己写一个接口的话太麻烦，所以`Java`自己提供了一些接口: 

 - `Consumer< T >con` 消费性 接口:  `void accept(T t)`；
 - `Supplier< T >sup`供给型接口 :  `T get()`；
 -  `Function< T , R >fun ` 函数式接口 :   `R apply (T t)`；
 -  `Predicate< T >`： 断言形接口 : `boolean test(T t)`；

### 1、`Consumer< T >con`消费性接口-`void accept(T t)`

```java
@Test
public void test1(){
    apply(1000,(num) -> System.out.println("消费了" + num + "元!"));
}
public void apply(double num,Consumer<Double>con){
    con.accept(num);
}
```

***
### 2、`Supplier< T >sup`供给型接口-`T get()`
例子: 产生指定个数的整数，并放入集合中；
```java
public void test2(){
    ArrayList<Integer> res = getNumList(10, () -> (int) (Math.random() * 100));
    System.out.println(res);
}
//需求，产生指定个数的整数，并放入集合中
public ArrayList<Integer> getNumList(int num, Supplier<Integer>sup){
    ArrayList<Integer>list = new ArrayList<>();
    for(int i = 0; i < num; i++){
        Integer e = sup.get();
        list.add(e);
    }
    return list;
}
```
### 3、`Function< T, R >fun`函数式接口-` R apply (T t)`

```java
public void test3(){
    String newStr = strHandler("abc", (str) -> str.toUpperCase());
    System.out.println(newStr);
    newStr = strHandler("   abc  ", (str) -> str.trim());
    System.out.println(newStr);
}
public String strHandler(String str, Function<String,String>fun){
    return fun.apply(str);
}
```
### 4、`Predicate< T >`断言形接口-`boolean test(T t)`
判断一些字符串数组判断长度`>2`的字符串: 
```java
public void test4(){
    List<String> list = Arrays.asList("Hello", "atguiu", "lambda", "ok", "www", "z");
    List<String> res = filterStr(list, (str) -> str.length() > 2);
    System.out.println(res);
}
//需求
public List<String> filterStr(List<String>list, Predicate<String>pre){
    ArrayList<String>res = new ArrayList<>();
    for(String str : list){
        if(pre.test(str)){
            res.add(str);
        }
    }
    return res;
}
```
***
## 六、方法引用和构造器引用
### 1、方法引用

使用前提: **`Lambda`体中调用方法的参数列表和返回值类型，要和函数式接口中抽象方法的参数列表和返回值类型保持一致；**</font>

#### 1)、语法格式(一) 对象::实例方法名

```java
public void test1(){
	//普通写法
     PrintStream ps = System.out;
     Consumer<String>con = (x) -> ps.println(x);
     con.accept("hello !");

     System.out.println("----------------------");
	//简写
     Consumer<String>con1 = ps::println;
     con1.accept("hello ! ");

     System.out.println("----------------------");
	//更简单的写法
     Consumer<String>con2 = System.out::println;
     con2.accept("hello ! ");
}
```
**注意，这样写的前提: `Consumer`中的`accept()`方法和`println()`方法的参数列表和返回类型要完全一致:** 



再看一个例子: 
三种写法的效果是一样的: 

```java
public class TestLambda {

    public static void main(String[] args) {

        // method 1
        Consumer<String> consumer = s -> System.out.println(s);
        useConsumer(consumer,"123");

        //method 2
        useConsumer(s -> System.out.println(s),"123");

        //method3   method reference (方法引用)
        useConsumer(System.out::println,"123"); //因为println和 accept 是同样的只有一个入参，没有返回值
    }

    public static <T> void useConsumer(Consumer<T> consumer,T t){
        consumer.accept(t);
    }
}
```
再看一个例子: 

```java
public static void main(String[] args) {
    //都是输出 字符 'l'
    BiFunction<String,Integer,Character> bf = String::charAt; //这里第一个必须传入　String
    Character c = bf.apply("hello,", 2);
    System.out.println(c);

    //注意这里使用的是Function 接口
    String str = new String("hello");
    Function<Integer,Character> f = str::charAt; //这里不需要String
    Character c2 = f.apply(2);
    System.out.println(c2);
}

```

再看一个例子: 

```java
public void test2(){
    Employee emp = new Employee("zx",23,5555);

    Supplier<String>sup = () -> emp.getName();
    System.out.println(sup.get());

    //简写
    Supplier<String>sup2 = emp::getName;
    System.out.println(sup2.get());
}
```



#### 2)、语法格式(二)  类名::静态方法

```java
public void test3(){
     Comparator<Integer>com = (x,y) -> Integer.compare(x,y);

     Comparator<Integer>com2 = Integer::compare;
}
```




#### 3)、语法格式(三) 类::实例方法名</font>

使用注意: **若Lambda参数列表中的第一个参数是实例方法的第一个调用者，而第二个参数是实例方法的参数时，可以使用`ClassName :: method`**。
```java
public void test4(){
    BiPredicate<String,String>bp = (x,y) -> x.equals(y);

    BiPredicate<String,String>bp2 = String::equals;
}
```


再举一列:

```java
public class Student {

    private String name;
    private int score;
    
    //..省略构造方法和setter、getter

    //当前的和传入的比较
    public int compareByScore(Student student){
        return getScore() - student.getScore();
    }
}
```

测试:

```java
public class T1 {

    public static void main(String[] args) {

        List<Student> stus = Arrays.asList(
                new Student("zhangsan", 93),
                new Student("wangwu", 95),
                new Student("lisi", 94)
        );

        // 3、 类名:: 实例方法名
        stus.sort(Student::compareByScore);

        stus.forEach(s -> System.out.println(s.getName()));
    }
}
```

 总结: **第一个参数作为调用者，其他参数是真正的参数**。

### 2)、构造器引用
**需要调用构造器的参数列表，要与函数式接口中的抽象方法的参数列表保持一致；**

```java
public void test5(){
    Supplier<Employee>sup = () -> new Employee();

    Supplier<Employee>sup2 = Employee::new; //调用的是默认的
    System.out.println(sup2.get());
}
```
输出: 

```java
name='null', age=0, salary=0.0
```


再看构造器一个参数的: 

```java
public void test6(){
    Function<String,Employee>fun = Employee::new;
    System.out.println(fun.apply("zx"));
}
```
输出：

```java
name='zx', age=0, salary=0.0
```



如果想要匹配多个的，(两个的可以使用`BiFunction`)，下面看一个三个的: 
例如想匹配这个: 

```java
public class ComplexApple {

    private String name;
    private int weight;
    private String color;

    public ComplexApple() {
    }

    //匹配这个构造方法
    public ComplexApple(String name, int weight, String color) {
        this.name = name;
        this.weight = weight;
        this.color = color;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getWeight() {
        return weight;
    }

    public void setWeight(int weight) {
        this.weight = weight;
    }

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }
}

```
自己建一个接口:

```java
@FunctionalInterface
public interface ThreeFunction<A,B,C,R> {
    R apply(A a,B b,C c);
}

```
测试: 

```java

public class Test {

    public static void main(String[] args) {

        ThreeFunction<String,Integer,String,ComplexApple> tf = ComplexApple::new;

        ComplexApple apple = tf.apply("蓝色", 12, "好苹果");
        
    }
}

```

