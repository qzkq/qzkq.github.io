---
title: SQL常用语句 笔记
date: 2026-01-03 08:10:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿[https://github.com/QInzhengk/Math-Model-and-Machine-Learning](https://github.com/QInzhengk/Math-Model-and-Machine-Learning)
@[TOC](SQL笔记)
## 1.常用MySQL命令
```sql
# 查看所有数据库
SHOW DATABASES;
# 切换指定数据库
USE test;
# 查看当前库中所有的表
SHOW TABLES;
# 查看表结构
DESC departments;
# 查看当前所处的数据库
SELECT DATABASE();
# 查看当前登陆用户
SELECT USER();
# 查看版本
SELECT VERSION();
```

## 2.语法规范

> 关键字不区分大小写，但建议关键字大写
> 表名、列名建议小写
> 每条命令最好用分号结尾
> 每条命令根据需要，可以进行缩进或换行
> 最好是关键字单独占一行

## 3.语句分类

> 数据查询语言（Data Query Language, ）DQL
> 负责进行数据查询而不会对数据本身进行修改的语句。

> 数据定义语言 (Data Definition Language,）DDL
> 负责数据结构定义与数据库对象定义的语言，由CREATE、ALTER与DROP三个语法所组成

> 数据操纵语言（Data Manipulation Language,）DML
> 负责对数据库对象运行数据访问工作的指令集，以INSERT、UPDATE、DELETE三种指令为核心，分别代表插入、更新与删除。

> 数据控制语言 (Data Control Language)
> 它可以控制特定用户账户对数据表、查看表、预存程序、用户自定义函数等数据库对象的控制权。由 GRANT 和 REVOKE 两个指令组成。

### MySQL索引
索引是帮助MySQL高效获取数据的数据结构
索引数据结构：二叉树、红黑树、hash表、B-Tree

 1. 普通索引：最基本的索引，没什么限制。
 2. 唯一索引：索引列的值必须唯一，但允许有空值。
 3. 主键索引：一种特殊的索引，一个表只能有一个主键，不允许有空值。
 4. 组合索引：指多个字段上创建的索引，使用组合索引遵循最左前缀原则。
 5. 全文索引：主要用来查找文本中的关键字，而不是直接与索引中的值比较。

### 数据查询语言
#### 基础查询
```sql
# 查单个字段
select dept_name from departments;
# 查多个字段
select name, email from employees;
# 查所有字段
select * from departments;
# 使用表达式
select date, employee_id, basic+bonus from salary;
# 查询函数，统计salary共有多少行记录
select count(*) from salary;
# 使用别名，字段名和别名之间可以用空格或关键字AS与as指定别名
select dept_id 部门编号, dept_name AS 部门名 from departments;
# 去重 distinct
select dept_id from employees;
select distinct dept_id from employees;
# 使用concat函数进行字符串拼接
select concat(name, '-', phone_number) from employees;
```
#### 条件查询

```sql
select * from departments where dept_id>3; 
select * from departments where dept_id<3;
select * from departments where dept_id=3;
select * from departments where dept_id!=3;
select * from departments where dept_id>=3;
select * from departments where dept_id<=3;

select * from departments where dept_id>1 and dept_id<5;
select * from departments where dept_id<3 or dept_id>6;
select * from departments where not dept_id<=6;
```
#### 模糊查询

- like: 包含
- between x and y :        在x和y之间的
- in：在列表中的
- is null：为空，相当于python的None
- is not null：非空
- %匹配0到多个任意字符
-  _匹配一个字符

```sql
select name, email from employees where name like '张%';

select name, email from employees where name like '张_';

select * from departments where dept_id between 3 and 5;

select * from departments where dept_id in (1, 3, 5, 8);
# 匹配部门名为空的记录
select * from departments where dept_name is null;
# 查询部门名不为空的记录
select * from departments where dept_name is not null;
```
#### 排序（默认升序）

```sql
select name, birth_date from employees where birth_date>'19980101';
# 默认升序排列
select name, birth_date from employees where birth_date>'19980101' order by birth_date;
# 降序排列
select name, birth_date from employees where birth_date>'19980101' order by birth_date desc;
```

#### 函数
##### 字符函数
**LENGTH(str)**：返字符串长度，以(字节)为单位
```sql
select length('abc');
select length('你好');
select name, email, length(email) from employees where name='李平';
```
**CONCAT(s1,s2，...):** 返回连接参数产生的字符串，一个或多个待拼接的内容，任意一个为NULL则返回值为NULL

```sql
select concat(dept_id, '-', dept_name) from departments;
```
**UPPER(str)**和UCASE(str): 将字符串中的字母全部转换成大写

```sql
select name, upper(email) from employees where name like '李%';
```
**LOWER(str)**和LCASE(str):将str中的字母全部转换成小写

**SUBSTR(s, start, length)**: 从子符串s的start位置开始，取出length长度的子串，位置(从1)开始计算

```sql
select substr('hello world', 7);
# 取子串，下标从7开始取出3个
select substr('hello world', 7, 3);
```

**INSTR(str,str1)**：返回str1参数，在str参数内的位置

```sql
# 子串在字符串中的位置
select instr('hello world', 'or');
select instr('hello world', 'ol');
```

**TRIM(s)**: 返回字符串(s删除了两边空格之后的字符串)

```sql
select trim('  hello world.  ');
```
**LEFT(str, length)** ：从左开始截取字符串，length 是截取的长度。

**group_concat**语法

```sql
group_concat([DISTINCT] 要连接的字段 [Order BY ASC/DESC 排序字段] [Separator '分隔符'])
```

##### 数学函数
ABS(x)：返回x的绝对值

```sql
select abs(-10);
```
MOD(x,y): 返回x被y除后的余数

```sql
select mod(10, 3);
```

CEIL(x)、CEILING(x): 返回不小于x的最小整数

```sql
select ceil(10.1);
```
FLOOR(x): 返回不大于x的最大整数

```sql
select floor(10.9);
```

ROUND(x)、ROUND(x,y): 前者返回最接近于x的整数，即对x进行四舍五入；后者返回最接近x的数，其值保留到小数点后面y位，若y为负值，则将保留到x到小数点左边y位

```sql
select round(10.6666);返回最接近于x的整数，即对x进行四舍五入
select round(10.6666, 2);返回最接近x的数，其值保留到小数点后面y位
```

##### 日期和时间函数
**CURDATE()、CURRENT_DATE()**: 将当前日期按照"YYYY-MM-DD"或者"YYYYMMDD"格式的值返回，具体格式根据函数用在字符串或是数字语境中而定

```sql
select curdate();当前日期按照"YYYY-MM-DD"
select curdate() + 0;格式根据函数用在字符串或是数字语境中而定
```

**NOW()**: 返回当前日期和时间值，格式为"YYYY_MM-DD HH:MM:SS"或"YYYYMMDDHHMMSS"，具体格式根据函数用在字符串或数字语境中而定

```sql
select now();式为"YYYY_MM-DD HH:MM:SS"
select now() + 0;具体格式根据函数用在字符串或数字语境中而定
```

**UNIX_TIMESTAMP()、UNIX_TIMESTAMP(date)**: 前者返回一个格林尼治标准时间1970-01-01 00:00:00到现在的秒数，后者返回一个格林尼治标准时间1970-01-01 00:00:00到指定时间的秒数

```sql
select unix_timestamp();
```
**FROM_UNIXTIME(date)**: 和UNIX_TIMESTAMP互为反函数，把UNIX时间戳转换为普通格式的时间

```sql
select from_unixtime(0);
```

**MONTH(date)和MONTHNAME(date)**:前者返回指定日期中的月份，后者返回指定日期中的月份的名称

```sql
select month('20211001120000');返回指定日期中的月份
select monthname('20211001120000');返回指定日期中的月份的名称
```

**DAYNAME(d)、DAYOFWEEK(d)、WEEKDAY(d)**: DAYNAME(d)返回d对应的工作日的英文名称，如Sunday、Monday等；DAYOFWEEK(d)返回的对应一周中的索引，1表示周日、2表示周一；WEEKDAY(d)表示d对应的工作日索引，0表示周一，1表示周二

```sql
select dayname('20211001120000');返回星期*
select dayname('20211001');
```

**WEEK(d)**: 计算日期d是一年中的第几周

```sql
select week('20211001');
```
**DAYOFYEAR(d)、DAYOFMONTH(d)**： 前者返回d是一年中的第几天，后者返回d是一月中的第几天

```sql
select dayofyear('20211001');
```
**YEAR(date)、QUARTER(date)、MINUTE(time)、SECOND(time)**: YEAR(date)返回指定日期对应的年份，范围是1970到2069；QUARTER(date)返回date对应一年中的季度，范围是1到4；MINUTE(time)返回time对应的分钟数，范围是0~59；SECOND(time)返回制定时间的秒值

```sql
select year('20211001');返回指定日期对应的年份
select quarter('20211001');回date对应一年中的季度
```
**datediff(日期1, 日期2)**：得到的结果是日期1与日期2相差的天数。
如果日期1比日期2大，结果为正；如果日期1比日期2小，结果为负。

```sql
SELECT DATEDIFF('2007-12-31 23:59:59','2007-12-30');
1
SELECT DATEDIFF('2010-11-30 23:59:59','2010-12-31');
-31
```

##### 流程控制函数
IF(expr,v1,v2): 如果expr是TRUE则返回v1，否则返回v2

```sql
select if(3>0, 'yes', 'no');
select name, dept_id, if(dept_id=1, '人事部', '非人事部')  from employees where name='张亮';
```
IFNULL(v1,v2): 如果v1不为NULL，则返回v1，否则返回v2

```sql
select dept_id, dept_name, ifnull(dept_name, '未设置') from departments;
insert into departments(dept_id) values(9);
select dept_id, dept_name, ifnull(dept_name, '未设置') from departments; 
```

CASE expr (WHEN v1)( THEN r1) [WHEN v2 THEN v2] [ELSE rn] END: 如果expr等于某个vn，则返回对应位置THEN后面的结果，如果与所有值都不想等，则返回ELSE后面的rn

```sql
select dept_id, dept_name,
case dept_nam
when '运维部' then '技术部门'
when '开发部' then '技术部门'
when null then '未设置'
else '非技术部门'
end as '部门类型'
from departments;
```

```sql
select dept_id, dept_name,
case 
when dept_name='运维部' then '技术部门'
when dept_name='开发部' then '技术部门'
when dept_name is null then '未设置'
else '非技术部门'
end as '部门类型'
from departments;
```
##### 分组函数
用于统计，又称为聚合函数或统计函数

```sql
# sum/min/count/avg
select employee_id, max(basic+bonus) from salary where employee_id=10 and year(date)=2018;
```
#### 分组查询
语法格式

 -  **查询列表必须是分组函数和出现在（GROUP BY）后面的字段**
 -  通常而言，**分组前的数据筛选放在where子句中，分组后的数据筛选放在having子句中**

```sql
SELECT 字段名1(要求出现在group by后面)，分组函数(),……
FROM 表名
WHERE 条件
GROUP BY 字段名1，字段名2
HAVING 过滤条件
ORDER BY 字段;
```


```sql
查询每个部门的人数
select dept_id, count(*) from employees group by dept_id;

查询每个部门中年龄最大的员工
select dept_id, min(birth_date) from employees group by dept_id;

查询每个部门入职最晚员工的入职时间
select dept_id, max(hire_date) from employees group by dept_id;

统计各部门使用tedu.cn邮箱的员工人数
select dept_id, count(*) from employees where email like '%@tedu.cn' group by dept_id;
+---------+----------+
| dept_id | count(*) |
+---------+----------+
|       1 |        5 |
|       2 |        2 |
|       3 |        4 |
|       4 |       32 |
|       5 |        7 |
|       6 |        5 |
|       7 |       15 |
|       8 |        1 |
+---------+----------+
8 rows in set (0.00 sec)

查看员工2018年工资总收入，按总收入进行降序排列
select employee_id, sum(basic+bonus) as total from salary where year(date)=2018 group by employee_id order by total desc;

查询部门人数少于10人
select dept_id, count(*) from employees where count(*)<10 group by dept_id;
ERROR 1111 (HY000): Invalid use of group function
 
select dept_id, count(*) from employees group by dept_id having count(*)<10;
+---------+----------+
| dept_id | count(*) |
+---------+----------+
|       1 |        8 |
|       2 |        5 |
|       3 |        6 |
|       6 |        9 |
|       8 |        3 |
+---------+----------+
5 rows in set (0.00 sec)
```
**查询结果中如果有where，group by（包含having），order by，使用的顺序group by（包含having）必须在where之后，order by之前。**
#### 连接查询
也叫多表查询。常用于查询字段来自于多张表

```sql
如果直接查询两张表，将会得到笛卡尔积
select name, dept_name from employees, departments;

通过添加有效的条件可以进行查询结果的限定
select name, dept_name from employees, departments where employees.dept_id=departments.dept_id;
```
语法格式

```sql
SELECT 字段... 
FROM 表1 [AS] 别名 [连接类型]
JOIN 表2 [AS] 别名
ON 连接条件
WHERE 分组前筛选条件
GROUP BY 分组
HAVING 分组后筛选条件
ORDER BY 排序字段
```
##### 内连接

```sql
select 查询列表
from 表1 别名
inner join 表2 别名 on 连接条件
inner join 表3 别名 on 连接条件
[where 筛选条件]
[group by 分组]
[having 分组后筛选]
[order by 排序列表]
```
##### 等值连接
查询每个员工所在的部门名，使用别名。两个表中的同名字段，必须指定表名
```sql
select name, d.dept_id, dept_name
from employees as e
inner join departments as d
on e.dept_id=d.dept_id;
```

查询2018年总工资大于30万的员工，按工资降序排列

```sql
select name, sum(basic+bonus) as total from employees as e
inner join salary as s
on e.employee_id=s.employee_id
where year(s.date)=2018
group by name
having total>300000
order by total desc;
```
##### 非等值连接 between ... and ...（前面包括后面不包括）
##### 创建表语法：

```sql
CREATE TABLE 表名称
(
列名称1 数据类型,
列名称2 数据类型,
列名称3 数据类型,
....
)
```


```sql
mysql> use test;
mysql> create table age_grade
    -> (
    -> id int, #主键。仅作为表的行号
    -> grade char(1), #工资级别，共ABCDE五类
    -> low int, #该级别最低工资
    -> high int, #该级别最高工资
    -> primary key (id));
```

##### 向表中插入数据语法：

```sql
INSERT INTO 表名称 VALUES (值1, 值2,....);
```

```sql
insert into age_grade values
(1, 'A', 5000, 8000),
(2, 'B', 8001, 10000),
(3, 'C', 10001, 15000);
```
查询2018年12月员工各基本工资级别的人数
```sql
select grade, count(*)
from salary as s
inner join wage_grade as g
on s.basic between g.low and g.high
where year(date)=2018 and month(date)=12
group by grade;
```
查询2018年12月员工基本工资级别，员工需要显示姓名

```sql
select name, date, basic, grade
from salary as s
inner join employees as e
on s.employee_id=e.employee_id
inner join wage_grade
on basic between low and high
where date='20181210'
order by grade, basic;
```
##### 自连接

 - 将一张表作为两张使用
 -  每张表起一个别名

查看哪些员的生日月份与入职月份相同

```sql
select e.name, e.hire_date, em.birth_date
from employees as e
inner join employees as em
on month(e.hire_date)=month(em.birth_date)
and e.employee_id=em.employee_id;
```
##### 外连接的概述

 - 常用于查询一个表中有，另一个表中没有的记录
 - 如果从表中有和它匹配的，则显示匹配的值
 -  如要从表中没有和它匹配的，则显示NULL
 - 外连接查询结果=内连接查询结果+主表中有而从表中没有的记录
 - 左外连接中，left join左边的是主表left outer join
 - 右外连接中，right join右边的是主表right outer join
 - 左外连接和右外连接可互换，实现相同的目标

##### 左外连接
语法

```sql
SELECT tb1.字段..., tb2.字段
FROM table1 AS tb1
LEFT OUTER JOIN table2 AS tb2 
ON tb1.字段=tb2.字段
```
查询所有部门的人员以及没有员工的部门

```sql
select d.*, e.name
from departments as d
left outer join employees as e
on d.dept_id=e.dept_id;
```
##### 右外连接
查询所有部门的人员以及没有员工的部门

```sql
select d.*, e.name
    -> from employees as e
    -> right outer join departments as d
    -> on d.dept_id=e.dept_id;
```
##### left join和right join 的区别
left join（左连接）：查询结果为两个表匹配到的数据，左表特有的数据，对于右表中不存在的数据使用null填充。
##### 交叉连接 cross join
返回笛卡尔积

```sql
SELECT <字段名> FROM <表1> CROSS JOIN <表2> [WHERE子句]
```


#### 子查询
子查询就是指的在一个完整的查询语句之中，嵌套若干个不同功能的小查询，从而一起完成复杂查询的一种编写形式

子查询返回的数据分类
       

 - 单行单列：返回的是一个具体列的内容，可以理解为一个单值数据
 - 单行多列：返回一行数据中多个列的内容
 - 多行单列：返回多行记录之中同一列的内容，相当于给出了一个操作范围
 - 多行多列：查询返回的结果是一张临时表

子查询常出现的位置
        
 - select之后：仅支持单行单列        
 - from之后：支持多行多列
 - where或having之后：支持单行单列、单行多列、多行单列

##### 单行单列
查询运维部所有员工信息

```sql
#  首先从departments表中查出运维部的编号
select dept_id from departments where dept_name='运维部';
+---------+
| dept_id |
+---------+
|       3 |
+---------+
1 row in set (0.00 sec)
# 再从employees表中查找该部门编号和运维部编号相同的员工
select *
from employees
where dept_id=(
   select dept_id from departments where dept_name='运维部'
);
```
查询每个部门的人数

```sql
# 查询所有部门的信息
select d.* from departments as d;
+---------+-----------+
| dept_id | dept_name |
+---------+-----------+
|       1 | 人事部    |
|       2 | 财务部    |
|       3 | 运维部    |
|       4 | 开发部    |
|       5 | 测试部    |
|       6 | 市场部    |
|       7 | 销售部    |
|       8 | 法务部    |
|       9 | NULL      |
+---------+-----------+
9 rows in set (0.00 sec)
# 查询每个部门的人数
select d.*, (
select count(*) from employees as e
   where d.dept_id=e.dept_id
) as amount
from departments as d;
+---------+-----------+--------+
| dept_id | dept_name | amount |
+---------+-----------+--------+
|       1 | 人事部    |      8 |
|       2 | 财务部    |      5 |
|       3 | 运维部    |      6 |
|       4 | 开发部    |     55 |
|       5 | 测试部    |     12 |
|       6 | 市场部    |      9 |
|       7 | 销售部    |     35 |
|       8 | 法务部    |      3 |
|       9 | NULL      |      0 |
+---------+-----------+--------+
9 rows in set (0.00 sec)
```

##### 多行单列
##### 单行多列
查找2018年12月基本工资和奖金都是最高的工资信息
```sql
# 查询2018年12月最高的基本工资
select max(basic) from salary
where year(date)=2018 and month(date)=12;
+------------+
| max(basic) |
+------------+
|      25524 |
+------------+
1 row in set (0.00 sec)

# 查询2018年12月最高的奖金
select max(bonus) from salary
where year(date)=2018 and month(date)=12;
+------------+
| max(bonus) |
+------------+
|      11000 |
+------------+
1 row in set (0.00 sec)

mysql> select * from salary
    -> where year(date)=2018 and month(date)=12 and basic=(
    ->   select max(basic) from salary
    ->   where year(date)=2018 and month(date)=12
    -> ) and bonus=(
    ->   select max(bonus) from salary
    ->   where year(date)=2018 and month(date)=12
    -> );
+------+------------+-------------+-------+-------+
| id   | date       | employee_id | basic | bonus |
+------+------------+-------------+-------+-------+
| 6368 | 2018-12-10 |         117 | 25524 | 11000 |
+------+------------+-------------+-------+-------+
1 row in set (0.01 sec)
```

##### 多行多列
查询3号部门及其部门内员工的编号、名字和email

```sql
# 查询3号部门和员工的所有信息
select d.dept_name, e.*
from departments as d
inner join employees as e
on d.dept_id=e.dept_id;

# 将上述结果当成一张临时表，必须为其起别名。再从该临时表中查询
mysql> select dept_id, dept_name, employee_id, name, email
    -> from (
    ->   select d.dept_name, e.*
    ->   from departments as d
    ->   inner join employees as e
    ->   on d.dept_id=e.dept_id
    -> ) as tmp_table
    -> where dept_id=3;
+---------+-----------+-------------+-----------+--------------------+
| dept_id | dept_name | employee_id | name      | email              |
+---------+-----------+-------------+-----------+--------------------+
|       3 | 运维部    |          14 | 廖娜      | liaona@tarena.com  |
|       3 | 运维部    |          15 | 窦红梅    | douhongmei@tedu.cn |
|       3 | 运维部    |          16 | 聂想      | niexiang@tedu.cn   |
|       3 | 运维部    |          17 | 陈阳      | chenyang@tedu.cn   |
|       3 | 运维部    |          18 | 戴璐      | dailu@tedu.cn      |
|       3 | 运维部    |          19 | 陈斌      | chenbin@tarena.com |
+---------+-----------+-------------+-----------+--------------------+
6 rows in set (0.00 sec)
```
##### 分页查询

```sql
# 按employee_id排序，取出前15至20号员姓名
select employee_id, name from employees
order by employee_id
limit 15, 5;
+-------------+--------+
| employee_id | name   |
+-------------+--------+
|          16 | 聂想   |
|          17 | 陈阳   |
|          18 | 戴璐   |
|          19 | 陈斌   |
|          20 | 蒋红   |
+-------------+--------+
5 rows in set (0.00 sec)
```
##### 联合查询UNION
作用：将多条select语句的结果，合并到一起，称之为联合操作。

语法：( ) UNION ( )

 - 要求查询时，多个select语句的检索到的字段数量必须一致
 -  每一条记录的各字段类型和顺序最好是一致的
 - UNION关键字默认去重，可以使用UNION ALL包含重复项

查询1972年前或2000年后出生的员工
```sql
select name, birth_date from employees
where year(birth_date)<1972 or year(birth_date)>2000;
+-----------+------------+
| name      | birth_date |
+-----------+------------+
| 梁伟      | 1971-08-19 |
| 张建平    | 1971-11-02 |
| 窦红梅    | 1971-09-09 |
| 温兰英    | 1971-08-14 |
| 朱文      | 1971-08-15 |
| 和林      | 1971-12-10 |
+-----------+------------+
6 rows in set (0.01 sec)


mysql> (
    -> select name, birth_date from employees
    ->   where year(birth_date)<1972
    -> )
    -> union
    -> (
    ->   select name, birth_date from employees
    ->   where year(birth_date)>=2000
    -> );
```
### 插入语句
#### 不指定列名的插入
语法格式：

```sql
INSERT INTO 表名称 VALUES (值1, 值2,....)
```

 - 需要为所有列指定值
 - 值的顺序必须与表中列的顺序一致

#### 指定列名的插入
语法格式：

```sql
INSERT INTO table_name (列1, 列2,...) VALUES (值1, 值2,....)
```

 - 列和值的顺序要一致
 - 列名先后顺序不重要


主键由于是自动增长的，也可以不指定主键的值

支持子查询

```sql
mysql> insert into employees
    -> (name, hire_date, birth_date, email, phone_number, dept_id)
    -> (
    ->   select name, hire_date, birth_date, email, phone_number, dept_id
    ->   from employees
    ->   where name='张三'
    -> );
Query OK, 1 row affected (0.00 sec)
Records: 1  Duplicates: 0  Warnings: 0
```
#### 使用set语句
语法格式：

```sql
INSERT INTO 表名 SET 列名1=列值1, 列名2=列值2, ...
```

#### 修改语句
修改单表记录

语法：
```sql
UPDATE 表名称 SET 列名称=新值, 列名称=新值, ... WHERE 筛选条件
```
修改多表记录

语法：

```sql
UPDATE 表1 AS 表1别名
INNER | LEFT | RIGHT JOIN 表2 AS 表2别名
ON 连接条件
SET 列=值, 列=值, ...
WHERE 连接条件
```

```sql
# 修改李四所在部门为企划部
update departments as d
inner join employees as e
on d.dept_id=e.dept_id
set d.dept_name='企划部'
where e.name='李四';
```
#### 删除记录
删除单表记录
语法：

```sql
DELETE FROM 表名 WHERE 筛选条件;
```
删除的是满足条件的整行记录，而不是某个字段
##### 删除重复的电子邮箱
表: Person

```sql
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| email       | varchar |
+-------------+---------+
id是该表的主键列。
该表的每一行包含一封电子邮件。电子邮件将不包含大写字母。
```

编写一个 SQL **删除语句**来 **删除** 所有重复的电子邮件，只保留一个id最小的唯一电子邮件。

以 **任意顺序** 返回结果表。 （注意： 仅需要写删除语句，将自动对剩余结果进行查询）

查询结果格式如下所示。

示例 1:

```sql
输入: 
Person 表:
+----+------------------+
| id | email            |
+----+------------------+
| 1  | john@example.com |
| 2  | bob@example.com  |
| 3  | john@example.com |
+----+------------------+
输出: 
+----+------------------+
| id | email            |
+----+------------------+
| 1  | john@example.com |
| 2  | bob@example.com  |
+----+------------------+
解释: john@example.com重复两次。我们保留最小的Id = 1。
```

```sql
delete p1.* 
from Person p1,Person p2 
where p1.email=p2.email and p1.id>p2.id
```

#### 删除多表记录
语法：

```sql
DELETE 表1别名, 表2别名
FROM 表1 AS 表1别名
INNER | LEFT | RIGHT JOIN 表2 AS 表2别名
ON 连接条件
WHERE 筛选条件
```

```sql
# 删除9号部门中所有的员工
delete e
from employees as e
inner join departments as d
on e.dept_id=d.dept_id
where d.dept_id=9;
Query OK, 2 rows affected (0.00 sec)
```
#### 清空表
语法：

```sql
TRUNCATE TABLE 表名
TRUNCATE不支持WHERE条件
```

 - 自增长列，TRUNCATE后从1开始；DELETE继续编号
 - TRUNCATE不能回滚，DELETE可以
 - 效率略高于DELETE

## 数据库管理
### 创建数据库
语法：

```sql
CREATE DATABASE [IF NOT EXISTS] <数据库名>
[[DEFAULT] CHARACTER SET <字符集名>] 
[[DEFAULT] COLLATE <校对规则名>];
```

 - [ ]中的内容是可选的
 - <数据库名>：创建数据库的名称。MySQL 的数据存储区将以目录方式表示 MySQL 数据库，因此数据库名称必须符合操作系统的文件夹命名规则，不能以数字开头，尽量要有实际意义。
 - IF NOT EXISTS：在创建数据库之前进行判断，只有该数据库目前尚不存在时才能执行操作。此选项可以用来避免数据库已经存在而重复创建的错误。
 - [DEFAULT] CHARACTER SET：指定数据库的字符集。指定字符集的目的是为了避免在数据库中存储的数据出现乱码的情况。如果在创建数据库时不指定字符集，那么就使用系统的默认字符集。
 - [DEFAULT] COLLATE：指定字符集的默认校对规则。
 - MySQL 的字符集（CHARACTER）和校对规则（COLLATION）是两个不同的概念。字符集是用来定义 MySQL 存储字符串的方式，校对规则定义了比较字符串的方式。
### 修改数据库
语法：

```sql
ALTER DATABASE [数据库名] { 
[ DEFAULT ] CHARACTER SET <字符集名> |
[ DEFAULT ] COLLATE <校对规则名>}
```

 - ALTER DATABASE 用于更改数据库的全局特性。
 - 使用 ALTER DATABASE 需要获得数据库 ALTER 权限。
 - 数据库名称可以忽略，此时语句对应于默认数据库。
 - CHARACTER SET 子句用于更改默认的数据库字符集。

### 删除数据库
语法：

```sql
DROP DATABASE [ IF EXISTS ] <数据库名>
```

 - <数据库名>：指定要删除的数据库名。
 - IF EXISTS：用于防止当数据库不存在时发生错误。
 - DROP DATABASE：删除数据库中的所有表格并同时删除数据库。
 - 如果要使用 DROP DATABASE，需要获得数据库 DROP 权限。

## 关系数据库的规范化
#### 数据库设计的三大范式
**第一范式**：要求表的每个字段必须是不可分割的独立单元。
**第二范式**：在第一范式的基础上，要求每张表只表达一个意思。表的每个字段都和表的主键有依赖。
**第三范式**：在第二范式的基础上，要求每张表的主键之外的其它字段都只能和主键有直接决定依赖关系。

#### 修改表
修改列名

语法：

```sql
ALTER TABLE 表
CHANGE [COLUMN] 列表 数据类型
```
#### 修改列的类型或约束
语法：

```sql
ALTER TABLE 表
MODIFY [COLUMN] 列名 类型
```
#### 添加新列
语法：

```sql
ALTER TABLE 表
ADD [COLUMN] 列名 类型
```
#### 删除列
语法：

```sql
ALTER TABLE 表
DROP [COLUMN] 列名
```
#### 修改表名
语法：

```sql
ALTER TABLE 表名
RENAME TO 新表名
```
#### 删除表
语法：

```sql
DROP TABLE [IF EXISTS] 表名
```
#### 表复制
仅复制表结构

语法：

```sql
CREATE TABLE 待创建的表名 LIKE 已有表名
```
#### 复制表结构及数据
语法：

```sql
CREATE TABLE 待创建的表名
SELECT 字段, ... FROM 已有表名
```
## 约束
### 约束分类

 - PRIMARY KEY：主键，用于保证该字段的值具有唯一性并且非空。
 - NOT NULL ：非空，用于保证该字段的值不能为空。
 - DEFAULT：默认值，用于保证该字段有默认值。
 - UNIQUE：唯一，用于保证该字段的值具有唯一性，可以为空。
 - FOREIGN KEY：外键，用于限制两个表的关系，用于保证该字段的值必须来自于主表的关联列的值，在从表添加外键约束，用于引用主表中某些的值。

约束可应用在列级或表级。列表所有约束均支持，但外键约束没有效果；表级约束可以支持主键、唯一、外键约束。
#### 删除约束
语法：

```sql
ALTER TABLE <表名> DROP FOREIGN KEY <外键约束名>
```
## 事务控制语言
### 事务

 - 也称工作单元，是由一个或多个SQL语句所组成的操作序列，这些SQL语句作为一个完整的工作单元，要么全部执行成功，要么全部执行失败。在数据库中，通过事务来保证数据的一致性。
### 事务的特性（ACID）
 - 原子性(Atomicity)：事务就像原子一样，不可被分割，组成事务的DML操作语句要么全成功，要么全失败，不可能出现部分成功部分失败的情况。
 - 一致性(Consistency)：一旦事务完成，不管是成功的，还是失败的，整个系统处于数据一致的状态。
 - 隔离性(Isolation)：一个事务的执行不会被另一个事务所干扰。比如两个人同时从一个账户取钱，通过事物的隔离性，确保账户余额的正确性。
 - 持久性(Durability)：也称永久性，指事务一旦提交，对数据的改变就是永久的，不可以被在回滚。

## 参考链接
[https://blog.csdn.net/kali_yao/article/details/120209248?spm=1001.2014.3001.5506](https://blog.csdn.net/kali_yao/article/details/120209248?spm=1001.2014.3001.5506)

[https://leetcode.cn/problems/delete-duplicate-emails/](https://leetcode.cn/problems/delete-duplicate-emails/)

[https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_datediff](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_datediff)
