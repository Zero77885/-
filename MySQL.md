# sql语句的执行顺序

格式:

SELECT 字段名1,字段名2
FROM 表名
WHERE 过滤条件
GROUP BY 字段名1,字段名2,.......
HAVING 过滤条件
ORDER BY 字段名1 ASC/DESC,字段名2 ASC/DESC,.......
LIMIT 索引值,数据的条数

## -----执行的顺序

FROM 表名
WHERE 过滤条件 人-
GROUP BY 字段名1,字段名2,.......
HAVING 过滤条件

SELECT 字段名1,字段名2

ORDER BY 字段名1 ASC/DESC,字段名2 ASC/DESC,.......
LIMIT 索引值,数据的条数;







# 1.查看所有的库

SHOW DATABASES;

 

# 2.选库

USE 库名;

 

# 3.查看所有的表

SHOW TABLES;

 

# 4.查看表中的内容 

*表示所有的字段

select 字段名1,字段名2,.......

from 表名;

 

# 5.给字段起别名

select 字段名1 as 别名1,字段名2 别名2,......

from 表名;

 

 

# 6.过滤：

格式：

​    select 字段名1,字段名2,.....

​    from 表名

​    where 过滤条件

 

# 7.模糊查找

 关键字 like

 

# 8.排序

关键字 ORDER BY

DESC 降序

ASC 升序

 

# 9.多表查询 

如果要查询的多个字段不是在同一张表中那么就需要用到多表查询

select 字段名1,字段名2,........

​    from 表名1 join 表名2

​    on 连接条件

​    join 表名3

​    on 连接条件

​    .......

​    where 过滤条件

​    order by 字段名1 asc/desc,字段名2 asc/desc,.....

 

## 9.1表和表的连接方式分类：

​    自连接 vs 非自连接

​    等值连接 vs 非等值连接

​    内连接 vs 外连接

\#等值连接:连接的条件用的是等号

\#非自连接：连接的表不是同一张表

\#自连接：连接的表同一张表

\#非等值连接：连接条件用的不是等号

 

## 9.2左外连接

除了两张表中匹配的内容还包括左表中不匹配的内容

关键字: LEFT JOIN

 

## 9.3右外连接

除了两张表中匹配的内容还包括右表中不匹配的内容

关键字: RIGHT JOIN

 

## 9.4满外连接

除了两张表中匹配的内容还包括左表和右表中不匹配的内容

\#full join : 但是mysql不支持 不过可以通过union关键字实现该效果

格式：

​    select ....

​    ....

​    union（去重）/union all（不去重）

​    select ....

​    ....

 

# 10. 字母转成大小写

LOWER('SQL Course') : 将所有字母转成小写

UPPER('SQL Course') ：将所有字母转成大写

 

# 11.字符串拼接

CONCAT('Hello', 'World') :

 

# 12.截取子串 

SUBSTR('HelloWorld',1,5) : 截取子串 1表示索引位置(索引位置从1开始) 5表示长度

 

# 13.求字符串的长度

LENGTH('HelloWorld') ：字符串的长度

 

# 14.字符串中首次出现位置

INSTR('HelloWorld', 'W') : W在当前字符串中首次出现位置

 

# 15.长度不够补"*"

如果字段的内容长度不够10在内容的左边用*补

RPAD(salary, 10, '*')

如果字段的内容长度不够10在内容的右边用*补

 

# 16.去除字符串两端指定的字符

 TRIM('H' FROM 'HelloWorld') : 

 

# 17. 将字符串的所有b字符替换成m

REPLACE('abcd','b','m') ：

 

# 18.四舍五入

ROUND: 四舍五入

 

# 19. 截断

TRUNCATE: 

 

# 20. 求余

MOD: 

 

# 21.日期函数

 SELECT NOW(); 日期函数

 

 

# 22.通用函数

case 字段名

when 值1 then 返回值1

when 值2 then 返回值2

when 值3 then 返回值3

.......

else 返回值n

end

 

 

 

# 23.如果字段的内容为null就用默认值替换 

ifnull(字段名,默认值) ：

 

# 24.求和 

sum():

 

# 25. 求平均值

avg():求平均值

 

# 26.求最大值

 max():求最大值

 

# 27. 求最小值

min():求最小值

 

# 28.统计

count():统计结果的条数

count(*)：对查询的结果进行统计。

count(字段名): 对查询的结果中 该字段不为null的数据进行统计

count(具体的数值)：可以理解成count(*)

 

# 29.去重 

格式：

select distinct   字段名1，字段名2,......

from xxx

 

# 30.分组

格式：

select xxxx

from 表名

where 过滤条件

group by 字段名1,字段名2,........

having 过滤条件

order by 字段名1 asc/desc,字段名2 asc/desc,........

 

select xxxx

from 表名1 join 表名2

on 连接条件

join 表名3

on 连接条件

........

where 过滤条件

group by 字段名1,字段名2,........

having 过滤条件

order by 字段名1 asc/desc,字段名2 asc/desc,........

 

 

# 31. where和having的区别？

1.where是在分组前过滤 having是在分组后过滤

2.where后面不能用组函数 having后面可以写组函数

\#注意：select后面一旦出现组函数就不能再出现其它字段除非该字段出现在group by的后面。

 

 

# 32.子查询

子查询：在一条查询语句a中 再嵌套另一条查询语句b b语句叫作子查询（内查询） a语句叫作主查询（外查询）

 

