---
title: Python redis 使用(笔记)
date: 2026-01-02 08:02:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿[https://github.com/QInzhengk/Math-Model-and-Machine-Learning](https://github.com/QInzhengk/Math-Model-and-Machine-Learning)
@[TOC](Python redis 使用（源码）)
## Python redis 使用介绍（通过源码查看怎么用）
**redis 是一个 Key-Value 数据库，Value 支持 string(字符串)，list(列表)，set(集合)，zset(有序集合)，hash(哈希类型)等类型。**

### 1、安装启动 redis
#### 1.1 用brew安装
**1.查看系统是否已经安装了Redis**

```python
brew info redis
```
这个命令会展示此系统下的redis信息，如果没有安装，会展示not install
**2.输入命令安装Redis**

```python
brew install redis
```

可能需要等一会，系统下载完redis的包，会自动进行安装

**3.启动redis**

```python
brew services start redis
```

这个命令会在后台启动redis服务，并且每一次登录系统，都会自动重启

如果你不想/不需要后台服务，你可以运行:

```python
/usr/local/opt/redis/bin/redis-server /usr/local/etc/redis.conf
```
**4.查看redis服务进程**

可以通过下面命令查看redis是否正在运行
```python
ps axu | grep redis
```
**默认端口号为 6379，ctrl+D 退出**

这个命令会读取redis的配置文件，并且在redis运行的过程中也会看到实时的日志打印。
### 2、redis 模块（Python）
redis 提供两个类 Redis 和 StrictRedis
 1. StrictRedis 用于实现大部分官方的命令 
 2. Redis 是 StrictRedis 的子类，用于向后兼用旧版本。

redis 取出的结果默认是字节，可以设定 decode_responses=True 改成字符串。

#### 2.1 连接池
redis-py 使用 connection pool 来管理对一个 redis server 的所有连接，避免每次建立、释放连接的开销。
默认，每个Redis实例都会维护一个自己的连接池。可以直接建立一个连接池，然后作为参数 Redis，这样就可以实现多个 Redis 实例共享一个连接池。

### 3、redis 基本命令 String

```python
set(name, value, ex=None, px=None, nx=False, xx=False)
```

在 Redis 中设置值，默认，不存在则创建，存在则修改。

参数：
 - ex - 过期时间（秒） 
 - px - 过期时间（毫秒） 
 - nx - 如果设置为True，则只有name不存在时，当前set操作才执行 
 - xx - 如果设置为True，则只有name存在时，当前set操作才执行

1.**setnx(name, value) 设置值，只有name不存在时，执行设置操作（添加）**

2.**setex(name, time, value) 设置值**
 - time - 过期时间（数字秒 或 timedelta对象）


```python
import redis
import time
pool = redis.ConnectionPool(host='localhost', port=6379, decode_responses=True)
r = redis.Redis(connection_pool=pool)
r.setex("fruit2", 5, "orange")
print(r.get('fruit2'))
time.sleep(5)
print(r.get('fruit2'))  # 5秒后，取值就从orange变成None
# 输出结果
orange
None
```

3.**psetex(name, time_ms, value) 设置值**
 - time_ms - 过期时间（数字毫秒 或 timedelta对象）

4.**mset(self, mapping) 批量设置值**  

```python
r.mset({'k1': 'v1', 'k2': 'v2'})
print(r.mget("k1", "k2"))   # 一次取出多个键对应的值
print(r.mget("k1"))
# 输出结果
['v1', 'v2']
['v1']
```
5.mget(keys, *args) **批量获取**

```python
print(r.mget('k1', 'k2'))
print(r.mget(['k1', 'k2']))
print(r.mget("fruit", "fruit1", "fruit2", "k1", "k2"))  # 将目前redis缓存中的键对应的值批量取出来
# 输出结果
['v1', 'v2']
['v1', 'v2']
['watermelon', None, 'orange', 'v1', 'v2']
```

6.**getset(name, value) 设置新值并获取原来的值**

```python
print(r.getset("food", "barbecue"))  # 设置的新值是barbecue 设置前的值是beef
```

7.**getrange(key, start, end) 获取子序列（根据字节获取，非字符）**

 - start - 起始位置（字节） 
 - end - 结束位置（字节）

```python
r.set("cn_name", "君惜大大") # 汉字
print(r.getrange("cn_name", 0, 2))   # 取索引号是0-2 前3位的字节 君 切片操作 （一个汉字3个字节 1个字母一个字节 每个字节8bit）
print(r.getrange("cn_name", 0, -1))  # 取所有的字节 君惜大大 切片操作
r.set("en_name","junxi") # 字母
print(r.getrange("en_name", 0, 2))  # 取索引号是0-2 前3位的字节 jun
print(r.getrange("en_name", 0, -1)) # 取所有的字节 junxi
```

8.**setrange(name, offset, value) 修改字符串内容，从指定字符串索引开始向后替换（新值太长时，则向后添加）**

 - offset - 字符串的索引，字节（一个汉字三个字节）
 -  value - 要设置的值

```python
r.setrange("en_name", 1, "ccc")
print(r.get("en_name"))    # jccci 原始值是junxi 从索引号是1开始替换成ccc 变成 jccci
```
9.**strlen(name) 返回name对应值的字节长度（一个汉字3个字节）**

```python
print(r.strlen("foo"))  # 4 'goo1'的长度是4
```

10.**incr(self, name, amount=1) 自增 name 对应的值，当 name 不存在时，则创建 name＝amount，否则，则自增。**

 - name - Redis的name 
 - amount - 自增数（必须是整数）

注：同 incrby


```python
r.set("foo", 123)
print(r.mget("foo", "foo1", "foo2", "k1", "k2"))
r.incr("foo", amount=1)
print(r.mget("foo", "foo1", "foo2", "k1", "k2"))
```

**应用场景 – 页面点击数**
假定对一系列页面需要记录点击次数。例如论坛的每个帖子都要记录点击次数，而点击次数比回帖的次数的多得多。如果使用关系数据库来存储点击，可能存在大量的行级锁争用。所以，点击数的增加使用redis的INCR命令最好不过了。

当redis服务器启动时，可以从关系数据库读入点击数的初始值（12306这个页面被访问了34634次）

```python
r.set("visit:12306:totals", 34634)
print(r.get("visit:12306:totals"))
```

每当有一个页面点击，则使用INCR增加点击数即可。

```python
r.incr("visit:12306:totals")
r.incr("visit:12306:totals")
```

页面载入的时候则可直接获取这个值

```python
print(r.get("visit:12306:totals"))
```

11.**decr(self, name, amount=1) 自减 name 对应的值，当 name 不存在时，则创建 name＝amount，否则，则自减。**

 - name - Redis的name 
 - amount - 自减数（整数)

