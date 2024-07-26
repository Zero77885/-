



# 库的操作

```hive
[IF NOT EXISTS] :如果库不存在就创建 库存在则不创建 如果没有该字段库存在则报错。

CREATE DATABASE [IF NOT EXISTS] 库名

[COMMENT ‘库的说明-注释’]

[LOCATION HDFS对应的路径] #默认路径是/user/hive/warehouse

[WITH DBPROPERTIES ('属性名'='属性值', ...)]; #给库设置属性
```

***案例\***

```hive
create database d1
comment 'this is d1'
location '/demo/d1'
with dbproperties('ver'='1.0');
```



## 查看库的信息

```hive
#[extended]加上后可以查看库的属性
desc database [extended] 库名;
```



## 查询数据库

```hive
#[LIKE '匹配规则'] ：模糊查询--但是不要当成mysql中的模糊查询 -- 只能用*和|
SHOW DATABASES [LIKE '匹配规则'];
```

## 删除数据库

```hive
[IF EXISTS] ：如果库存在则删除 不存在则不删 如果没有该字段库不存在则报错
[RESTRICT|CASCADE] : 
CASCADE用来删除非空的库。
RESTRICT默认的只能删除空库

DROP DATABASE [IF EXISTS] 库名 [RESTRICT|CASCADE];
```



## 切换库

```hive
use 库名;
```

## 修改库的属性

```hive
ALTER DATABASE 库名 SET DBPROPERTIES ('属性名'='属性值', ...);
```

## 修改location

```hive
ALTER DATABASE 库名 SET LOCATION hdfs的绝对路径;

alter database d1 set location'hdfs://hadoop102:8020/user/hive/warehouse/d1';
```



## 修改owner user

```hive
ALTER DATABASE 库名 SET OWNER USER 用户名;

alter database d1 set owner user longge;
```



# 表的操作

## 创建表的格式

```hive
[IF NOT EXISTS] ：如果表不存在则创建 存在则不创建 如果没有该字段 表存在则报错。
[TEMPORARY] ：创建临时表-当客户端退出 临时表被删除
[EXTERNAL] ：加上该字段创建的是外部表 否则是管理表

CREATE [TEMPORARY] [EXTERNAL] TABLE [IF NOT EXISTS] [库名.]表名  
[(
字段名 字段类型 [COMMENT 字段的描述信息-注释], 
...
)]
[COMMENT 表的描述信息-注释]
[PARTITIONED BY (字段名 字段类型 [COMMENT 字段的描述信息-注释], ...)] #创建分区表
[CLUSTERED BY (分桶字段名1, 分桶字段名2, ...) #创建分桶表
[SORTED BY (排序的字段名 [ASC|DESC], ...)] INTO 桶的数量 BUCKETS]
[ROW FORMAT row_format] #用来对数据和元数据的匹配进行说明
[FIELDS TERMINATED BY 分隔字符] #每个字段用什么隔开
[COLLECTION ITEMS TERMINATED BY 分隔字符] #复杂数据类型中的元素之间用什么隔开
[MAP KEYS TERMINATED BY 分隔字符] #map中的元素的key和value用什么隔开
[LINES TERMINATED BY 分隔字符] #每条数据之间用什么隔开
[NULL DEFINED AS char] #表中的null值在HDFS上的文件中用什么字符表示 默认\N
[STORED AS file_format]# 数据存储的格式默认是textfile
[LOCATION HDFS的路径]# 创建表的在HDFS的路径--创建库和表在HDFS上都有对应的目录(默认是在库的下面)
[TBLPROPERTIES ('属性名'='属性值', ...)]
```

***案例\***

```hive
create table if not exists db1.employee
(
id int comment 'this is id',
name string comment 'this is name'
)
comment 'this is table'
location '/demo/employee'
tblproperties('ver'='1.0');   
```



## 创建临时表

```hive
create TEMPORARY table tem_tbl
(
id int,
name string
);
```

