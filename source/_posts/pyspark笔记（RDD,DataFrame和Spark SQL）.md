---
title: pyspark笔记（RDD,DataFrame和Spark SQL）
date: 2026-01-02 08:01:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿[https://github.com/QInzhengk/Math-Model-and-Machine-Learning](https://github.com/QInzhengk/Math-Model-and-Machine-Learning)
@[TOC](PySpark)

# RDD和DataFrame
## 1.SparkSession 介绍
SparkSession 本质上是SparkConf、SparkContext、SQLContext、HiveContext和StreamingContext这些环境的集合，避免使用这些来分别执行配置、Spark环境、SQL环境、Hive环境和Streaming环境。SparkSession现在是读取数据、处理元数据、配置会话和管理集群资源的入口。

## 2.SparkSession创建RDD 
```python
from pyspark.sql.session import SparkSession

if __name__ == "__main__":
    spark = SparkSession.builder.master("local") \
        .appName("My test") \
        .getOrCreate()
    sc = spark.sparkContext

    data = [1, 2, 3, 4, 5, 6, 7, 8, 9]
    rdd = sc.parallelize(data)
```
SparkSession实例化参数：通过静态类Builder来实例化。Builder 是 SparkSession 的构造器。 通过 Builder, 可以添加各种配置。可以通SparkSession.builder 来创建一个 SparkSession 的实例,并通过 stop 函数来停止 SparkSession。

**Builder 的主要方法如下**：

```python
（1）appName函数
appName(String name)
用来设置应用程序名字，会显示在Spark web UI中

（2）master函数
master(String master)
设置Spark master URL 连接，比如"local" 设置本地运行，"local[4]"本地运行4cores，或则"spark://master:7077"运行在spark standalone 集群。

（3）config函数

（4）getOrCreate函数
getOrCreate()
获取已经得到的 SparkSession，或则如果不存在则创建一个新的基于builder选项的SparkSession

（5）enableHiveSupport函数
表示支持Hive，包括 链接持久化Hive metastore, 支持Hive serdes, 和Hive用户自定义函数
```

## 3.直接创建DataFrame
```python
# 直接创建Dataframe
df = spark.createDataFrame([
        (1, 144.5, 5.9, 33, 'M'),
        (2, 167.2, 5.4, 45, 'M'),
        (3, 124.1, 5.2, 23, 'F'),
    ], ['id', 'weight', 'height', 'age', 'gender']) 
```
##  4.从字典创建DataFrame

```python
df = spark.createDataFrame([{'name':'Alice','age':1},
    {'name':'Polo','age':1}]) 
```

## 4.指定schema创建DataFrame

```python
schema = StructType([
    StructField("id", LongType(), True),   
    StructField("name", StringType(), True),
    StructField("age", LongType(), True),
    StructField("eyeColor", StringType(), True)
])
df = spark.createDataFrame(csvRDD, schema)
```
## 5.读文件创建DataFrame

```python
testDF = spark.read.csv(FilePath, header='true', inferSchema='true', sep='\t')
```
## 6.从pandas dataframe创建DataFrame

```python
import pandas as pd
from pyspark.sql import SparkSession

colors = ['white','green','yellow','red','brown','pink']
color_df=pd.DataFrame(colors,columns=['color'])
color_df['length']=color_df['color'].apply(len)

color_df=spark.createDataFrame(color_df)
color_df.show()
```

## 7.RDD与DataFrame的转换
```sql
RDD转变成DataFrame df.toDF(['col1','col2'])
DataFrame转变成RDD df.rdd.map(lambda x: (x.001,x.002))
```

# DataFrames常用
## Row
DataFrame 中的一行。可以访问其中的字段：

 - 类似属性(row.key)
 - 像字典值(row[key])

![在这里插入图片描述](/img/37b517777b5e1cbbf1f9353402fe9564.png)
## 查看列名/行数

```python
# 查看有哪些列 ，同pandas
df.columns
# ['color', 'length']

# 行数
df.count()

# 列数
len(df.columns)
```
## 统计频繁项目
```python
# 查找每列出现次数占总的30%以上频繁项目
df.stat.freqItems(["id", "gender"], 0.3).show()
+------------+----------------+
|id_freqItems|gender_freqItems|
+------------+----------------+
|      [5, 3]|          [M, F]|
+------------+----------------+
```
## select选择和切片筛选
### 选择几列
```python
color_df.select('length','color').show()
```
### 多列选择和切片
```python
color_df.select('length','color')
        .select(color_df['length']>4).show()
```
### between 范围选择

```python
color_df.filter(color_df.length.between(4,5) )
        .select(color_df.color.alias('mid_length')).show()
```
### 联合筛选

```python
# 这里使用一种是 color_df.length, 另一种是color_df[0]
color_df.filter(color_df.length>4)
        .filter(color_df[0]!='white').show()
```
### filter运行类SQL

```python
color_df.filter("color='green'").show()

color_df.filter("color like 'b%'").show()
```
### where方法的SQL

```python
color_df.where("color like '%yellow%'").show()
```
### 直接使用SQL语法

```python
# 首先dataframe注册为临时表，然后执行SQL查询
color_df.createOrReplaceTempView("color_df")
spark.sql("select count(1) from color_df").show()
```
## 新增、修改列
### lit新增一列常量

```python
import pyspark.sql.functions as F
df = df.withColumn('mark', F.lit(1))
```
### 聚合后修改

```python
# 重新命名聚合后结果的列名(需要修改多个列名就跟多个：withColumnRenamed)
# 聚合之后不修改列名则会显示：count(member_name)
df_res.agg({'member_name': 'count', 'income': 'sum', 'num': 'sum'})
      .withColumnRenamed("count(member_name)", "member_num").show()


from pyspark.sql import functions as F
df_res.agg(
    F.count('member_name').alias('mem_num'),
    F.sum('num').alias('order_num'),
    F.sum("income").alias('total_income')
).show()

```
## cast修改列数据类型

```python
from pyspark.sql.types import IntegerType

# 下面两种修改方式等价
df = df.withColumn("height", df["height"].cast(IntegerType()))
df = df.withColumn("weight", df.weight.cast('int'))
print(df.dtypes)
```
## 排序
### 混合排序

```python
color_df.sort(color_df.length.desc(),color_df.color.asc())                               
        .show()
```

### orderBy排序

```python
color_df.orderBy('length','color').show()
```
## 缺失值
### 计算列中的空值数目

```python
# 计算一列空值数目
df.filter(df['col_name'].isNull()).count()

# 计算每列空值数目
for col in df.columns:
    print(col, "\t", "with null values: ", 
          df.filter(df[col].isNull()).count())
```

### 平均值填充缺失值

```python
from pyspark.sql.functions import when
import pyspark.sql.functions as F

# 计算各个数值列的平均值
def mean_of_pyspark_columns(df, numeric_cols):
    col_with_mean = []
    for col in numeric_cols:
        mean_value = df.select(F.avg(df[col]))
        avg_col = mean_value.columns[0]
        res = mean_value.rdd.map(lambda row: row[avg_col]).collect()
        col_with_mean.append([col, res[0]])
    return col_with_mean

# 用平均值填充缺失值
def fill_missing_with_mean(df, numeric_cols):
    col_with_mean = mean_of_pyspark_columns(df, numeric_cols)
    for col, mean in col_with_mean:
        df = df.withColumn(col, when(df[col].isNull() == True, F.lit(mean)).otherwise(df[col]))
    return df

if __name__ == '__main__':
    # df需要自行创建
    numeric_cols = ['age2', 'height2']  # 需要填充空值的列
    df = fill_missing_with_mean(df, numeric_cols)  # 空值填充
    df.show()
```
## 替换值
### replace 全量替换

```python
# 替换pyspark dataframe中的任何值，而无需选择特定列
df = df.replace（'？'，None）
df = df.replace（'ckd \t'，'ckd'）
```
### functions 部分替换

```python
# 只替换特定列中的值,则不能使用replace.而使用pyspark.sql.functions
# 用classck的notckd替换no
import pyspark.sql.functions as F
df = df.withColumn('class',
                   F.when(df['class'] == 'no', F.lit('notckd'))
                    .otherwise(df['class']))
```

## groupBy + agg 聚合
作为聚合函数agg，通常是和分组函数groupby一起使用，表示对分组后的数据进行聚合操作；如果没有分组函数，默认是对整个dataframe进行聚合操作。

## explode分割

```python
# 为给定数组或映射中的每个元素返回一个新行
from pyspark.sql.functions import split, explode

df = sc.parallelize([(1, 2, 3, 'a b c'),
                     (4, 5, 6, 'd e f'),
                     (7, 8, 9, 'g h i')])
        .toDF(['col1', 'col2', 'col3', 'col4'])
df.withColumn('col4', explode(split('col4', ' '))).show()
+----+----+----+----+
|col1|col2|col3|col4|
+----+----+----+----+
|   1|   2|   3|   a|
|   1|   2|   3|   b|
|   1|   2|   3|   c|
|   4|   5|   6|   d|
|   4|   5|   6|   e|
|   4|   5|   6|   f|
|   7|   8|   9|   g|
|   7|   8|   9|   h|
|   7|   8|   9|   i|
+----+----+----+----+

# 示例二
from pyspark.sql import Row
from pyspark.sql.functions import explode

eDF = spark.createDataFrame([Row(
    a=1, 
    intlist=[1, 2, 3], 
    mapfield={"a": "b"})])
eDF.select(explode(eDF.intlist).alias("anInt")).show()
+-----+
|anInt|
+-----+
|    1|
|    2|
|    3|
+-----+
```
## isin

```python
# 如果自变量的求值包含该表达式的值，则该表达式为true
df[df.name.isin("Bob", "Mike")].collect()
# [Row(age=5, name='Bob')]
df[df.age.isin([1, 2, 3])].collect()
# [Row(age=2, name='Alice')]
```
## 读取
### 从hive中读取数据

```python
from pyspark.sql import SparkSession
myspark = SparkSession.builder \
    .appName('compute_customer_age') \
    .config('spark.executor.memory','2g') \
    .enableHiveSupport() \
    .getOrCreate()

sql = """
      SELECT id as customer_id,name, register_date
      FROM [db_name].[hive_table_name]
      limit 100
      """
df = myspark.sql(sql)
df.show(20)

```
### 将数据保存到数据库中

```python
DataFrame.write.mode("overwrite").saveAsTable("test_db.test_table2")
```
### 读写csv/json

```python
from pyspark import SparkContext
from pyspark.sql import SQLContext
sc = SparkContext()
sqlContext = SQLContext(sc)
csv_content = sqlContext.read.format('com.databricks.spark.csv').options(header='true', inferschema='true').load(r'./test.csv')
csv_content.show(10)  #读取
df.select("year", "model").save("newcars.csv", "com.databricks.spark.csv",header="true") #保存
```

```python
df_sparksession_read = spark.read.csv(r"E: \数据\欺诈数据集\PS_7_log.csv",header=True)
df_sparksession_read.show(10)
或：
df_sparksession_read = spark.read.json(r"E: \数据\欺诈json数据集\PS_7_log.json",header=True)
df_sparksession_read.show(10)
```

# pyspark.sql
```python
pyspark.sql.SQLContext DataFrame和SQL方法的主入口
pyspark.sql.DataFrame 将分布式数据集分组到指定列名的数据框中
pyspark.sql.Column DataFrame中的列
pyspark.sql.Row DataFrame数据的行
pyspark.sql.HiveContext 访问Hive数据的主入口
pyspark.sql.functions DataFrame可用的内置函数
pyspark.sql.types 可用的数据类型列表
pyspark.sql.Window 用于处理窗口函数
```
## pyspark.sql.functions常见内置函数
### 1.pyspark.sql.functions.abs(col)

计算绝对值。

### 2.pyspark.sql.functions.acos(col)
计算给定值的反余弦值; 返回的角度在0到π的范围内。

### 3.pyspark.sql.functions.add_months(start, months)
返回start后months个月的日期。

```python
df = sqlContext.createDataFrame([('2015-04-08',)], ['d'])
df.select(add_months(df.d, 1).alias('d')).collect()

[Row(d=datetime.date(2015, 5, 8))]
```

### 4.pyspark.sql.functions.array_contains(col, value)
集合函数：如果数组包含给定值，则返回True。 收集元素和值必须是相同的类型。

```python
>>> df = sqlContext.createDataFrame([(["a", "b", "c"],), ([],)], ['data'])
>>> df.select(array_contains(df.data, "a")).collect()
[Row(array_contains(data,a)=True), Row(array_contains(data,a)=False)]
```

### 5.pyspark.sql.functions.ascii(col)
计算字符串列的第一个字符的数值。

### 6.pyspark.sql.functions.avg(col)
聚合函数：返回组中的值的平均值。

### 7.pyspark.sql.functions.cbrt(col)
计算给定值的立方根。

### 9.pyspark.sql.functions.coalesce(*cols)
返回不为空的第一列。

### 10.pyspark.sql.functions.col(col)
根据给定的列名返回一个列。

**col函数的作用相当于python中的dataframe格式的提取data[‘id’]**
### 11.pyspark.sql.functions.collect_list(col)
聚合函数：返回重复对象的列表。

### 12.pyspark.sql.functions.collect_set(col)
聚合函数：返回一组消除重复元素的对象。

### 13.pyspark.sql.functions.concat(*cols)
将多个输入字符串列连接成一个字符串列。

```python
>>> df = sqlContext.createDataFrame([('abcd','123')], ['s', 'd'])
>>> df.select(concat(df.s, df.d).alias('s')).collect()
[Row(s=u'abcd123')]
```

### 14.pyspark.sql.functions.concat_ws(sep, *cols)
使用给定的分隔符将多个输入字符串列连接到一个字符串列中。

```python
>>> df = sqlContext.createDataFrame([('abcd','123')], ['s', 'd'])
>>> df.select(concat_ws('-', df.s, df.d).alias('s')).collect()
[Row(s=u'abcd-123')]
```

### 15.pyspark.sql.functions.corr(col1, col2)
返回col1和col2的皮尔森相关系数的新列。

### 16.pyspark.sql.functions.cos(col)
计算给定值的余弦。

### 17.pyspark.sql.functions.cosh(col)
计算给定值的双曲余弦。

### 18.pyspark.sql.functions.count(col)
聚合函数：返回组中的项数量。

### 19.pyspark.sql.functions.countDistinct(col, *cols)
返回一列或多列的去重计数的新列。

```python
>>> l=[('Alice',2),('Bob',5)]
>>> df = sqlContext.createDataFrame(l,['name','age'])
>>> df.agg(countDistinct(df.age, df.name).alias('c')).collect()
[Row(c=2)]
>>> df.agg(countDistinct("age", "name").alias('c')).collect()
[Row(c=2)]
```

### 20.pyspark.sql.functions.current_date()
以日期列的形式返回当前日期。

### 21.pyspark.sql.functions.current_timestamp()
将当前时间戳作为时间戳列返回。

### 22.pyspark.sql.functions.date_add(start, days)
返回start后days天的日期

```python
>>> df = sqlContext.createDataFrame([('2015-04-08',)], ['d'])
>>> df.select(date_add(df.d, 1).alias('d')).collect()
[Row(d=datetime.date(2015, 4, 9))]
```

### 23.pyspark.sql.functions.date_format(date, format)
将日期/时间戳/字符串转换为由第二个参数给定日期格式指定格式的字符串值。
一个模式可能是例如dd.MM.yyyy，可能会返回一个字符串，如“18 .03.1993”。 可以使用Java类java.text.SimpleDateFormat的所有模式字母。
注意：尽可能使用像年份这样的专业功能。 这些受益于专门的实施。

```python
>>> df = sqlContext.createDataFrame([('2015-04-08',)], ['a'])
>>> df.select(date_format('a', 'MM/dd/yyy').alias('date')).collect()
[Row(date=u'04/08/2015')]
```

### 24.pyspark.sql.functions.date_sub(start, days)
返回start前days天的日期

```python
>>> df = sqlContext.createDataFrame([('2015-04-08',)], ['d'])
>>> df.select(date_sub(df.d, 1).alias('d')).collect()
[Row(d=datetime.date(2015, 4, 7))]
```

### 25.pyspark.sql.functions.datediff(end, start)
返回从start到end的天数。

```python
>>> df = sqlContext.createDataFrame([('2015-04-08','2015-05-10')], ['d1', 'd2'])
>>> df.select(datediff(df.d2, df.d1).alias('diff')).collect()
[Row(diff=32)]
```

### 26.pyspark.sql.functions.dayofmonth(col)
将给定日期的月份的天解压为整数。

```python
>>> df = sqlContext.createDataFrame([('2015-04-08',)], ['a'])
>>> df.select(dayofmonth('a').alias('day')).collect()
[Row(day=8)]
```

### 27.pyspark.sql.functions.dayofyear(col)
将给定日期的年份中的某一天提取为整数。

```python
>>> df = sqlContext.createDataFrame([('2015-04-08',)], ['a'])
>>> df.select(dayofyear('a').alias('day')).collect()
[Row(day=98)]
```

### 28.pyspark.sql.functions.desc(col)
基于给定列名称的降序返回一个排序表达式。

### 29.pyspark.sql.functions.exp(col)
计算给定值的指数。

### 30.pyspark.sql.functions.expm1(col)
计算给定值的指数减1。

### 31.pyspark.sql.functions.factorial(col)
计算给定值的阶乘。

```python
>>> df = sqlContext.createDataFrame([(5,)], ['n'])
>>> df.select(factorial(df.n).alias('f')).collect()
[Row(f=120)]
```

### 34.pyspark.sql.functions.format_string(format, *cols)
以printf样式格式化参数，并将结果作为字符串列返回。
参数:●  format – 要格式化的格式
         ●  cols - 要格式化的列
```python
>>> from pyspark.sql.functions import *
>>> df = sqlContext.createDataFrame([(5, "hello")], ['a', 'b'])
>>> df.select(format_string('%d %s', df.a, df.b).alias('v')).collect()
[Row(v=u'5 hello')]
```

### 35.pyspark.sql.functions.hex(col) 
计算给定列的十六进制值，可以是StringType，BinaryType，IntegerType或LongType

```python
>>> sqlContext.createDataFrame([('ABC', 3)], ['a', 'b']).select(hex('a'), hex('b')).collect()
[Row(hex(a)=u'414243', hex(b)=u'3')]
```

### 36.pyspark.sql.functions.hour(col)
将给定日期的小时数提取为整数。

```python
>>> df = sqlContext.createDataFrame([('2015-04-08 13:08:15',)], ['a'])
>>> df.select(hour('a').alias('hour')).collect()
[Row(hour=13)]
```

### 38.pyspark.sql.functions.initcap(col)
在句子中将每个单词的第一个字母翻译成大写。

```python
>>> sqlContext.createDataFrame([('ab cd',)], ['a']).select(initcap("a").alias('v')).collect()
[Row(v=u'Ab Cd')]
```

### 39.pyspark.sql.functions.isnan(col)
如果列是NaN，则返回true的表达式。

```python
>>> df = sqlContext.createDataFrame([(1.0, float('nan')), (float('nan'), 2.0)], ("a", "b"))
>>> df.select(isnan("a").alias("r1"), isnan(df.a).alias("r2")).collect()
[Row(r1=False, r2=False), Row(r1=True, r2=True)]
```

### 40.pyspark.sql.functions.kurtosis(col)
聚合函数：返回组中的值的峰度。

### 41.pyspark.sql.functions.last(col)
聚合函数：返回组中的最后一个值。

### 42.pyspark.sql.functions.last_day(date)
返回给定日期所属月份的最后一天。

### 43.pyspark.sql.functions.lit(col)
创建一个文字值的列

```python
from pyspark.sql import Row
from pyspark.sql import functions as sf
rdd = sc.parallelize([Row(name='Alice', level='a', age=5, height=80),Row(name='Bob', level='a', age=5, height=80),Row(name='Cycy', level='b', age=10, height=80),Row(name='Didi', level='b', age=12, height=75),Row(name='EiEi', level='b', age=10, height=70)])
df = rdd.toDF()
print df.show()
"""
+---+------+-----+-----+
|age|height|level| name|
+---+------+-----+-----+
|  5|    80|    a|Alice|
|  5|    80|    a|  Bob|
| 10|    80|    b| Cycy|
| 12|    75|    b| Didi|
| 10|    70|    b| EiEi|
+---+------+-----+-----+
"""
df2 = df.select("name", (df.age+1).alias("new_age"), sf.lit(2))
print df2.show()
"""
+-----+-------+---+
| name|new_age|  2|
+-----+-------+---+
|Alice|      6|  2|
|  Bob|      6|  2|
| Cycy|     11|  2|
| Didi|     13|  2|
| EiEi|     11|  2|
+-----+-------+---+
"""
# 也可以重命名
df2 = df.select("name", (df.age+1).alias("new_age"), sf.lit(2).alias("constant"))
print df2.show()
"""
+-----+-------+--------+
| name|new_age|constant|
+-----+-------+--------+
|Alice|      6|       2|
|  Bob|      6|       2|
| Cycy|     11|       2|
| Didi|     13|       2|
| EiEi|     11|       2|
+-----+-------+--------+
"""
```

### 44.pyspark.sql.functions.log(arg1, arg2=None)
返回第二个参数的第一个基于参数的对数。
如果只有一个参数，那么这个参数就是自然对数。

```python
>>> df.select(log(10.0, df.age).alias('ten')).map(lambda l: str(l.ten)[:7]).collect()
['0.30102', '0.69897']
>>> df.select(log(df.age).alias('e')).map(lambda l: str(l.e)[:7]).collect()
['0.69314', '1.60943']
```

### 45.pyspark.sql.functions.log1p(col)
计算给定值的自然对数加1。

### 46.pyspark.sql.functions.log2(col)
返回参数的基数为2的对数。

```python
>>> sqlContext.createDataFrame([(4,)], ['a']).select(log2('a').alias('log2')).collect()
[Row(log2=2.0)]
```

### 47.pyspark.sql.functions.lower(col)
将字符串列转换为小写。

### 48.pyspark.sql.functions.ltrim(col)
从左端修剪指定字符串值的空格。

### 49.pyspark.sql.functions.minute(col)
提取给定日期的分钟数为整数

```python
>>> df = sqlContext.createDataFrame([('2015-04-08 13:08:15',)], ['a'])
>>> df.select(minute('a').alias('minute')).collect()
[Row(minute=8)]
```

### 51.pyspark.sql.functions.month(col)
将给定日期的月份提取为整数

```python
>>> df = sqlContext.createDataFrame([('2015-04-08',)], ['a'])
>>> df.select(month('a').alias('month')).collect()
[Row(month=4)]
```

### 52.pyspark.sql.functions.months_between(date1, date2)
返回date1和date2之间的月数。

```python
>>> df = sqlContext.createDataFrame([('1997-02-28 10:30:00', '1996-10-30')], ['t', 'd'])
>>> df.select(months_between(df.t, df.d).alias('months')).collect()
[Row(months=3.9495967...)]
```

### 53.pyspark.sql.functions.rand(seed=None)
生成一个随机列，其中包含均匀分布在 [0.0, 1.0) 中的独立且同分布 (i.i.d.) 样本。

### 54.pyspark.sql.functions.randn(seed=None)
从标准正态分布生成具有独立且同分布 (i.i.d.) 样本的列。

### 55.pyspark.sql.functions.reverse(col)
反转字符串列并将其作为新的字符串列返回

### 56.pyspark.sql.functions.rtrim(col)
从右端修剪指定字符串值的空格

### 57.pyspark.sql.functions.skewness(col)
聚合函数：返回组中值的偏度

### 58.pyspark.sql.functions.sort_array(col, asc=True)
集合函数：按升序对给定列的输入数组进行排序。
参数:col – 列或表达式名称

```python
>>> df = sqlContext.createDataFrame([([2, 1, 3],),([1],),([],)], ['data'])
>>> df.select(sort_array(df.data).alias('r')).collect()
[Row(r=[1, 2, 3]), Row(r=[1]), Row(r=[])]
>>> df.select(sort_array(df.data, asc=False).alias('r')).collect()
[Row(r=[3, 2, 1]), Row(r=[1]), Row(r=[])]
```

### 59.pyspark.sql.functions.split(str, pattern)
将模式分割（模式是正则表达式）。
注：pattern是一个字符串表示正则表达式。

```python
>>> df = sqlContext.createDataFrame([('ab12cd',)], ['s',])
>>> df.select(split(df.s, '[0-9]+').alias('s')).collect()
[Row(s=[u'ab', u'cd'])]
```

### 60.pyspark.sql.functions.sqrt(col)
计算指定浮点值的平方根

### 61.pyspark.sql.functions.stddev(col)
聚合函数：返回组中表达式的无偏样本标准差

### 62.pyspark.sql.functions.sumDistinct(col)
聚合函数：返回表达式中不同值的总和

### 63.pyspark.sql.functions.to_date(col)
将StringType或TimestampType的列转换为DateType

```python
>>> df = sqlContext.createDataFrame([('1997-02-28 10:30:00',)], ['t'])
>>> df.select(to_date(df.t).alias('date')).collect()
[Row(date=datetime.date(1997, 2, 28))]
```

### 64.pyspark.sql.functions.trim(col)
修剪指定字符串列的两端空格。

### 65.pyspark.sql.functions.trunc(date, format)
返回截断到格式指定单位的日期

参数: format – ‘year’, ‘YYYY’, ‘yy’ or ‘month’, ‘mon’, ‘mm’

```python
>>> df = sqlContext.createDataFrame([('1997-02-28',)], ['d'])
>>> df.select(trunc(df.d, 'year').alias('year')).collect()
[Row(year=datetime.date(1997, 1, 1))]
>>> df.select(trunc(df.d, 'mon').alias('month')).collect()
[Row(month=datetime.date(1997, 2, 1))]
```

### 66.pyspark.sql.functions.var_samp(col)
聚合函数：返回组中值的无偏差

### 67.pyspark.sql.functions.variance(col)
聚合函数：返回组中值的总体方差

### 68.pyspark.sql.functions.array(*cols)
创建一个新的数组列。
参数: cols – 列名（字符串）列表或具有相同数据类型的列表达式列表。

```python
>>> df.select(array('age', 'age').alias("arr")).collect()
[Row(arr=[2, 2]), Row(arr=[5, 5])]
>>> df.select(array([df.age, df.age]).alias("arr")).collect()
[Row(arr=[2, 2]), Row(arr=[5, 5])]
```
### 69.pyspark.sql.functions.bin(col)
返回给定列的二进制值的字符串表示形式

```python
>>> l=[('Alice',2),('Bob',5)]
>>> df = sqlContext.createDataFrame(l,['name','age'])
>>> df.select(bin(df.age).alias('c')).collect()
[Row(c=u'10'), Row(c=u'101')]
```
### 70.pyspark.sql.functions.conv(col, fromBase, toBase)
将字符串列中的数字从一个基数转换为另一个基数。

```python
>>> df = sqlContext.createDataFrame([("010101",)], ['n'])
>>> df.select(conv(df.n, 2, 16).alias('hex')).collect()
[Row(hex=u'15')]
```
### 71.pyspark.sql.functions.expr(str)
将表达式字符串分析到它表示的列中

```python
>>> l=[('Alice',2),('Bob',5)]
>>> df = sqlContext.createDataFrame(l,['name','age'])
>>> df.select(expr("length(name)")).collect()
[Row(length(name)=5), Row(length(name)=3)]
```
### 72.pyspark.sql.functions.from_utc_timestamp(timestamp, tz)
假设时间戳是UTC，并转换为给定的时区

```python
>>> df = sqlContext.createDataFrame([('1997-02-28 10:30:00',)], ['t'])
>>> df.select(from_utc_timestamp(df.t, "PST").alias('t')).collect()
[Row(t=datetime.datetime(1997, 2, 28, 2, 30))]
```
### 73.pyspark.sql.functions.greatest(*cols)
返回列名称列表的最大值，跳过空值。 该功能至少需要2个参数。 如果所有参数都为空，它将返回null

```python
>>> df = sqlContext.createDataFrame([(1, 4, 3)], ['a', 'b', 'c'])
>>> df.select(greatest(df.a, df.b, df.c).alias("greatest")).collect()
[Row(greatest=4)]
```
### 74.pyspark.sql.functions.instr(str, substr)
找到给定字符串中第一次出现substr列的位置。 如果其中任一参数为null，则返回null。
注：位置不是从零开始的，但是基于1的索引，如果在str中找不到substr，则返回0。

```python
>>> df = sqlContext.createDataFrame([('abcd',)], ['s',])
>>> df.select(instr(df.s, 'b').alias('s')).collect()
[Row(s=2)]
```
### 75.pyspark.sql.functions.isnull(col)
如果列为null，则返回true的表达式

```python
>>> df = sqlContext.createDataFrame([(1, None), (None, 2)], ("a", "b"))
>>> df.select(isnull("a").alias("r1"), isnull(df.a).alias("r2")).collect()
[Row(r1=False, r2=False), Row(r1=True, r2=True)]
```
### 76.pyspark.sql.functions.least(*cols)
返回列名称列表的最小值，跳过空值。 该功能至少需要2个参数。 如果所有参数都为空，它将返回null

```python
>>> df = sqlContext.createDataFrame([(1, 4, 3)], ['a', 'b', 'c'])
>>> df.select(least(df.a, df.b, df.c).alias("least")).collect()
[Row(least=1)]
```
### 77.pyspark.sql.functions.length(col)
计算字符串或二进制表达式的长度

```python
>>> sqlContext.createDataFrame([('ABC',)], ['a']).select(length('a').alias('length')).collect()
[Row(length=3)]
```
### 78.pyspark.sql.functions.locate(substr, str, pos=0)
找到第一个出现的位置在位置pos后面的字符串列中。
注：位置不是从零开始，而是从1开始。 如果在str中找不到substr，则返回0。
参数: substr – 一个字符串
         str – 一个StringType的列
         pos – 起始位置（基于零）

```python
>>> df = sqlContext.createDataFrame([('abcd',)], ['s',])
>>> df.select(locate('b', df.s, 1).alias('s')).collect()
[Row(s=2)]
```

### 79.pyspark.sql.functions.max(col)
聚合函数：返回组中表达式的最大值。

### 80.pyspark.sql.functions.mean(col)
聚合函数：返回组中的值的平均值

### 81.pyspark.sql.functions.min(col)
聚合函数：返回组中表达式的最小值。

### 82.pyspark.sql.functions.next_day(date, dayOfWeek)
返回晚于日期列值的第一个日期
星期几参数不区分大小写，并接受：“Mon”, “Tue”, “Wed”, “Thu”, “Fri”, “Sat”, “Sun”.

```python
>>> df = sqlContext.createDataFrame([('2015-07-27',)], ['d'])
>>> df.select(next_day(df.d, 'Sun').alias('date')).collect()
[Row(date=datetime.date(2015, 8, 2))]
```
### 83.pyspark.sql.functions.repeat(col, n)
重复一个字符串列n次，并将其作为新的字符串列返回

```python
>>> df = sqlContext.createDataFrame([('ab',)], ['s',])
>>> df.select(repeat(df.s, 3).alias('s')).collect()
[Row(s=u'ababab')]
```
### 84.pyspark.sql.functions.round(col, scale=0)
如果scale> = 0，将e的值舍入为小数点的位数，或者在scale <0的时候将其舍入到整数部分。

```python
>>> sqlContext.createDataFrame([(2.546,)], ['a']).select(round('a', 1).alias('r')).collect()
[Row(r=2.5)]
```
### 85.pyspark.sql.functions.row_number()
窗口函数：返回窗口分区内从1开始的连续编号。

```python
from pyspark.sql.window import Window
df_r = df.withColumn('row_number', sf.row_number().over(Window.partitionBy("level").orderBy("age")).alias("rowNum"))
# 其他写法
df_r = df.withColumn('row_number', sf.row_number().over(Window.partitionBy(df.level).orderBy(df.age)).alias("rowNum"))
print df_r.show()
"""
+---+------+-----+-----+----------+                                             
|age|height|level| name|row_number|
+---+------+-----+-----+----------+
| 10|    80|    b| Cycy|         1|
| 10|    70|    b| EiEi|         2|
| 12|    75|    b| Didi|         3|
|  5|    80|    a|  Bob|         1|
|  5|    80|    a|Alice|         2|
"""
```
表示逆序，或者根据多个字段分组

```python
df_r = df.withColumn('row_number', sf.row_number().over(Window.partitionBy(df.level, df.age).orderBy(sf.desc("name"))).alias("rowNum"))
# 另一种写法
df_r = df.withColumn('row_number', sf.row_number().over(Window.partitionBy("level", "age").orderBy(sf.desc("name"))).alias("rowNum"))
print df_r.show()
"""
+---+------+-----+-----+----------+
|age|height|level| name|row_number|
+---+------+-----+-----+----------+
|  5|    80|    a|  Bob|         1|
|  5|    80|    a|Alice|         2|
| 10|    70|    b| EiEi|         1|
| 10|    80|    b| Cycy|         2|
| 12|    75|    b| Didi|         1|
+---+------+-----+-----+----------+
"""
```

### 86.pyspark.sql.functions.second(col)
将给定日期的秒数提取为整数

```python
>>> df = sqlContext.createDataFrame([('2015-04-08 13:08:15',)], ['a'])
>>> df.select(second('a').alias('second')).collect()
[Row(second=15)]
```

### 87.pyspark.sql.functions.size(col)
集合函数：返回存储在列中的数组或映射的长度
参数:col – 列或表达式名称

```python
>>> df = sqlContext.createDataFrame([([1, 2, 3],),([1],),([],)], ['data'])
>>> df.select(size(df.data)).collect()
[Row(size(data)=3), Row(size(data)=1), Row(size(data)=0)]
```

### 88.pyspark.sql.functions.substring(str, pos, len)
子字符串从pos开始，长度为len，当str是字符串类型时，或者返回从字节pos开始的字节数组的片段，当str是二进制类型时，长度
为len

```python
>>> df = sqlContext.createDataFrame([('abcd',)], ['s',])
>>> df.select(substring(df.s, 1, 2).alias('s')).collect()
[Row(s=u'ab')]
```
### 89.pyspark.sql.functions.sum(col)
聚合函数：返回表达式中所有值的总和。

### 90.pyspark.sql.functions.to_utc_timestamp(timestamp, tz)
假定给定的时间戳在给定的时区并转换为UTC

```python
>>> df = sqlContext.createDataFrame([('1997-02-28 10:30:00',)], ['t'])
>>> df.select(to_utc_timestamp(df.t, "PST").alias('t')).collect()
[Row(t=datetime.datetime(1997, 2, 28, 18, 30))]
```
### 91.pyspark.sql.functions.year(col)
将给定日期的年份提取为整数

```python
>>> df = sqlContext.createDataFrame([('2015-04-08',)], ['a'])
>>> df.select(year('a').alias('year')).collect()
[Row(year=2015)]
```
### 92.pyspark.sql.functions.when(condition, value)
评估条件列表并返回多个可能的结果表达式之一。 如果不调用Column.otherwise（），则不匹配条件返回None

参数:condition – 一个布尔的列表达式.
        value – 一个文字值或一个Column表达式

```python
>>> df.select(when(df['age'] == 2, 3).otherwise(4).alias("age")).collect()
[Row(age=3), Row(age=4)]

>>> df.select(when(df.age == 2, df.age + 1).alias("age")).collect()
[Row(age=3), Row(age=None)]
```

```python
df3 = df.withColumn("when", sf.when(df.age<7, "kindergarten").when((df.age>=7)&(df.age<11), 'low_grade').otherwise("high_grade"))
print df3.show()
"""
+---+------+-----+-----+------------+
|age|height|level| name|        when|
+---+------+-----+-----+------------+
|  5|    80|    a|Alice|kindergarten|
|  5|    80|    a|  Bob|kindergarten|
| 10|    80|    b| Cycy|   low_grade|
| 12|    75|    b| Didi|  high_grade|
| 10|    70|    b| EiEi|   low_grade|
+---+------+-----+-----+------------+
"""
```

### 93.pyspark.sql.functions.udf(f, returnType=StringType)
创建一个表示用户定义函数（UDF）的列表达式。

```python
>>> from pyspark.sql.types import IntegerType
>>> slen = udf(lambda s: len(s), IntegerType())
>>> df.select(slen(df.name).alias('slen')).collect()
[Row(slen=5), Row(slen=3)]
```
udf只能对每一行进行操作，无法对groupBy后的数据处理。

```python
from pyspark.sql import types as st
def ratio(a, b):
    if a is None or b is None or b == 0:
        r = -1.0
    else:
        r = 1.0 * a / b
    return r
col_ratio = udf(ratio, st.DoubleType())
df_udf = df.withColumn("ratio", col_ratio(df.age, df.height))
print df_udf.show()
"""
+---+------+-----+-----+-------------------+
|age|height|level| name|              ratio|
+---+------+-----+-----+-------------------+
|  5|    80|    a|Alice|             0.0625|
|  5|    80|    a|  Bob|             0.0625|
| 10|    80|    b| Cycy|              0.125|
| 12|    75|    b| Didi|               0.16|
| 10|    70|    b| EiEi|0.14285714285714285|
+---+------+-----+-----+-------------------+
"""
```

## 参考链接
[pyspark官方api](https://spark.apache.org/docs/latest/api/python/reference/index.html)

[RDD](https://spark.apache.org/docs/latest/api/python/_modules/pyspark/rdd.html)

[DataFrame](https://spark.apache.org/docs/3.0.1/api/python/_modules/pyspark/sql/dataframe.html)

[https://blog.csdn.net/htbeker/article/details/86233819](https://blog.csdn.net/htbeker/article/details/86233819)

[https://www.cnblogs.com/wonglu/p/8390710.html](https://www.cnblogs.com/wonglu/p/8390710.html)

[https://www.jianshu.com/p/42d90f93c262](https://www.jianshu.com/p/42d90f93c262)

[https://blog.csdn.net/wapecheng/article/details/107472312](https://blog.csdn.net/wapecheng/article/details/107472312)

[https://blog.csdn.net/qq_31400717/article/details/105820203](https://blog.csdn.net/qq_31400717/article/details/105820203)



