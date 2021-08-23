 ## 《SQL必知必会-第四版》<br>
> + "-- 注释内容"这种注释方法一般适用于所有DBMS
### 第二课 检索数据
- 注意：不能部分使用DISTINCT ; DISTINCT 关键字作用于所有的列，不仅仅是跟在其后的那一列。
```sql
-- 去重不仅作用于score,而是所有查询列!!!
SELECT DISTINCT score, Cid;

-- 这里去重作用于括号的S2.score
SELECT
    S1.score 'Score',
    COUNT( DISTINCT S2.score ) 'Rank'
```

### 第三课 排序检索数据
- ORDER BY 默认是升序(正序)ASC,DESC是降序(反序);
- 注意: 在多个列上降序排序 如果想在多个列上进行降序排序，必须对每一列指定DESC 关键字。

### 第四课 过滤数据
- WHERE prod_price **BETWEEN** 5 **AND** 10;
- WHERE prod_price **IS NULL**;

### 第五课 高级数据过滤
- 注意: AND在求值过程中优先级必OR更高
```sql
WHERE vend_id = 'DLL01' OR vend_id = 'BRS01' AND prod_price >= 10;
-- 等价为
WHERE vend_id = 'DLL01' OR (vend_id = 'BRS01' AND prod_price >= 10);
```
- 建议: 任何时候使用具有AND 和OR 操作符的WHERE 子句，**都应该使用圆括号明确地分组操作符**。不要过分依赖默认求值顺序，即使它确实如你 =希望的那样。使用圆括号没有什么坏处，它能消除歧义。
- IN的最大优点是可以包含其他SELECT语句，能够更动态地建立WHERE子句。IN()适合于A表必B表数据大的情况!因为它会把B表数据全部遍历一遍与A表比较

### 第六课 通配符过滤
- WHERE prod_name LIKE '%'不会匹配为NULL的行
- 与%能匹配0个字符不同，_总是刚好匹配一个字符，不能多也不能少。
```sql
-- 匹配前面两个字符任意,后面字符为" 电风扇"的字符串!
where sname like '__ 电风扇';
```

### 第七课 创建计算字段
- 注意:使用CONCAT函数连接字符串时,字段有一个为null,那么CONCAT的这一列结果也为null;
- TRIM函数去除字段前后空格;
```sql
SELECT CONCAT(TRIM(sname),'【',TRIM(age),'】') name
--  赵雷【1990-01-01】 
```

### 第八课 使用函数处理数据

### 第九课 聚合函数
- SQL聚合函数: 前面4个函数都会忽略列值为NULL的行,COUNT(column)也会忽略,COUNT(*)不会忽略;

函数 | 说明 
------  | ------
AVG() | 返回某列的平均值
MAX() | 返回某列的最大值
MIN() | 返回某列的最小值
SUM() | 返回某列值之和
COUNT() | 返回某列的行数

- 使用COUNT(*)对表中行的数目进行计数，不管表列中包含的是空值（NULL）还是非空值;
- 使用COUNT(column)对特定列中具有值的行进行计数，忽略NULL 值;
- COUNT(1)等同于COUNT(主键),统计数据行数使用COUNT(*);
- 提示：虽然MAX()一般用来找出最大的数值或日期值，但许多（并非所有）DBMS允许将它用来返回任意列中的最大值，包括返回文本列中的最大值。在用于文本数据时，MAX()返回按该列排序后的最后一行。
---
- 聚合函数使用DISTINCT参数,，DISTINCT必须使用列名，不能用于计算或表
  达式。
```sql
SELECT AVG(DISTINCT prod_price) AS avg_price;
SELECT SUM(DISTINCT prod_price) AS avg_price;
SELECT COUNT(DISTINCT prod_price) AS count_price;
```

### 第十课 分组数据
1. GROUP BY (创建分组)
- 【了解】：除聚集计算语句外，SELECT 语句中的每一列都必须在GROUP BY 子句中给出。(MySQL8.0以上支持每列不必须在GROUP BY 子句中给出)
2. HAVING (分组后过滤)
- 提示：HAVING支持所有WHERE操作符
- 说明：HAVING和WHERE 的差别
> 这里有另一种理解方法，WHERE 在数据分组前进行过滤，HAVING 在数
据分组后进行过滤。这是一个重要的区别，WHERE 排除的行不包括在
分组中。这可能会改变计算值，从而影响HAVING 子句中基于这些值
过滤掉的分组

### 第十一课 使用子查询
- IN(子查询)
```sql
SELECT cust_name, cust_contact
FROM Customers
WHERE cust_id IN (SELECT cust_id
                  FROM Orders
                  WHERE order_num IN (SELECT order_num
                                      FROM OrderItems
                                      WHERE prod_id = 'RGAN01'));
```
- 使用计算字段做为子查询
```sql
SELECT cust_name,
       cust_state,
       (SELECT COUNT(*)
        FROM Orders
        WHERE Orders.cust_id = Customers.cust_id) AS orders
FROM Customers
ORDER BY cust_name;
```
### 第十二、十三课 联结表

