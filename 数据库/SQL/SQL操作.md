

[toc]


# 操作
# 一、基础

模式定义了数据如何存储、存储什么样的数据以及数据如何分解等信息，数据库和表都有模式。

主键的值不允许修改，也不允许复用（不能将已经删除的主键值赋给新数据行的主键）。

SQL（Structured Query Language)，标准 SQL 由 ANSI 标准委员会管理，从而称为 ANSI SQL。各个 DBMS 都有自己的实现，如 PL/SQL、Transact-SQL 等。

SQL 语句不区分大小写，但是数据库表名、列名和值是否区分依赖于具体的 DBMS 以及配置。

SQL 支持以下三种注释：

```sql
# 注释
SELECT *
FROM mytable; -- 注释
/* 注释1
   注释2 */
```

数据库创建与使用：

```sql
CREATE DATABASE test;
USE test;
```

### 数据类型
数值型
- 数值型数据：都是数值
- 系统将数值型分为整数型和小数型。

整数型
存放整型数据：在SQL中因为要考虑如何节省磁盘空间，所以系统将整型又细分为了5类：
- Tinyint：迷你整形，使用1个字节存储，表示的状态最多为256种（常用）
- Smallint：小整型，使用2个字节存储，表示的状态最多为65536种
- Mediumint：中整形，使用3个字节存储
- Int：标准整型，使用4个字节存储（常用）
- Bigint：大整型，使用8个字节存储


小数型：带有小数或者范围超出整型的数值类型，SQL中，将小数型细分为两种：浮点型和定点型：

- 定点型：小数点固定，精度固定，不过会丢失精度
- 浮点型：小数点浮动，精度有限，而且会丢失精度
    因为超出指定范围之后，会丢失精度（自动四舍五入）
    浮点型：理论分为两种精度
    - 	Float：单精度，占4个字节存储数据，精度范围大概为7位左右
    - 	Double：双精度，占8个字节存储数据，精度范围大概为15位左右

### 日期时间类型
- Datatime：时间日期，格式YYYY-mm-dd HH：ii：ss，表示的范围是从1000到9999年，有零值：0000-00-00 00:00:00
- Date：日期，就是datetime中的date部分
- Time：时间（段），指定的某个区间之间，-时间（过去的某段时间），+时间（将来的某段时间）
- Timestamp：时间戳，并不是时间戳，只是从1970年开始的YYYY-mm-dd HH：ii：ss，格式与datetime完全一致
- Year：年份，有两种形式，year(2)和year(4)
### 字符串类型
在SQL中，将字符串类型分为6类：char，varchar，text，blob，enum，set

- ==定长字符串==

定长字符串：char，磁盘（二维表）在定义结构的时候，就已经确定了最终数据的存储长度。
Char(L)：L代表length，可以存储的长度，单位为字符，最大长度值可以为255。
Char(4)：在UTF8环境下，需要4 * 3 = 12个字节

- ==变长字符串==

变长字符串：varchar，在分配空间的时候，按照最大空间分配：但实际上最终用了多少，是根据具体的数据来确定的。
Varchar(L)：L代表字符长度，理论长度是65536个字符，但是会多出1~2个字节来确定存储的实际长度：但是实际上如果长度超过255，既不用定长也不用变长而是使用文本字符串Text。
Varchar[10]：的确存了10个汉字，utf8环境下，10 * 3 + 1 = 31(byts)  --多加1个字节是因为一个字节可以表示的最大长度值是255 。
>如何选择定长或者变长字符串呢？
定长的磁盘空间比较浪费，但是效率高：如果数据基本上确定长度都一样，就选择定长
如：身份证，电话号码，手机号码等
变长的磁盘空间比较节省，但是效率低：如果数据不能确定长度（不同数据有变化）
如：姓名，家庭住址等

- ==文本字符串==

如果数据量非常大，通常说超过255个字符就会使用文本字符串
文本字符串根据存储的数据格式进行分类：text和blob
    - Text:存储文字（二进制数据实际上都是存储路径）
    - Blob：存储二进制数据（通常不用）

- 枚举字符串
    - 枚举：enum，事先将所有可能出现的结果都设计好，实际上存储的数据必须是规定好的数据中的一个。
    - 枚举的使用方式
    - 定义：enum(可能出现的元素列表);    //如：enum(‘男’,’女’);
    - 使用：存储数据，只能存储上面定义好的数据

- ==集合字符串==

