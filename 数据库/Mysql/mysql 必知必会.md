[toc]
## 第3章 使用MySQL


```
# 使用指定的数据库
use mysql_need_know;

# 显示所有的数据库
show databases;

# 显示所有数据库中的所有表
show tables;

# 显示指定表的所有列信息
show columns from customers;
desc customers;
describe customers;

# 显示mysql服务状态信息
show status;

# 显示指定的数据库或者数据表的创建SQL语句
SHOW CREATE DATABASE mysql_need_know;
SHOW CREATE TABLE customers;

# 显示授予用户的安全权限
SHOW GRANTS;

# 显示错误信息
SHOW ERRORS;

# 显示警告信息
SHOW WARNINGs;
```

## 第4章 检索数据

```
# 从指定表中查询所有的列的信息
SELECT prod_name FROM products;

# 从指定表中查询指定列的信息
SELECT prod_name FROM products;

# 从指定表中查询多个列的信息
SELECT prod_id, prod_name, prod_price FROM products;

# 去重(会应用于所有的列, 而不是只有第一列)
SELECT DISTINCT vend_id, prod_price FROM products;

# 限制查询返回的行数(一个参数为返回的行数)
SELECT prod_name FROM products LIMIT 5;
# 限制查询返回的行数(二个参数中第一个为跳过的行数, 第二个参数为要显示的行数)
SELECT prod_name FROM products LIMIT 5, 5;
# 从第0行开始取5行返回, 和上面的语句相反
SELECT prod_name FROM products LIMIT 5 OFFSET 0;

# 全限定表名和列名
SELECT products.prod_name FROM mysql_need_know.products;
```

## 第5章 排序检索数据


```
# 排序(默认正序)
SELECT prod_name FROM products ORDER BY prod_name;
# 正序
SELECT prod_name FROM products ORDER BY prod_name ASC;
# 逆序
SELECT prod_name FROM products ORDER BY prod_name DESC;
# 只对其前面的列名排序
SELECT prod_id, prod_price, prod_name FROM products ORDER BY prod_price DESC, prod_name;


# 多列排序，会按列的顺序排，先排价格，如果有价格相同的行，这些行再按姓名排
SELECT prod_id, prod_price, prod_name FROM products ORDER BY prod_price, prod_name;


# 限制和排序结合，找出指定列的最大值和最小值（LIMIT要在ORDER BY子句之后）
SELECT prod_price FROM products ORDER BY prod_price LIMIT 1;
SELECT prod_price FROM products ORDER BY prod_price DESC LIMIT 1;
```

## 第6章 过滤数据


```
SELECT prod_name, prod_price FROM products WHERE prod_price = 2.50;

# WHERE 子句操作符 =, !=, <>, <, <=, >, >=, BETWEEN a AND b （包含a和b）
# 大小写不区分（字符串要用小括号括起来）
SELECT prod_name, prod_price FROM products WHERE prod_name = 'fuses';

# 检测指定列是否包含null值
SELECT prod_name FROM products WHERE prod_price IS NULL;
SELECT cust_id FROM customers WHERE cust_email IS NULL;
```

## 第7章 数据过滤


```
# 多条件WHERE过滤
# AND
SELECT prod_id, prod_price, prod_name FROM products WHERE vend_id = 1003 AND prod_price <= 10;

# OR 
SELECT vend_id, prod_name, prod_price FROM products WHERE vend_id = 1002 OR vend_id = 1003;

# 组合OR和AND（下面是错误的组合方法，AND的优先级比OR要高）
SELECT prod_name, prod_price FROM products WHERE vend_id = 1002 OR vend_id = 1003 AND prod_price >= 10;

# 组合OR和AND（使用括号提升包含OR的筛选条件的优先级）
SELECT prod_name, prod_price FROM products WHERE (vend_id = 1002 OR vend_id = 1003) AND prod_price >= 10;

# IN 指定条件范围
SELECT vend_id, prod_name, prod_price FROM products WHERE vend_id IN (1002, 1003) ORDER BY prod_name;
# 使用OR和上面用IN的效果一样，但是IN更快
SELECT vend_id, prod_name, prod_price FROM products WHERE vend_id = 1002 OR vend_id = 1003;
# NOT 否定后面跟的条件
SELECT vend_id, prod_name, prod_price FROM products WHERE vend_id NOT IN (1002, 1003) ORDER BY prod_name;
```

## 第8章 用通配符进行过滤


```
# 通配符 % 表示任何字符出现任何次数(当前是不区分大小写的，可以配置区分大小写)
SELECT prod_id, prod_name FROM products WHERE prod_name LIKE 'jet%';
# 使用多个通配符
SELECT prod_id, prod_name FROM products WHERE prod_name LIKE '%anvil%';

SELECT prod_name FROM products WHERE prod_name LIKE 's%e';

# 注意，用%不能匹配null

# 下划线(_)匹配一个字符
SELECT prod_id, prod_name FROM products WHERE prod_name LIKE '_ ton anvil';
```
```
## 第9章 用正则表达式进行搜索

# 在子句中使用正则匹配
SELECT prod_name FROM products WHERE prod_name REGEXP '1000' ORDER BY prod_name;

# 使用.匹配任意一个字符
SELECT prod_name FROM products WHERE prod_name REGEXP '.000' ORDER BY prod_name;
# 区分大小写
SELECT prod_name FROM products WHERE prod_name REGEXP BINARY 'Jet' ORDER BY prod_name;
# 匹配两个串(类似OR的功能)
SELECT prod_name FROM products WHERE prod_name REGEXP '1000|2000' ORDER BY prod_name;
# 匹配一组字符
SELECT prod_name FROM products WHERE prod_name REGEXP '[123] Ton' ORDER BY prod_name;

# 错误的匹配，这等于匹配 1 或者 2 或者 3 Ton
SELECT prod_name FROM products WHERE prod_name REGEXP '1|2|3 Ton' ORDER BY prod_name;
# 否匹配
SELECT prod_name FROM products WHERE prod_name REGEXP '[^123] Ton' ORDER BY prod_name;
# 匹配范围
SELECT prod_name FROM products WHERE prod_name REGEXP '[1-5] Ton' ORDER BY prod_name;
# 匹配特殊字符(在特殊字符前加双斜杠(\\))
SELECT vend_name FROM vendors WHERE vend_name REGEXP '\\.' ORDER BY vend_name;

#匹配字符类(预定义的字符集)
# 任意字母或者数字
SELECT vend_name FROM vendors WHERE vend_name REGEXP '[[:alnum:]]' ORDER BY vend_name;
#任意字符
SELECT vend_name FROM vendors WHERE vend_name REGEXP '[[:alpha:]]' ORDER BY vend_name;
#任意数字
SELECT vend_name FROM vendors WHERE vend_name REGEXP '[[:digit:]]' ORDER BY vend_name;
#任意小写字母
SELECT vend_name FROM vendors WHERE vend_name REGEXP '[[:lower:]]' ORDER BY vend_name;
#任意大写字母
SELECT vend_name FROM vendors WHERE vend_name REGEXP '[[:upper:]]' ORDER BY vend_name;

SELECT prod_name FROM products WHERE prod_name REGEXP '\\([0-9] sticks?\\)' ORDER BY prod_name;

# 匹配连在一起的4位数字
SELECT prod_name FROM products WHERE prod_name REGEXP '[[:digit:]]{4}' ORDER BY prod_name;

# 使用定位符 匹配以数字或者.开关的所有产品
SELECT prod_name FROM products WHERE prod_name REGEXP '^[0-9\\.]' ORDER BY prod_name;


# 不使用数据库检测正则
SELECT 'hello1' REGEXP '[0-9]';
```