子查询分类： 单行子查询 vs 多行子查询

 

单行子查询：子查询返回的结果只有一条

多行子查询：子查询返回的结果有多条

 

 

注意：1.单行子查询用的运算符有: > < >= <= <> =

​     如果用的是单行子查询的运算符而子查询返回的是多行就会报错。

​     

   2.子查询返回的字段只能有一个

   

   3.多行子查询使用的运算符 ： in any all

​       in (多行子查询) ： 等于其中的某一个值即可

​        \> any (多行子查询) : 大于其中的任意一个即可

​        \> all （多行子查询） : 大于所有的才可以

 

# 33. 改名

AS 改名

 

# 34.不等于

  <> 

# 35 or 等价于 || 或

 

 

# 36 IN(1,2) 等于 1和2

 

# 37 AND 等价于 && 与

 

# 38 between 1 and 2包含边界 表示 >=1 and <=2

 

# 39. 判断是否为null 用 is null 或 is not null

 

# 40.[if not exists] :

如果库不存在则创建 存在则不创建。如果没有该字段库存在则报错。

 

# 41. 创建库

create database [if not exists] 库名 [character set '编码集']

 

# 42. #查看库的信息-库的创建语句

SHOW CREATE DATABASE xxx

 

# 43.修改编码集

ALTER DATABASE db2 CHARACTER SET 'utf8';

 

# 44 删除库

 drop database [if exists] 库名;

 

[if exists] :如果库存在则删除 不存在则不删。如果没有该字段库不存在则报错。

 

 

# 45. 查看所有的表

SHOW TABLES;

 

# 46. 查看表结构

DESC employees;

 

# 47. 查看表的创建语句

SHOW CREATE TABLE employees;

 

# 48.1创建表的方式一：

 

create table [if not exists] 表名(

字段名 字段类型,

字段名2 字段类型,

字段类名n 字段类型 #注意：如果是最后一个字段不加逗号 除非下面有其它约束

) [character set '编码集']

 

注意：创建表时如果不指定编码集那么默认和库的编码集一致。

 

## 48.2 创建表方式二：基于现有的表创建新表

 

create table 新表的表名 like 已经存在的表的表名

 

## 48.3 创建表方式三 ：将查询结果创建成一张新表

create table 表名

select....

from xxx

xxxx

 

 

# 49. #删除表

drop table [if exists] 表名

 

 

# 50. 向表中插入字段：

alter table 表名 add [column] 字段名 字段类型;

 

# 51. 修改字段的名字:

 alter table 表名 change [column] 原字段名 新字段名 字段类型;

 

# 52. 修改字段的类型：

\#格式一 ：alter table 表名 modify [column] 字段名 新字段类型;

\#格式二： alter table 表名 change [column] 字段名 字段名 新字段类型;

 

 

# 53. 删除字段 : 

alter table 表名 drop [column] 字段名;

 

 

# 54. 修改表名:

 alter table 原表名 rename to 新表名;

 

# 55. 向表中插入一条数据:

\#格式: insert into 表名(字段名1，字段名2,...) values(值1，值2,....),(值1，值2,....),........

 

# 56. 将查询的数据插入到表中

insert into 表名(字段名1，字段名2,....)

select 字段名1，字段名2,.....

from xxxx

xxxx

 

 

# 57. #修改数据:

 update 表名 set 字段名1=值1，字段名2=值2,.....[where ]

 

 

# 58. 删除数据:

 delete from 表名 [where 过滤条件]

 

# 59.清空表中的数据 ：

truncate table 表名;





# 60.事务(数据回滚)

开启事务:

方式一:SET autocommit=FALSE

方式二:START TRANSACTION;

回滚:ROLLBACK;

提交:COMMIT;(一旦提交 就不可以再回滚)

关闭事务:SET autocommit=TRUE;

## 60.1 delete from 和 truncate table 的区别

1.delete from可以使用事务而truncate table不可以使用事务。
2.如果要删除的数据确定不需要回滚操作 那么使用truncate table效率更高一些。





# 61.约束(给表加约束)

NOT NULL 非空约束，规定某个字段不能为空
UNIQUE  唯一约束，规定某个字段在整个表中是唯一的
PRIMARY KEY  主键(非空且唯一)
FOREIGN KEY  外键
CHECK  检查约束
DEFAULT  默认值


约束分为：列级约束 vs 表级约束
列级约束：同时只约束一列
表级约束：同时可以约束多列


添加约束：创建表时添加约束（了解） vs  创建表后添加约束（知道即可）

AUTO_INCREMENT : 自增

## 1外键约束

外键约束 CONSTRAINT 索引名 FOREIGN KEY（本表的字段名）REFERENCES 主表（主表的字段名）

## 2.级联删除

DELETE FROM dept WHERE

# 62.创建表后添加约束

## primary key

添加约束 : alter table 表名 add primary key (字段名)
修改约束 : alter table 表名 modify  字段名 类型 primary key
删除约束 : alter table 表名 drop primary key

## unique

添加约束 : alter table 表名  add unique(字段名)
添加约束 : alter table 表名 add constraint  索引名 unique(字段名)
修改约束 ：alter table 表名 modify 字段名 类型  unique
删除约束 ：alter table 表名 drop index 索引名

# 63.分页

limit 索引值,数据的条数

 #注意：索引值从0开始

分页公式 ：limit （页数-1）*数据的条数  ,数据的条数