1. 内连接
```sql
SELECT vend_name, prod_name, prod_price
FROM Vendors,
     Products
WHERE Vendors.vend_id = Products.vend_id;

-- 等价于
SELECT vend_name, prod_name, prod_price
FROM Vendors
       INNER JOIN Products
                  ON Vendors.vend_id = Products.vend_id;
```
2. 外连接
- (左连接)left join ... on;<br>
  连接时,保留A表所有数据,B表数据不一定全保留,根据on条件而定!
  
```sql
-- 1.2 查询存在" 01 "课程但可能不存在" 02 "课程的情况(不存在时显示为 null )
select s1.Sid,s1.score score1,s2.score score2 from
(select Sid,score from SC where Cid = "01") s1 left join
(select Sid,score from SC where Cid = "02") s2 on s1.Sid = s2.Sid 
where s1.score > s2.score
```

- (右连接)right join ... on;

3. 全连接 
> 关键字: union/union all
- 使用注意条件: 
  * UNION 必须由两条或两条以上的SELECT 语句组成，语句之间用关键
  字UNION 分隔（因此，如果组合四条SELECT 语句，将要使用三个UNION
  关键字）。
  * UNION 中的每个查询必须包含相同的列、表达式或聚集函数（不过，
  各个列不需要以相同的次序列出）。
  * 列数据类型必须兼容：类型不必完全相同，但必须是DBMS 可以隐含
  转换的类型（例如，不同的数值类型或不同的日期类型）。
  
```sql
SELECT cust_name, cust_contact, cust_email
FROM Customers
WHERE cust_state IN ('IL', 'IN', 'MI')
UNION
SELECT cust_name, cust_contact, cust_email
FROM Customers
WHERE cust_name = 'Fun4All';
```
- UNION从查询结果集中自动去除了重复的行,而UNION ALL是合并所有行(包括重复的),所以UNION ALL执行时间更短!

### 第十八课 使用视图
1. 为什么使用视图?
- 重用SQL语句。
- 使用表的一部分而不是整个表。这样就可以保护数据。可以授予用户访问表的特定部分的权限，而不是整个表的
  访问权限。
- 更改数据格式和表示。视图可返回与底层表的表示和格式不同的数据。
2. 创建视图
- 基本语法
```sql
CREATE VIEW ProductCustomers AS
SELECT cust_name, cust_contact, prod_id
FROM Customers, Orders, OrderItems
WHERE Customers.cust_id = Orders.cust_id
AND OrderItems.order_num = Orders.order_num;
```
3. 小结
> 视图为虚拟的表。它们包含的不是数据而是根据需要检索数据的查询。
视图提供了一种封装SELECT 语句的层次，可用来简化数据处理，重新
格式化或保护基础数据。

