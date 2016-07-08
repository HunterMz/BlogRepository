---
title: SQL语句的简单使用
tags:
  - 数据库
category:
  - 编程基础
date: 2016-07-07 14:30:32
---


* 字段类型

``` SQL
integer: 整形值
real: 浮点值
text: 文本字符串
blob: 二进制数据(比如文件)
```

<!--more-->

* 创建表

``` SQL
create table 表名(字段名1 字段类型， 字段名2 字段类型， ...)
```

* 删表

``` SQL
drop table 表名 ;

drop table if exists 表名 ;
```

* 插入数据 (insert)

``` SQL
insert into 表名 (字段1, 字段2, …) values (字段1的值, 字段2的值, …) ;

insert into test_table (name, age) values (‘mj’, 10) ;
```

* 更新数据

``` SQL
update 表名 set 字段1 = 字段1的值, 字段2 = 字段2的值, … ;

update test_table set name = ‘jack’, age = 20 ;
```

* 删除数据

``` SQL
delete from 表名 ;

delete from test_table ;  // 会删除test_table中的所有数据
```

* 条件语句


``` SQL
条件语句的常见格式:

where 字段 = 某个值 ;   // 不能用两个 =

where 字段 is 某个值 ;   // is 相当于 =

where 字段 != 某个值 ;

where 字段 is not 某个值 ;   // is not 相当于 !=

where 字段 > 某个值 ;

where 字段1 = 某个值 and 字段2 > 某个值 ;  // and相当于C语言中的 &&

where 字段1 = 某个值 or 字段2 = 某个值 ;  //  or 相当于C语言中的 ||

示例:
update t_student set age = 5 where age > 10 and name != ‘jack’ ;

delete from t_student where age <= 10 or age > 30 ;
```

* DQL语句

``` SQL
格式

select 字段1, 字段2, … from 表名 ;

select * from 表名;   //  查询所有的字段


示例

select name, age from test_table ;

select * from test_table ;

select * from test_table where age > 10 ;  //  条件查询
```

* 起别名

``` SQL
格式(字段和表都可以起别名)

select 字段1 别名 , 字段2 别名 , … from 表名 别名 ;

select 字段1 别名, 字段2 as 别名, … from 表名 as 别名 ;

select 别名.字段1, 别名.字段2, … from 表名 别名 ;


示例

select name myname, age myage from test_table ;  // 给name起个叫做myname的别名，给age起个叫做myage的别名

select s.name, s.age from test_table s ;  //给test_table表起个别名叫做s，利用s来引用表中的字段
```

* 查询表中某一记录的数量

``` SQL
格式

select count (字段) from 表名 ;

select count ( * ) from 表名 ;


示例

select count (age) from test_table ;

select count ( * ) from test_table where score >= 60;
```

* 排序

``` SQL
查询出来的结果可以用order by进行排序

select * from t_student order by 字段 ;

select * from t_student order by age ;


默认是按照升序排序（由小到大），也可以变为降序（由大到小）

select * from t_student order by age desc ;  //降序

select * from t_student order by age asc ;   // 升序（默认）


也可以用多个字段进行排序

select * from t_student order by age asc, height desc ;

先按照年龄排序（升序），年龄相等就按照身高排序（降序）
```

* 使用limit

``` SQL
使用limit可以精确地控制查询结果的数量，比如每次只查询10条数据

格式

select * from 表名 limit 数值1, 数值2 ;  // 数值1表示从第几个数开始，数值2表示每次查询多少个

示例

select * from t_student limit 4, 8 ;

可以理解为：跳过最前面4条语句，然后取8条记录

limit常用来做分页查询，比如每页固定显示5条数据，那么应该这样取数据

第1页：limit 0, 5

第2页：limit 5, 5

第3页：limit 10, 5

…

第n页：limit 5*(n-1), 5

select * from t_student limit 7 ;这条语句的作用相当于select * from t_student limit 0, 7 ;表示取最前面的7条记录
```