注：同 decrby
```python
r.decr("foo4", amount=3) # 递减3
r.decr("foo1", amount=1) # 递减1
print(r.mget("foo1", "foo4"))
# 输出结果
['124', '-3']
```

12.**append(key, value) 在redis name对应的值后面追加内容**

 - key - redis的name 
 - value - 要追加的字符串

```python
r.append("name", "haha")    # 在name对应的值junxi后面追加字符串haha
print(r.mget("name"))
```

### 4、redis 基本命令 hash
1、**单个增加--修改(单个取出)--没有就新增，有的话就修改**

```python
hset(name, key, value, mapping=None)
```
name对应的hash中设置一个键值对（不存在，则创建；否则，修改）
 - name - redis的name 
 - key - name对应的hash中的key 
 - value - name对应的hash中的value
 - **mapping - 接受一个由键/值对组成的字典**

注：hsetnx(name, key, value) 当name对应的hash中不存在当前key时则创建（相当于添加）

```python
import redis
import time
pool = redis.ConnectionPool(host='localhost', port=6379, decode_responses=True)
r = redis.Redis(connection_pool=pool)
r.hset("hash1", "k1", "v1")
r.hset("hash1", "k2", "v2")
print(r.hkeys("hash1")) # 取hash中所有的key
print(r.hget("hash1", "k1"))    # 单个取hash的key对应的值
print(r.hmget("hash1", "k1", "k2")) # 多个取hash的key对应的值
print(r.hsetnx("hash1", "k2", "v3")) # 只能新建 如果HSETNX创建了一个字段，返回1，否则返回0
print(r.hget("hash1", "k2"))
# 输出结果
['k1', 'k2']
v1
['v1', 'v2']
0
v2
```

2、**批量增加（取出）**

```python
hmset(name, mapping)
```

在name对应的hash中批量设置键值对
 - name - redis的name
 - mapping - 字典

```python
r.hmset("hash2", {"k2": "v2", "k3": "v3"})
```

```python
hget(name,key)
```

在name对应的hash中获取根据key获取value

```python
hmget(name, keys, *args)
```

在name对应的hash中获取多个key的值
 - name - reids对应的name 
 - keys - 要获取key集合，如：['k1', 'k2', 'k3']
 -  *args - 要获取的key，如：k1,k2,k3


```python
print(r.hget("hash2", "k2"))  # 单个取出"hash2"的key-k2对应的value
print(r.hmget("hash2", "k2", "k3"))  # 批量取出"hash2"的key-k2 k3对应的value --方式1
print(r.hmget("hash2", ["k2", "k3"]))  # 批量取出"hash2"的key-k2 k3对应的value --方式2
```
**3、取出所有的键值对**