## 第10章 创建计算字段
```

# 使用concat()函数拼接列
SELECT CONCAT(vend_name, ' (', vend_country, ')') AS vend_title FROM vendors ORDER BY vend_name;

# 使用RTRIM()函数来删除字段右边的空格
SELECT CONCAT(RTRIM(vend_name), '(', RTRIM(vend_country), ')') AS vend_title FROM vendors ORDER BY vend_name;

# 查询指定订单号的物品项并计算每项的总价
SELECT
    prod_id,
    quantity,
    item_price,
    quantity * item_price AS expanded_price
FROM
    orderitems 
WHERE
    order_num = 20005;

# 常用检测方法
SELECT 3 * 2;
SELECT TRIM('abc');
SELECT NOW();
```

第11章 使用数据处理函数
```
# 转换大小写函数
SELECT vend_name, UPPER(vend_name) AS vend_name_upcase FROM vendors ORDER BY vend_name;
SELECT vend_name, LOWER(vend_name) AS vend_name_lower FROM vendors ORDER BY vend_name;

# 返回串左边和返回串右边指定位数的字符
SELECT vend_name, LEFT(vend_name, 5) AS vend_name_left FROM vendors ORDER BY vend_name;
SELECT vend_name, RIGHT(vend_name, 5) AS vend_name_right FROM vendors ORDER BY vend_name;

# 返回串的长度
SELECT vend_name, LENGTH(vend_name) AS vend_name_length FROM vendors ORDER BY vend_name;

# 返回串的子串第一次出现的位置
SELECT vend_name, LOCATE('A', vend_name) AS vend_name_locate FROM vendors ORDER BY vend_name;

# 返回子串的字符(截取子串)
SELECT vend_name, SUBSTRING(vend_name, 1, 5) AS vend_name_locate FROM vendors ORDER BY vend_name;

# 返回串的SOUNDEX值
SELECT vend_name, SOUNDEX(vend_name) AS vend_name_locate FROM vendors ORDER BY vend_name;
# 发音相近匹配
SELECT cust_name, cust_contact FROM customers WHERE SOUNDEX(cust_contact) = SOUNDEX('Y. Lie');


SELECT CURDATE(); # 当前日期
SELECT CURTIME(); # 当前时间

SELECT DATEDIFF('2018-10-10', "2018-10-11"); # 比较日期(计算日期差值)
SELECT ADDDATE(NOW(), 10); # 给指定的日期增加指定的天数
SELECT OrderId,DATEADD(day,2,OrderDate) AS OrderPayDate FROM Orders # 向OrderDate增加2天
SELECT DATE_ADD(NOW(), INTERVAL 60 SECOND) # 高精度增加时间
SELECT DATE_FORMAT(NOW(),'%Y-%m-%d %H:%i:%S'); # 使用指定格式格式化日期
SELECT DATE('2018-10-10 12:12'); # 截取日期部分
SELECT TIME('2018-10-10 12:12:12'); #返回日期的时间部分
SELECT YEAR('2018-10-10 12:12:12'); #返回日期的年份部分
SELECT MONTH('2018-10-10 12:12:12'); #返回日期的月份部分
SELECT DAY('2018-10-10 12:12:12'); #返回日期的天数部分
SELECT HOUR('2018-10-10 12:12:12'); #返回日期的时针部分
SELECT MINUTE('2018-10-10 12:12:12'); #返回日期的分针部分
SELECT SECOND('2018-10-10 12:12:12'); #返回日期的秒部分

# 根据日期筛选
SELECT cust_id, order_num FROM orders WHERE order_date = '2005-09-01';
# 更可靠的根据日期筛选
SELECT cust_id, order_num FROM orders WHERE DATE(order_date) = '2005-09-01';
# 筛选一个日期范围内的结果
SELECT cust_id, order_num FROM orders WHERE DATE(order_date) BETWEEN '2005-09-01' AND '2005-09-30';
# 更可靠的日期范围筛选
SELECT cust_id, order_num FROM orders WHERE Year(order_date) = 2005 AND Month(order_date) = 9;

# 返回0-1之间不包括1的随机数
SELECT RAND();
```
## 第12章 汇总数据
```
# 返回指定列的平均值
SELECT AVG(prod_price) AS avg_price FROM products;
# 返回指定列符合条件的行的平均值
SELECT AVG(prod_price) AS avg_price FROM products WHERE vend_id = 1003;

# 计算指定表的行数(包括null值行)
SELECT COUNT(*) AS num_cust FROM customers;

# 计算指定列的行数(不包括null值行)
SELECT COUNT(cust_email) AS num_cust FROM customers;

# 返回指定列的最大值（忽略null值）
SELECT MAX(prod_price) AS max_price FROM products;

# 返回指定列的最小值（忽略null值）
SELECT MIN(prod_price) AS min_price FROM products;

# 对指定列求合
SELECT SUM(quantity) AS items_ordered FROM orderitems WHERE order_num = 20005;
# 在求全函数中计算
SELECT SUM(quantity*item_price) AS total_price FROM orderitems WHERE order_num = 20005;

# 在函数中使用DISTINCT
SELECT AVG(DISTINCT prod_price) AS avg_price FROM products WHERE vend_id = 1003;

# 使用多个函数
SELECT
    COUNT( * ) AS num_items,
    MIN( prod_price ) AS price_min,
    MAX( prod_price ) AS price_max,
    AVG( prod_price ) AS price_avg 
FROM
    products;
```