集合跟枚举很类似：实际存储的是数值，而不是字符串（集合是多选）

集合使用方式：
定义：Set(元素列表) 如果有8个元素，使用1个字节存储，如果有16元素，就使用2个字节来存，以此类推。

### MySQL记录长度
MySQL中规定：任何一条记录不能超过65536个字节。（varchar永远达不到其理论值）

### 属性
- ====列属性====

列属性：真正约束字段的是数据类型，但是数据类型的约束很单一，需要有一些额外的约束来更加保证数据的合法性。
列属性有很多：NULL/NOT NULL,default,primary key,unique key,auto_increment,comment

- ==空属性==

两个值：NULL（默认的）和 NOT NULL（不为空）

虽然默认的，数据库基本都是字段为空，但实际上在真实开发的时候，尽可能地要保证所有的数据都不应该为空：空数据没有意义；空数据没办法参与运算。

- ==列描述==

列描述：comment，描述，没有实际含义：是专门用来描述字段，会根据表创建语句保存：用来给程序员（数据库管理员）进行了解的。

- ==默认值==

默认值：某一种数据会经常性地出现某个具体的值，可以在一开始就指定好，在需要真实数据的时候，用户可以选择性地使用默认值

默认值的生效：使用：在数据进行插入时不给该字段赋值就ok了。

想要使用默认值，可以不一定去指定列表，故意不使用字段列表，可以使用default关键字代替值


- ==字段属性==
主键：唯一键和自增长。

主键：primary key，主要的键。一张表只能有一个字段可以使用对应的键，用来唯一约束该字段里面的数据，不能重复：这种键称之为主键。一张表只能有最多一个主键。

增加主键：
1. 在创建表的时候直接在字段之后跟 primary key关键字,优点:非常直接:缺点:只能使用一个字段作为主键
2. 在创建表的时候,在所有的字段之后,使用 primary key(主键字段列表)来创建主键(如果有多个字段作为主键可以是复合主键)
3. 当表已经创建好之后,再次额外追加主键:可以通过修改表字段属性,也可以直接追加。

- ==主键约束==

主键约束：主键对应的字段中的数据不允许重复：一旦重复，数据操作会失败。

- ==主键删除==

更新主键&删除主键
实际上，没有办法更新主键：主键必须先删除，才能增加。

```
Alter table 表名 drop primary;
```

- ==主键分类==

在实际创建表的过程中,很少使用真实业务数据作为主键字段(业务主键，如学号，课程号)
==大部分的时候是使用逻辑性的字段(字段没有业务含义,值是什么都没有关系)==，将这种字段主键称之为逻辑主键。

```
Create table my student(
Id int primary key auto_ increment comment ‘逻辑主键：自增长’ 
Number char(10) not null comment ‘学号’
Name varchar(10)not null
)
```
- ==自动增==长
自增长：当对应的字段不给值或者说给默认值，或者给NULL的时候,会自动的被系统触发，系统会从当前字段中已有的最大值再进行+1操作，得到一个新的在不同的字段。自增长通常是跟主键搭配。

自增长特点: auto_increment
1. 任何一个字段要做自增长必须前提是本身是一个索引(key一栏有值) 
2. 自增长字段必须是数字(整型) 
3. 一张表最多只能有一个自增长

自增长如果是涉及到字段改变：必须先删除自增长，后增加(一张表只能有一个自增长) 修改当前自增长已经存在的值：修改只能比当前已有的自增长的最大值大，不能小(小不生效)
```
Alter table表名 auto_increment = 值
```
自增长是字段的一个属性：可以通过modify来进行修改（保证字段没有auto_increment即可）

```
Alter table 表名 modify 字段类型;
```

- ==增加唯一键==

一张表往往有很多字段需要具有唯一性，数据不能重复：但是一张表中只能有一个主键：唯一键（unique key）就可以解决表中有多个字段需要唯一性约束的问题。

唯一键的本质与主键差不多：唯一键默认的允许字段为空，而且可以多个为空（空字段不参与唯一性比较）

增加唯一键：三种方式

1. 方案1：在创建表的时候，字段之后直接跟unique/unique key
2. 方案2：在所有的字段之后加unique key(字段列表);  --复合唯一键
3. 方案3：在创建表之后增加唯一

- ==唯一键约束==

唯一键与主键本质相同：唯一的区别就是唯一键默认允许为空，而且是多个为空。




# 二、创建表