```python
hgetall(name)
```

获取name对应hash的所有键值

```python
print(r.hgetall("hash1"))
# 输出结果
{'k1': 'v1', 'k2': 'v2'}
```

**4、得到所有键值对的格式 hash长度**

```python
hlen(name)
```

获取name对应的hash中键值对的个数

```python
print(r.hlen("hash1"))
```

**5、得到所有的keys（类似字典的取所有keys）**

```python
hkeys(name)
```

获取name对应的hash中所有的key的值

```python
print(r.hkeys("hash1"))
```

**6、得到所有的value（类似字典的取所有value）**

```python
hvals(name)
```

获取name对应的hash中所有的value的值

```python
print(r.hvals("hash1"))
```

**7、判断成员是否存在（类似字典的in）**

```python
hexists(name, key)
```

检查 name 对应的 hash 是否存在当前传入的 key

```python
print(r.hexists("hash1", "k4"))  # False 不存在
print(r.hexists("hash1", "k1"))  # True 存在
```

**8、删除键值对**

```python
hdel(name,*keys)
```

将name对应的hash中指定key的键值对删除

```python
print(r.hgetall("hash1"))
r.hset("hash1", "k2", "v222")   # 修改已有的key k2
r.hset("hash1", "k11", "v1")   # 新增键值对 k11
r.hdel("hash1", "k1")    # 删除一个键值对
print(r.hgetall("hash1"))
```

**9、自增自减整数(将key对应的value--整数 自增1或者2，或者别的整数 负数就是自减)**

```python
hincrby(name, key, amount=1)
```

自增name对应的hash中的指定key的值，不存在则创建key=amount

参数：

 - name - redis中的name 
 - key - hash对应的key 
 - amount - 自增数（整数）

```python
r.hset("hash1", "k3", 123)
r.hincrby("hash1", "k3", amount=-1)
print(r.hgetall("hash1"))
r.hincrby("hash1", "k4", amount=1)  # 不存在的话，value默认就是1
print(r.hgetall("hash1"))
```

**10、自增自减浮点数(将key对应的value--浮点数 自增1.0或者2.0，或者别的浮点数 负数就是自减)**

```python
hincrbyfloat(name, key, amount=1.0)
```

自增name对应的hash中的指定key的值，不存在则创建key=amount

参数：
 - name - redis中的name 
 - key - hash对应的key 
 - amount，自增数（浮点数）

自增 name 对应的 hash 中的指定 key 的值，不存在则创建 key=amount。

```python
r.hset("hash1", "k5", "1.0")
r.hincrbyfloat("hash1", "k5", amount=-1.5)    # 已经存在，递减-1.5
print(r.hgetall("hash1"))
r.hincrbyfloat("hash1", "k6", amount=-1.0)    # 不存在，value初始值是-1.0 每次递减1.0
print(r.hgetall("hash1"))
# 输出结果
{'k2': 'v222', 'k11': 'v1', 'k3': '123', 'k4': '1', 'k5': '-0.5', 'k6': '-1'}
{'k2': 'v222', 'k11': 'v1', 'k3': '123', 'k4': '1', 'k5': '-0.5', 'k6': '-2'}
```
**11、取值查看--分片读取**

```python
hscan(name, cursor=0, match=None, count=None)
```

增量式迭代获取，对于数据大的数据非常有用，hscan可以实现分片的获取数据，并非一次性将数据全部获取完，从而防止内存被撑爆

 - name - redis的name 
 - cursor - 游标（基于游标分批取获取数据） 
 - match - 匹配指定key，默认None 表示所有的key 
 - count - 每次分片最少获取个数，默认None表示采用Redis的默认分片个数

```python
第一次：cursor1, data1 = r.hscan('xx', cursor=0, match=None, count=None)
第二次：cursor2, data1 = r.hscan('xx', cursor=cursor1, match=None, count=None)
...
直到返回值cursor的值为0时，表示数据已经通过分片获取完毕

print(r.hscan("hash1"))
# 输出结果
(0, {'k2': 'v222', 'k11': 'v1', 'k3': '123', 'k4': '1', 'k5': '-0.5', 'k6': '-5'})
```

**12、hscan_iter(name, match=None, count=None)**
利用yield封装hscan创建生成器，实现分批去redis中获取数据

 - match - 匹配指定key，默认None 表示所有的key 
 - count - 每次分片最少获取个数，默认None表示采用Redis的默认分片个数