## 第13章 分组数据

```
# 使用分组计算所有供应商对应都有几个产品
SELECT vend_id, COUNT(*) AS num_prods FROM products GROUP BY vend_id;

-- 在具体使用 GROUP BY 子句前，需要知道一些重要的规定。
--  GROUP BY 子句可以包含任意数目的列。这使得能对分组进行嵌套，
-- 为数据分组提供更细致的控制。
--  如果在 GROUP BY 子句中嵌套了分组，数据将在最后规定的分组上
-- 进行汇总。换句话说，在建立分组时，指定的所有列都一起计算
-- （所以不能从个别的列取回数据）。
--  GROUP BY 子句中列出的每个列都必须是检索列或有效的表达式
-- （但不能是聚集函数）。如果在 SELECT 中使用表达式，则必须在
-- GROUP BY 子句中指定相同的表达式。不能使用别名。
--  除聚集计算语句外， SELECT 语句中的每个列都必须在 GROUP BY 子
-- 句中给出。
--  如果分组列中具有 NULL 值，则 NULL 将作为一个分组返回。如果列
-- 中有多行 NULL 值，它们将分为一组。
--  GROUP BY 子句必须出现在 WHERE 子句之后， ORDER BY 子句之前

# 对分组汇总
SELECT vend_id, COUNT(*) AS num_prods FROM products GROUP BY vend_id WITH ROLLUP;

# 使用HAVING过滤分组
SELECT cust_id, COUNT(*) AS orders FROM orders GROUP BY cust_id HAVING COUNT(*) >= 2;

# 查询出具有2个或以上，价格为10或以上产品的供应商
SELECT vend_id, COUNT(*) AS num_prods FROM products WHERE prod_price >= 10 GROUP BY vend_id HAVING COUNT(*) >= 2;

SELECT vend_id, COUNT(*) AS num_prods FROM products GROUP BY vend_id HAVING COUNT(*) >= 2;

# GROUP BY 和 ORDER BY 结合使用
SELECT order_num, SUM(quantity*item_price) AS ordertotal FROM orderitems GROUP BY order_num HAVING SUM(quantity*item_price) >= 50 ORDER BY ordertotal;


# SELECT子句的顺序
SELECT > FROM > WHERE > GROUP BY > HAVING > ORDER BY > LIMIT;
```
## 第14章 使用子查询
```
# 列出订购物品 TNT2 的所有客户
-- 方式一：分条查询
SELECT order_num FROM orderitems WHERE prod_id = 'TNT2';
SELECT cust_id FROM orders WHERE order_num IN (20005, 20007);

-- 方法二：子查询
SELECT cust_name, cust_contact FROM customers WHERE cust_id IN (
        SELECT cust_id FROM orders WHERE    order_num IN ( 
                    SELECT order_num FROM orderitems WHERE prod_id = 'TNT2'));


# 计算字段中使用子查询
SELECT
    cust_name,
    cust_state,
    ( SELECT COUNT( * ) FROM orders WHERE orders.cust_id = customers.cust_id ) AS orders 
FROM
    customers 
ORDER BY
    cust_name;
```
## 第15章 联结表
```
# 联结查询（等值联结果或者内部连接）
SELECT
    vend_name,
    prod_name,
    prod_price 
FROM
    vendors,
    products 
WHERE
    vendors.vend_id = products.vend_id 
ORDER BY
    vend_name,
    prod_name;
    
# 内部连接的另一种更明确的写法
SELECT vend_name, prod_name, prod_price FROM vendors INNER JOIN products ON vendors.vend_id = products.vend_id;


# 同时连接多个表
SELECT
    prod_name,
    vend_name,
    prod_price,
    quantity 
FROM
    orderitems,
    products,
    vendors 
WHERE
    products.vend_id = vendors.vend_id 
    AND orderitems.prod_id = products.prod_id 
    AND order_num = 20005;
```
## 第16章 创建高级联结
```
# 在连接查询中使用别名
SELECT
    cust_name,
    cust_contact 
FROM
    customers AS c,
    orders AS o,
    orderitems AS oi 
WHERE
    c.cust_id = o.cust_id 
    AND oi.order_num = o.order_num 
    AND prod_id = 'TNT2';

# 自连接
SELECT p1.prod_id, p1.prod_name
FROM products AS p1, products AS p2
WHERE p1.vend_id = p2.vend_id
AND p2.prod_id = 'DTNTR';

# 自然连接 排除多次出现，使每个列只返回一次 (一般内部连接都是自然连接)
SELECT
    c.*,
    o.order_num,
    o.order_date,
    oi.prod_id,
    oi.quantity,
    oi.item_price 
FROM
    customers AS c,
    orders AS o,
    orderitems AS oi 
WHERE
    c.cust_id = o.cust_id 
    AND oi.order_num = o.order_num 
    AND prod_id = 'FB';


# 外部连接(左外和右外)
SELECT customers.cust_id, orders.order_num FROM customers LEFT OUTER JOIN orders ON customers.cust_id = orders.cust_id;
SELECT customers.cust_id, orders.order_num FROM customers RIGHT OUTER JOIN orders ON orders.cust_id = customers.cust_id;


# 带聚集函数的联结
SELECT
    customers.cust_name,
    customers.cust_id,
    COUNT(orders.order_num) AS num_ord 
FROM
    customers
    INNER JOIN orders ON customers.cust_id = orders.cust_id 
GROUP BY
    customers.cust_id;

SELECT
    customers.cust_name,
    customers.cust_id,
    COUNT( orders.order_num ) AS num_ord 
FROM
    customers
    LEFT OUTER JOIN orders ON customers.cust_id = orders.cust_id 
GROUP BY
    customers.cust_id;
```
## 第17章 组合查询

