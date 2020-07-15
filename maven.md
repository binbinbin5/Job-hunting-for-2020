

## Maven 
maven是一款服务于java平台的自动化构建工具.(在POM.XML中配置jar包，解决jar依赖)

优点：
- 依赖管理:不再需要手动导入 Jar依赖包，并且可以自动处理依赖关系，也就是说某个依赖如果依赖于其它依赖，构建工具可以帮助我们自动处理这种依赖关系。
- 运行单元测试:不再需要在项目代码中添加测试代码，从而避免了污染项目代码。
- 将源代码转化为可执行文件:包含预处理、编译、汇编、链接等步骤。
- 将可执行文件进行打包:不再需要使用 IDE 将应用程序打包成 Jar 包。
- 发布到生产服务器上:不再需要通过 FTP 将 Jar 包上传到服务器上。


### Maven仓库

仓库的搜索顺序为：本地仓库、中央仓库、远程仓库。
- 本地仓库用来存储项目的依赖库；
- 中央仓库是下载依赖库的默认位置；
- 远程仓库，因为并非所有的依赖库都在中央仓库，或者中央仓库访问速度很慢，远程仓库是中央仓库的补充。

![](https://raw.githubusercontent.com/binbinbin5/myPics/master/imgs/5273240e4256449eabec215d5189449b.jpeg)

- 你要jar包，不可能每次都要联网去下载吧，多费劲，所以本地仓库就是相当于加了一层jar包缓存，先到这里来查。如果这里查不到，那么就去私服上找，如果私服也找不到，那么去中央仓库去找，找到jar后，会把jar的信息同步到私服和本地仓库中。

- 私服，就是公司内部局域网的一台服务器而已，你想一下，当你的工程Project-A依赖别人的Project-B的接口，怎么做呢？没有Maven的时候，当然是copy Project-B jar到你的本地lib中引入，那么Maven的方式，很显然需要其他人把Project-B deploy到私服仓库中供你使用。因此私服中存储了本公司的内部专用的jar！不仅如此，私服还充当了中央仓库的镜像，说白了就是一个代理！

- 中央仓库：该仓库存储了互联网上的jar，由Maven团队来维护，地址是：==http://repo1.maven.org/maven2/==，一般需要什么都去中央仓库拿。




>POM.XML文件内容：提供了项目对象模型（POM）文件来管理项目的构建。

可以在工程中的pom.xml里面开始添加<dependency>标签来管理jar包，在Maven规范的目录结构下进行编写代码，最后你会通过插件的方式来进行测试、打包（jar or war）、部署、运行。




```xml
//例如要在项目中引入 Junit，Maven 的代码如下：
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
   <modelVersion>4.0.0</modelVersion>
 
   <groupId>jizg.study.maven.hello</groupId>
   <artifactId>hello-first</artifactId>
   <version>0.0.1-SNAPSHOT</version>

   <dependencies>//这些可以去中央仓库拷贝
          <dependency>
               <groupId>junit</groupId>
               <artifactId>junit</artifactId>
               <version>4.10</version>
               <scope>test</scope>
          </dependency>
   </dependencies>

</project>
```

```
[groupId, artifactId, version, packaging, classifier] 称为一个项目的坐标，其中 groupId、artifactId、version 必须定义，packaging 可选（默认为 Jar），classifier 不能直接定义的，需要结合插件使用。

- groupId：项目组 Id，必须全球唯一；
- artifactId：项目 Id，即项目名；
- version：项目版本；
- packaging：项目打包方式。

```

## 依赖原则

1. 依赖路径最短优先原则

```html
A -> B -> C -> X(1.0)
A -> D -> X(2.0)
```
由于 X(2.0) 路径最短，所以使用 X(2.0)。

2. 声明顺序优先原则

```html
A -> B -> X(1.0)
A -> C -> X(2.0)
```

在 POM 中最先声明的优先，上面的两个依赖如果先声明 B，那么最后使用 X(1.0)。

3. 覆写优先原则

子 POM 内声明的依赖优先于父 POM 中声明的依赖。

>解决依赖冲突

找到 Maven 加载的 Jar 包版本，使用 `mvn dependency:tree` 查看依赖树，根据依赖原则来调整依赖在 POM 文件的声明顺序。

## 问题
>version分为开发版本（Snapshot）和发布版本（Release），那么为什么要分呢？

在实际开发中，我们经常遇到这样的场景，比如A服务依赖于B服务，A和B同时开发，B在开发中发现了BUG，修改后，将版本由1.0升级为2.0，那么A必须也跟着在POM.XML中进行版本升级。过了几天后，B又发现了问题，进行修改后升级版本发布，然后通知A进行升级...可以说这是开发过程中的版本不稳定导致了这样的问题。

Maven，已经替我们想好了解决方案，就是使用Snapshot版本，在开发过程中B发布的版本标志为Snapshot版本，A进行依赖的时候选择Snapshot版本，那么每次B发布的话，会在私服仓库中，形成带有时间戳的Snapshot版本，而A构建的时候会自动下载B最新时间戳的Snapshot版本！

>既然Maven进行了依赖管理，为什么还会出现依赖冲突？处理依赖冲突的手段是?

首先来说，对于Maven而言，同一个groupId同一个artifactId下，只能使用一个version！按照从上到下的顺序，一般选择下的配置。

现在，我们可以思考下了，比如工程中需要引入A、B，而A依赖1.0版本的C，B依赖2.0版本的C，那么问题来了，C使用的版本将由引入A、B的顺序而定？这显然不靠谱！如果A的依赖写在B的依赖后面，将意味着最后引入的是1.0版本的C，很可能在运行阶段出现类（ClassNotFoundException）、方法（NoSuchMethodError）找不到的错误（因为B使用的是高版本的C）！

这里其实涉及到了2个概念：依赖传递（transitive）、Maven的最近依赖策略。

依赖传递：如果A依赖B，B依赖C，那么引入A，意味着B和C都会被引入。

Maven的最近依赖策略：==如果一个项目依赖相同的groupId、artifactId的多个版本，那么在依赖树（mvn dependency:tree）中离项目最近的那个版本将会被使用==。（从这里可以看出Maven是不是有点小问题呢？能不能选择高版本的进行依赖么？据了解，Gradle就是version+策略）

现在，我们可以想想如何处理依赖冲突呢？

想法1：要使用哪个版本，我们是清楚的，那么能不能不管如何依赖传递，都可以进行版本锁定呢？

使用<dependencyManagement>  [这种主要用于子模块的版本一致性中]

想法2：在依赖传递中，能不能去掉我们不想依赖的？

使用<exclusions> [在实际中我们可以在IDEA中直接利用插件帮助我们生成]

想法3：既然是最近依赖策略，那么我们就直接使用显式依赖指定版本，那不就是最靠近项目的么？Maven继承与聚合，这个你需要掌握。

使用<dependency>

>依赖解决

如果我们新加入一个依赖的话，那么先通过mvn dependency:tree命令形成依赖树，看看我们新加入的依赖，是否存在传递依赖，传递依赖中是否和依赖树中的版本存在冲突，如果存在多个版本冲突，利用上文的方式进行解决！

## 构建项目
简单Java工程目录结构

这里需要注意2点：
- 第一：src/main下内容最终会打包到Jar/War中，而src/test下是测试内容，并不会打包进去。
- 第二：src/main/resources中的资源文件会COPY至目标目录，这是Maven的默认生命周期中的一个规定动作。（想一想，hibernate/mybatis的映射XML需要放入resources下，而不能在放在其他地方了）

>构建的各个环节

Maven生命周期
我们只需要注意一点：执行后面的命令时，前面的命令自动得到执行。
- 清理clean：将以前编译得到的旧文件class字节码文件删除
- 编译compile：将java源程序编译成class字节码文件
- 测试test：自动测试，自动调用junit程序
- 报告report：测试程序执行的结果
- 打包package：动态Web工程打War包，java工程打jar包
- 安装install：Maven特定的概念-----将打包得到的文件复制到“仓库”中的指定位置
- 部署deploy：将动态Web工程生成的war包复制到Servlet容器下，使其可以运行




## Maven命令
maven 命令的格式为 mvn [plugin-name]:[goal-name]，可以接受的参数如下。

```
-D 指定参数，如 -Dmaven.test.skip=true 跳过单元测试；

-P 指定 Profile 配置，可以用于区分环境；

-e 显示maven运行出错的信息；

-o 离线执行命令,即不去远程仓库更新包；

-X 显示maven允许的debug信息；

-U 强制去远程更新snapshot的插件或依赖，默认每天只更新一次。
```
常用命令

```

创建maven项目：mvn archetype:create 

指定 group： -DgroupId=packageName 

指定 artifact：-DartifactId=projectName

创建web项目：-DarchetypeArtifactId=maven-archetype-webapp  

创建maven项目：mvn archetype:generate

验证项目是否正确：mvn validate

maven 打包：mvn package

只打jar包：mvn jar:jar

生成源码jar包：mvn source:jar

产生应用需要的任何额外的源代码：mvn generate-sources

编译源代码： mvn compile

编译测试代码：mvn test-compile

运行测试：mvn test

运行检查：mvn verify

清理maven项目：mvn clean

生成eclipse项目：mvn eclipse:eclipse

清理eclipse配置：mvn eclipse:clean

生成idea项目：mvn idea:idea

安装项目到本地仓库：mvn install

发布项目到远程仓库：mvn:deploy

在集成测试可以运行的环境中处理和发布包：mvn integration-test

显示maven依赖树：mvn dependency:tree

显示maven依赖列表：mvn dependency:list

下载依赖包的源码：mvn dependency:sources

安装本地jar到本地仓库：mvn install:install-file -DgroupId=packageName -DartifactId=projectName -Dversion=version -Dpackaging=jar -Dfile=path
```
WEB相关命令

```
启动tomcat：mvn tomcat:run

启动jetty：mvn jetty:run

运行打包部署：mvn tomcat:deploy

撤销部署：mvn tomcat:undeploy

启动web应用：mvn tomcat:start

停止web应用：mvn tomcat:stop

重新部署：mvn tomcat:redeploy

部署展开的war文件：mvn war:exploded tomcat:exploded    
```