```python
for item in r.hscan_iter('hash1'):
    print(item)
print(r.hscan_iter("hash1"))    # 生成器内存地址
# 输出结果
('k2', 'v222')
('k11', 'v1')
('k3', '123')
('k4', '1')
('k5', '-0.5')
('k6', '-6')
<generator object ScanCommands.hscan_iter at 0x7f94486cc270>
```

### 5、redis基本命令 list
**1.增加（类似于list的append，只是这里是从左边新增加）--没有就新建**

```python
lpush(name,values)
```

在name对应的list中添加元素，每个新的元素都添加到列表的最左边

```python
import redis
import time
pool = redis.ConnectionPool(host='localhost', port=6379, decode_responses=True)
r = redis.Redis(connection_pool=pool)
r.lpush("list1", 11, 22, 33)
print(r.lrange('list1', 0, -1))
保存顺序为: 33,22,11
```

rpush:增加（从右边增加）--没有就新建
```python
r.rpush("list2", 44, 55, 66)    # 在列表的右边，依次添加44,55,66
print(r.llen("list2"))  # 列表长度
print(r.lrange("list2", 0, -1)) # 切片取出值，范围是索引号0到-1(最后一个元素)
```

**2.往已经有的name的列表的左边添加元素，没有的话无法创建**

```python
lpushx(name,value)
```

在name对应的list中添加元素，只有name已经存在时，值添加到列表的最左边

```python
r.lpushx("list10", 10)   # 这里list10不存在
print(r.llen("list10"))  # 0
print(r.lrange("list10", 0, -1))  # []
r.lpushx("list2", 77)   # 这里"list2"之前已经存在，往列表最左边添加一个元素，一次只能添加一个
print(r.llen("list2"))  # 列表长度
print(r.lrange("list2", 0, -1)) # 切片取出值，范围是索引号0到-1(最后一个元素
```

rpushx:往已经有的name的列表的右边添加元素，没有的话无法创建

```python
r.rpushx("list2", 99)   # 这里"foo_list1"之前已经存在，往列表最右边添加一个元素，一次只能添加一个
print(r.llen("list2"))  # 列表长度
print(r.lrange("list2", 0, -1)) # 切片取出值，范围是索引号0到-1(最后一个元素)
```

**3.新增（固定索引号位置插入元素）**

```python
linsert(name, where, refvalue, value))
```

在name对应的列表的某一个值前或后插入一个新值

 - name - redis的name 
 - where - before/after
 - refvalue - 标杆值，即：在它前后插入数据
 -  value - 要插入的数据


```python
r.linsert("list2", "before", "11", "00")   # 往列表中左边第一个出现的元素"11"前插入元素"00"
print(r.lrange("list2", 0, -1))   # 切片取出值，范围是索引号0-最后一个元素
```

**4.修改（指定索引号进行修改）**

```python
r.lset(name, index, value)
```

对name对应的list中的某一个索引位置重新赋值

 - name - redis的name 
 - index - list的索引位置 
 - value - 要设置的值

```python
r.lset("list2", 0, -11)    # 把索引号是0的元素修改成-11
print(r.lrange("list2", 0, -1))
```

**5.删除（指定值进行删除）**

```python
r.lrem(name, count, value)
```

在name对应的list中删除指定的值

 - name - redis的name
 - count
	 - count=0，删除列表中所有的指定值；
	 - count=1，从前到后，删除左边第1个
	 - count=-2，从后向前，删除2个
 -  value - 要删除的值

```python
r.lrem("list2", 1, "11")    # 将列表中左边第一次出现的"11"删除
print(r.lrange("list2", 0, -1))
r.lrem("list2", -1, "99")    # 将列表中右边第一次出现的"99"删除
print(r.lrange("list2", 0, -1))
r.lrem("list2", 0, "22")    # 将列表中所有的"22"删除
print(r.lrange("list2", 0, -1))
```

**6.删除并返回**

```python
lpop(name)
```

在name对应的列表的左侧获取第一个元素并在列表中移除，返回值则是第一个元素

```python
rpop(name) 表示从右向左操作
```

```python
r.lpop("list2")    # 删除列表最左边的元素，并且返回删除的元素
print(r.lrange("list2", 0, -1))
print(r.rpop("list2"))    # 删除列表最右边的元素，并且返回删除的元素
print(r.lrange("list2", 0, -1))
```

**7.删除索引之外的值**

```python
ltrim(name, start, end)
```

在name对应的列表中移除没有在start-end索引之间的值

 - name - redis的name 
 - start - 索引的起始位置 
 - end - 索引结束位置


```python
r.ltrim("list2", 0, 2)    # 删除索引号是0-2之外的元素，值保留索引号是0-2的元素
print(r.lrange("list2", 0, -1))
```

**8.取值（根据索引号取值）**

```python
lindex(name, index)
```