```
# 使用UNION组合多条语句查询
SELECT vend_id, prod_id, prod_price FROM products WHERE prod_price <= 5
UNION
SElECT vend_id, prod_id, prod_price FROM products WHERE vend_id IN (1001, 1002);

# 不自动取消重复行
SELECT vend_id, prod_id, prod_price FROM products WHERE prod_price <= 5
UNION ALL
SElECT vend_id, prod_id, prod_price FROM products WHERE vend_id IN (1001, 1002);

# 只能在最后一条SQL后写排序
SELECT vend_id, prod_id, prod_price FROM products WHERE prod_price <= 5
UNION ALL
SElECT vend_id, prod_id, prod_price FROM products WHERE vend_id IN (1001, 1002);
ORDER BY vend_id, prod_price;
```
## 第18章 全文本搜索
```
# MyISAM支持全文搜索, InnoDB不支持全文搜索
#为了进行全文本搜索，必须索引被搜索的列，而且要随着数据的改变不断地重新索引
# 创建表时使用FULLTEXT对指定的列进行索引

# 使用函数进行搜索
SELECT note_text FROM productnotes WHERE Match(note_text) Against('rabbit');
# 使用LIKE进行搜索
SELECt note_text FROM productnotes WHERE note_text LIKE '%rabbit%';

# 观察搜索结果排序
SELECT note_text, Match(note_text) Against('rabbit') AS rank FROM productnotes;


# 不使用查询扩展
SELECT note_text FROM productnotes WHERE Match(note_text) Against('anvils');

# 使用查询扩展(会扫描两遍，第一遍精确查找，第二遍模糊查找)
SELECT note_text FROM productnotes WHERE Match(note_text) Against('anvils' WITH QUERY EXPANSION);

# 布尔文本搜索
SELECT note_text FROM productnotes WHERE Match(note_text) Against('heavy' IN BOOLEAN MODE);
SELECT note_text FROM productnotes WHERE Match(note_text) Against('heavy -rope*' IN BOOLEAN MODE);

# 搜索匹配包含词 rabbit 和 bait 的行
SELECT note_text FROM productnotes WHERE Match(note_text) Against('+rabbit +bait' IN BOOLEAN MODE);
# 没有指定操作符，这个搜索匹配包含 rabbit 和 bait 中的至少一个词的行
SELECT note_text FROM productnotes WHERE Match(note_text) Against('rabbit bait' IN BOOLEAN MODE);
# 这个搜索匹配短语 rabbit bait 而不是匹配两个词 rabbit 和bait
SELECT note_text FROM productnotes WHERE Match(note_text) Against('"rabbit bait"' IN BOOLEAN MODE);
# 匹配 rabbit 和 carrot ，增加前者的等级，降低后者的等级
SELECT note_text FROM productnotes WHERE Match(note_text) Against('>rabbit <bait' IN BOOLEAN MODE);
# 这个搜索匹配词 safe 和 combination ，降低后者的等级
SELECT note_text FROM productnotes WHERE Match(note_text) Against('"+safe +(<combination)' IN BOOLEAN MODE);
```
## 第19章 插入数据
```
# 在指定的表中插入一行数据(INSERT语句一般不会产生输出, 但会返回影响的行数)
# 这种方式不保险, 哪里列的顺序改变了就会出错
INSERT INTO Customers
VALUES
    ( NULL, 'Pep E.LaPew', '100 Main Street', 'Los Angeles', 'CA', '90046', 'USA', 'NULL', 'NULL' );

# 指定列名插入, 即使以后列的顺序改变了也不会出错
INSERT INTO customers ( cust_name, cust_address, cust_city, cust_state, cust_zip, cust_country, cust_contact, cust_email )
VALUES
    ( 'Pep E. LaPew', '100 Main Street', 'Los Angeles', 'CA', '90046', 'USA', NULL, NULL);


# 插入一行, 只插入指定的列值, 其它为默认值或者NULL
# 没有默认值或者不能为NULl时会报错，并且插入不成功
INSERT INTO customers (cust_name) VALUES ('Jack song');

# 降低INSERT语句的优先级
INSERT LOW_PRIORITY INTO customers ( cust_name )
VALUES
    ( 'Jone Li. Main' );

# 一次插入多行数据 方式一(用分号分隔)
INSERT INTO customers ( cust_name, cust_address, cust_city, cust_state, cust_zip, cust_country )
VALUES
    ( 'Pep E. LaPew', '100 Main Street', 'Los Angeles', 'CA', '90046', 'USA' );
INSERT INTO customers ( cust_name, cust_address, cust_city, cust_state, cust_zip, cust_country )
VALUES
    ( 'M. Martian', '42 Galaxy Way', 'New York', 'Ny', '11213', 'USA' );


# 一次插入多行数据 方式二(多行数用括号包裹，逗号分隔)
# 此技术可以提高数据库处理的性能,因为MySQL用单条 INSERT 语句处理多个插入比使用多条 INSERT语句快
INSERT INTO customers ( cust_name, cust_address, cust_city, cust_state, cust_zip, cust_country )
VALUES
    ( 'Pep E. LaPew', '100 Main Street', 'Los Angeles', 'CA', '90046', 'USA' ),
    ( 'M. Martian', '42 Galaxy Way', 'New York', 'NY', '11213', 'USA' );


# INSERT和SELECT结合插入其它表检索出来的数据
INSERT INTO customers ( cust_id, cust_contact, cust_email, cust_name, cust_address, cust_city, cust_state, cust_zip, cust_country ) SELECT
cust_id,
cust_contact,
cust_email,
cust_name,
cust_address,
cust_city,
cust_state,
cust_zip,
cust_country 
FROM
    custnew;
```
## 第20章 更新和删除数据
```
# 不要省略 WHERE 子句 在使用 UPDATE 时一定要注意细心,
# 因为稍不注意，就会更新表中所有行. 

# 更新符合筛选条件的行中的指定列
UPDATE customers SET cust_email = 'elemer@fudd.com' WHERE cust_id = 10005;

# 更新多列
UPDATE customers SET cust_name = 'The Fudds', cust_email = 'elmer@fudd.com' WHERE cust_id = 10005;

# 更新多行时, 使用IGNORE忽略错误, 继续更新下去
UPDATE IGNORE customers SET cust_email = 'elemer1@fudd.com' WHERE cust_id = '10005';


# 可以用来删除指定列(可以为NULL的情况)
UPDATE customers SET cust_email = NULL WHERE cust_id = 10005;



# 在使用 DELETE 时一定要注意细心。因为稍不注意，就会错误地删除表中所有行
# 删除符合筛选条件的行
DELETE FROM customers WHERE cust_id = 10006;

# DELETE 语句从表中删除行, 甚至是删除表中所有行。但是, DELETE 不删除表本身

-- 更新和删除的指导原则
-- 前一节中使用的 UPDATE 和 DELETE 语句全都具有 WHERE 子句，这样做的
-- 理由很充分。如果省略了 WHERE 子句，则 UPDATE 或 DELETE 将被应用到表中
-- 所有的行。换句话说，如果执行 UPDATE 而不带 WHERE 子句，则表中每个行
-- 都将用新值更新。类似地，如果执行 DELETE 语句而不带 WHERE 子句，表的
-- 所有数据都将被删除。
-- 下面是许多SQL程序员使用 UPDATE 或 DELETE 时所遵循的习惯。
--  除非确实打算更新和删除每一行，否则绝对不要使用不带 WHERE
-- 子句的 UPDATE 或 DELETE 语句。
--  保证每个表都有主键（如果忘记这个内容，请参阅第15章），尽可能
-- 像 WHERE 子句那样使用它（可以指定各主键、多个值或值的范围）。
--  在对 UPDATE 或 DELETE 语句使用 WHERE 子句前，应该先用 SELECT 进
-- 行测试，保证它过滤的是正确的记录，以防编写的 WHERE 子句不
-- 正确。
--  使用强制实施引用完整性的数据库（关于这个内容，请参阅第15
-- 章），这样MySQL将不允许删除具有与其他表相关联的数据的行
```
## 第21章 创建和操纵表
```
# 使用SQL创建新表
CREATE TABLE customers1 (
    cust_id INT NOT NULL AUTO_INCREMENT,
    cust_name char(50) NOT NULL,
    cust_address char(50) NULL,
    cust_city char(50) NULL,
    cust_state char(5) NULL,
    cust_zip char(10) NULL,
    cust_country char(50) NULL,
    cust_contact char(50) NULL,
    cust_email char(255) NULL,
    PRIMARY KEY (cust_id)
) ENGINE=InnoDB;


# 不允许NULL值
CREATE TABLE orders1 (
    order_num   int NOT NULL AUTO_INCREMENT,
    order_date datetime NOT NULL,
    cust_id int NOT NULL,
    PRIMARY KEY (order_num)
) ENGINE=InnoDB;

# 多列主键
CREATE TABLE orderitems1 (
    order_num   int NOT NULL,
    order_item int NOT NULL,
    prod_id char(10) NOT NULL,
    quantity int NOT NULL,
    item_price decimal(8, 2) NOT NULL,
    PRIMARY KEY (order_num, order_item)
) ENGINE=InnoDB;


# 使用函数获得最后自动生成的id
SELECT LAST_INSERT_ID();


# 创建表时给默认值
CREATE TABLE IF NOT EXISTS orderitems2 (
    order_num int NOT NULL,
    order_item int NOT NULL,
    prod_id char(10) NOT NULL,
    quantity int NOT NULL DEFAULT 1,
    item_price decimal(8, 2) NOT NULL,
    PRIMARY KEY (order_num, order_item)
) ENGINE=InnoDB;


# 更新表结构(添加列)
ALTER TABLE vendors ADD vend_phone CHAR(20);

# 更新表结构(删除列)
ALTER TABLE vendors DROP COLUMN vend_phone;

# 更新表(添加外键)
ALTER TABLE orderitems ADD CONSTRAINT fk_orderitems_orders FOREIGN KEY (order_num) REFERENCES orders (order_num);

ALTER TABLE orderitems ADD CONSTRAINT fk_orderitems_products FOREIGN KEY (prod_id) REFERENCES products (prod_id);

ALTER TABLE orders ADD CONSTRAINT fk_orders_customers FOREIGN KEY (cust_id) REFERENCES customers (cust_id);

ALTER TABLE products ADD CONSTRAINT fk_products_vendors FOREIGN KEY (vend_id) REFERENCES vendors (vend_id);


##复杂的表结构更改一般需要手动删除过程，它涉及以下步骤：
--  用新的列布局创建一个新表；
--  使用 INSERT SELECT 语句（关于这条语句的详细介绍，请参阅第
--      19章）从旧表复制数据到新表。如果有必要，可使用转换函数和
--      计算字段；
--  检验包含所需数据的新表；
--  重命名旧表（如果确定，可以删除它）；
--  用旧表原来的名字重命名新表；
--  根据需要，重新创建触发器、存储过程、索引和外键。


# 删除表
DROP TABLE orderitems2

# 重命名表
RENAME TABLE customers1 TO customer3;

# 同时重命名多个表
RENAME TABLE customer3 TO customers1, orderitems1 TO orderitems2;
```
第22章 使用视图
```
# 视图是虚拟的表。与包含数据的表不一样，视图只包含使用时动态检索数据的查询


# 普通方法检索需要的数据
SELECT
    cust_name,
    cust_contact 
FROM
    customers,
    orders,
    orderitems 
WHERE
    customers.cust_id = orders.cust_id 
    AND orderitems.order_num = orders.order_num 
    AND prod_id = 'TNT2';

# 使用视图检索(假如可以把整个查询包装成一个名为 productcustomers 的虚拟表)
SELECT
    cust_name,
    cust_contact 
FROM
    productcustomers 
WHERE
    prod_id = 'TNT2';

-- 这就是视图的作用。 productcustomers 是一个视图，作为视图，它
-- 不包含表中应该有的任何列或数据，它包含的是一个SQL查询（与上面用
-- 以正确联结表的相同的查询


# 创建视图
CREATE VIEW productcustomers AS SELECT
cust_name,
cust_contact,
prod_id 
FROM
    customers,
    orders,
    orderitems 
WHERE
    customers.cust_id = orders.cust_id 
    AND orderitems.order_num = orders.order_num;

# 创建视图
CREATE VIEW vendorlocations AS SELECT
Concat ( RTrim( vend_name ), '(', RTrim( vend_country ), ')' ) AS vend_title 
FROM
    vendors 
ORDER BY
    vend_name;

# 在视图中查询
SELECT * FROM vendorlocations;


# 创建视图
CREATE VIEW customeremaillist AS SELECT
cust_id,
cust_name,
cust_email 
FROM
    customers 
WHERE
    cust_email IS NOT NULL;


# 检索视图
SELECT * FROM customeremaillist;

# 双WHERE子句(视图中一个, SQL中一个)
SELECT * FROM customeremaillist WHERE cust_id = 10003;


# 创建视图(和计算字段结合)
CREATE VIEW orderitemsexpanded AS SELECT
order_num,
prod_id,
quantity,
item_price,
quantity * item_price AS expanded_price 
FROM
    orderitems;

# 检索有计算字段的视图
SELECT * FROM orderitemsexpanded WHERE order_num = 20005;

-- 如果视图定义中有以下操作，则不能进行视图的更新：
--  分组（使用 GROUP BY 和 HAVING ）；
--  联结；
--  子查询；
--  并；
--  聚集函数（ Min() 、 Count() 、 Sum() 等）
--      DISTINCT；
--  导出（计算）列
```
## 第23章 使用存储过程