```sql
CREATE TABLE mytable (
  # int 类型，不为空，自增
  id INT NOT NULL AUTO_INCREMENT,
  # int 类型，不可为空，默认值为 1，不为空
  col1 INT NOT NULL DEFAULT 1,
  # 变长字符串类型，最长为 45 个字符，可以为空
  col2 VARCHAR(45) NULL,
  # 日期类型，可为空
  col3 DATE NULL,
  # 设置主键为 id
  PRIMARY KEY (`id`));
```

查看：

1. 查看所有数据库

```
show databases
```

2. 查看指定部分的数据库：模糊查询

```
show databases like ‘pattern’;
```

-  pattern是匹配模式
-  %：表示匹配多个字符
-  _：表示匹配单个字符
-  当精确字段含有_时，必须进行转义，如：show databases like 'mydata\_%';查找名称以mydata_开头的数据库

3. 查看数据库的创建语句：

```
show create database 数据库名称;
```


4.查看表结构：查看表中的字段信息

```
Desc/describe/show columns from表名;
```


## 新增数据表

```
Create table [if not exists] 表名(
字段名称 数据类型,
字段名称 数据类型 –最后一行没有逗号
)[表选项];
```

- If not exists: 如果表名不存在，那么就创建，否则不执行创建代码(检查功能)
- 表选项：控制表的表现   
- 字符集：Charset/character set具体的字符集 -- 保证表中数据存储的字符集
- 校对集：Collate 具体的校对集
- 存储引擎：engine具体的存储引擎（innodb和myisam）

## 进入数据库环境

```
use 数据库名字;
```


# 三、修改表
==注意：MySQL会自动寻找分号：作为它的语句结束符。==

- 修改表本身
    1. 表本身可以修改：表名和表选项
    1. 修改表名：rename table 旧表名 to 新表名
    1. 修改表选项：字符集和存储引擎
    1. Alter table 表名 表选项 [=] 值;

- 修改字段
    1. 字段 操作很多：新增，修改，重命名，删除

```
//修改字段：修改通常是修改属性或者数据类型
Alter table 表名 modify 字段名 数据类型 [属性][位置];
//重命名字段
Alter table 表名 change 旧字段名 新字段名 数据类型[列属性][位置];
//删除字段
Alter table 表名 drop 字段名;
小心：如果表中已经存在数据，那么删除字段会清空该字段的所有数据（不可逆）
```
```sql
//添加列
ALTER TABLE mytable
ADD col CHAR(20);
```



```sql
//删除列
ALTER TABLE mytable
DROP COLUMN col;
```



```sql
//删除表
DROP TABLE mytable;
//
Drop table 表名1，表名2...;  -- 可以一次性删除多张表
```

- 新增字段

```
Alter table表名 add[column] 字段名 数据类型 [列属性][位置];
```

位置：
- 字段名乐意存放于表中的任意位置
- First：第一个位置
- After：在哪个字段之后：after 字段名;默认的是在最后一个字段之后





# 四、插入
两种方案
- 方案1：给全表字段插入数据，不需要指定字段列表，要求数据的值出现的顺序必须与表中设计的字段出现的顺序一致：凡是非数值数据，都需要使用引号（建议单引号）包裹

```
Insert into 表名 values(值列表)[,(值列表)];   --可以一次性插入多条记录
```

- 方案2：给部分字段插入数据，需要选定字段列表：字段列表出现的顺序与字段出现顺序无关；但是值列表的顺序必须与选定的字段的顺序一致。

```
Insert into 表名(字段列表) values(值列表)[,(值列表)];
```


```sql
//普通插入
INSERT INTO mytable(col1, col2) VALUES(val1, val2);
```



```sql
//插入检索出来的数据
INSERT INTO mytable1(col1, col2) SELECT col1, col2 FROM mytable2;
```



```sql
//将一个表的内容插入到一个新表
CREATE TABLE newtable AS SELECT * FROM mytable;
```

# 五、更新

```
Update 表名 set 字段 = 值[where 条件];  
-- 建议都有where：要不然就是全部更新
-- 更新不一定成功：如没有真正要更新的数据 
```

```sql
UPDATE mytable
SET col = val
WHERE id = 1;
```

# 六、删除
删除数据
```
Delete from 表名 [where条件];
```

```sql
DELETE FROM mytable
WHERE id = 1;
```

**TRUNCATE TABLE**  可以清空表，也就是删除所有行。

