[toc]

## JDBC
Jdbc连接数据库一般分为以下几步：

```
//声明数据库驱动，数据源的url，用于登录数据库的账户和密码（将其他功能封装成方法的时候方便使用）
String driver = "数据库驱动名称"；
String url = "数据库连接地址"
String user = "用来连接数据库的用户名"；
String pwd = "用来连接数据库的密码"；
//加载数据库驱动 
Class.forName(driver);
//根据url创建数据库连接对象Connection
Connection con = DriverManage.getConnection(url,user,pwd);
//用数据库连接对象创建Statement对象(或PrepareStatement)
Statement s = con.createStatement();
或
PrepareStatement ps = con.PrepareStatement(sql);
//做数据库的增删改查工作
ResultSet rs = s.executeQuery();
//关闭结果集对象Resultset，statement对象，connection对象，
rs.close();
s.close();
con.close();
//各个步骤的异常处理
```



缺点
- 连接参数，SQL语句的硬编码（写死），SQL发生变化就需要重新编译
- 每次操作数据库都需要连接，完成后再断开，数据库的频繁断开和连接
- 查询结果集取数据的硬编码，数据库表字段发送变动时候，需要重新编译



## Mybaits

- 通过其提供的输入参数映射方式，将参数自由灵活的配置再SQL语句中，解决了JDBC中参数在java类手工配置的问题。==mybatis不会对应用程序或者数据库的现有设计强加任何影响。 sql写在xml里，便于统一管理和优化。通过sql基本上可以实现我们不使用数据访问框架可以实现的所有功能，或许更多。==
-   查询的结果集与java对象自动映射：将结果集的监测自动映射成相应的的java对象，避免JDBC中堆结果集的手工检索
-   ==Mybaits还可以创建自己的数据库连接池==，使用XML文件配置，堆数据库连接数据进行管理，避免了JDBC的数据库连接参数的硬编码问题。
-   ==解除sql与程序代码的耦合==：通过提供DAO层，将业务逻辑和数据访问逻辑分离，使系统的设计更清晰，更易维护，更易单元测试。sql和代码的分离，提高了可维护性。
 
==架构== 
1. 基础支撑层，主要是用来做连接管理、事务管理、配置加载、缓存管理等最基础组件，为上层提供最基础的支撑。
1. 数据处理层，主要是用来做参数映射、sql解析、sql执行、结果映射等处理，可以理解为请求到达，完成一次数据库操作的流程。
1. API接口层，主要对外提供API，提供诸如数据的增删改查、获取配置等接口。

**MyBatis的功能架构**：

我们把Mybatis的功能架构分为三层：

- API接口层：提供给外部使用的接口API，开发人员通过这些本地API来操纵数据库。接口层一接收到调用请求就会调用数据处理层来完成具体的数据处理。 
- 数据处理层：负责具体的SQL查找、SQL解析、SQL执行和执行结果映射处理等。它主要的目的是根据调用的请求完成一次数据库操作。 
- 基础支撑层：负责最基础的功能支撑，包括连接管理、事务管理、配置加载和缓存处理，这些都是共用的东西，将他们抽取出来作为最基础的组件。为上层的数据处理层提供最基础的支撑。 

**MyBatis的优缺点**

优点：
- 简单易学：本身就很小且简单。没有任何第三方依赖，最简单安装只要两个jar文件+配置几个sql映射文件易于学习，易于使用，通过文档和源代码，可以比较完全的掌握它的设计思路和实现。 
- 灵活：mybatis不会对应用程序或者数据库的现有设计强加任何影响。 sql写在xml里，便于统一管理和优化。通过sql基本上可以实现我们不使用数据访问框架可以实现的所有功能，或许更多。 
- 解除sql与程序代码的耦合：通过提供DAL层，将业务逻辑和数据访问逻辑分离，使系统的设计更清晰，更易维护，更易单元测试。sql和代码的分离，提高了可维护性。 
- 提供映射标签，支持对象与数据库的orm字段关系映射 
- 提供对象关系映射标签，支持对象关系组建维护 
- 提供xml标签，支持编写动态sql。 

缺点：
- 编写SQL语句时工作量很大，尤其是字段多、关联表多时，更是如此。 
- SQL语句依赖于数据库，导致数据库移植性差，不能更换数据库。 
- 框架还是比较简陋，功能尚有缺失，虽然简化了数据绑定代码，但是整个底层数据库查询实际还是要自己写的，工作量也比较大，而且不太容易适应快速数据库修改。 
- 二级缓存机制不佳 





