---
title: Python连接MySQL及查询实战
date: 2026-01-02 08:15:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿@[TOC](Python连接MySQL及查询实战)

# 1Python DB-API
## 1.1概述

 - Python 标准数据库接口为 Python DB-API，Python DB-API为开发人员提供了数据库应用编程接口。
 - PyMySQL 是在 Python3.x 版本中用于连接 MySQL 服务器的一个实现库，Python2中则使用mysqldb。
 - PyMySQL 遵循 Python 数据库 API v2.0 规范，并包含了 pure-Python MySQL 客户端库。

## 1.2安装PyMySQL

 - 在使用 PyMySQL 之前，我们需要确保 PyMySQL 已安装。
 - PyMySQL下载地址：https://github.com/PyMySQL/PyMySQL。
 - 如果还未安装，我们可以使用以下命令安装最新版的 PyMySQL： pip install PyMySQL

## 1.3连接数据库

 - 数据库准备，连接数据库之前，请确保已经创建了python数据库，以及students表
 - **创建Connection对象**：用于建立与数据库的连接

```bash
from pymysql import * 
# 导入pymysql模块 
# 创建连接对象 Connection对象 
# host:数据库主机地址 
# user:数据库账号 
# password：数据库密码 
# database : 需要连接的数据库的名称 
# port: mysql的端口号 
# charset: 通信采用编码格式 
conn = connect(host='127.0.0.1', user='root', password='mysql', database='python', port=3306, charset='utf8')
```

### Connection 连接对象拥有的方法

 - close 关闭连接，连接数据库跟打开文件一样，操作完成之后需要关闭，否则会占用连接。
 - commit()提交，pymysql 默认开启事物，所以每次更新数据库都要提交
 - rollback()回滚，事物回滚
 - cursor()返回Cursor对象，用于执行sql语句并获得结果

### 获取cursor对象

```bash
cur = conn.cursor() # cursor对象用于执行sql语句
```

**cursor对象拥有的方法**

 - close()关闭cursor对象
 - execute(operation [, parameters ])执行语句，返回受影响的行数，可以执行所有语句
 - fetchone()获取查询结果集的第一个行数据，返回一个元组
 - fetchall()执行查询时，获取结果集的所有行，一行构成一个元组，再将这些元组装入一个元组返回

**插入数据**

```bash
res = cur.execute('insert into students VALUES (0,"白里守约",0,"广州",2,18);’) 
print(res) # 查看受影响的行数 
conn.commit() # 提交事物
```

**查询数据：**

```bash
cur.execute('select * from students;') # 执行sql语句，select * from students; 
res = cur.fetchone() # 获取结果集的第一条数据 
res1 = cur.fetchall() # 获取结果及的所有数据 
print(res) # 将获取的查询结果打印 
print(res1)
```

 - 执行sql语句参数化，参数化sql语句中使用%s占位。
 - execute(operation [parameters])
 - 执行语句，返回受影响的行数，可以执行所有语句 [parameters] 参数列表

```bash
sql = 'select * from students where id=%s and gender= %s;' # sql语句中使用%s占位	
#执行sql语句					
cur.execute(sql,[15,0])
```

### 实例：

```bash
from pymysql import *
#创建数据库的连接
conn=connect(host='192.168.117.128',user='root',password='111111',
             database='stuDB',charset='utf8')
#创建一个游标对象 可以利用这个对象进行数据库的操作
try:
    cur=conn.cursor()
    insertsql='''
    insert into student(id,name,hometown) values (66,'钱之坑','北京市')
    '''
    cur.execute('select * from student where id=%s',[66])
    # conn.commit()
    res=cur.fetchall()
    for item in res:
        print('姓名;{0} 地址{1}'.format(item[1],item[3]))
    print(res)
    #print('sucess')
except Exception as ex:
    print(ex)
finally:
    cur.close()
    conn.close()
```

# 2查询实战
## 2.1准备数据
### 创建表

```bash
create table goods( 
id int unsigned primary key auto_increment not null, 
name varchar(150) not null, 
cate varchar(40) not null, 
brand_name varchar(40) not null, 
price decimal(10,3) not null default 0 
);
```

**使用insert语句，插入多条数据**

```bash
insert into goods values(0,' Apple MacBook Air 13.3英寸笔记本电脑','笔记本','苹果','6588’); 
insert into goods values(0,'联想(Lenovo)拯救者R720 15.6英寸大屏','笔记本','联想','6099’); 
insert into goods values(0,'法国酒庄直采原瓶原装进口AOC级艾落干红葡萄酒','红酒','法国','499’); 
insert into goods values(0,'x550cc 15.6英寸笔记本','笔记本','华硕','2799’); 
insert into goods values(0,'清扬(CLEAR)洗发水','洗发水','清扬','35’); 
......
```

## 2.2查询
### 查询goods表中所有的商品

```bash
select * from goods;
```

### 查询所有产品的平均价格,并且保留两位小数

```bash
select round(avg(price),2) as avg_price from goods;
```

### 通过子查询来实现，查询所有价格大于平均价格的商品，并且按价格降序排序

```bash
select id,name,price from goods 
where price > (select round(avg(price),2) as avg_price from goods) 
order by price desc;
```

### 查询所有 “联想” 的产品

```bash
select * from goods where brand_name='联想';
```

### 查询价格大于或等于"联想"价格的商品，并且按价格降序排列

```bash
select id,name,price from goods  where price >= any(select price from goods where brand_name = '联想’)  order by price desc;
```

### 查询每个产品类型的最低价格的，通过cate字段进行分组。

```bash
select cate,min(price) from goods group by cate;
```

### 查询价格区间在4500-6500之间的笔记本

```bash
select * from goods where price between 4500 and 6500 and cate='笔记本';
```

## 2.3查询数据分表
### 创建一个商品表
create table if not exists goods_cates( 

```bash
cate_id int unsigned primary key auto_increment, 
cate_name varchar(40) 
);
```

**1、查询goods表中所有的商品，并且按"类别"分组**

```bash
select cate from goods group by cate;
```

**2、将分组后的结果写入到刚才创建的表中**

```bash
insert into goods_cates (cate_name) select cate from goods group by cate;
```

**3、通过goodscates数据表来更新goods表，将goods表中的cate字段，修改成goodscates的id字段**

```bash
update goods as g inner join goods_cates as c on g.cate = c.cate_name 
set cate = cate_id;
```

**4、字段 brand_name 进行分表。**

```bash
create table if not exists goods_brands( 
brand_id int unsigned primary key auto_increment, 
brand_name varchar(40) 
); 

insert into goods_brands(brand_name) select brand_name from goods group by brand_name;
```

**5、通过goodsbrands数据表来更新goods表，将goods表中的barndname字段，修改成goods_brands的id字段**

```bash
update goods as g inner JOIN goods_brands as j on g.brand_name=j.brand_name 
set g.brand_name=j.brand_id;
```

**6、查看goods表结构，发现 cate， 、brand_name 两个字段都是varchar字段，需要修改成int类型字段。**

```bash
desc goods; 
alter table goods change cate cate_id int unsigned not null, 
change brand_name brand_id int unsigned not null;
```

**7、通过左连接查询所有商品的信息**

```bash
select id,name,cate_name,brand_name,price from goods as g 
left join goods_cates as c on g.cate_id = c.cate_id 
left join goods_brands as b on g.brand_id = b.brand_id;
```

**8、通过右连接查询所有商品的信息**

```bash
select id,name,cate_name,brand_name,price from goods as g 
right join goods_cates as c on g.cate_id = c.cate_id 
right join goods_brands as b on g.brand_id = b.brand_id;
```