```sql
TRUNCATE TABLE mytable;
```

使用更新和删除操作时一定要用 WHERE 子句，不然会把整张表的数据都破坏。可以先用 SELECT 语句进行测试，防止错误删除。

# 七、查询

查看数据
```
Select */字段列表 from 表名 [where条件];
```

## DISTINCT

相同值只会出现一次。它作用于所有列，也就是说所有列的值都相同才算相同。

```sql
SELECT DISTINCT col1, col2
FROM mytable;
```

## LIMIT

限制返回的行数。可以有两个参数，第一个参数为起始行，从 0 开始；第二个参数为返回的总行数。

返回前 5 行：

```sql
SELECT *
FROM mytable
LIMIT 5;
```

```sql
SELECT *
FROM mytable
LIMIT 0, 5;
```

返回第 3 \~ 5 行：

```sql
SELECT *
FROM mytable
LIMIT 2, 3;
```

# 八、排序

-  **ASC** ：升序（默认）
-  **DESC** ：降序

可以按多个列进行排序，并且为每个列指定不同的排序方式：

```sql
SELECT *
FROM mytable
ORDER BY col1 DESC, col2 ASC;
```

# 九、过滤

不进行过滤的数据非常大，导致通过网络传输了多余的数据，从而浪费了网络带宽。因此尽量使用 SQL 语句来过滤不必要的数据，而不是传输所有的数据到客户端中然后由客户端进行过滤。

```sql
SELECT *
FROM mytable
WHERE col IS NULL;
```

下表显示了 WHERE 子句可用的操作符

|  操作符 | 说明  |
| :---: | :---: |
| = | 等于 |
| &lt; | 小于 |
| &gt; | 大于 |
| &lt;&gt; != | 不等于 |
| &lt;= !&gt; | 小于等于 |
| &gt;= !&lt; | 大于等于 |
| BETWEEN | 在两个值之间 |
| IS NULL | 为 NULL 值 |

应该注意到，NULL 与 0、空字符串都不同。

**AND 和 OR**  用于连接多个过滤条件。优先处理 AND，当一个过滤表达式涉及到多个 AND 和 OR 时，可以使用 () 来决定优先级，使得优先级关系更清晰。

**IN**  操作符用于匹配一组值，其后也可以接一个 SELECT 子句，从而匹配子查询得到的一组值。

**NOT**  操作符用于否定一个条件。

# 十、通配符

通配符也是用在过滤语句中，但它只能用于文本字段。

-  **%**  匹配 >=0 个任意字符；

-  **\_**  匹配 ==1 个任意字符；

-  **[ ]**  可以匹配集合内的字符，例如 [ab] 将匹配字符 a 或者 b。用脱字符 ^ 可以对其进行否定，也就是不匹配集合内的字符。

使用 Like 来进行通配符匹配。

```sql
SELECT *
FROM mytable
WHERE col LIKE '[^AB]%'; -- 不以 A 和 B 开头的任意文本
```

不要滥用通配符，通配符位于开头处匹配会非常慢。

# 十一、计算字段

在数据库服务器上完成数据的转换和格式化的工作往往比客户端上快得多，并且转换和格式化后的数据量更少的话可以减少网络通信量。

计算字段通常需要使用  **AS**  来取别名，否则输出的时候字段名为计算表达式。

```sql
SELECT col1 * col2 AS alias
FROM mytable;
```

**CONCAT()**  用于连接两个字段。许多数据库会使用空格把一个值填充为列宽，因此连接的结果会出现一些不必要的空格，使用 **TRIM()** 可以去除首尾空格。

```sql
SELECT CONCAT(TRIM(col1), '(', TRIM(col2), ')') AS concat_col
FROM mytable;
```

# 十二、函数

各个 DBMS 的函数都是不相同的，因此不可移植，以下主要是 MySQL 的函数。

## 汇总

|函 数 |说 明|
| :---: | :---: |
| AVG() | 返回某列的平均值 |
| COUNT() | 返回某列的行数 |
| MAX() | 返回某列的最大值 |
| MIN() | 返回某列的最小值 |
| SUM() |返回某列值之和 |

AVG() 会忽略 NULL 行。

使用 DISTINCT 可以汇总不同的值。

```sql
SELECT AVG(DISTINCT col1) AS avg_col
FROM mytable;
```

## 文本处理