我们常用的操作数据库语言SQL语句在执行的时候需要要先编译，然后执行，而存储过程是一组为了完成特定功能的SQL语句集，经编译后存储在数据库中，用户通过指定存储过程的名字并给定参数（如果该存储过程带有参数）来调用执行它。

```sql
# 调用存储过程并返回数据
CALL productpricing(@pricelow, @pricehigh, @priceaverage);


# 创建存储过程
CREATE PROCEDURE productpricing()
BEGIN
    SELECT Avg(prod_price) AS priceaverage FROM products;
END


# 调用存储过程
CALL productpricing();


# 为了在命令行中不出错，改变分隔符
DELIMITER //
CREATE PROCEDURE productpricing1()
BEGIN
    SELECT Max(prod_price) AS pricehigh FROM products;
END //
DELIMITER ;

CALL productpricing1();


# 删除存储过程
DROP PROCEDURE IF EXISTS productpricing1 ;


# 使用OUT参数
CREATE PROCEDURE productpricingwithavg(OUT pl DECIMAL(8,2), OUT ph DECIMAL(8,2), OUT pa DECIMAL(8,2))
BEGIN
    SELECT Min(prod_price) INTO pl FROM products;
    SELECT Max(prod_price) INTO ph FROM products;
    SELECT Avg(prod_price) INTO pa FROM products;
END;

# 使用变量接收
CALL productpricingwithavg(@pricelow, @pricehigh, @priceaverage);
# 显示出来 
SELECT @pricelow;
SELECT @pricelow, @pricehigh, @priceaverage;


# 使用IN OUT参数
CREATE PROCEDURE ordertotal(IN onumber INT, OUT ototal DECIMAL(8,2))
BEGIN
    SELECT Sum(item_price*quantity) FROM orderitems WHERE order_num = onumber INTO ototal;
END;

CALL ordertotal(20005, @total);
SELECT @total;

# 可以用不同的参数反复调用存储过程
CALL ordertotal(20009, @total);
SELECT @total;


# 一个更智能的存储过程(带税求和和不带税求和)
-- Name: ordertotal
-- Parameters: onumber = oreder number
--                       taxable = 0 if not taxable, 1 if taxable
--             ototal = order total variable

CREATE PROCEDURE ordertotalsmart ( IN onumber INT, IN taxable BOOLEAN, OUT ototal DECIMAL ( 8, 2 ) ) COMMENT 'Obtain order, optionally adding tax' 
BEGIN
    
    -- Declare variable for total
    DECLARE total DECIMAL(8,2);
    -- DEclare tax percentage
    DECLARE taxrate INT DEFAULT 6;
    
    -- Get the order total
    SELECT Sum(item_price*quantity)
    FROM orderitems
    WHERE order_num = onumber
    INTO total;
    
    -- Is this taxable?
    IF taxable THEN
        -- Yes, so add taxrate to the total
        SELECT total+(total/100*taxrate) INTO total;
    END IF;
    
    -- And finally, save to out variable
    SELECT total INTO ototal;
    
END;

# 调用上面的存储过程
CALL ordertotalsmart(20005, 0, @total);
SELECT @total;
CALL ordertotalsmart(20005, 1, @total);
SELECT @total;

# 显示创建存储过程的语句
SHOW CREATE PROCEDURE ordertotalsmart;

# 显示所有存储过程的状态
SHOW PROCEDURE STATUS;

# 显示筛选后的存储过程的状态
SHOW PROCEDURE STATUS LIKE 'ordertotalsmart';
```
## 第24章 使用游标
提供对查询结果的详细处理:游标的一个常见用途就是保存查询结果，以便以后使用。游标的结果集是由SELECT语句产生，如果处理过程需要重复使用一个记录集，那么创建一次游标而重复使用若干次，比重复查询数据库要快的多。