## 将查询的结果创建成一张表

 ```hive
 create table 新表名
 as
 select * from 表名;
 ```



## 基于现有的表创建一张新表(没有数据只有表结构)

```hive
#格式：create table 新表的表名 like 存在的表的表名;

create table student3 like student;
```

## 建表（数据只有value）

```hive
create table 表名(
name string,#字段和字段类型
friends array<string>,#字段和字段类型
students map<string,int>,#字段和字段类型
address struct<street:string,city:string,postal_code:int>#字段和字段类型
)
row format delimited #用来对数据和元数据的匹配进行说明
fields terminated BY ','#复杂数据类型中的元素之间用什么隔开
collection items terminated BY '-' #map中的元素的key和value用什么隔开
map keys terminated BY ':'; #每条数据之间用什么隔开
```

## 建表（数据是json）

```hive
create table 表名(
name string,
friends array\<string>,
students map<string,int>,
address struct<street:string,city:string,postal_code:int>
)
row format serde 'org.apache.hadoop.hive.serde2.JsonSerDe';
'org.apache.hadoop.hive.serde2.JsonSerDe' #如果是json数据由它来帮我们解析—一行是一条数据
```

 

## 查询复杂数据

```hive
select 数组[索引] 或 map['key值'] 或struct.字段名 
from 表名;
```

## 创建外部表

```hive
create **external** table 表名
 (
id int,
name string
)
row format delimited fields terminated by '\t';
```

## 创建管理表(内部表)

```hive
create table 表名
(
id int,
name string
)
row format delimited fields terminated by '\t';
```



#  外部表和管理表的转换。

#查看表信息

```hive
desc formatted 表名;

```

 #通过修改表的属性 修改表的类型

```hive
alter table 表名 set tblproperties('EXTERNAL'='TRUE/FALSE');
```

## 查看表

```hive
#[IN 库名]：查看哪个库中的表
#LIKE ['匹配规则'] : 模糊查询
**SHOW TABLES [IN 库名] LIKE ['匹配规则'];**
```

## 查看表信息

```hive
\# desc 表名: 查看字段信息
\# desc formatted 表名 : 查看表的详细信息
\# desc EXTENDED 表名 : 查看表的详细信息但是显示的内容没有格式化
DESCRIBE [EXTENDED | FORMATTED] [库名.]表名
```

## 修改表名

```hive
ALTER TABLE 原表名 RENAME TO 新表名;
```

## 添加列

```hive
ALTER TABLE 表名 ADD COLUMNS (字段名 字段类型 [COMMENT 字段的描述信息], ...)
```

## 更新列-更新列的名字

```hive
ALTER TABLE 表名 CHANGE [COLUMN] 原字段名 新字段名 字段的类型 [COMMENT 字段的描述信息]
```

## 更新列-更新列的类型

```hive
#注意：一定要注意字段的类型是否可以转。比如 int可以转string  但string不能转int
ALTER TABLE 表名 CHANGE [COLUMN] 字段名 字段名 字段的新类型 [COMMENT 字段的描述信息]
```

## 更新列-更新列时可以调字段的位置

```hive
#注意：1.在交换位置时只能交换元数据，数据不会交换。 2.还要注意数据类型
ALTER TABLE 表名 CHANGE [COLUMN] 字段名 字段名 字段的新类型 [COMMENT 字段的描述信息]  [FIRST | AFTER 字段的名字]
```

## 替换列

```hive
#1.替换是依次替换   
#2.替换时一定要注意字段的类型   
#3.如果替换的字段的数量少于原数量 直接缺少该字段
ALTER TABLE 表名 REPLACE COLUMNS (字段名 字段类型 [COMMENT 字段的描述信息], ...)
```

## 删除表

```hive
#[IF EXISTS]:如果表存在则删除 不存在则不删。我们发现加不加该字段没区别。
DROP TABLE [IF EXISTS] 表名;
```

## 清空表

```hive
#注意：不能清空外部表
TRUNCATE [TABLE] 表名
```