| 函数  | 说明  |
| :---: | :---: |
|  LEFT() |  左边的字符 |
| RIGHT() | 右边的字符 |
| LOWER() | 转换为小写字符 |
| UPPER() | 转换为大写字符 |
| LTRIM() | 去除左边的空格 |
| RTRIM() | 去除右边的空格 |
| LENGTH() | 长度 |
| SOUNDEX() | 转换为语音值 |

其中， **SOUNDEX()**  可以将一个字符串转换为描述其语音表示的字母数字模式。

```sql
SELECT *
FROM mytable
WHERE SOUNDEX(col1) = SOUNDEX('apple')
```

## 日期和时间处理


- 日期格式：YYYY-MM-DD
- 时间格式：HH:<zero-width space>MM:SS

|函 数 | 说 明|
| :---: | :---: |
| ADDDATE() | 增加一个日期（天、周等）|
| ADDTIME() | 增加一个时间（时、分等）|
| CURDATE() | 返回当前日期 |
| CURTIME() | 返回当前时间 |
| DATE() |返回日期时间的日期部分|
| DATEDIFF() |计算两个日期之差|
| DATE_ADD() |高度灵活的日期运算函数|
| DATE_FORMAT() |返回一个格式化的日期或时间串|
| DAY()| 返回一个日期的天数部分|
| DAYOFWEEK() |对于一个日期，返回对应的星期几|
| HOUR() |返回一个时间的小时部分|
| MINUTE() |返回一个时间的分钟部分|
| MONTH() |返回一个日期的月份部分|
| NOW() |返回当前日期和时间|
| SECOND() |返回一个时间的秒部分|
| TIME() |返回一个日期时间的时间部分|
| YEAR() |返回一个日期的年份部分|

```sql
mysql> SELECT NOW();
```

```
2018-4-14 20:25:11
```

## 数值处理

| 函数 | 说明 |
| :---: | :---: |
| SIN() | 正弦 |
| COS() | 余弦 |
| TAN() | 正切 |
| ABS() | 绝对值 |
| SQRT() | 平方根 |
| MOD() | 余数 |
| EXP() | 指数 |
| PI() | 圆周率 |
| RAND() | 随机数 |

# 十三、分组

把具有相同的数据值的行放在同一组中。

可以对同一分组数据使用汇总函数进行处理，例如求分组数据的平均值等。

指定的分组字段除了能按该字段进行分组，也会自动按该字段进行排序。

```sql
SELECT col, COUNT(*) AS num
FROM mytable
GROUP BY col;
```

GROUP BY 自动按分组字段进行排序，ORDER BY 也可以按汇总字段来进行排序。

```sql
SELECT col, COUNT(*) AS num
FROM mytable
GROUP BY col
ORDER BY num;
```

WHERE 过滤行，HAVING 过滤分组，行过滤应当先于分组过滤。

```sql
SELECT col, COUNT(*) AS num
FROM mytable
WHERE col > 2
GROUP BY col
HAVING num >= 2;
```

分组规定：

- GROUP BY 子句出现在 WHERE 子句之后，ORDER BY 子句之前；
- 除了汇总字段外，SELECT 语句中的每一字段都必须在 GROUP BY 子句中给出；
- NULL 的行会单独分为一组；
- 大多数 SQL 实现不支持 GROUP BY 列具有可变长度的数据类型。

# 十四、子查询

子查询中只能返回一个字段的数据。

可以将子查询的结果作为 WHRER 语句的过滤条件：

```sql
SELECT *
FROM mytable1
WHERE col1 IN (SELECT col2
               FROM mytable2);
```

下面的语句可以检索出客户的订单数量，子查询语句会对第一个查询检索出的每个客户执行一次：

```sql
SELECT cust_name, (SELECT COUNT(*)
                   FROM Orders
                   WHERE Orders.cust_id = Customers.cust_id)
                   AS orders_num
FROM Customers
ORDER BY cust_name;
```

# 十五、连接

连接用于连接多个表，使用 JOIN 关键字，并且条件语句使用 ON 而不是 WHERE。

连接可以替换子查询，并且比子查询的效率一般会更快。

可以用 AS 给列名、计算字段和表名取别名，给表名取别名是为了简化 SQL 语句以及连接相同表。

## 内连接

内连接又称等值连接，使用 INNER JOIN 关键字。

```sql
SELECT A.value, B.value
FROM tablea AS A INNER JOIN tableb AS B
ON A.key = B.key;
```