```
-- 使用游标涉及几个明确的步骤
--  在能够使用游标前，必须声明（定义）它。这个过程实际上没有
--  检索数据，它只是定义要使用的 SELECT 语句。
-- 一旦声明后，必须打开游标以供使用。这个过程用前面定义的
--  SELECT 语句把数据实际检索出来。
-- 对于填有数据的游标，根据需要取出（检索）各行。
-- 在结束游标使用时，必须关闭游标


# 创建游标(存储过程处理完成后，游标就消失（因为它局限于存储过程）)
CREATE PROCEDURE processorders()
BEGIN
    DECLARE ordernumbers CURSOR
    FOR
    SELECT order_num FROM ordres;
    
    -- 打开游标
    OPEN ordernumbers;
    
    
    -- 关闭游标 
    CLOSE ordernumbers;
    
END;

# 使用游标(FETCH)
CREATE PROCEDURE processorders1()
BEGIN
    
    -- Declare local variables
    DEClARE o INT;

    -- Declare the cursor
    DECLARE ordernumbers CURSOR
    FOR
    SELECT order_num FROM orders;
    
    -- open cursor
    OPEN ordernumbers;
    
    
    -- Get order number
    FETCH ordernumbers INTO o;
    
    
    -- close cursor
    CLOSE ordernumbers;
    
    
END;


# 用游标循环获取行
CREATE PROCEDURE processorders2 ()
BEGIN
    -- Declare local variables
    -- 变量要申明在游标或者句柄之前
    DECLARE done BOOLEAN DEFAULT 0;
    DECLARE o INT;
    
    -- Declare the cursor
    DECLARE ordernumbers CURSOR
    FOR 
    SELECT order_num FROM orders;
    
    -- Declare continue handler
    DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET done=1;
    
    -- open cursor
    OPEN ordernumbers;
    
    -- loop through all rows
    REPEAT
        
        -- Get order number
        FETCH ordernumbers INTO o;
    
    -- End of loop
    UNTIL done END REPEAT;
    
    -- close cursor
    CLOSE ordernumbers;
    
END;


# 实例
CREATE PROCEDURE porcessoredres4()
BEGIN

    -- Declare loca variables
    DECLARE done BOOLEAN DEFAULT 0;
    DECLARE o INT;
    DECLARE t DECIMAL(8,2);
    
    -- Declare the cursor
    DECLARE ordernumbers CURSOR
    FOR
    SELECT order_num FROM orders;
    
    -- Declare continue handler
    DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET done=1;
    
    -- Create a table to store the results
    CREATE TABLE IF NOT EXISTS ordertotals (order_num INT, total DECIMAL(8,2));
    
    -- Open cursor
    OPEN ordernumbers;
    
    -- Loop through all rows
    REPEAT
        
        -- Get order number
        FETCH ordernumbers INTO o;
        
        -- Get the total for this order
        CALL ordertotalsmart(o, 1, t);
        
        -- Insert order and total into ordertotals
        INSERT INTO ordertotals(order_num, total) VALUES (o, t);
    
        -- End of loop
        UNTIL done END REPEAT;
    
        -- Close cursor
        ClOSE ordernumbers;

END;


# 调用实例
CALL porcessoredres4();
```
## 第25章 使用触发器