### 第十九课 使用存储过程[&](https://blog.csdn.net/qq_33157666/article/details/87877246?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-12.pc_relevant_baidujshouduan&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-12.pc_relevant_baidujshouduan)
> 资料学习:[mysql存储过程学习笔记](https://blog.csdn.net/qq_33157666/article/details/87877246?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-12.pc_relevant_baidujshouduan&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-12.pc_relevant_baidujshouduan)

> 存储过程就是为以后使用而保存的一条
或多条SQL 语句。可将其视为批文件，虽然它们的作用不仅限于批处理。

#### 1. 为什么使用存储过程?
- 通过把处理封装在一个易用的单元中，可以简化复杂的操作
- 简单: 由于不要求反复建立一系列处理步骤，因而保证了数据的一致性。如果所有开发人员和应用程序都使用同一存储过程，则所使用的代码都是相同的。
- 安全性: 简化对变动的管理。如果表名、列名或业务逻辑（或别的内容）有变化，那么只需要更改存储过程的代码。使用它的人员甚至不需要知道这些变化。
- 高性能: 因为存储过程通常以编译过的形式存储，所以DBMS 处理命令所需的工作量少，提高了性能。

#### 2. 创建存储过程
1. 基本语法:
  > create procedure 名称([IN|OUT|INOUT] 参数名 参数数据类型 <br> begin <br>
  ... <br>
  end;

  
2. 存储过程的传入参数in
- 说明：
  1. 传入参数：类型为in,表示该参数的值必须在调用存储过程事指定，如果不显示指定为in,那么默认就是in类型。
  2. IN类型参数一般只用于传入，在调用过程中一般不作为修改和返回
  3. 如果调用存储过程中需要修改和返回值，可以使用OUT类型参数

<details>
<summary>代码演示:</summary>

```sql
-- varchar类型参数需要指定长度
create procedure test4(userId varchar (150)) 
begin
      declare username varchar(32) default ''; -- declare:声明变量
        -- select A into B:查询A字段赋值给B变量!
        select name into username from users where id=userId; 
        select username;
end;

-- 调用: call test4("OD1528989");
```
</details>

3. 存储过程的传出参数out
- 概括：
   + 传出参数：在调用存储过程中，可以改变其值，并可返回；
   + out是传出参数，不能用于传入参数值；
   + 调用存储过程时，out参数也需要指定，但必须是变量，不能是常量；
   + 如果既需要传入，同时又需要传出，则可以使用INOUT类型参数
<details>
<summary>代码演示:</summary>

```sql
create procedure test5(in userId int,out username varchar(32))
begin
select name into username from users where id=userId;
end;
```
```sql
-- 执行存储过程
-- 定义一个变量 set @变量名
set @uname = '';
call test5(1,@uname);
select @uname as username;
```
</details>

4. 存储过程的可变参数INOUT
- 概括:
   + 可变变量INOUT:调用时可传入值，在调用过程中，可修改其值，同时也可返回值；
   + INOUT参数集合了IN和OUT类型的参数功能；
   + INOUT调用时传入的是变量，而不是常量；


### 第20 课 管理事务处理
#### 1.事务处理
- 使用事务处理，通过确保成批的SQL 操作要么完全执行，要么完全不执行，来维护数据库的完整性
  + 事务（transaction）指一组SQL 语句；
  + 回退（rollback）指撤销指定SQL 语句的过程；
  + 提交（commit）指将未存储的SQL 语句结果写入数据库表；
  + 保留点（savepoint）指事务处理中设置的临时占位符（placeholder），可以对它发布回退（与回退整个事务处理不同）。
- 可以回退哪些语句？<br>
> 事务处理用来管理INSERT、UPDATE 和DELETE 语句。不能回退SELECT
语句（回退SELECT 语句也没有必要），也不能回退CREATE 或DROP 操
作。事务处理中可以使用这些语句，但进行回退时，这些操作也不撤销。

### 第21 课 使用游标

### 第22 课 SQL的高级特性
#### 1.约束
> 约束（constraint） :管理如何插入或处理数据库数据的规则。

1. 主键(PRIMARY KEY)
   + 任意两行的主键值都不相同。
   + 每行都具有一个主键值（即列中不允许NULL 值）。
   + 包含主键值的列从不修改或更新。（大多数DBMS 不允许这么做，但如果你使用的DBMS 允许这样做，好吧，千万别！）
   + 主键值不能重用。如果从表中删除某一行，其主键值不分配给新行。
2. 外键
- 外键是表中的一列，其值必须列在另一表的主键中。外键是保证引用完整性的极其重要部分
- 外键有助防止意外删除,如,A表主键为B表的外键,删除A表的一行数据时,必须保证删除A表管理的所有外键数据(不止B表)!
3. 唯一约束
> 唯一约束用来保证一列（或一组列）中的数据是唯一的。它们类似于主键，但存在以下重要区别。
+ 表可包含多个唯一约束，但每个表只允许一个主键。
+ 唯一约束列可包含NULL 值。
+ 唯一约束列可修改或更新。
+ 唯一约束列的值可重复使用。
+ 与主键不一样，唯一约束不能用来定义外键。
4. 检查约束
+ 检查最小或最大值。例如，防止0 个物品的订单（即使0 是合法的数）。
+ 指定范围。例如，保证发货日期大于等于今天的日期，但不超过今天起一年后的日期。
+ 只允许特定的值。例如，在性别字段中只允许M或F。
5. 索引
> 索引用来排序数据以加快搜索和排序操作的速度。

- 在开始创建索引前，应该记住以下内容:
  + 索引改善检索操作的性能，但降低了数据插入、修改和删除的性能。在执行这些操作时，DBMS 必须动态地更新索引。
  + 索引数据可能要占用大量的存储空间。
  + 并非所有数据都适合做索引。取值不多的数据（如州）不如具有更多可能值的数据（如姓或名），能通过索引得到那么多的好处。
  + 索引用于数据过滤和数据排序。如果你经常以某种特定的顺序排序数据，则该数据可能适合做索引。
  + 可以在索引中定义多个列（例如，州加上城市）。这样的索引仅在以州加城市的顺序排序时有用。如果想按城市排序，则这种索引没有用处。
6. 触发器
> 触发器是特殊的存储过程，它在特定的数据库活动发生时自动执行。触发器可以与特定表上的INSERT、UPDATE 和DELETE 操作（或组合）相关联。

- 下面是触发器的一些常见用途。
  + 保证数据一致。例如，在INSERT 或UPDATE 操作中将所有州名转换
    为大写。
  + 基于某个表的变动在其他表上执行活动。例如，每当更新或删除一行
    时将审计跟踪记录写入某个日志表。
  + 进行额外的验证并根据需要回退数据。例如，保证某个顾客的可用资金不超限定，如果已经超出，则阻塞插入。
  + 计算计算列的值或更新时间戳。
   
7. 数据库安全
> 一般说来，需要保护的操作有：
> - 对数据库管理功能（创建表、更改或删除已存在的表等）的访问；
> - 对特定数据库或表的访问；
> - 访问的类型（只读、对特定列的访问等）；
> - 仅通过视图或存储过程对表进行访问；
> - 创建多层次的安全措施，从而允许多种基于登录的访问和控制；
> - 限制管理用户账号的能力。