可以不明确使用 INNER JOIN，而使用普通查询并在 WHERE 中将两个表中要连接的列用等值方法连接起来。

```sql
SELECT A.value, B.value
FROM tablea AS A, tableb AS B
WHERE A.key = B.key;
```

## 自连接

自连接可以看成内连接的一种，只是连接的表是自身而已。

一张员工表，包含员工姓名和员工所属部门，要找出与 Jim 处在同一部门的所有员工姓名。

子查询版本

```sql
SELECT name
FROM employee
WHERE department = (
      SELECT department
      FROM employee
      WHERE name = "Jim");
```

自连接版本

```sql
SELECT e1.name
FROM employee AS e1 INNER JOIN employee AS e2
ON e1.department = e2.department
      AND e2.name = "Jim";
```

## 自然连接

自然连接是把同名列通过等值测试连接起来的，同名列可以有多个。

内连接和自然连接的区别：内连接提供连接的列，而自然连接自动连接所有同名列。

```sql
SELECT A.value, B.value
FROM tablea AS A NATURAL JOIN tableb AS B;
```

## 外连接

外连接保留了没有关联的那些行。分为左外连接，右外连接以及全外连接，左外连接就是保留左表没有关联的行。

检索所有顾客的订单信息，包括还没有订单信息的顾客。

```sql
SELECT Customers.cust_id, Orders.order_num
FROM Customers LEFT OUTER JOIN Orders
ON Customers.cust_id = Orders.cust_id;
```

customers 表：

| cust_id | cust_name |
| :---: | :---: |
| 1 | a |
| 2 | b |
| 3 | c |

orders 表：

| order_id | cust_id |
| :---: | :---: |
|1    | 1 |
|2    | 1 |
|3    | 3 |
|4    | 3 |

结果：

| cust_id | cust_name | order_id |
| :---: | :---: | :---: |
| 1 | a | 1 |
| 1 | a | 2 |
| 3 | c | 3 |
| 3 | c | 4 |
| 2 | b | Null |

# 十六、组合查询

使用  **UNION**  来组合两个查询，如果第一个查询返回 M 行，第二个查询返回 N 行，那么组合查询的结果一般为 M+N 行。

每个查询必须包含相同的列、表达式和聚集函数。

默认会去除相同行，如果需要保留相同行，使用 UNION ALL。

只能包含一个 ORDER BY 子句，并且必须位于语句的最后。

```sql
SELECT col
FROM mytable
WHERE col = 1
UNION
SELECT col
FROM mytable
WHERE col =2;
```

# 十七、视图

视图是虚拟的表，本身不包含数据，也就不能对其进行索引操作。

对视图的操作和对普通表的操作一样。

视图具有如下好处：

- 简化复杂的 SQL 操作，比如复杂的连接；
- 只使用实际表的一部分数据；
- 通过只给用户访问视图的权限，保证数据的安全性；
- 更改数据格式和表示。

```sql
CREATE VIEW myview AS
SELECT Concat(col1, col2) AS concat_col, col3*col4 AS compute_col
FROM mytable
WHERE col5 = val;
```

# 十八、存储过程

存储过程可以看成是对一系列 SQL 操作的批处理。

使用存储过程的好处：

- 代码封装，保证了一定的安全性；
- 代码复用；
- 由于是预先编译，因此具有很高的性能。

命令行中创建存储过程需要自定义分隔符，因为命令行是以 ; 为结束符，而存储过程中也包含了分号，因此会错误把这部分分号当成是结束符，造成语法错误。

包含 in、out 和 inout 三种参数。

给变量赋值都需要用 select into 语句。

每次只能给一个变量赋值，不支持集合的操作。

```sql
delimiter //

create procedure myprocedure( out ret int )
    begin
        declare y int;
        select sum(col1)
        from mytable
        into y;
        select y*y into ret;
    end //

delimiter ;
```

```sql
call myprocedure(@ret);
select @ret;
```

# 十九、游标

在存储过程中使用游标可以对一个结果集进行移动遍历。

游标主要用于交互式应用，其中用户需要对数据集中的任意行进行浏览和修改。

使用游标的四个步骤：

1. 声明游标，这个过程没有实际检索出数据；
2. 打开游标；
3. 取出数据；
4. 关闭游标；