在name对应的列表中根据索引获取列表元素

```python
print(r.lindex("list2", 0))  # 取出索引号是0的值
print(type(r.lindex("list2", 0)))
# 输出结果
00
<class 'str'>
```

**9.移动 元素从一个列表移动到另外一个列表**

```python
rpoplpush(src, dst)
```

从一个列表取出最右边的元素，同时将其添加至另一个列表的最左边

 - src - 要取数据的列表的 name 
 - dst - 要添加数据的列表的 name

```python
r.rpoplpush("list1", "list2")
print(r.lrange("list2", 0, -1))
```

**10.移动 元素从一个列表移动到另外一个列表 可以设置超时**

```python
brpoplpush(src, dst, timeout=0)
```

从一个列表的右侧移除一个元素并将其添加到另一个列表的左侧

 - src - 取出并要移除元素的列表对应的name 
 - dst - 要插入元素的列表对应的name
 - timeout - 当src对应的列表中没有数据时，阻塞等待其有数据的超时时间（秒），0 表示永远阻塞

```python
r.brpoplpush("list1", "list2", timeout=2)
print(r.lrange("list2", 0, -1))
```

**11.一次移除多个列表**

```python
blpop(keys, timeout)
```

将多个列表排列，按照从左到右去pop对应列表的元素

 - keys - redis的name的集合
 - timeout - 超时时间，当元素所有列表的元素获取完之后，阻塞等待列表内有数据的时间（秒）, 0 表示永远阻塞

`r.brpop(keys, timeout)` 同 blpop，将多个列表排列,按照从右像左去移除各个列表内的元素

```python
r.lpush("list10", 3, 4, 5)
r.lpush("list11", 3, 4, 5)
while True:
    r.blpop(["list10", "list11"], timeout=2)
    print(r.lrange("list10", 0, -1), r.lrange("list11", 0, -1))
```

**12.自定义增量迭代**

由于redis类库中没有提供对列表元素的增量迭代，如果想要循环name对应的列表的所有元素，那么就需要获取name对应的所有列表。

循环列表

但是，如果列表非常大，那么就有可能在第一步时就将程序的内容撑爆，所以有必要自定义一个增量迭代的功能：

```python
def list_iter(name):
    """
    自定义redis列表增量迭代
    :param name: redis中的name，即：迭代name对应的列表
    :return: yield 返回 列表元素
    """
    list_count = r.llen(name)
    for index in range(list_count):
        yield r.lindex(name, index)

# 使用
for item in list_iter('list2'): # 遍历这个列表
    print(item)
```

### 6、redis基本命令 set
**1.新增**

```python
sadd(name,values)
```
 - name - 对应的集合中添加元素


```python
r.sadd("set1", 33, 44, 55, 66)  # 往集合中添加元素
print(r.scard("set1"))  # 集合的长度是4
print(r.smembers("set1"))   # 获取集合中所有的成员
# 输出结果
4
{'66', '55', '33', '44'}
```

**2.获取元素个数 类似于len**

```python
scard(name)
```

获取name对应的集合中元素个数


**3.获取集合中所有的成员**

获取name对应的集合的所有成员

```python
smembers(name)
```

获取集合中所有的成员--元组形式

```python
sscan(name, cursor=0, match=None, count=None)
```

```python
print(r.sscan("set1"))
# 输出结果
(0, ['33', '44', '55', '66'])
```

获取集合中所有的成员--迭代器的方式

```python
sscan_iter(name, match=None, count=None)
```

同字符串的操作，用于增量迭代分批获取元素，避免内存消耗太大

```python
for i in r.sscan_iter("set1"):
    print(i)
# 输出结果
33
44
55
66
```

**4.差集**

```python
sdiff(keys, *args)
```

在第一个name对应的集合中且不在其他name对应的集合的元素集合

```python
r.sadd("set2", 11, 22, 33)
print(r.smembers("set1"))   # 获取集合中所有的成员
print(r.smembers("set2"))
print(r.sdiff("set1", "set2"))   # 在集合set1但是不在集合set2中
print(r.sdiff("set2", "set1"))   # 在集合set2但是不在集合set1中
```

差集--差集存在一个新的集合中

```python
sdiffstore(dest, keys, *args)
```

获取第一个name对应的集合中且不在其他name对应的集合，再将其新加入到dest对应的集合中，**返回值为新集合中键的数目**。

```python
print(r.sdiffstore("set3", "set1", "set2"))   # 在集合set1但是不在集合set2中
print(r.smembers("set3"))   # 获取集合3中所有的成员
# 输出结果
3
{'66', '44', '55'} #无序
```

**5.交集**

```python
sinter(keys, *args)
```

获取多一个name对应集合的交集