表发生改动的时候自动进行处理，比如每当订购一个产品时，都从数据库中删去订购的量。


```
-- 触发器按每个表每个事件每次地定义，每个表每个事件每次只允许一个触发器。
-- 因此，每个表最多支持6个触发器（每条 INSERT 、 UPDATE和 DELETE 的之前和之后）
-- 只有表才支持触发器，视图不支持（临时表也不支持


# 创建一个触发器
CREATE TRIGGER newproduct AFTER INSERT ON products FOR EACH ROW
BEGIN

END;


# 删除一个触发器
DROP TRIGGER newproduct;


# 创建一个触发器
CREATE TRIGGER neworder AFTER INSERT ON orders
FOR EACH ROW 
BEGIN
    
END;


# DELETE触发器
CREATE TRIGGER deleteorder BEFORE DELETE ON orders
FOR EACH ROW
BEGIN
    INSERT INTO archive_orders(order_num, ordre_date, cust_id)
    VALUES(OLD.order_num, OLD.order_date, OLD.cust_id);
END;


# UPDATE触发器
CREATE TRIGGER updatevendor BEFORE UPDATE ON vendors
FOR EACH ROW SET NEW.vend_state = Upper(NEW.vend_state);
```
## 第26章 管理事务处理
```
# 事务 使用ROLLBACK
SELECT * FROM ordertotals;
START TRANSACTION;              
DELETE FROM ordertotals;
SELECT * FROM ordertotals;
ROLLBACK;
SELECT * FROM ordertotals;


# 使用COMMIT提交事务
START TRANSACTION;
DELETE FROM orderitems WHERE order_num = 20009;
DELETE FROM orders WHERE order_num = 20010;
COMMIT;


# 设置保留点并回滚到保留点
START TRANSACTION;
SELECT * FROM ordertotals;
SAVEPOINT delete1;
DELETE FROM ordertotals;
SELECT * FROM ordertotals;
ROLLBACK TO delete1;
SELECT * FROM ordertotals;


# 取消MYSQL的自动提交（MYSQL默认每条语句是自动提交的）
# autocommit 标志是针对每个连接而不是服务器的
SET autocommit=0;
```
## 第27章 全球化和本地化
 ```
# 查看支持的字符集
SHOW CHARACTER SET;


# 查看校对表
SHOW COLLATION;

# 查看所使用的字符集和校对
SHOW VARIABLES LIKE 'character%';
SHOW VARIABLES LIKE 'collation%';


# 创建表时指定字符集
CREATE TABLE mytable (
    columnn1 INT,
    columnn2 VARCHAR(10)
) DEFAULT CHARACTER SET hebrew COLLATE hebrew_general_ci;


# 对指定的列设置字符集
CREATE TABLE mytable1(
    columnn1 INT,
    columnn2 VARCHAR(10),
    columnn3 VARCHAR(10) CHARACTER SET latin1 COLLATE latin1_general_ci
) DEFAULT CHARACTER SET hebrew COLLATE hebrew_general_ci;


# 在检索的时候指定校对
SELECT * FROM customers
ORDER BY lastname, firstname COLLATE latin1_general_cs;
```
## 第28章 安全管理


