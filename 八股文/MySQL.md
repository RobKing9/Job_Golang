# MySQL使用指南

## 安装和配置

1. [安装MySQL 地址](https://dev.mysql.com/downloads/mysql/)

2. 将下载好的文件进行解压

3. 新建一个my.ini配置文件，内容如下

   ```ini
   [mysqld]
   basedir ="D:\AStudy\MySQL\mysql-8.0.27-winx64"
   datadir ="D:\AStudy\MySQL\mysql-8.0.27-winx64\data"
   port=3306
   server_id =10
   character-set-server=gbk
   character_set_filesystem=gbk
   [client]
   port=3306
   default-character-set=gbk
   [mysqld_safe]
   timezone="CST"
   [mysql]
   default-character-set=utf8
   ```

4. 使用管理员身份运行cmd并cd D:\AStudy\MySQL\mysql-8.0.27-winx64\bin

5. 输入 `mysqld --initialize-insecure` 生成data文件

6. 输入 `mysqld -install` 安装MySQL

7. 启动服务 `net start mysql`

## 基础指令

- `mysql -u root -p` 之后输入密码进入mysql命令行
- `show databases;`显示所有的数据库
- `use <databaseName>`使用某个数据库
- `show tables;`显示该数据库所有的表
- `desc <tablename>`查看表结构
- `exit`退出mysql

### 索引

建表的时候创建索引：

```sql
CREATE TABLE ... {
	unique index (列名),
	index(列名),
	index(列名，列名)
}	

ALTER TABLE 表名 ADD index(列名)
// 唯一索引
create unique index index_name on table(column, ...)

// 联合索引
CREATE INDEX index_product_no_name ON product(product_no, name);
```

查询索引：`SHOW index FROM 表名;`

在一条查询语句加上`EXPLAIN`可以查看这条语句走的索引

删除索引：`drop index 索引名 on 表名;`或者`Alter table 表名 drop index 索引名;`

## 增删改指令(DML)

- `insert into <表名>(列名，列名...) values(val, val...), (), ()... `添加一行或者多行数据

  `insert ignore into`表示如果存在该字段则忽略

- `update <表名> set <列名>=<val> .... where ...`更新指定的数据

- `delete from <表名> where <列名>=<val>`删除指定的行

## 查询指令(DQL)

### 检索数据(SELECT)

- `SELECT <列名> FROM <表名>`

可以在列名前面使用`DISTINCT`去除重复，在表名后面加上`LIMIT 5`可以限制出现的记录

列名可以是*通配符，但是一般不用，降低检索和应用程序的性能

### 排序检索数据(DESC)

- `ORDER BY <列名> <ASC(默认升序), DESC>`

可以按照两个列排序

### 过滤数据(WHERE, ADD, OR, LIKE)

- `WHERE <列名> BETWEEN AND`

可以是等于，不等于，大于，小于，范围等

- `AND OR` 用来过滤数据，AND的优先级高于OR
- `SELECT * FROM products WHERE prod_name LIKE '%rot%';`

使用LIKE，%可以匹配单个，多个字符；_下划线只能匹配一个。'%'不能匹配NULL数据，去掉多余的空格。

### 创建计算字段(Concat, AS)

- `SELECT Concat(vend_name, '(', vend_country, ')') AS vend_info FROM vendors ORDER BY vend_name;`

`Concat()`函数用于拼接串。`Trim()`，`RTrim()`， `LTrim()`用于去掉左右空格。

`AS`用于赋予新的替换名

### 使用函数处理数据(针对项)

#### 文本处理

- `SELECT vend_name, Upper(vend_name) AS vend_name_Upper FROM vendors ORDER BY vend_name;`

Upper()将文本转化为大写；Left()返回串左边的字符；Length()返回串的长度；Locate()串的字串；Soundex()发音类似。

#### 日期处理

- `SELECT * FROM orders WHERE Year(order_date) = 2005;`

Date()获取日期(年月日)；Year()，Month(), Day(), Hour(), Minute(), Time()获取日期的年份

`DATEDIFF(日期一，日期二)`结果是日期一和日期二相差的天数，大于为正

#### 数值处理

Abs()绝对值；Sqrt()根；Rand()随机数

### 聚集函数(针对表)

- `SELECT COUNT(*) AS num_items, MIN(prod_price) AS price_min, MAX(prod_price) AS price_max, AVG(prod_price) AS price_avg FROM products;`

COUNT(*) 用来计数；MIN()，MAX(), AVG()求最小，最大，平均值；SUM() 指定列值的和

### 分组(GROUP BY, HAVING)

- `SELECT vend_id, COUNT(*) AS num_prods FROM products WHERE prod_price >= 10 GROUP BY vend_id HAVING COUNT(*) >= 2;`

先价格筛选，将筛选后的按照vend_id进行分组，满足数量要大于等于2

- `SELECT order_num, SUM(quantity * item_price) AS ordertotal FROM orderitems GROUP BY order_num HAVING SUM(quantity * item_price) >= 50 ORDER BY ordertotal;`

先分组过滤，然后按照总价格排序

`GROUP BY`用于有聚集函数的场景，出现在`WHERE`字句之后，`ORDER BY`字句之前。HAVING用于分组过滤。

#### 子查询(IN)

嵌套查询

### 联表查询(INNTER JOIN)

- `SELECT * FROM products INNER JOIN vendors on vendors.vend_id = products.vend_id;`
- `SELECT prod_name, prod_price, vend_name FROM products, vendors WHERE products.vend_id = vendors.vend_id ORDER BY vend_name, prod_name;`

使用`INNTER JOIN ON <两个表的列值相等>`而不使用WHERE,提高性能，联结的表越多，查询性能越差。

- `SELECT t.team_id, employee_name, team_name FROM t_employee t RIGHT JOIN t_team tt ON t.team_id = tt.team_id;` 

  外连接：`LEFT JOIN，RIGHT JOIN` 分别是以左边(或者右边为准)，如果右边没有数据会置为空，左表所有行数都会显示
  
- ```
  SELECT * FROM t_blog LEFT JOIN t_type ON t_blog.typeId=t_type.id
  UNION
  SELECT * FROM t_blog RIGHT JOIN t_type ON t_blog.typeId=t_type.id;
  ```

  外连接就是求两个集合的并集。从笛卡尔积的角度讲就是从笛卡尔积中挑出ON子句条件成立的记录，然后加上左表中剩余的记录，最后加上右表中剩余的记录。另外MySQL不支持OUTER JOIN，但是我们可以对左连接和右连接的结果做UNION操作来实现。

![](https://raw.githubusercontent.com/RobKing9/Blog_Pic/master/Git/20220914105502.png)

## 执行一条 SQL 查询语句，期间发生了什么？

- 连接器：建立连接，管理连接、校验用户身份；
- 查询缓存：查询语句如果命中查询缓存则直接返回，否则继续往下执行。MySQL 8.0 已删除该模块；
- 解析 SQL，通过解析器对 SQL 查询语句进行词法分析、语法分析，然后构建语法树，方便后续模块读取表名、字段、语句类型；
- 执行 SQL：执行 SQL 共有三个阶段：
  - 预处理阶段：检查表或字段是否存在；将 `select *` 中的 `*` 符号扩展为表上的所有列。
  - 优化阶段：基于查询成本的考虑， 选择查询成本最小的执行计划；
  - 执行阶段：根据执行计划执行 SQL 查询语句，从存储引擎读取记录，返回给客户端；

![](https://raw.githubusercontent.com/RobKing9/Blog_Pic/master/20230325122824.png)

## 索引的原理、分类和使用

![](https://raw.githubusercontent.com/RobKing9/Blog_Pic/master/20230326103844.png)

![](https://raw.githubusercontent.com/RobKing9/Blog_Pic/master/20230326110522.png)

为了高效查询记录所在的数据页，InnoDB 采用 b+ 树作为索引，**每个节点都是一个数据页**。

如果叶子节点存储的是实际数据的就是聚簇索引，一个表只能有一个聚簇索引；如果叶子节点存储的不是实际数据，而是主键值则就是二级索引，一个表中可以有多个二级索引。

在使用二级索引进行查找数据时，如果查询的数据能在二级索引找到，那么就是「**索引覆盖**」操作，如果查询的数据不在二级索引里，就需要先在二级索引找到主键值，需要去聚簇索引中获得数据行，这个过程就叫作「**回表**」。

## 其他

- MySQL查询语句的正确执行顺序：`FROM(including JOINs) ---> WHERE ---> GROUP BY ---> HAVING ---> SELECT ---> DISTINCT ---> ORDER BY ---> LIMIT/OFFSET`
  - 先找到要查询表格或连接要查询的表格，因此FROM才是第一步；
  - 接下来是进行条件筛选，所以是WHERE紧随其后；
  - 然后如果遇到表格有分组的需要，则需要先GROUP BY；
  - 分组时如果也存在筛选条件，这里就要用HAVING进行分组筛选；
  - 这些执行过后才是查询操作SELECT；
  - SELECT的时候如果遇到重复数据，就需要去重，即使用DISTINCT;
  - 接下来如果要对查询后的数据进行排序，会用到ORDER BY；
  - 最后如果要指定返回的查询数据范围、条数则要用LIMIT/OFFSET函数。

## 