```python
print(r.sinter("set1", "set2")) # 取2个集合的交集
```

交集--交集存在一个新的集合中，**返回值为新集合中键的数目**。

```python
sinterstore(dest, keys, *args)
```

获取多一个name对应集合的并集，再将其加入到dest对应的集合中

```python
print(r.sinterstore("set3", "set1", "set2")) # 取2个集合的交集
print(r.smembers("set3"))
# 输出结果
6
{'55', '33', '22', '44', '66', '11'}
```

**6.并集**

```python
sunion(keys, *args)
```

获取多个name对应的集合的并集，**返回值为新集合中键的数目**。

```python
print(r.sunion("set1", "set2")) # 取2个集合的并集
# 输出结果
{'44', '66', '55', '33', '22', '11'}
```

并集--并集存在一个新的集合

```python
sunionstore(dest,keys, *args)
```

获取多一个name对应的集合的并集，并将结果保存到dest对应的集合中

```python
print(r.sunionstore("set3", "set1", "set2")) # 取2个集合的并集
print(r.smembers("set3"))
# 输出结果
3
{'55', '66', '44'}
```

**8.判断是否是集合的成员 类似in**

```python
sismember(name, value)
```

检查value是否是name对应的集合的成员，结果为True和False

```python
print(r.sismember("set1", 33))  # 33是集合的成员
print(r.sismember("set1", 23))  # 23不是集合的成员
```

**9.移动**

```python
smove(src, dst, value)
```

将某个成员从一个集合中移动到另外一个集合

```python
r.smove("set1", "set2", 44)
print(r.smembers("set1"))
print(r.smembers("set2"))
# 输出结果
{'55', '33', '66'}
{'11', '33', '44', '22'}
```

**10.删除--随机删除并且返回被删除值**

```python
spop(name)
```

从集合移除一个成员，并将其返回,说明一下，集合是无序的，所有是随机删除的

```python
print(r.spop("set2"))   # 这个删除的值是随机删除的，集合是无序的
print(r.smembers("set2"))
```

**11.删除--指定值删除**

```python
srem(name, values)
```

在name对应的集合中删除某些值

```python
print(r.srem("set2", 11))   # 从集合中删除指定值 11
print(r.smembers("set2"))
```

### 7、redis基本命令 有序set
Set操作，Set集合就是不允许重复的列表，本身是无序的。

有序集合，在集合的基础上，为每个元素排序；元素的排序需要根据另外一个值来进行比较，所以，对于有序集合，每一个元素有两个值，即：值和分数，分数专门用来做排序。

**1.新增**

```python
zadd(name, mapping, nx=False, xx=False, ch=False, incr=False, gt=None, lt=None）
```

在name对应的有序集合中添加元素

```python
import redis
import time
pool = redis.ConnectionPool(host='localhost', port=6379, decode_responses=True)
r = redis.Redis(connection_pool=pool)
r.zadd("zset1", {"n1": 11, "n2": 22})
r.zadd("zset2", {'m1': 22, 'm2': 44})
print(r.zcard("zset1")) # 集合长度
print(r.zcard("zset2")) # 集合长度
print(r.zrange("zset1", 0, -1))   # 获取有序集合中所有元素
print(r.zrange("zset2", 0, -1, withscores=True))   # 获取有序集合中所有元素和分数
# 输出结果
2
2
['n1', 'n2']
[('m1', 22.0), ('m2', 44.0)]
```

**2.获取有序集合元素个数 类似于len**

```python
zcard(name)
```

获取name对应的有序集合元素的数量



**3.获取有序集合的所有元素**

```python
r.zrange( name, start, end, desc=False, withscores=False, score_cast_func=float)
```

按照索引范围获取name对应的有序集合的元素

 - name - redis的name 
 - start - 有序集合索引起始位置（非分数） 
 - end - 有序集合索引结束位置（非分数） 
 - desc - 排序规则，默认按照分数从小到大排序 
 - withscores - 是否获取元素的分数，默认只获取元素的值 
 - score_cast_func - 对分数进行数据转换的函数

3-1 从大到小排序(同zrange，集合是从大到小排序的)

```python
zrevrange(name, start, end, withscores=False, score_cast_func=float)
```

```python
print(r.zrevrange("zset1", 0, -1))    # 只获取元素，不显示分数
print(r.zrevrange("zset1", 0, -1, withscores=True)) # 获取有序集合中所有元素和分数,分数倒序
```

3-2 按照分数范围获取name对应的有序集合的元素

```python
zrangebyscore(name, min, max, start=None, num=None, withscores=False, score_cast_func=float)
```

