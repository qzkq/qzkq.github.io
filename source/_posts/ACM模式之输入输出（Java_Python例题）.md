---
title: ACM模式之输入输出（Java_Python例题）
date: 2026-01-01 08:01:00
tags:
  - "算法"
  - "ACM"
  - "输入输出"
categories:
  - "算法"
---

﻿[微信公众号：数学建模与人工智能](https://mp.weixin.qq.com/s?__biz=MzI5MTY1MzU1Mg==&mid=2247487453&idx=1&sn=4659cb0f09714bebafd389a016e152cc&scene=19#wechat_redirect)

[https://github.com/QInzhengk/Math-Model-and-Machine-Learning](https://github.com/QInzhengk/Math-Model-and-Machine-Learning)

@[TOC](目录)
​力扣刷题用的是核心代码模式，而**牛客用的是ACM模式**；由于ACM竞赛题目的输入数据和输出数据一般有多组（不定），并且格式多种多样，所以，如何处理题目的输入输出是对大家的一项最基本的要求。这也是困扰初学者的一大问题。
## 一、Java
### 1. 输入:

```java
import java.util.Scanner;
Scanner sc = new Scanner (System.in);
```

读一个整数：int n = sc.nextInt(); 
读一个字符串：String s = sc.next();（以空格作为分隔符）
读一个浮点数：double t =sc.nextDouble();
读一整行：String s = sc.nextLine(); 
判断是否有下一个输入可以用sc.hasNext()或sc.hasNextInt()或sc.hasNextDouble()或sc.hasNextLine()
### 2. 输出 
System.out.println();//换行打印，输出之后会自动换行
System.out.print();//不换行打印
System.out.printf();//按格式输出
### 3. 字符串处理 String
String 类用来存储字符串，可以用charAt方法来取出其中某一字节，计数从0开始：

```java
String a = "Hello"; // a.charAt(1) = 'e'
```

用substring方法可得到子串，如上例

```java
System.out.println(a.substring(0, 4)) // output "Hell"
```

注意第2个参数位置上的字符不包括进来。这样做使得 s.substring(a, b) 总是有 b-a个字符。
字符串连接可以直接用 + 号，如

```java
String a = "Hello";
String b = "world";
System.out.println(a + ", " + b + "!"); // output "Hello, world!"
```

如想直接将字符串中的某字节改变，可以使用另外的StringBuffer类。
### 4. 高精度
BigInteger和BigDecimal可以说是acmer选择java的首要原因。
函数：add, subtract, divide, mod, compareTo等，其中加减乘除模都要求是BigInteger(BigDecimal)和BigInteger(BigDecimal)之间的运算，所以需要把int(double)类型转换为BigInteger(BigDecimal)，用函数BigInteger.valueOf().
### 5. 进制转换
String st = Integer.toString(num, base); // 把num当做10进制的数转成base进制的st(base <= 35).
int num = Integer.parseInt(st, base); // 把st当做base进制，转成10进制的int(parseInt有两个参数,第一个为要转的字符串,第二个为说明是什么进制). 
BigInter m = new BigInteger(st, base); // st是字符串，base是st的进制.
### 6. 数组排序
函数：`Arrays.sort();`
## 二、Python
### 1.输入：
input为字符串，int强制转换成整形；多组例子使用while

```python
n=int(input()) #3
s=input().split(',')  #1,2,3
try:
    n,m = map(int,input().split()) #2 5
    except:
       break
b = list(map(int,input().split())) #2 3 5 6 9 
```

### 2.输出:

> print(*objects, sep=' ', end='\n', file=sys.stdout)

参数的具体含义如下：

 - objects --表示输出的对象。输出多个对象时，需要用 , （逗号）分隔。 
 - sep -- 用来间隔多个对象。 
 - end --用来设定以什么结尾。默认值是换行符 \n，我们可以换成其他字符。 
 - file -- 要写入的文件对象。

### 3.sort和sorted
python中列表的内置函数sort（）可以对列表中的元素进行排序，而全局性的sorted（）函数则对所有可迭代的序列都是适用的；并且sort（）函数是内置函数，会改变当前对象，而sorted（）函数只会返回一个排序后的当前对象的副本，而不会改变当前对象。
#### 1、内置函数sort（fun，key，reverse=False）

 1. 参数fun是表明此sort函数是基于何种算法进行排序的，一般默认情况下python中用的是归并排序
 2. 参数key用来指定一个函数，此函数在每次元素比较时被调用，此函数代表排序的规则，也就是你按照什么规则对你的序列进行排序
 3. 参数reverse是用来表明是否逆序，默认的False情况下是按照升序的规则进行排序的，当reverse=True时，便会按照降序进行排序

```python
#coding:utf-8
list1 = [(2,'huan',23),(12,'the',14),(23,'liu',90)]
​
#使用默认参数进行排序，即按照元组中第一个元素进行排序
list1.sort()
print list1
#输出结果为[(2, 'huan', 23), (12, 'the', 14), (23, 'liu', 90)]
​
#使用匿名表达式重写key所代表的函数,按照元组的第二个元素进行排序
list1.sort(key=lambda x:(x[1]))
print list1
​
#[(2, 'huan', 23), (23, 'liu', 90), (12, 'the', 14)]
​
#使用匿名函数重写key所代表的函数，先按照元组中下标为2的进行排序，
# 对于下标2处元素相同的，则按下标为0处的元素进行排序
list1.sort(key=lambda x:(x[2],x[0]))
print list1
#[(12, 'the', 14), (2, 'huan', 23), (23, 'liu', 90)]
```

#### 2、全局函数sorted（）

> sorted(iterable, key=None, reverse=False)

对于sorted（）函数中key的重写，和sort（）函数中是一样的，所以刚刚对于sort（）中讲解的方法，都是适用于sorted（）函数。
![在这里插入图片描述](/img/83c31b57176df8e26573c2c5f7c57913.png)


### 4.幂运算：
**
### 5.index
Python index() 方法检测字符串中是否包含子字符串 str ，如果指定 beg（开始） 和 end（结束） 范围，则检查是否包含在指定范围内，该方法与 python find()方法一样，只不过如果str不在 string中会报一个异常。
index()方法语法：

> str.index(str, beg=0,end=len(string))

参数:

 - str -- 指定检索的字符串 
 - beg -- 开始索引，默认为0。 
 - end -- 结束索引，默认为字符串的长度。

返回值：如果包含子字符串返回开始的索引值，否则抛出异常。
### 6.any
any(iterable)：参数iterable -- 元组或列表。 
返回值:如果都为空、0、false，则返回false，如果不都为空、0、false，则返回true。
### 7.isdigit()方法
isdigit() 方法检测字符串是否只由数字组成，只对 0 和 正数有效。
### 8.浮点数输出
#### 1、格式化输出

 - %f ——保留小数点后面六位有效数字 
 - %.3f，保留3位小数位

```python
print('%f' % 1.11)  # 默认保留6位小数
1.110000
print('%.1f' % 1.11)  # 取1位小数
1.1
print("%d,%d"%(1,2))
1,2
```
#### 2、format 格式化函数
格式化字符串函数 str.format()，基本语法是通过 {} 和 : 来代替以前的 % 。format 函数可以接受不限个参数，位置可以不按顺序。

```python
print("{} {}".format("hello", "world"))    # 不设置指定位置，按默认顺序
hello world
 
print("{0} {1}".format("hello", "world"))  # 设置指定位置
hello world
 
print("{1} {0} {1}".format("hello", "world"))  # 设置指定位置
world hello world

print("{:.2f}".format(3.1415926))
3.14
```

#### 3、内置round(number[, ndigits])

 - number - 这是一个数字表达式。 
 - ndigits - 表示从小数点到最后四舍五入的位数。默认值为0。
 - 该方法返回x的小数点舍入为n位数后的值。

### 9.set
![在这里插入图片描述](/img/5c56f1c9def157a2b34220266d1af5d2.png)

 - 交集 & : x&y，返回一个新的集合，包括同时在集合 x 和y中的共同元素。 
 - 并集 | : x|y，返回一个新的集合，包括集合 x 和
   y 中所有元素。
   
 - 差集 - : x-y，返回一个新的集合,包括在集合 x 中但不在集合 y 中的元素。 
 -  补集 ^ : x^y，返回一个新的集合，包括集合 x 和 y 的非共同元素。

### 10.ord() 
ord() 函数是 chr() 函数（对于 8 位的 ASCII 字符串）的配对函数，它以一个字符串（Unicode 字符）作为参数，返回对应的 ASCII 数值，或者 Unicode 数值。
## 三、编程题
### 1、矩阵元素相乘
A[n,m]是一个n行m列的矩阵，a[i,j]表示A的第i行j列的元素，定义x[i,j]为A的第i行和第j列除了a[i,j]之外所有元素(共n+m-2个)的乘积，即x[i,j]=a[i,1]*a[i,2]*...*a[i,j-1]*...*a[i,m]*a[1,j]*a[2,j]...*a[i-1,j]*a[i+1,j]...*a[n,j],现输入非负整形的矩阵A[n,m]，求MAX(x[i,j])，即所有的x[i,j]中的最大值。
**输入描述:**

> 第一行两个整数n和m。之后n行输入矩阵，均为非负整数。

**输出描述:**

> 一行输出答案。

示例1
输入

```python
3 5
5 1 8 5 2
1 3 10 3 3
7 8 5 5 16
```

输出

```python
358400
```

Java

```java
import java.util.Scanner;
public class Main{
    public static void main(String[] args){
        Scanner sc = new Scanner(System.in);
        while(sc.hasNext()){
            int n = sc.nextInt();
            int m = sc.nextInt();
            int[][] a = new int[n][m];
            for(int i=0;i<n;i++){
                for(int j=0;j<m;j++){
                    a[i][j] = sc.nextInt();
                }
            }
            int max=0;
            for(int i=0;i<n;i++){
                for(int j=0;j<m;j++){
                  int res=1;
                    for(int k=0;k<m;k++){
                        if(k!=j){
                            res*=a[i][k];
                        } 
                    }
                    for(int k=0;k<n;k++){
                        if(k!=i){
                            res*=a[k][j];
                        }
                    }
                    if(max<res){
                        max=res;
                    }
                }
            } 
            System.out.println(max);  
        }
    }
}
```

Python

```python
while True:
    try:
        n,m = map(int,input().split())
    except:
       break
    a = []
    for i in range(n):
        b = list(map(int,input().split()))
        a.append(b)   
    ma = 0
    for i in range(n):
        for j in range(m):
            res=1
            for k in range(m):
                if k!=j:
                    res*=a[i][k]
            for k in range(n):
                if k!=i:
                    res*=a[k][j]
            if ma<res:
                ma=res                
    print(ma)
```

### 2、有序数组去重
给定一个字符串，字符串是有序的整数集合，逗号相连，移除相同的数字，使每个数字只出现一次，输出最终的数字个数。
**输入描述:**

> 1,2,2

**输出描述:**

> 2

示例1
输入

```python
1,2,2
```

输出

```python
2
```

示例2
输入

```python
0,0,1,1,1,2,2,3,3,4
```

输出

```python
5
```

**备注:**
有序整数字符串集合请在控制台一行内完成输入，并用英文逗号(,)相隔，如：1,2,2
Java

```java
import java.util.Scanner;
public class Main{
    public static void main(String arg[]){
        Scanner sc = new Scanner(System.in);
        String[] str=sc.nextLine().split(","); 
        int count = 0;
        for (int i = 0; i < str.length-1; i++) {
            if (str[i].equals(str[i+1])){
                count++;
            }
        }
        System.out.println(str.length-count);
    }
}
```

Python

```python
s=input().split(',')
s=set(s)
print(len(s))
```

### 3、最大子序列和
给一个长度为N的序列a1,a2,...,an,求最大连续和。也即，寻找1<=i<=j<=N,使得ai+...+aj尽量大。
**输入描述:**

> 一行, 整数序列, 逗号分隔

**输出描述:**

> 一行, 整数, 表示最大子序列和

示例1
输入

```python
1, 2, -5, 3, 4
```

输出

```python
7
```

Java

```java
import java.util.Scanner;
public class Main{
    public static void main(String arg[]){
        Scanner sc = new Scanner (System.in);
        String[] str=sc.nextLine().replace(" ","").split(",");
        int[] a = new int[str.length];
        int k = 0;
        for (String temp : str) {
            a[k++] = Integer.parseInt(String.valueOf(temp));
        }
        if(a.length==1){
            System.out.print(a[0]);
        }else{
          int temp=0,ma=a[0];
            for(int i:a){
                temp=Math.max(temp+i,i);
                ma=Math.max(ma,temp);
            }
            System.out.print(ma);
        }
    }
}
```

Python

```python
A=list(map(int,input().strip().split(',')))
temp=0
ma=A[0]
for a in A:
    temp=max(a+temp,a)
    ma=max(temp,ma)
print(ma)
```

### 4、回文串
回文串是指字符串无论从左读还是从右读，所读的顺序是一样的；简而言之，回文串是左右对称的。
现给定一个字符串，求出它的最长回文子串。你可以假定只有一个满足条件的最长回文串。
**输入描述:**

> 一行, 字符串

**输出描述:**

> 一行, 字符串

示例1
输入

```python
yabccbau
```

输出

```python
abccba
```

Python

```python
st=input()
a=''
for i in range(len(st)):
    for j in range(i+1,len(st)+1):
        s=st[i:j]
        if(s==s[::-1]):
            if(len(a)<len(s)):
                a=s
print(a)
```

### 5、变形词
对于两个字符串A和B，如果A和B中出现的字符种类相同且每种字符出现的次数相同，则A和B互为变形词，请设计一个高效算法，检查两给定串是否互为变形词。
给定两个字符串A和B，请返回一个bool值，代表他们是否互为变形词。
**输入描述:**

```python
两行，每行各一个字符串s，s长度小于1000
```

**输出描述:**

> bool 值

示例1
输入

```python
bcbc
cbcb
```

输出

```python
1
```

Python

```python
s1=input()
s2=input()
st1=[]
st2=[]
if len(s1)!=len(s2):
    print(0)
else:
    for i in range(len(s1)):
        if s1[i] not in st1:
            st1.append(s1[i])
    for j in range(len(s2)):
        if s2[j] not in st2:
            st2.append(s2[j])
    flag=0
    for k in range(len(st1)):
        if s1.count(st1[k])!=s2.count(st1[k]):
            flag=1
            print(0)
            break
    if(flag==0):
        print(1) 
```

### 6、连续质数表示
一些正数能被表示成一个或者多个连续质数的和。那一个数会有多少种这样的表示方式呢？比如说数字41能有3种表示方式：2+3+5+7+11+13，11+13+17，和41；数字3只有本身这一种表示方式；而20没有这样的表示方式。写一个程序生成给定数字的表示方式数量吧。数字大小范围从2到10，000。 
**输入描述:**

> 一行，包含一个2到10000的正整数

**输出描述:**

> 一行, 非负整数, 给定数字的表示方式数量

示例1
输入

```python
41
```

输出

```python
3
```

Python

```python
def IsPrime(num):
    for i in range(2,num):
        if num%i==0:
            return False
    return True
n=int(input())
a=[]
for i in range(2,n+1):
    if IsPrime(i):
        a.append(i)
count=0
for k in range(len(a)):
    sum=0
    j=k
    while sum<n and j<len(a):
        sum+=a[j]
        j+=1
    if sum==n:
        count+=1
print(count)   
```

## 四、Python编程之算法岗笔试题
### 1、找零钱的所有可能数
**对于1-100之间任意钱，换成等值的10元、5元、2元、1元的小钞票，编程输出所有可能的总数**

**问题分析**
对于零钱n，条件x * 1 + y * 2 + z * 5 + m * 10 = n，找出符合条件的（x,y,z,m）的总数。

```python
#对于1-100之间任意钱，换成等值的10元、5元、2元、1元的小钞票
#编程输出所有可能的总数

#a代表10元的张数
#b代表5元的张数
#c代表2元的张数
#d代表1元的张数

#方法：for循环
count=0
for a in range(0,int(n/10)+1):
    for b in range(0,int(n/5)+1):
    	for c in range(0,int(n/2)+1):
        	for d in range(0,n+1):
            	if a*10+b*5+c*2+d==n:
                	count+=1
return count
```
### 2、重排序列
给定一个长度为N的序列A1到AN，现在要对序列进行M次操作，每次操作对序列的前若干项进行升序或降序排列，求经过这M次操作后得到的序列。
输入描述

> 第一行包含两个整数N和M，1≤N，M≤10^5。 第二行包含N个空格隔开的整数A1到AN，1≤Ai≤10^9。
> 接下来M行，每行包含两个整数t和x，0≤t≤1，1≤x≤N。若t=0，则表示对A1到Ax进行升序排列；若t=1，则表示对A1到Ax进行降序排列。操作执行顺序与输入顺序一致。

输出描述

> 输出N个空格隔开的整数，即经过M次操作后得到的序列。

```python
n,m=map(int,input().split())
l1=list(map(int,input().split()))
l2=[]
for i in range(m):
    nums=list(map(int,input().split()))
    l2.append(nums)
for j in range(len(l2)):
    if l2[j][0]==1:
        l1[:l2[j][1]]=sorted(l1[:l2[j][1]],reverse=True)
    else:
        l1[:l2[j][1]]=sorted(l1[:l2[j][1]],reverse=False)
for k in l1:
    print(k,end=' ')
```

参考：