```sql
delimiter //
create procedure myprocedure(out ret int)
    begin
        declare done boolean default 0;

        declare mycursor cursor for
        select col1 from mytable;
        # 定义了一个 continue handler，当 sqlstate '02000' 这个条件出现时，会执行 set done = 1
        declare continue handler for sqlstate '02000' set done = 1;

        open mycursor;

        repeat
            fetch mycursor into ret;
            select ret;
        until done end repeat;

        close mycursor;
    end //
 delimiter ;
```

# 二十、触发器

触发器会在某个表执行以下语句时而自动执行：DELETE、INSERT、UPDATE。

触发器必须指定在语句执行之前还是之后自动执行，之前执行使用 BEFORE 关键字，之后执行使用 AFTER 关键字。BEFORE 用于数据验证和净化，AFTER 用于审计跟踪，将修改记录到另外一张表中。

INSERT 触发器包含一个名为 NEW 的虚拟表。

```sql
CREATE TRIGGER mytrigger AFTER INSERT ON mytable
FOR EACH ROW SELECT NEW.col into @result;

SELECT @result; -- 获取结果
```

DELETE 触发器包含一个名为 OLD 的虚拟表，并且是只读的。

UPDATE 触发器包含一个名为 NEW 和一个名为 OLD 的虚拟表，其中 NEW 是可以被修改的，而 OLD 是只读的。

MySQL 不允许在触发器中使用 CALL 语句，也就是不能调用存储过程。

# 二十一、事务管理

基本术语：

- 事务（transaction）指一组 SQL 语句；
- 回退（rollback）指撤销指定 SQL 语句的过程；
- 提交（commit）指将未存储的 SQL 语句结果写入数据库表；
- 保留点（savepoint）指事务处理中设置的临时占位符（placeholder），你可以对它发布回退（与回退整个事务处理不同）。

不能回退 SELECT 语句，回退 SELECT 语句也没意义；也不能回退 CREATE 和 DROP 语句。

MySQL 的事务提交默认是隐式提交，每执行一条语句就把这条语句当成一个事务然后进行提交。当出现 START TRANSACTION 语句时，会关闭隐式提交；当 COMMIT 或 ROLLBACK 语句执行后，事务会自动关闭，重新恢复隐式提交。

设置 autocommit 为 0 可以取消自动提交；autocommit 标记是针对每个连接而不是针对服务器的。

如果没有设置保留点，ROLLBACK 会回退到 START TRANSACTION 语句处；如果设置了保留点，并且在 ROLLBACK 中指定该保留点，则会回退到该保留点。

```sql
START TRANSACTION
// ...
SAVEPOINT delete1
// ...
ROLLBACK TO delete1
// ...
COMMIT
```

# 二十二、字符集

基本术语：

- 字符集为字母和符号的集合；
- 编码为某个字符集成员的内部表示；
- 校对字符指定如何比较，主要用于排序和分组。

除了给表指定字符集和校对外，也可以给列指定：

```sql
CREATE TABLE mytable
(col VARCHAR(10) CHARACTER SET latin COLLATE latin1_general_ci )
DEFAULT CHARACTER SET hebrew COLLATE hebrew_general_ci;
```

可以在排序、分组时指定校对：

```sql
SELECT *
FROM mytable
ORDER BY col COLLATE latin1_general_ci;
```

# 二十三、权限管理

MySQL 的账户信息保存在 mysql 这个数据库中。

```sql
USE mysql;
SELECT user FROM user;
```

**创建账户** 

新创建的账户没有任何权限。

```sql
CREATE USER myuser IDENTIFIED BY 'mypassword';
```

**修改账户名** 

```sql
RENAME myuser TO newuser;
```

**删除账户** 

```sql
DROP USER myuser;
```

**查看权限** 

```sql
SHOW GRANTS FOR myuser;
```

**授予权限** 

账户用 username@host 的形式定义，username@% 使用的是默认主机名。

```sql
GRANT SELECT, INSERT ON mydatabase.* TO myuser;
```

**删除权限** 

GRANT 和 REVOKE 可在几个层次上控制访问权限：

- 整个服务器，使用 GRANT ALL 和 REVOKE ALL；
- 整个数据库，使用 ON database.\*；
- 特定的表，使用 ON database.table；
- 特定的列；
- 特定的存储过程。

```sql
REVOKE SELECT, INSERT ON mydatabase.* FROM myuser;
```

**更改密码** 

必须使用 Password() 函数进行加密。

```sql
SET PASSWROD FOR myuser = Password('new_password');
```