```python
for i in range(1, 30):
    element = 'n' + str(i)
    r.zadd("zset3", {element: i})
print(r.zrangebyscore("zset3", 15, 25))  # # 在分数是15-25之间，取出符合条件的元素
print(r.zrangebyscore("zset3", 12, 22, withscores=True))  # 在分数是12-22之间，取出符合条件的元素（带分数）
# 输出结果
['n15', 'n16', 'n17', 'n18', 'n19', 'n20', 'n21', 'n22', 'n23', 'n24', 'n25']
[('n12', 12.0), ('n13', 13.0), ('n14', 14.0), ('n15', 15.0), ('n16', 16.0), ('n17', 17.0), ('n18', 18.0), ('n19', 19.0), ('n20', 20.0), ('n21', 21.0), ('n22', 22.0)]
```

3-3 按照分数范围获取有序集合的元素并排序（默认从大到小排序）

```python
zrevrangebyscore(name, max, min, start=None, num=None, withscores=False, score_cast_func=float)
```

```python
print(r.zrevrangebyscore("zset3", 22, 11, withscores=True)) # 在分数是22-11之间，取出符合条件的元素 按照分数倒序
```

3-4 获取所有元素--默认按照分数顺序排序

```python
zscan(name, cursor=0, match=None, count=None, score_cast_func=float)
```

```python
print(r.zscan("zset3"))
# 输出结果
(0, [('n1', 1.0), ('n2', 2.0), ('n3', 3.0), ('n4', 4.0), ('n5', 5.0), ('n6', 6.0), ('n7', 7.0), ('n8', 8.0), ('n9', 9.0), ('n10', 10.0), ('n11', 11.0), ('n12', 12.0), ('n13', 13.0), ('n14', 14.0), ('n15', 15.0), ('n16', 16.0), ('n17', 17.0), ('n18', 18.0), ('n19', 19.0), ('n20', 20.0), ('n21', 21.0), ('n22', 22.0), ('n23', 23.0), ('n24', 24.0), ('n25', 25.0), ('n26', 26.0), ('n27', 27.0), ('n28', 28.0), ('n29', 29.0)])
```

3-5 获取所有元素--迭代器

```python
zscan_iter(name, match=None, count=None,score_cast_func=float)
```

```python
for i in r.zscan_iter("zset3"): # 遍历迭代器
    print(i)
```

**4.zcount(name, min, max)**

获取name对应的有序集合中分数 在 [min,max] 之间的个数

```python
print(r.zrange("zset3", 0, -1, withscores=True))
print(r.zcount("zset3", 11, 22))
```

**5.获取值的索引号**

```python
zrank(name, value)
```

获取某个值在 name对应的有序集合中的索引（从 0 开始）

```python
zrevrank(name, value)，从大到小排序。
```

```python
print(r.zrange("zset3", 0, -1))
print(r.zrank("zset3", "n1"))   # n1的索引号是0 这里按照分数顺序（从小到大）
print(r.zrank("zset3", "n6"))   # n6的索引号是5
print(r.zrevrank("zset3", "n1"))    # n1的索引号是28 这里安照分数倒序（从大到小）
# 输出结果
['n1', 'n2', 'n3', 'n4', 'n5', 'n6', 'n7', 'n8', 'n9', 'n10', 'n11', 'n12', 'n13', 'n14', 'n15', 'n16', 'n17', 'n18', 'n19', 'n20', 'n21', 'n22', 'n23', 'n24', 'n25', 'n26', 'n27', 'n28', 'n29']
0
5
28
```

**6.删除--指定值删除**

```python
zrem(name, values)
```

删除name对应的有序集合中值是values的成员

```python
r.zrem("zset3", "n3")   # 删除有序集合中的元素n3 删除单个
print(r.zrange("zset3", 0, -1))
```

**7.删除--根据排行范围删除，按照索引号来删除**

```python
zremrangebyrank(name, min, max)
```

根据排行范围删除

```python
r.zremrangebyrank("zset3", 0, 1)  # 删除有序集合中的索引号是0, 1的元素
print(r.zrange("zset3", 0, -1))
```

**8.删除--根据分数范围删除**

```python
zremrangebyscore(name, min, max)
```

根据分数范围删除

```python
r.zremrangebyscore("zset3", 11, 22)   # 删除有序集合中的分数是11-22的元素
print(r.zrange("zset3", 0, -1))
```

**9.获取值对应的分数**

```python
zscore(name, value)
```

获取name对应有序集合中 value 对应的分数

```python
print(r.zscore("zset3", "n27"))   # 获取元素n27对应的分数27
```

### 8、其他常用操作
**1.删除**

```python
delete(*names)
```

根据删除redis中的任意数据类型（string、hash、list、set、有序set）