### sqlMapConfig.xml
[--> **参数配置不用记，具体参考MyBaits官方文档**](http://www.mybatis.org/mybatis-3/zh/index.html)


MyBaits中，数据库的数据源是配置再sqlMapConfig.xml配置文件中，**其中配置了数据库驱动、数据库连接地址、数据库用户名和密码、事务管理参数等。** 在与Spring MVC建立数据库连接池就不需要为MyBaits单独配置连接池了。

SqlMapConfig.xml中配置的内容和顺序如下：（一般不修改采用默认配置）
- properties（属性）
- settings（全局配置参数）
- typeAliases（类型别名）
- typeHandlers（类型处理器）
- objectFactory（对象工厂）
- plugins（插件）
- environments（环境集合属性对象）
- environment（环境子属性对象）
- transactionManager（事务管理）
- dataSource（数据源）
- mappers（映射器）

```
//properties 属性
//SqlMapConfig.xml可以引用java属性文件中的配置信息如下：
//我们将 数据库的配置语句写在 db.properties 文件中（key-value形式注意不要出现空格）

jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf-8
jdbc.username=root
jdbc.password=root

```


```
//SqlMapConfig.xml中所有配置
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<properties resource="db.properties"></properties>
	 <settings>  
	 <!-- 转驼峰命名-->
        <setting name="mapUnderscoreToCamelCase" value="true" />  
    </settings>  
	<typeAliases> 
		<!-- 1/定义单个pojo类别名
		type:类的全路劲名称
		alias:别名
		 -->
<!-- <typeAlias type="cn.chuantao.pojo.User" alias="user"/> -->
		
		<!-- 2/使用包扫描的方式批量定义别名 
		定以后别名等于类名,不区分大小写,但是建议按照java命名规则来,首字母小写,以后每个单词的首字母大写
		-->
		<package name="cn.chuantao.pojo"/>

	</typeAliases>
 
	<!-- 和spring整合后 environments配置将废除-->
	<environments default="development">
		<environment id="development">
		<!-- 使用jdbc事务管理-->
		<transactionManager type="JDBC" />
		<!-- 数据库连接池-->
		<dataSource type="POOLED">
			<property name="driver" value="${jdbc.driver}" />
			<property name="url" value="${jdbc.url}" />
			<property name="username" value="${jdbc.username}" />
			<property name="password" value="${jdbc.password}" />
		</dataSource>
		</environment>
	</environments>
	
	<mappers>(引入mapper，sql语句)
		<!-- 1、使用class属性引入接口的全路径名称:
		    有绝对路径和相对路径
		-->
		<mapper resource="User.xml"/> 
        <mapper url="file:///var/mappers/User.xml">
		
		<!-- 2、使用包扫描的方式批量引入Mapper接口 -->
		<package name="org.myBatis.mapper"/>
		
		<!-- 3、使用接口信息进行配置Mapper接口 -->
	    <mapper class="org.myBatis.mapper.UserMapper"/>
	    
	</mappers>
</configuration>
```

### Mapper.xml
MyBaits将sql配置再独立的配置文件Mapper.xml中。包括select,update,inset,delete。==MyBaits的核心就是sql配置的Mapper。xml映射文件。==

SQL 映射文件有很少的几个顶级元素（按照它们应该被定义的顺序）：
1. cache – 给定命名空间的缓存配置。
1. cache-ref – 其他命名空间缓存配置的引用。
1. resultMap – 是最复杂也是最强大的元素，用来描述如何从数据库结果集中来加载对象。
1. parameterMap – 已废弃！老式风格的参数映射。内联参数是首选,这个元素可能在将来被移除，这里不会记录。
1. sql – 可被其他语句引用的可重用语句块。
1. insert – 映射插入语句
1. update – 映射更新语句
1. delete – 映射删除语句
1. select – 映射查询语句


```

<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!-- namespace:命名空间,做sql隔离,分类化管理 -->
<mapper namespace="test">
 
	<!-- 
	id:sql语句唯一标识
	parameterType:指定传入参数类型
	resultType:返回结果集类型
	#{}占位符:起到占位作用,如果传入的是基本类型(string,long,double,int,boolean,float等),那么#{}中的变量名称可以随意写.
	 -->
	<select id="findUserById" parameterType="java.lang.Integer" resultType="cn.chuantao.pojo.User">
		select * from user where id=#{id}
	</select>
	
	<!-- 
	如果返回结果为集合,可以调用selectList方法,这个方法返回的结果就是一个集合,所以映射文件中应该配置成集合泛型的类型
	${}拼接符:字符串原样拼接,如果传入的参数是基本类型(string,long,double,int,boolean,float等),那么${}中的变量名称必须是value
	注意:拼接符有sql注入的风险,所以慎重使用
	 -->
	<select id="findUserByUserName" parameterType="java.lang.String" resultType="cn.chuantao.pojo.User">
		select * from user where username like '%${value}%'
	</select>
	
	<!-- 
	#{}:如果传入的是pojo类型,那么#{}中的变量名称必须是pojo中对应的属性.属性.属性.....
	如果要返回数据库自增主键:可以使用select LAST_INSERT_ID()
	 -->
	<insert id="insertUser" parameterType="cn.chuantao.pojo.User" >
		<!-- 执行 select LAST_INSERT_ID()数据库函数,返回自增的主键
		keyProperty:将返回的主键放入传入参数的Id中保存.
		order:当前函数相对于insert语句的执行顺序,在insert前执行是before,在insert后执行是AFTER
		resultType:id的类型,也就是keyproperties中属性的类型
		-->
		<selectKey keyProperty="id" order="AFTER" resultType="java.lang.Integer">
			select LAST_INSERT_ID()
		</selectKey>
		insert into user (username,birthday,sex,address) values(#{username},#{birthday},#{sex},#{address})
	</insert>
	
	<delete id="delUserById" parameterType="int">
		delete from user where id=#{id}
	</delete>
	
	<update id="updateUserById" parameterType="cn.chuantao.pojo.User">
		update user set username=#{username} where id=#{id}
	</update>
</mapper>


```

使用Mapper动态接口的UserMapper.xml
```

<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- 
mapper接口代理实现编写规则:
	1. 映射文件中namespace要等于接口的全路径名称
	2. 映射文件中sql语句id要等于接口的方法名称
	3. 映射文件中传入参数类型要等于接口方法的传入参数类型
	4. 映射文件中返回结果集类型要等于接口方法的返回值类型
 -->
<mapper namespace="cn.chuantao.mapper.UserMapper">
 
 
	<select id="findUserById" parameterType="int" resultType="cn.chuantao.pojo.User">
		select * from user where id=#{id}
	</select>
	
	<!-- 
	如果返回结果为集合,可以调用selectList方法,这个方法返回的结果就是一个集合,所以映射文件中应该配置成集合泛型的类型
	${}拼接符:字符串原样拼接,如果传入的参数是基本类型(string,long,double,int,boolean,float等),那么${}中的变量名称必须是value
	注意:拼接符有sql注入的风险,所以慎重使用
	 -->
	<select id="findUserByUserName" parameterType="string" resultType="user">
		select * from user where username like '%${value}%'
	</select>
	
	<!-- 
	#{}:如果传入的是pojo类型,那么#{}中的变量名称必须是pojo中对应的属性.属性.属性.....
	如果要返回数据库自增主键:可以使用select LAST_INSERT_ID()
	 -->
	<insert id="insertUser" parameterType="cn.chuantao.pojo.User" >
		<!-- 执行 select LAST_INSERT_ID()数据库函数,返回自增的主键
		keyProperty:将返回的主键放入传入参数的Id中保存.
		order:当前函数相对于insert语句的执行顺序,在insert前执行是before,在insert后执行是AFTER
		resultType:id的类型,也就是keyproperties中属性的类型
		-->
		<selectKey keyProperty="id" order="AFTER" resultType="java.lang.Integer">
			select LAST_INSERT_ID()
		</selectKey>
		insert into user (username,birthday,sex,address) values(#{username},#{birthday},#{sex},#{address})
	</insert>
</mapper>

```
### sqlSessionFactory
MyBaits中处理读取加载XML文件的就是核心对象：会话工厂与会话。

==会话工厂（sqlSessionFactory）会根据Resources资源信息加载对象，获取项目配置的数据库sqlMapConfig.xml信息，从而产生一直可以和数据库交互的会话实例类-sqlSession==
### 注意
>1
- #{} 是预处理语句属性
- %{} 是字符串替换，尽量不要使用，引发SQL注入攻击。

>2
Mapper配置输出映射：
- resultType:使用resultType进行输出映射，只有查询出来的列名和pojo（实体bean）中的属性名一致，该列才可以映射成功。
    - 如果查询出来的列名和pojo中的属性名全部不一致，没有创建pojo对象。
    - 只要查询出来的列名和pojo中的属性有一个一致，就会创建pojo对象。
- resultMap：如果查询出来的列名和返回结果的属性名不一致，通过定义一个resultMap对列名和pojo属性名之间作一个映射关系。

1. 在MyBatis进行查询映射时，其实查询出来的每一个属性都是放在一个对应的Map里面的，其中键是列名，值则是其对应的值。
1. 当提供的返回类型属性是resultType时，MyBatis会将Map里面的键值对取出赋给resultType所指定的对象对应的属性。
1. 所以其实MyBatis的每一个查询映射的返回类型都是ResultMap，只是当提供的返回类型属性是resultType的时候，MyBatis会自动把对应的值赋给resultType所指定对象的属性。

### resultMap
myBaits对于处理简单的单表查询一般使用resultType就可以解决了，但是对于多表联合查询一般旺旺需要使用resultMap进行详细描述，告诉MyBaits规则自己定义。


- 配置文件支持继承 <父类>
```xml
 <resultMap id="DatabaseTemp" type="com.leike.pojo.UserWithDetail">
        <result property="id" column="uid"/>
        <result property="password" column="password"/>
        <result property="phone" column="phone"/>
        <result property="createDate" column="createDate"/>
        <result property="status" column="status"/>
    </resultMap>
```
三种映射方式<resultMap>
```xml
<!--第一种方法：官方推荐：-->
<resultMap id="UserWithDetailMap" extends="DatabaseTemp" type="com.leike.pojo.UserWithDetail">
        <!--已经继承了-->
        <association property="userDetail" javaType="com.leike.pojo.UserDetail">
            <id property="id" column="udId"/>
            <result property="address" column="address"/>
            <result property="cid" column="cid"/>
        </association>
    </resultMap>

 <!--第二种方法   连缀写法-->
    <resultMap id="UserWithDetailMap1" extends="DatabaseTemp" type="com.leike.pojo.UserWithDetail">
        <result property="userDetail.id" column="udId"/>
        <result property="userDetail.address" column="address"/>
        <result property="userDetail.cid" column="cid"/>
    </resultMap>
    
    
    <!--第三种方法 分布查询-->
     <resultMap id="UserWithDetailMap2" extends="DatabaseTemp" type="com.leike.pojo.UserWithDetail" >
        <association property="userDetail" select="com.leike.mapper.UserDetailMapper.queryByUid" column="uid"></association>
    </resultMap>
```


### 实体关系

MyBatis实现一对一有几种方式?具体怎么操作的？==Association:  一对一查询的方式==
- 有联合查询和嵌套查询,联合查询是几个表联合查询,只查询一次, 通过在resultMap里面配置association节点配置一对一的类就可以完成；
- 嵌套查询是先查一个表，根据这个表里面的结果的 外键id，去再另外一个表里面查询数据,也是通过association配置，但另外一个表的查询通过select属性配置。

```xml
<mapper namespace="one.to.one.mapper.OrdersMapper">
    <!--
    嵌套结果：使用嵌套结果映射来处理重复的联合结果的子集
                               封装联表查询的数据(去除重复的数据)
     select * from orders o,user u where o.user_id=u.id and o.id=#{id}
     -->
    <select id="selectOrderAndUserByOrderID" resultMap="getOrderAndUser">
        select * from orders o,user u where o.user_id=u.id and o.id=#{id}
    </select>
    
    <resultMap type="com.ys.po.Orders" id="getOrderAndUser">
        <!--
            id:指定查询列表唯一标识，如果有多个唯一标识，则配置多个id
            column:数据库对应的列
            property:实体类对应的属性名
          -->
        <id column="id" property="id"/>
        <result column="user_id" property="userId"/>
        <result column="number" property="number"/>
        <!--association:用于映射关联查询单个对象的信息
            property:实体类对应的属性名
            javaType:实体类对应的全类名
          -->
        <association property="user" javaType="com.ys.po.User">
            <!--
                association映射关联了查询的单个对象信息
                property是java的属性
                要以JavaType对应的数据类型封装，这边是User
                即column对于getOrderAndUser,property对应User
              -->
            <id column="id" property="id"/>
            <result column="username" property="username"/>
            <result column="sex" property="sex"/>
        </association>
    </resultMap>
     
     
    <!--
         方式二：嵌套查询：通过执行另外一个SQL映射语句来返回预期的复杂类型
         select user_id from order WHERE id=1;//得到user_id
         select * from user WHERE id=1   //1 是上一个查询得到的user_id的值
         property:别名(属性名)    column：列名 -->
    <select id="getOrderByOrderId" resultMap="getOrderMap">
        select * from order where id=#{id}
    </select>
   
    <resultMap type="com.ys.po.Orders" id="getOrderMap">
        <id column="id" property="id"/>
        <result column="number" property="number"/>
        <association property="userId"  column="id" select="getUserByUserId">
         
        </association>
    </resultMap>
    
    <select id="getUserByUserId" resultType="com.ys.po.User">
        select * from user where id=#{id}
    </select>
     
</mapper>
```

 
MyBatis实现一对多有几种方式,怎么操作的？==Collection:  一对多查询的方式==

- 有联合查询和嵌套查询。联合查询是几个表联合查询,只查询一次,通过在resultMap里面的collection节点配置一对多的类就可以完成；
- 嵌套查询是先查一个表,根据这个表里面的 结果的外键id,去再另外一个表里面查询数据,也是通过配置collection,但另外一个表的查询通过select节点配置。

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="one.to.many.mapper.UserMapper">
    <!--
    方式一：嵌套结果：使用嵌套结果映射来处理重复的联合结果的子集
                               封装联表查询的数据(去除重复的数据)
     select * from user u,orders o where u.id=o.user_id and u.id=#{id}
     -->
    <select id="selectUserAndOrdersByUserId" resultMap="getUserAndOrders">
        select u.*,o.id oid,o.number number from user u,orders o where u.id=o.user_id and u.id=#{id}
    </select>
    <resultMap type="com.ys.po.User" id="getUserAndOrders">
        <!--id:指定查询列表唯一标识，如果有多个唯一标识，则配置多个id
            column:数据库对应的列
            property:实体类对应的属性名 -->
        <id column="id" property="id"/>
        <result column="username" property="username"/>
        <result column="sex" property="sex"/>
        <!--
            property:实体类中定义的属性名
            ofType：指定映射到集合中的全类名
          -->
        <collection property="orders" ofType="com.ys.po.Orders">
            <id column="oid" property="id"/>
            <result column="number" property="number"/>
        </collection>
    </resultMap>
</mapper>
```

多对多：collection+collection+association,多对多主要是关联关系要找好，然后根据关联去查询。
```
<!-- 一个用户对应有多个订单信息 -->
<collection property="orders" ofType="com.hp.bean.Orders">
	<id column="o_id" property="o_id"/>
	<result column="o_name" property="o_name"/>
	<result column="u_id" property="u_id"/>
	
	<!-- 一个商品订单有多个商品信息 -->
	<collection property="ordersDemos" ofType="com.hp.bean.OrdersDemo">
	<id column="d_id" property="d_id"/>
	<result  column="d_num" property="d_num"/>
		<result  column="d_price" property="d_price"/>
		<result  column="orders_id" property="orders_id"/>
		
		<!-- 一个订单详细 有一个商品信息 -->
		<association property="items" javaType="com.hp.bean.Items">
		<id column="i_id" property="i_id"/>
		<result column="i_text" property="i_text" />
		<result column="i_price" property="i_price" />
		<result column="i_desc" property="i_desc" />
		</association>
		
	</collection>
	
</collection>
 
</resultMap>

```
### 懒加载

>需求：查询订单信息，有时候需要关联查出用户信息。

第一种方法：我们直接关联查询出所有订单和用户的信息
```
select * from orders o ,user u where o.user_id = u.id;
```
分析：
1. 这里我们一次查询出所有的信息，需要什么信息的时候直接从查询的结果中筛选。但是如果订单和用户表都比较大的时候，这种关联查询肯定比较耗时。
2. 我们的需求是有时候需要关联查询用户信息，这里不是一定需要用户信息的。即有时候不需要查询用户信息，我们也查了，程序进行了多余的耗时操作。

 
第二种方法：分步查询，首先查询出所有的订单信息，然后如果需要用户的信息，我们在根据查询的订单信息去关联用户信息

```
//查询所有的订单信息，包括用户id
select * from orders;
//如果需要用户信息，我们在根据上一步查询的用户id查询用户信息
select * from user where id=user_id
```
分析：
1. 这里两步都是单表查询，执行效率比关联查询要高很多
2. 分为两步，如果我们不需要关联用户信息，那么我们就不必执行第二步，程序没有进行多余的操作。

那么我们说，这第二种方法就是mybatis的懒加载。==通俗的讲就是按需加载，我们需要什么的时候再去进行什么操作。而且先从单表查询，需要时再从关联表去关联查询，能大大提高数据库性能，因为查询单表要比关联查询多张表速度要快==。在mybatis中，resultMap可以实现高级映射(使用association、collection实现一对一及一对多映射)，association、collection具备延迟加载功能。


### 缓存
缓存是为了提高速度，直接把已经加载的东西存入内存，后续使用从内存中拿。（不怎么变的才做缓存）

- 一级缓存是SqlSession级别的缓存。在操作数据库时需要构造sqlSession对象，在对象中有一个数据结构（HashMap）用于存储缓存数据。不同的sqlSession之间的缓存数据区域（HashMap）是互相不影响的。
- 二级缓存是mapper级别的缓存，多个SqlSession去操作同一个Mapper的sql语句，多个SqlSession可以共用二级缓存，二级缓存是跨SqlSession的。

![](https://raw.githubusercontent.com/binbinbin5/myPics/master/imgs/huancun.jpg)
每个sqlSession类的实例对象自身有一个一级缓存，而查询同一个Mapper映射文件的sqlSession类的实例对象之间又共享一个二级缓存。
 
>一级缓存：会话级别的
- 1、第一次发起查询息，先去找缓存中查询，如果没有，从数据库查询用户信息。得到用户信息，将用户信息存储到一级缓存中。 
- 2、如果中间sqlSession去执行commit操作（==插入、更新、删除==），则会清空SqlSession中的一级缓存，这样做的目的为了让缓存中存储的是最新的信息，避免脏读。
- 3、第二次发起查询信息，先去找缓存中信息，缓存中有，直接从缓存中获取用户信息。

```
sqlSession.clearCache() //强制清空缓存，缓存失效
sqlSession.flushStatements() //刷新，只是清空sql语句
```

![](https://raw.githubusercontent.com/binbinbin5/myPics/master/imgs/20190716160954.png)

>二级缓存：不是会话级别的
- 一级缓存是会话(session)级别的,言下之意就是要在同一个会话(session)当中才有效
- 如果开启了二级缓存
- 先去二级缓存当中尝试命中
- 如果也无法命中
- 尝试去一级缓存中命中
- 还不命中,再去数据库查询
![](https://raw.githubusercontent.com/binbinbin5/myPics/master/imgs/20190716160117.png)

和一级缓存默认开启不一样，二级缓存需要我们手动开启：

首先在全局配置文件 mybatis-configuration.xml 文件中加入如下代码：
```
<settings>
    <setting name="cacheEnabled" value="true"/>
</settings>

<!--其次在 UserMapper.xml 文件中开启缓存-->

<cache
  eviction="FIFO"
  flushInterval="60000"
  size="512"
  readOnly="true"/>
```
这个更高级的配置创建了一个 FIFO 缓存，每隔 60 秒刷新，最多可以存储结果对象或列表的 512 个引用，而且返回的对象被认为是只读的，因此对它们进行修改可能会在不同线程中的调用者产生冲突。

可用的清除策略有：

- LRU – 最近最少使用：移除最长时间不被使用的对象。
- FIFO – 先进先出：按对象进入缓存的顺序来移除它们。
- SOFT – 软引用：基于垃圾回收器状态和软引用规则移除对象。
- WEAK – 弱引用：更积极地基于垃圾收集器状态和弱引用规则移除对象。
>二级缓存整合ehcache：

mybatis提供了一个cache接口，如果要实现自己的缓存逻辑，实现cache接口开发即可。mybatis本身默认实现了一个，但是这个缓存的实现无法实现分布式缓存，所以我们要自己来实现。==ehcache分布式缓存就可以，mybatis提供了一个针对cache接口的ehcache实现类，这个类在mybatis和ehcache的整合包中。==

```
在全局配置文件 mybatis-configuration.xml 开启缓存
<!--开启二级缓存  -->
<settings>
    <setting name="cacheEnabled" value="true"/>
</settings>
```

```
<!-- 开启本mapper的namespace下的二级缓存
    type:指定cache接口的实现类的类型，不写type属性，mybatis默认使用PerpetualCache
    要和ehcache整合，需要配置type为ehcache实现cache接口的类型
-->
<cache type="org.mybatis.caches.ehcache.EhcacheCache"></cache>
```
对于访问多的查询请求且用户对查询结果实时性要求不高，此时可采用mybatis二级缓存技术降低数据库访问量，提高访问速度，业务场景比如：耗时较高的统计分析sql、电话账单查询sql等。实现方法如下：通过设置刷新间隔时间，由mybatis每隔一段时间自动清空缓存，根据数据变化频率设置缓存刷新间隔flushInterval，比如设置为30分钟、60分钟、24小时等，根据需求而定。 

mybatis二级缓存对细粒度的数据级别的缓存实现不好，比如如下需求：对商品信息进行缓存，由于商品信息查询访问量大，但是要求用户每次都能查询最新的商品信息，此时如果使用mybatis的二级缓存就无法实现当一个商品变化时只刷新该商品的缓存信息而不刷新其它商品的信息，因为mybaits的二级缓存区域以mapper为单位划分的，当一个商品信息变化会将所有商品信息的缓存数据全部清空。解决此类问题可能需要在业务层根据需求对数据有针对性缓存。

### 动态SQL
mybatis 动态SQL，通过 ==if, choose, when, otherwise, trim, where, set, foreach==等标签，可组合成非常灵活的SQL语句，从而在提高 SQL 语句的准确性的同时，也大大提高了开发人员的效率。

```xml
<!--如果 #{username} 为空，那么查询结果也是空，使用 if 来判断解决-->
<!--where”标签会知道如果它包含的标签中有返回值的话，它就插入一个‘where’。
此外，如果标签返回的内容是以AND 或OR 开头的，则它会剔除掉。-->

<select id="selectUserByUsernameAndSex" resultType="user" parameterType="com.ys.po.User">
    select * from user
    <where>
    <!-- where 标签完美解决科研自动消除多余and-->
        <if test="username != null and test!=''">
           and username=#{username}
        </if>
         
        <if test="username != null and test!=''">
           and sex=#{sex}
        </if>
    </where>
</select>
```

```xml
<!-- 根据 id 更新 user 表的数据 -->
<update id="updateUserById" parameterType="com.ys.po.User">
    update user 
    <!--set只能处理后置“,”逗号-->
        <set>
            <if test="username != null and username != ''">
                u.username = #{username},
            </if>
            <if test="sex != null and sex != ''">
                u.sex = #{sex}
            </if>
        </set>
     
     where id=#{id}
</update>
```

```xml
<select id="selectUserByChoose" resultType="com.ys.po.User" parameterType="com.ys.po.User">
      select * from user
      <where>
          <choose>
          <!--choose when otherwise 表示if ,else if.
          choose 只能选择一个
          -->
              <when test="id !='' and id != null">
                  id=#{id}
              </when>
              <when test="username !='' and username != null">
                  and username=#{username}
              </when>
              <otherwise>
                  and sex=#{sex}
              </otherwise>
          </choose>
      </where>
  </select>
```
trim标记是一个格式化的标记，可以完成set或者是where标记的功能
- prefix：前缀　　　　　　
- prefixoverride：去掉第一个and或者是or
- suffix：后缀　　
- suffixoverride：去掉最后一个逗号（也可以是其他的标记，就像是上面前缀中的and一样）
```xml
<select id="selectUserByUsernameAndSex" resultType="user" parameterType="com.ys.po.User">
        select * from user
        <!-- <where>
            <if test="username != null">
               username=#{username}
            </if>
             
            <if test="username != null">
               and sex=#{sex}
            </if>
        </where>  -->
        
        <!--前缀价格where,前缀加and或者or-->
        <trim prefix="where" prefixOverrides="and | or">
            <if test="username != null">
               and username=#{username}
            </if>
            <if test="sex != null">
               and sex=#{sex}
            </if>
        </trim>
        
        <!--
           <trim prefix="set" suffixOverrides=",">
                <if test="username != null and username != ''">
                    u.username = #{username},
                </if>
                <if test="sex != null and sex != ''">
                    u.sex = #{sex},
                </if>
            </trim>
         -->
    </select>
　　
```

```xml
<!-- 定义 sql 片段 -->
<sql id="selectUserByUserNameAndSexSQL">
    <if test="username != null and username != ''">
        AND username = #{username}
    </if>
    <if test="sex != null and sex != ''">
        AND sex = #{sex}
    </if>
</sql>

<select id="selectUserByUsernameAndSex" resultType="user" parameterType="com.ys.po.User">
    select * from user
    <trim prefix="where" prefixOverrides="and | or">
        <!-- 引用 sql 片段，如果refid 指定的不在本文件中，那么需要在前面加上 namespace -->
        <include refid="selectUserByUserNameAndSexSQL"></include>
        <!-- 在这里还可以引用其他的 sql 片段 -->
    </trim>
</select>
```


```xml
<!--foreach 语句-->
<select id="selectUserByListId" parameterType="com.ys.vo.UserVo" resultType="com.ys.po.User">
        select * from user
        <where>
            <!--
                collection:指定输入对象中的集合属性list set map
                item:每次遍历生成的对象 代号
                open:开始遍历时的拼接字符串 拼接以什么开头
                close:结束时拼接的字符串 拼接以什么结尾
                separator:遍历对象之间需要拼接的字符串 
                select * from user where 1=1 and id in (1,2,3)
              -->
              
            <foreach collection="ids" item="id" open="and id in (" close=") " separator=",">
                #{id}
            </foreach>
             <!--
            <foreach collection="ids" item="id" open="and (" close=")" separator="or">
                id=#{id}
            </foreach>
              -->
        </where>
    </select>
```

```xml
<!--模糊查询-->
<select id="selectBymohu" resultType = "binbin.com" >
     <!--2bind重新绑定
     <bind name = "_city" value="'%'+city+'%'"/>
     -->
    select * form user
    where
    <!--#{}不支持拼接%-->
    city like concat('%',#{city],'%') <!--1-->
                                    
</select>
```

```xml
<sql id = "baseColumn">
county,state,city
<sql>
<!--通过包含sql来查询，简化工作量-->
<select id="listAll" resultType="binbin.com">
    <inlcude refid = "base">
</select>
```



### 注解

```
public interface UserMapper {
    //根据 id 查询 user 表数据
    @Select("select * from user where id = #{id}")
    public User selectUserById(int id) throws Exception;
 
    //向 user 表插入一条数据
    @Insert("insert into user(username,sex,birthday,address) value(#{username},#{sex},#{birthday},#{address})")
    public void insertUser(User user) throws Exception;
     
    //根据 id 修改 user 表数据
    @Update("update user set username=#{username},sex=#{sex} where id=#{id}")
    public void updateUserById(User user) throws Exception;
     
    //根据 id 删除 user 表数据
    @Delete("delete from user where id=#{id}")
    public void deleteUserById(int id) throws Exception;
     
}
```









## MyBatis定义的接口实现的？ && MyBaits工作流程
- 构建 SqlSessionFactory ( 通过 xml 配置文件 , 或者直接编写Java代码)
- 从 SqlSessionFactory 中获取 SqlSession
- 从SqlSession 中获取 Mapper
- 调用 Mapper 的方法 ，例如：blogMapper.selectBlog(int blogId)


![](https://raw.githubusercontent.com/binbinbin5/myPics/master/imgs/20190716100507.png)


```
 public class MyBatisUtil  {  
        private  final static SqlSessionFactory sqlSessionFactory;  
        static {  
           String resource = "resource/mybatis-config.xml";  
           Reader reader = null;  
           try {  
               reader = Resources.getResourceAsReader(resource);  
           } catch (IOException e) {  
               System.out.println(e.getMessage());  
                
           }  
           sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);  
        }  
         
        public static SqlSessionFactory getSqlSessionFactory() {  
           return sqlSessionFactory;  
        }  

```

```
sqlSession sqlsession = MybatisUtil.getSession();
Mapper mapper = sqlSession.getMapper(Mapper.class)
...
```