```
# 查看此MYSQL服务器上的所有用户
USE mysql;
USE mysql_need_know;
SELECT `user` FROM user;
SELECT * FROM user;

# 创建一个新的用户账号并指定密码(没有任何权限,要在之后分配权限)
CREATE USER bean IDENTIFIED BY '123456';

# 重命令用户
RENAME USER bean TO bforta;

# 删除一个用户(Mysql5.0之前只会删除账号，不会删除此账号的权限需要先用 REVOKE删除与账号相关的权限)
DROP USER bean;

# 查看用户权限
SHOW GRANTS FOR bforta;
SHOW GRANTS FOR root@localhost;

-- 用户定义为 user@host MySQL的权限用用户名和主机名结
-- 合定义。如果不指定主机名，则使用默认的主机名 % （授予用
-- 户访问权限而不管主机名

# 设置权限(bforta可以检索数据库crashcourse中的所有表)
GRANT SELECT ON crashcourse.* TO bforta;

# 移除权限
REVOKE SELECT ON crashcourse.* FROM bforta;

-- GRANT 和 REVOKE 可在几个层次上控制访问权限：
--  整个服务器，使用 GRANT ALL 和 REVOKE ALL；
--  整个数据库，使用 ON database.*；
--  特定的表，使用 ON database.table；
--  特定的列；
--  特定的存储过程


-- 未来的授权 在使用 GRANT 和 REVOKE 时，用户账号必须存在，
-- 但对所涉及的对象没有这个要求。这允许管理员在创建数据库
-- 和表之前设计和实现安全措施。
-- 这样做的副作用是，当某个数据库或表被删除时（用 DROP 语
-- 句），相关的访问权限仍然存在。而且，如果将来重新创建该
-- 数据库或表，这些权限仍然起作用


# 一次授予多个权限
GRANT SELECT, INSERT ON crashcourse.* TO bforta;


# 更改指定用户的密码(使用Password函数)
SET PASSWORD FOR bforta = Password('12345');

# 更改当前登陆用户的密码
SET PASSWORD = Password('123456');
```

## 第29章 数据库维护


```
# 使用在命令行工具中使用mysqldump备份所有数据库到外部文件中

-- 备份：mysqldump -u root -p --databases 数据库1 数据库2 > xxx.sql
-- 还原：MySQL -uroot -p123456 <f:\all.sql
-- 常见选项：
-- --all-databases, -A： 备份所有数据库
-- --databases, -B： 用于备份多个数据库，如果没有该选项，mysqldump把第一个名字参数作为数据库名，后面的作为表名。使用该选项，mysqldum把每个名字都当作为数据库名。
-- 
-- --force, -f：即使发现sql错误，仍然继续备份
-- --host=host_name, -h host_name：备份主机名，默认为localhost
-- --no-data, -d：只导出表结构
-- --password[=password], -p[password]：密码
-- --port=port_num, -P port_num：制定TCP/IP连接时的端口号
-- --quick, -q：快速导出
-- --tables：覆盖 --databases or -B选项，后面所跟参数被视作表名
-- --user=user_name, -u user_name：用户名
-- --xml, -X：导出为xml文件
-- 1.备份全部数据库的数据和结构
-- 
-- mysqldump -uroot -p123456 -A >F:\all.sql
-- 
-- 2.备份全部数据库的结构（加 -d 参数）
-- 
-- mysqldump -uroot -p123456 -A-d>F:\all_struct.sql
-- 1.还原全部数据库:
-- 
-- (1) mysql命令行：mysql>source f:\all.sql
-- 
-- (2) 系统命令行： mysql -uroot -p123456 <f:\all.sql


# 检查表键
ANALYZE TABLE orders;

# 检查多表键
CHECK TABLE orders, orderitems;
```

## 第30章 改善性能


```
-- 首先，MySQL（与所有DBMS一样）具有特定的硬件建议。在学
-- 习和研究MySQL时，使用任何旧的计算机作为服务器都可以。但
-- 对用于生产的服务器来说，应该坚持遵循这些硬件建议。
--  一般来说，关键的生产DBMS应该运行在自己的专用服务器上。
--  MySQL是用一系列的默认设置预先配置的，从这些设置开始通常
-- 是很好的。但过一段时间后你可能需要调整内存分配、缓冲区大
-- 小等。（为查看当前设置，可使用 SHOW VARIABLES; 和 SHOW
-- STATUS; 。）
--  MySQL一个多用户多线程的DBMS，换言之，它经常同时执行多
-- 个任务。如果这些任务中的某一个执行缓慢，则所有请求都会执
-- 行缓慢。如果你遇到显著的性能不良，可使用 SHOW PROCESSLIST
-- 显示所有活动进程（以及它们的线程ID和执行时间）。你还可以用
-- KILL 命令终结某个特定的进程（使用这个命令需要作为管理员登
-- 录）。
--  总是有不止一种方法编写同一条 SELECT 语句。应该试验联结、并、
-- 子查询等，找出最佳的方法。
--  使用 EXPLAIN 语句让MySQL解释它将如何执行一条 SELECT 语句。
--  一般来说，存储过程执行得比一条一条地执行其中的各条MySQL
-- 语句快。
--  应该总是使用正确的数据类型。
--  决不要检索比需求还要多的数据。换言之，不要用 SELECT * （除
-- 非你真正需要每个列）。
--  有的操作（包括 INSERT ）支持一个可选的 DELAYED 关键字，如果
-- 使用它，将把控制立即返回给调用程序，并且一旦有可能就实际
-- 执行该操作。
--  在导入数据时，应该关闭自动提交。你可能还想删除索引（包括
-- FULLTEXT 索引），然后在导入完成后再重建它们。
--  必须索引数据库表以改善数据检索的性能。确定索引什么不是一
-- 件微不足道的任务，需要分析使用的 SELECT 语句以找出重复的
-- WHERE 和 ORDER BY 子句。如果一个简单的 WHERE 子句返回结果所花
-- 的时间太长，则可以断定其中使用的列（或几个列）就是需要索
-- 引的对象。
--  你的 SELECT 语句中有一系列复杂的 OR 条件吗？通过使用多条
-- SELECT 语句和连接它们的 UNION 语句，你能看到极大的性能改
-- 进。
--  索引改善数据检索的性能，但损害数据插入、删除和更新的性能。
-- 如果你有一些表，它们收集数据且不经常被搜索，则在有必要之
-- 前不要索引它们。（索引可根据需要添加和删除。）
--  LIKE 很慢。一般来说，最好是使用 FULLTEXT 而不是 LIKE 。
--  数据库是不断变化的实体。一组优化良好的表一会儿后可能就面
-- 目全非了。由于表的使用和内容的更改，理想的优化和配置也会
-- 改变。
--  最重要的规则就是，每条规则在某些条件下都会被打破
```