```python
r.delete("gender")  # 删除key为gender的键值对
```

**2.检查名字是否存在**

```python
exists(name)
```

检测redis的name是否存在，存在就是True，False 不存在

```python
print(r.exists("zset1"))
```

**3.模糊匹配**

```python
keys(pattern='')
```

根据模型获取redis的name

```python
KEYS * 匹配数据库中所有 key 。
KEYS h?llo 匹配 hello ， hallo 和 hxllo 等。
KEYS hllo 匹配 hllo 和 heeeeello 等。
KEYS h[ae]llo 匹配 hello 和 hallo ，但不匹配 hillo
```

```python
print(r.keys("foo*"))
```

**4.设置超时时间**

```python
expire(name ,time)
```

为某个redis的某个name设置超时时间

```python
r.lpush("list5", 11, 22)
r.expire("list5", time=3)
print(r.lrange("list5", 0, -1))
time.sleep(3)
print(r.lrange("list5", 0, -1))
```

**5.重命名**

```python
rename(src, dst)
```

对redis的name重命名

```python
r.lpush("list5", 11, 22)
r.rename("list5", "list5-1")
```

**6.随机获取name**

```python
randomkey()
```

随机获取一个redis的name（不删除）

```python
print(r.randomkey())
```

**7.获取类型**

```python
type(name)
```

获取name对应值的类型

```python
print(r.type("set1"))
print(r.type("hash2"))
# 输出结果
set
hash
```

**8.查看所有元素**

```python
r.scan(cursor=0, match=None, count=None)
print(r.hscan("hash2"))
print(r.sscan("set3"))
print(r.zscan("zset2"))
print(r.getrange("foo1", 0, -1))
print(r.lrange("list2", 0, -1))
print(r.smembers("set3"))
print(r.zrange("zset3", 0, -1))
print(r.hgetall("hash1"))
# 输出结果
(0, {'k2': 'v2', 'k3': 'v3'})
(0, ['44', '55', '66'])
(0, [('m1', 22.0), ('m2', 44.0)])
125.5
['22', '11', '00']
{'55', '44', '66'}
['n4', 'n5', 'n6', 'n7', 'n8', 'n9', 'n10', 'n23', 'n24', 'n25', 'n26', 'n27', 'n28', 'n29']
{'k2': 'v222', 'k11': 'v1', 'k3': '123', 'k4': '1', 'k5': '-0.5', 'k6': '-6'}
```

**9.查看所有元素--迭代器**

```python
r.scan_iter(match=None, count=None)

for i in r.hscan_iter("hash1"):
    print(i)

for i in r.sscan_iter("set3"):
    print(i)

for i in r.zscan_iter("zset3"):
    print(i)
```

**other 方法**

```python
print(r.get('name'))    # 查询key为name的值
r.delete("gender")  # 删除key为gender的键值对
print(r.keys()) # 查询所有的Key
print(r.dbsize())   # 当前redis包含多少条数据
# r.save()    # 执行"检查点"操作，将数据写回磁盘。保存时阻塞
# r.flushdb()        # 清空r中的所有数据
```

**管道（pipeline）**
redis默认在执行每次请求都会创建（连接池申请连接）和断开（归还连接池）一次连接操作，如果想要在一次请求中指定多个命令，则可以使用pipline实现一次请求指定多个命令，并且默认情况下一次pipline 是原子性操作。

管道（pipeline）是redis在提供单个请求中缓冲多条服务器命令的基类的子类。它通过减少服务器-客户端之间反复的TCP数据库包，从而大大提高了执行批量命令的功能。

```python
import redis
import time
pool = redis.ConnectionPool(host='localhost', port=6379, decode_responses=True)
r = redis.Redis(connection_pool=pool)
# pipe = r.pipeline(transaction=False)    # 默认的情况下，管道里执行的命令可以保证执行的原子性，执行pipe = r.pipeline(transaction=False)可以禁用这一特性。
# pipe = r.pipeline(transaction=True)
pipe = r.pipeline() # 创建一个管道
pipe.set('name', 'jack')
pipe.set('role', 'sb')
pipe.sadd('faz', 'baz')
pipe.incr('num')    # 如果num不存在则vaule为1，如果存在，则value自增1
pipe.execute()
print(r.get("name"))
print(r.get("role"))
print(r.get("num"))
# 输出结果
jack
sb
3
```

管道的命令可以写在一起，如：

```python
pipe.set('hello', 'redis').sadd('faz', 'baz').incr('num').execute()
print(r.get("name"))
print(r.get("role"))
print(r.get("num"))
```

参考资料：[https://www.runoob.com/w3cnote/python-redis-intro.html](https://www.runoob.com/w3cnote/python-redis-intro.html)
