---
title: Linux常用基本命令
date: 2026-01-01 08:15:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿[https://github.com/QInzhengk/Math-Model-and-Machine-Learning](https://github.com/QInzhengk/Math-Model-and-Machine-Learning)
@[TOC](Linux常用基本命令)
## 一、常用快捷键
 - ctrl+c: 停止进程
 - ctrl+l: 清屏（之前的内容可以向上查看）；彻底清屏是：reset
 - tab: 提示
 - 上下键: 查找执行过的命令
## 二、文件目录类
### 1.pwd
显示当前工作目录的绝对路径

### 2.ls和ll
**ll 命令列出的信息更加详细，有时间，是否可读写等信息；ll不是命令，是ls -l的别名
ll会列出该文件下的所有文件信息，包括隐藏的文件；ls 只列出文件名或目录名**
```bash
(base) [qi@ip--185 q_mk_pess]$ ll
总用量 38328
-rw-rw-r-- 1 qi qi 12711452 1月  18 02:07 ceshi.gz
drwxrwxr-x 3 qi qi     4096 1月  13 06:43 conf
-rw-rw-r-- 1 qi qi     154 1月  13 06:43 __init__.py
```
ll －t 是降序， ll －t ｜ tac 是升序

```bash
(base) [qi@ip-17-185 q_sk_pess]$ ls
ceshi.gz     prss_a_lk.py     rupess_l.sh
```

**ls 命令**

 -  -a 列出目录下的所有文件，包括以 . 开头的隐含文件。
 -  -b 把文件名中不可输出的字符用反斜杠加字符编号(就象在C语言里一样)的形式列出。
 -   -c 输出文件的 i 节点的修改时间，并以此排序。
 -   -d 将目录象文件一样显示，而不是显示其下的文件。
 -   -i 输出文件的 i 节点的索引信息。
 -   -l 列出文件的详细信息。
 -   -m 横向输出文件名，并以“，”作分格符。
 -   -n 用数字的 UID,GID 代替名称。
 -   -o 显示文件的除组信息外的详细信息。
 -   -q 用?代替不可输出的字符。
 -   -r 对目录反向排序。
 -   -s 在每个文件名后输出该文件的大小。
 -   -t 以时间排序(以最近修改的日期进行排序)
 -    -u 以文件上次被访问的时间排序。
 -    -A 显示除 “.”和“..”外的所有文件。
 -   -B 不输出以 “~”结尾的备份文件。
 -   -L 列出链接文件名而不是链接到的文件。
 -   -N 不限制文件长度。
 -   -Q 把输出的文件名用双引号括起来。
 -   -R 列出所有子目录下的文件。
 -   -S 以文件大小排序。
 -   -X 以文件的扩展名(最后一个 . 后的字符)排序。
 -   -1 一行只输出一个文件。
 -   --color=no 不显示彩色文件名
 -   --help 在标准输出上显示帮助信息。
 -   --version 在标准输出上输出版本信息并退出。
   
   **显示彩色目录列表**
   打开/etc/bashrc, 加入如下一行:
```bash
 alias ls="ls --color"
 alias：别名
```
   下次启动bash时就可以像在Slackware里那样显示彩色的目录列表了, 其中颜色的含义如下:
 - 蓝色-->目录
 - 绿色-->可执行文件
 - 红色-->压缩文件
 - 浅蓝色-->链接文件
 - 灰色-->其他文件 
### 3.cd
语法 `cd [dirName]`
用于切换当前工作目录；其中 dirName 表示法可为绝对路径或相对路径。
若目录名称省略，则变换至使用者的 home 目录 (也就是刚 login 时所在的目录)。另外，~ 也表示为 home 目录 的意思， . 则是表示目前所在的目录， . .则表示目前目录位置的上一层目录，- 回到上一次所在目录。
```bash
跳到目前目录的上上两层 :
cd ../..
```
### 4.mkdir
创建目录，语法：`mkdir [-p] dirName`
参数说明：
 - -p 确保目录名称存在，不存在的就建一个。

实例
在工作目录下，建立一个名为 runoob 的子目录 :
```bash
mkdir runoob
```
在工作目录下的 runoob2 目录中，建立一个名为 test 的子目录。
若 runoob2 目录原本不存在，则建立一个。（注：本例若不加 -p 参数，且原本 runoob2 目录不存在，则产生错误。）
```bash
mkdir -p runoob2/test
```
### 5.rmdir
语法 `rmdir [-p] dirName`
删除空的目录。
参数：
 - -p 是当子目录被删除后使它也成为空目录的话，则顺便一并删除。

实例
将工作目录下，名为 AAA 的子目录删除 :
```bash
rmdir AAA
```
在工作目录下的 BBB 目录中，删除名为 Test 的子目录。若 Test 删除后，BBB 目录成为空目录，则 BBB 亦予删除。
```bash
rmdir -p BBB/Test
```
### 6.touch
创建新文件
### 7.cp
语法 `cp [options] source dest`
主要用于复制文件或目录。
参数说明：

 - -f：覆盖已经存在的目标文件而不给出提示。
 - -i：与 -f 选项相反，在覆盖目标文件之前给出提示，要求用户确认是否覆盖，回答 y 时目标文件将被覆盖。
 - -r：若给出的源文件是一个目录文件，此时将复制该目录下所有的子目录和文件。

实例
使用指令 cp 将当前目录 test/ 下的所有文件复制到新目录 newtest 下，输入如下命令：
```bash
$ cp –r test/ newtest        
```
注意：用户使用该指令复制目录时，必须使用参数 -r 或者 -R 。
### 8.rm
用于删除一个文件或者目录。

### 9.mv
用来为文件或目录改名、或将文件或目录移入其它位置。
语法
```powershell
mv [options] source dest
```
mv 参数设置与运行结果

```powershell
mv source_file(文件) dest_file(文件)	将源文件名 source_file 改为目标文件名 dest_file
mv source_file(文件) dest_directory(目录)	将文件 source_file 移动到目标目录 dest_directory 中
mv source_directory(目录) dest_directory(目录)	目录名 dest_directory 已存在，将 source_directory 移动到目录名 dest_directory 中；目录名 dest_directory 不存在则 source_directory 改名为目录名 dest_directory
```
实例
将 info 目录放入 logs 目录中。注意，如果 logs 目录不存在，则该命令将 info 改名为 logs。
```powershell
mv info/ logs 
```
再如将 /usr/runoob 下的所有文件和目录移到当前目录下，命令行为：
```powershell
$ mv /usr/runoob/*  . 
```
### 10.cat
用于连接文件并打印到标准输出设备上。
语法格式 `cat [-AbeEnstTuv] [--help] [--version] fileName`
参数说明：
 - -n ：由 1 开始对所有输出的行数编号。  
 - -b ：和 -n 相似，只不过对于空白行不编号。
 - -s ：当遇到有连续两行以上的空白行，就代换为一行的空白行。

实例：
把 textfile1 的文档内容加上行号后输入 textfile2 这个文档里：
```powershell
cat -n textfile1 > textfile2
```
把 textfile1 和 textfile2 的文档内容加上行号（空白行不加）之后将内容附加到 textfile3 文档里：
```powershell
cat -b textfile1 textfile2 >> textfile3
```
清空 /etc/test.txt 文档内容：(/dev/null)
```powershell
cat /dev/null > /etc/test.txt
```
### 11.more
以一页一页的形式显示
按空白键（space）就往下一页显示，按 b 键就会往回（back）一页显示 。
语法 `more [-dlfpcsu] [-num] [+/pattern] [+linenum] [fileNames..]`
参数：

 - -num 一次显示的行数
 -  -d 提示使用者，在画面下方显示 [Press space to continue, 'q' to quit.] ，如果使用者按错键，则会显示 [Press 'h' for instructions.] 而不是 '哔' 声  （请按“h”键获取指示）
 -  -f 计算行数时，以实际上的行数，而非自动换行过后的行数（有些单行字数太长的会被扩展为两行或两行以上）
 -  -s 当遇到有连续两行以上的空白行，就代换为一行的空白行
 -  +/pattern 在每个文档显示前搜寻该字串（pattern），然后从该字串之后开始显示
 -  +num 从第 num 行开始显示
 - fileNames 欲显示内容的文档，可为复数个数

常用操作命令

 - Enter 向下1行 
 - Ctrl+F 向下滚动一屏 
 - 空格键 向下滚动一屏 
 - Ctrl+B 返回上一屏
 -  = 输出当前行的行号 
 - :f 输出文件名和当前行的行号 
 - q 退出more

### 12.less
less 支持翻页和搜索，支持向上翻页和向下翻页。
**语法**：less [参数] 文件 
**参数说明：**
 -  -e 当文件显示结束后，自动离开
 -  -N 显示每行的行号
 -  -o <文件名> 将less 输出的内容在指定文件中保存起来
 -  -Q 不使用警告音
 -  -s 显示连续空行为一行
 - /字符串：向下搜索"字符串"的功能;n:向下查找；N:向上查找
 - ? 字符串：向上搜索"字符串"的功能 ;n:向下查找；N:向上查找
 - b 向上翻一页 
 - d 向后翻半页 
 - h 显示帮助界面 
 - Q 退出less 命令 
 - u 向前滚动半页 
 - y 向前滚动一行 
 - 空格键 滚动一页 
 - 回车键 滚动一行 
 - [pagedown]： 向下翻动一页 
 - [pageup]： 向上翻动一页

实例
1、ps查看进程信息并通过less分页显示
```bash
ps -ef |less
```
2、查看命令历史使用记录并通过less分页显示
```bash
[root@localhost test]# history | less
22  scp -r tomcat6.0.32 root@192.168.120.203:/opt/soft
23  cd ..
24  scp -r web root@192.168.120.203:/opt/
25  cd soft
26  ls
……省略……
```
3、浏览多个文件
```bash
less log2013.log log2014.log
```
说明：
输入 ：n后，切换到log2014.log
输入 ：p后，切换到log2013.log

附加备注
1.全屏导航
 - ctrl + F - 向前移动一屏 
 - ctrl + B - 向后移动一屏 
 - ctrl + D - 向前移动半屏 
 - ctrl + U - 向后移动半屏
 
2.单行导航
 - j - 下一行 
 - k - 上一行
 
3.其它导航
 - G - 移动到最后一行 
 - g - 移动到第一行 
 - q / ZZ - 退出 less 命令
 
4.其它有用的命令
 - v - 使用配置的编辑器编辑当前文件 
 - h - 显示 less 的帮助文档 
 - &pattern - 仅显示匹配模式的行，而不是整个文件
 
5.标记导航
当使用 less 查看大文件时，可以在任何一个位置作标记，可以通过命令导航到标有特定标记的文本位置：
 - ma - 使用 a 标记文本的当前位置 
 - 'a - 导航到标记 a 处

### 13.head
用于查看文件的开头部分的内容，有一个常用的参数 -n 用于显示行数，默认为 10，即显示 10 行的内容。
**命令格式：**
```powershell
head [参数] [文件]  
```
**参数：**
 - -q 隐藏文件名
 -  -v 显示文件名
 - -c<数目> 显示的字节数
 - -n<行数> 显示的行数

例
显示文件前 20 个字节:
```powershell
head -c 20 runoob_notes.log
```
### 14.tail 
```powershell
tail -F /achin/lo/lg.sprk.cter.ent_pv.202
```
tail 命令可用于查看文件的内容，有一个常用的参数 -f 常用于查阅正在改变的日志文件。

tail -f filename 会把 filename 文件里的最尾部的内容显示在屏幕上，并且不断刷新，只要 filename 更新就可以看到最新的文件内容。

**命令格式：**
```powershell
tail [参数] [文件]  
```
**参数：**
 - -f 循环读取 
 - -q 不显示处理信息
 - -v 显示详细的处理信息
 - -c<数目> 显示的字节数
 - -n<行数> 显示文件的尾部 n 行内容

例：要显示 notes.log 文件的最后 10 行，请输入以下命令：
`tail notes.log`         # 默认显示最后 10 行
要跟踪名为 notes.log 的文件的增长情况，请输入以下命令：
```powershell
tail -f notes.log
```
此命令显示 notes.log 文件的最后 10 行。当将某些行添加至 notes.log 文件时，tail 命令会继续显示这些行。 显示一直继续，直到您按下（**Ctrl-C**）组合键停止显示。
显示文件 notes.log 的内容，从第 20 行至文件末尾:
```powershell
tail -n +20 notes.log
```
显示文件 notes.log 的最后 10 个字符:
```powershell
tail -c 10 notes.log
```
### 15.> 覆盖 和 >> 追加
echo “内容” >> 文件
1 将history命令执行的结果保存到history.log文件中
```powershell
# history > history.log
```
2 使用 >> 向 hosts.log中追加 当前日期
```powershell
# echo "当前日期是 `date`" >> hosts.log
```
### 16.history
查看已经执行过的命令

## 三、时间日期类
### 1.date
```powershell
date [-u|--utc|--universal] [MMDDhhmm[[CC]YY][.ss]]
```
date 可以用来显示或设定系统的日期与时间。
1.在显示方面，使用者可以设定欲显示的格式，格式设定为一个加号后接数个标记，其中可用的标记列表如下: % 
%H : 小时(00..23)
%M : 分钟(00..59)
%p : 显示本地 AM 或 PM
%r : 直接显示时间 (12 小时制，格式为 hh:mm:ss [AP]M)
%s : 从 1970 年 1 月 1 日 00:00:00 UTC 到目前为止的秒数
%S : 秒(00..61)
%T : 直接显示时间 (24 小时制)
%X : 相当于 %H:%M:%S
%Z : 显示时区 
%a : 星期几 (Sun..Sat)
%A : 星期几 (Sunday..Saturday)
%b : 月份 (Jan..Dec)
%B : 月份 (January..December)
%c : 直接显示日期与时间
%d : 日 (01..31)
%D : 直接显示日期 (mm/dd/yy)
%j : 一年中的第几天 (001..366)
%m : 月份 (01..12)
%U : 一年中的第几周 (00..53) (以 Sunday 为一周的第一天的情形)
%w : 一周中的第几天 (0..6)
%W : 一年中的第几周 (00..53) (以 Monday 为一周的第一天的情形)
%x : 直接显示日期 (mm/dd/yy)
%y : 年份的最后两位数字 (00.99)
%Y : 完整年份 (0000..9999)
2.加减
date +%Y%m%d         //显示现在天年月日
date +%Y%m%d --date="+1 day"  //显示后一天的日期
date +%Y%m%d --date="-1 day"  //显示前一天的日期
date +%Y%m%d --date="-1 month"  //显示上一月的日期
date +%Y%m%d --date="+1 year"  //显示下一年的日期

或者更简单点的  date=`date -d -${t}day '+%Y%m%d'` //为t为前几天

```powershell
YMD=$(date -d"-1 days" +%Y%m%d -u) #前一天
```
转换成时间戳：`$ date +%s -d 20211220`
1639958400
转换成日期：`$ date -d @1639958400`
2021年 12月 20日 星期一 00:00:00 UTC

### 2.cal 
查看日历

## 四、文件权限类
### 1.文件属性
在Linux中我们可以使用ll或者ls -l命令来显示一个文件的属性以及文件所属的用户和组。
```bash
(base) q@MacBook-Pro PycharmProjects % ls -l
total 0
drwxr-xr-x   7 q  staff  224  1 25 17:40 pythonProject
drwxr-xr-x   6 q  staff  192  1 27 15:55 qi
drwxr-xr-x@ 24 q  staff  768  1 25 17:40 人工智能
```
如果没有权限，就会出现减号[ - ]。
在 Linux 中第一个字符代表这个文件是目录、文件或链接文件等等 
 - -代表文件
 - d 代表目录
 - l 链接文档(link file)

Linux/Unix 的文件调用权限分为三级 : 文件所有者（Owner）、用户组（Group）、其它用户（Other Users）。
![在这里插入图片描述](/img/29bcfdce641b58571da3cb9766ed36ff.png)
**rxw 作用文件和目录的不同解释**
(1)作用到文件:

 - [ r ]代表可读(read): 可以读取，查看 
 - [ w ]代表可写(write): 可以修改，但是不代表可以删除该文件，删除一个文件的前提条件是对该文件所在的目录有写权限，才能删除该文件. 
 - [ x ]代表可执行(execute):可以被系统执行

(2)作用到目录:

 - [ r ]代表可读(read): 可以读取，ls 查看目录内容
 - [ w ]代表可写(write): 可以修改，目录内创建+删除+重命名目录 
 - [ x ]代表可执行(execute):可以进入该目录

![在这里插入图片描述](/img/09146400e8dd7075aa7c0f131b00ccc4.png)
其中链接数：
 - 如果查看到是文件:链接数指的是硬链接个数。  
 - 如果查看的是文件夹:链接数指的是子文件夹个数(包括隐藏文件夹，使用 ll -a 查 看)。

### 2.chmod（change mode）
控制用户对文件的权限的命令；
语法 `chmod [-cfvR] [--help] [--version] mode file...`
参数说明
mode : 权限设定字串，格式如下 :
```powershell
[ugoa...][[+-=][rwxX]...][,...]
```
其中：
 - u 表示该文件的拥有者，g 表示与该文件的拥有者属于同一个群体(group)者，o 表示其他以外的人，a 表示这三者皆是。
 - +表示增加权限、- 表示取消权限、= 表示唯一设定权限。
 - r 表示可读取，w 表示可写入，x 表示可执行，X 表示只有当该文件是个子目录或者该文件已经被设定过为可执行。

其他参数说明：
 - -R : 对目前目录下的所有文件与子目录进行相同的权限变更(即以递归的方式逐个变更)

**r=4 w=2 x=1 rwx=4+2+1=7**
实例
将文件 file1.txt 与 file2.txt 设为该文件拥有者，与其所属同一个群体者可写入，但其他以外的人则不可写入 :
```powershell
chmod ug+w,o-w file1.txt file2.txt
```

将目前目录下的所有文件与子目录皆设为任何人可读取 :
```powershell
chmod -R a+r *
```
## 五、搜素查找类
![在这里插入图片描述](/img/aacc60b20dc32f877e06226d28ee95c0.png)
![在这里插入图片描述](/img/947b3c1853b64e6ac2a56e7276c54360.png)

### 1.find 
将从指定目录向下递归地遍历其各个子目录，将满足条件的文件显示在终端。

```powershell
-amin n : 在过去 n 分钟内被读取过
-anewer file : 比文件 file 更晚被读取过的文件
-atime n : 在过去n天内被读取过的文件
-cmin n : 在过去 n 分钟内被修改过
-ctime n : 在过去n天内被修改过的文件
-name name, -iname name : 文件名称符合 name 的文件。iname 会忽略大小写
-size n : 默认单位是b,而它代表的是512字节，所以2表示1k，1M则是2048，如果不想自己转换，可以使用其他单位，如c(bytes)、k(Kilobytes)等，+n大于 -n小于 n等于
-type c : 文件类型是 c 的文件。
 - d: 目录
 - f: 一般文件
```
实例
```powershell
将当前目录及其子目录下所有文件后缀为 .c 的文件列出来:
# find . -name "*.c"

将当前目录及其子目录中的所有文件列出：
# find . -type f

将当前目录及其子目录下所有最近 20 天内更新过的文件列出:
# find . -ctime -20

查找 /var/log 目录中更改时间在 7 日以前的普通文件，并在删除之前询问它们：
# find /var/log -type f -mtime +7 -ok rm {} \;

查找当前目录中文件属主具有读、写权限，并且文件所属组的用户和其他用户具有读权限的文件：
# find . -type f -perm 644 -exec ls -l {} \;

查找系统中所有文件长度为 0 的普通文件，并列出它们的完整路径：
# find / -type f -size 0 -exec ls -l {} \;
```
### 2.grep 过滤查找及“|”管道符
```powershell
grep rn_spk_uster.sh *sh
注：哪些sh文件里有rn_spk_uster.sh
```
输出：
```powershell
n_ent_pv.sh:bash rn_spk_uster.sh "${JOB_PREFIX}" \
n_deo_tais_ily.sh:bash rn_spk_uster.sh "${JOB_PREFIX}" \
```
grep 指令用于查找内容包含指定的范本样式的文件，如果发现某文件的内容符合所指定的范本样式，预设 grep 指令会把含有范本样式的那一列显示出来。若不指定任何文件名称，或是所给予的文件名为 -，则 grep 指令会从标准输入设备读取数据。

语法
```bash
grep [-abcEFGhHilLnqrsvVwxy][-A<显示行数>][-B<显示列数>][-C<显示列数>][-d<进行动作>][-e<范本样式>][-f<范本文件>][--help][范本样式][文件或目录...]
```
参数：

> -A<显示行数> 或 --after-context=<显示行数> : 除了显示符合范本样式的那一列之外，并显示该行之后的内容。
> -B<显示行数> 或 --before-context=<显示行数> : 除了显示符合样式的那一行之外，并显示该行之前的内容。
> -c 或 --count : 计算符合样式的列数。
> -d <动作> 或 --directories=<动作> : 当指定要查找的是目录而非文件时，必须使用这项参数，否则grep指令将回报信息并停止动作。
> -i 或 --ignore-case : 忽略字符大小写的差别。
> -n 或 --line-number : 在显示符合样式的那一行之前，标示出该行的列数编号。
> -v 或 --invert-match : 显示不包含匹配文本的所有行。

**管道符，“|”**，表示将前一个命令的处理结果输出传递给后面的命令处理.
例：
```bash
(base) q@MacBook-Pro ~ % ls
Applications	Downloads	Music		PycharmProjects
Desktop		Library		Pictures	Sunlogin Files
Documents	Movies		Public		opt
(base) q@MacBook-Pro ~ % ls | grep D
Desktop
Documents
Downloads
(base) q@MacBook-Pro ~ % ls | grep -n D
2:Desktop
3:Documents
4:Downloads
```

1、在当前目录中，查找后缀有 file 字样的文件中包含 test 字符串的文件，并打印出该字符串的行。此时，可以使用如下命令：

```powershell
grep test *file 
```
结果如下所示：
```powershell
$ grep test test* #查找前缀有“test”的文件包含“test”字符串的文件  
testfile1:This a Linux testfile! #列出testfile1 文件中包含test字符的行  
testfile_2:This is a linux testfile! #列出testfile_2 文件中包含test字符的行  
testfile_2:Linux test #列出testfile_2 文件中包含test字符的行 
```
2、以递归的方式查找符合条件的文件。例如，查找指定目录/etc/acpi 及其子目录（如果存在子目录的话）下所有文件中包含字符串"update"的文件，并打印出该字符串所在行的内容，使用的命令为：

```powershell
grep -r update /etc/acpi 
```
输出结果如下：

```powershell
$ grep -r update /etc/acpi #以递归的方式查找“etc/acpi”下包含“update”的文件  
/etc/acpi/ac.d/85-anacron.sh:# (Things like the slocate updatedb cause a lot of IO.)  
Rather than  
/etc/acpi/resume.d/85-anacron.sh:# (Things like the slocate updatedb cause a lot of  
IO.) Rather than  
/etc/acpi/events/thinkpad-cmos:action=/usr/sbin/thinkpad-keys--update 
```
3、反向查找。前面各个例子是查找并打印出符合条件的行，通过"-v"参数可以打印出不符合条件行的内容。
查找文件名中包含 test 的文件中不包含test 的行，此时，使用的命令为：

```powershell
grep -v test *test*
```
结果如下所示：

```powershell
$ grep-v test* #查找文件名中包含test 的文件中不包含test 的行  
testfile1:helLinux!  
testfile1:Linis a free Unix-type operating system.  
testfile1:Lin  
testfile_1:HELLO LINUX!  
testfile_1:LINUX IS A FREE UNIX-TYPE OPTERATING SYSTEM.  
testfile_1:THIS IS A LINUX TESTFILE!  
testfile_2:HELLO LINUX!  
testfile_2:Linux is a free unix-type opterating system.  
```
### 3.which
which指令会在环境变量$PATH设置的目录里查找符合条件的文件。
语法：`which [文件...]`
例：使用指令"which"查看指令"bash"的绝对路径，输入如下命令：
```bash
$ which bash
```
上面的指令执行后，输出信息如下所示：
```bash
/bin/bash  #bash可执行程序的绝对路径 
```
## 六、压缩和解压类
### 1.gzip/gunzip压缩
gzip 文件 ：压缩文件，只能将文件压缩为*.gz文件(只能压缩文件不能压缩目录)
gunzip 文件.gz ：解压缩文件命令
不保留原来的文件（无论压缩还是解压）
### 2.zip/unzip压缩
zip [选项] xxx.zip:将要压缩的内容 ：压缩文件和目录的命令

 - -r :压缩目录
 - -d 从压缩文件内删除指定的文件

unzip [选项] xxx.zip ：解压缩文件

 - -d <目录> ：指定解压后文件的存放目录
 - -l ：显示压缩文件内所包含的文件
 - -v ：执行是时显示详细的信息

```powershell
解压xxx.zip到指定目录
unzip xxx.zip -d /home/test/
```
zip压缩命令可以压缩目录且保留源文件

查看压缩文件中包含的文件：

```powershell
# unzip -l abc.zip 
Archive: abc.zip
 Length   Date  Time  Name
--------  ----  ----  ----
  94618 05-21-10 20:44  a11.jpg
  202001 05-21-10 20:44  a22.jpg
    16 05-22-10 15:01  11.txt
  46468 05-23-10 10:30  w456.JPG
  140085 03-14-10 21:49  my.asp
--------          -------
  483188          5 files
```
-v 参数用于查看压缩文件目录信息，但是不解压该文件。

```powershell
# unzip -v abc.zip 
Archive: abc.zip
Length  Method  Size Ratio  Date  Time  CRC-32  Name
-------- ------ ------- -----  ----  ----  ------  ----
  94618 Defl:N  93353  1% 05-21-10 20:44 9e661437 a11.jpg
 202001 Defl:N  201833  0% 05-21-10 20:44 1da462eb a22.jpg
   16 Stored    16  0% 05-22-10 15:01 ae8a9910 ? +-|￥+-? (11).txt
  46468 Defl:N  39997 14% 05-23-10 10:30 962861f2 w456.JPG
 140085 Defl:N  36765 74% 03-14-10 21:49 836fcc3f my.asp
--------     ------- ---              -------
 483188      371964 23%              5 files
```
### 3.tar
tar [选项] xxx.tar.gz 将要打包进去的内容 
打包目录，压缩后的文件格式.tar.gz
选项说明

 - -z：打包同时压缩
 - -c：产生.tar打包文件
 - -v：显示详细信息
 - -f：指定压缩后的文件名
 - -x：解包.tar文件

```powershell
压缩多个文件
tar -zcvf xxx.tar.gz 1.txt 2.txt
解压到指定目录
tar -zxvf xxx.tar.gz -C d0
```
## 七、进程线程类
### 1.ps
process status：用于显示当前进程的状态，类似于 windows 的任务管理器。
语法 `ps [options] [--help]`
常用参数：
 -  -A 列出所有的进程   
 - -au 显示较详细的资讯   
 - -aux 显示所有包含其他使用者的行程

查找指定进程格式：（可以查看子父进程之间的关系）
```bash
ps -ef | grep 进程关键字
```
显示指定用户信息
```bash
# ps -u root //显示root进程用户信息
```

**(1)ps -aux 显示信息说明**

> USER:该进程是由哪个用户产生的 
> PID:进程的 ID 号 
> %CPU:该进程占用 CPU 资源的百分比，占用越高，进程越耗费资源;
> %MEM:该进程占用物理内存的百分比，占用越高，进程越耗费资源; 
> VSZ:该进程占用虚拟内存的大小，单位 KB;
> RSS:该进程占用实际物理内存的大小，单位 KB; 
> TTY:该进程是在哪个终端中运行的。其中 tty1-tty7 代表本地控制台终端，tty1-tty6 是 本地的字符界面终端，tty7 是图形终端。pts/0-255 代表虚拟终端。 
> STAT:进程状态。常见的状态有:R:运行、S:睡眠、T:停止状态、s:包含子进程、+: 位于后台 START:该进程的启动时间
> TIME:该进程占用 CPU 的运算时间，注意不是系统时间 COMMAND:产生此进程的命令名

**(2)ps -ef 显示信息说明** 

> UID:用户 ID 
> PID:进程 ID 
> PPID:父进程 ID 
> C:CPU 用于计算执行优先级的因子。数值越大，表明进程是 CPU
> 密集型运算，执行优先 级会降低;数值越小，表明进程是 I/O 密集型运算，执行优先级会提高 STIME:进程启动的时间
> TTY:完整的终端名称 
> TIME:CPU 时间 
> CMD:启动进程所用的命令和参数

如果想查看进程的 CPU 占用率和内存占用率，可以使用 aux; 如果想查看进程的父进程 ID 可以使用 ef。
### 2.kill
用于删除执行中的程序或工作。

kill 可将指定的信息送至程序。预设的信息为 SIGTERM(15)，可将指定程序终止。若仍无法终止该程序，可使用 SIGKILL(9) 信息尝试强制删除程序。程序或工作的编号可利用 ps 指令或 jobs 指令查看。

语法 `kill [-s <信息名称或编号>][程序]　或　kill [-l <信息编号>]`
参数说明：
 - -l <信息编号> 　若不加<信息编号>选项，则 -l 参数会列出全部的信息名称。
 -  -s <信息名称或编号> 　指定要送出的信息。
 -  [程序] 　[程序]可以是程序的PID或是PGID，也可以是工作编号。

使用 kill -l 命令列出所有可用信号。

最常用的信号是：

 - 1 (HUP)：重新加载进程。
 -  9 (KILL)：杀死一个进程。 
 - 15 (TERM)：正常停止一个进程。

实例
```bash
杀死进程
# kill 12345

强制杀死进程
# kill -KILL 123456

彻底杀死进程
# kill -9 123456

显示信号
# kill -l

杀死指定用户所有进程
#kill -9 $(ps -ef | grep hnlinux) //方法一 过滤出hnlinux用户进程 
#kill -u hnlinux //方法二
```
### 3.netstat 
显示网络统计信息和端口占用情况
1、基本语法
```powershell
netstat -anp | grep 进程号(查看该进程网络信息) 
netstat -nlp | grep 端口号 (查看网络端口号占用情况)
```
 2、选项说明
 -  -n ：拒绝显示别名，能显示数字的全部转化成数字
 -  -l ： 仅列出有在 listen(监听)的服务状态
 -  -p： 表示显示哪个进程在调用

3、案例实操
(1)通过 Tomcat 进程号查看该进程的网络信息
```powershell
netstat -anp | grep Tomcat 进程号
```
(2)查看某端口号是否被占用
```powershell
netstat -nlp | grep 8080
```
## 八、crond系统定时任务
### 1.crontab 
用来定期执行程序的命令。

linux 任务调度的工作主要分为以下两类：
 1. 系统执行的工作：系统周期性所要执行的工作，如备份系统数据、清理缓存
 2. 个人执行的工作：某个用户定期要做的工作，例如每隔10分钟检查邮件服务器是否有新信，这些工作可由每个用户自行设置

**语法**

```powershell
crontab [ -u user ] file
```
或

```powershell
crontab [ -u user ] { -l | -r | -e }
```
**说明：**
crontab 是用来让使用者在固定时间或固定间隔执行程序之用，换句话说，也就是类似使用者的时程表。
-u user 是指设定指定 user 的时程表，这个前提是你必须要有其权限(比如说是 root)才能够指定他人的时程表。如果不使用 -u user 的话，就是表示设定自己的时程表。

**参数说明：**
 - -e : 执行文字编辑器来设定时程表，内定的文字编辑器是 VI，如果你想用别的文字编辑器，则请先设定 VISUAL 环境变数来指定使用那个文字编辑器(比如说 setenv VISUAL joe)
 - -r : 删除目前的时程表 
 - -l : 列出目前的时程表

**时间格式如下：**

```powershell
f1 f2 f3 f4 f5 program
```
其中 f1 是表示分钟，f2 表示小时，f3 表示一个月份中的第几日，f4 表示月份，f5 表示一个星期中的第几天。program 表示要执行的程序。
当 f1 为 * 时表示每分钟都要执行 program，f2 为 * 时表示每小时都要执行程序，其余类推
当 f1 为 a-b 时表示从第 a 分钟到第 b 分钟这段时间内要执行，f2 为 a-b 时表示从第 a 到第 b 小时都要执行，其余类推
当 f1 为 */n 时表示每 n 分钟个时间间隔执行一次，f2 为 */n 表示每 n 小时个时间间隔执行一次，其余类推
当 f1 为 a, b, c,... 时表示第 a, b, c,... 分钟要执行，f2 为 a, b, c,... 时表示第 a, b, c...个小时要执行，其余类推

```powershell
*    *    *    *    *
-    -    -    -    -
|    |    |    |    |
|    |    |    |    +----- 星期中星期几 (0 - 6) (星期天 为0)
|    |    |    +---------- 月份 (1 - 12) 
|    |    +--------------- 一个月中的第几天 (1 - 31)
|    +-------------------- 小时 (0 - 23)
+------------------------- 分钟 (0 - 59)
```
使用者也可以将所有的设定先存放在文件中，用 crontab file 的方式来设定执行时间。
例：每一分钟执行一次 /bin/ls：

```powershell
* * * * * /bin/ls
```
在 12 月内, 每天的早上 6 点到 12 点，每隔 3 个小时 0 分钟执行一次 /usr/bin/backup：

```powershell
0 6-12/3 * 12 * /usr/bin/backup
```
周一到周五每天下午 5:00 寄一封信给 alex@domain.name：
```powershell
0 17 * * 1-5 mail -s "hi" alex@domain.name < /tmp/maildata
```
每月每天的午夜 0 点 20 分, 2 点 20 分, 4 点 20 分....执行 echo "haha"：

```powershell
20 0-23/2 * * * echo "haha"
```
下面再看看几个具体的例子：

```powershell
0 */2 * * * /sbin/service httpd restart  意思是每两个小时重启一次apache 

50 7 * * * /sbin/service sshd start  意思是每天7：50开启ssh服务 

50 22 * * * /sbin/service sshd stop  意思是每天22：50关闭ssh服务 

0 0 1,15 * * fsck /home  每月1号和15号检查/home 磁盘 

1 * * * * /home/bruce/backup  每小时的第一分执行 /home/bruce/backup这个文件 

00 03 * * 1-5 find /home "*.xxx" -mtime +4 -exec rm {} \;  每周一至周五3点钟，在目录/home中，查找文件名为*.xxx的文件，并删除4天前的文件。

30 6 */10 * * ls  意思是每月的1、11、21、31日是的6：30执行一次ls命令
```
注意：当程序在你所指定的时间执行后，系统会发一封邮件给当前的用户，显示该程序执行的内容，若是你不希望收到这样的邮件，请在每一行空一格之后加上 > /dev/null 2>&1 即可，如：
```powershell
20 03 * * * . /etc/profile;/bin/sh /var/www/runoob/test.sh > /dev/null 2>&1 
```

```powershell
crontab -e #下一行为显示内容
00 04 * * * cd /r_alysis && sh rut_v.sh >log.crontab.run_ 2>&1
# utc04是北京时间12点
```
## 九、补充
### 1、nohup 
no hang up（不挂起），用于在系统后台不挂断地运行命令，退出终端不会影响程序的运行。

nohup 命令，在默认情况下（非重定向时），会输出一个名叫 nohup.out 的文件到当前目录下，如果当前目录的 nohup.out 文件不可写，输出重定向到 $HOME/nohup.out 文件中。

**语法格式**

```powershell
 nohup Command [ Arg … ] [　& ]
```

**参数说明：**

 - Command：要执行的命令。
 - Arg：一些参数，可以指定输出文件。
 - &：让命令在后台执行，终端退出后命令仍旧执行。

例：在后台执行 root 目录下的 runoob.sh 脚本：

```powershell
nohup sh runpv.sh 20211228 &
nohup /root/runoob.sh &
```
在终端如果看到以下输出说明运行成功：

```powershell
appending output to nohup.out
```
这时我们打开 root 目录 可以看到生成了 nohup.out 文件。

如果要停止运行，你需要使用以下命令查找到 nohup 运行脚本到 PID，然后使用 kill 命令来删除：

```powershell
ps -aux | grep "runoob.sh" 
```
参数说明：

 - a : 显示所有程序 
 - u : 以用户为主的格式来显示 
 - x : 显示所有程序，不区分终端机

另外也可以使用 `ps -def | grep "runoob.sh"` 命令来查找。

找到 PID 后，就可以使用 kill PID 来删除。

```powershell
kill -9  进程号PID
```

以下命令在后台执行 root 目录下的 runoob.sh 脚本，并重定向输入到 runoob.log 文件：

```powershell
nohup /root/runoob.sh > runoob.log 2>&1 &
```

**2>&1 解释：**

将标准错误 2 重定向到标准输出 &1 ，标准输出 &1 再被重定向输入到 runoob.log 文件中。

 - 0 – stdin (standard input，标准输入) 
 - 1 – stdout (standard output，标准输出) 
 - 2 – stderr (standard error，标准错误输出)

### 2、zcat
```powershell
zcat cli211226/*.gz
```
zcat命令用于不真正解压缩文件，就能显示压缩包中文件的内容的场合。
**zcat [参数]**
 - -S	当后缀不是标准压缩包后缀时使用此选项
 - -c	将文件内容写到标注输出
 - -d	执行解压缩操作
 - -l	显示压缩包中文件的列表
 - -q	禁用警告信息
 - -r	在目录上执行递归操作
 - -t	测试压缩文件的完整性

例：
不解压缩文件的情况下，显示压缩包中文件的内容：

```powershell
[root@linux265 ~]# zcat file.gz
```

查看多个压缩文件：

```powershell
zcat file1.gz file2.gz
```
获取压缩文件的属性（压缩大小，未压缩大小，比率 -- 压缩率）：

```powershell
zcat -l file.gz
```

禁止所有警告：

```powershell
zcat -q file.gz
```


### 3、uniq
用于检查及删除文本文件中重复出现的行列，一般与 sort 命令结合使用。
uniq 可检查文本文件中重复出现的行列。
**语法**
```powershell
uniq [-cdu][-f<栏位>][-s<字符位置>][-w<字符位置>][--help][--version][输入文件][输出文件]
```
**参数：**

 - -c或--count 在每列旁边显示该行重复出现的次数。
 - -d或--repeated 仅显示重复出现的行列。
 - -u或--unique 仅显示出一次的行列。
 - [输入文件] 指定已排序好的文本文件。如果不指定此项，则从标准读取数据；、
 - [输出文件] 指定输出的文件。如果不指定此选项，则将内容显示到标准输出设备（显示终端）。

例：
文件testfile中第 2、3、5、6、7、9行为相同的行，使用 uniq 命令删除重复的行，可使用以下命令：

```powershell
uniq testfile 
```
testfile中的原有内容为：
```powershell
$ cat testfile      #原有内容  
test 30  
test 30  
test 30  
Hello 95  
Hello 95  
Hello 95  
Hello 95  
Linux 85  
Linux 85 
```
使用uniq 命令删除重复的行后，有如下输出结果：

```powershell
$ uniq testfile     #删除重复行后的内容  
test 30  
Hello 95  
Linux 85 
```
检查文件并删除文件中重复出现的行，并在行首显示该行重复出现的次数。使用如下命令：

```powershell
uniq -c testfile 
```
结果输出如下：

```powershell
$ uniq -c testfile      #删除重复行后的内容  
3 test 30             #前面的数字的意义为该行共出现了3次  
4 Hello 95            #前面的数字的意义为该行共出现了4次  
2 Linux 85            #前面的数字的意义为该行共出现了2次 
```
当重复的行并不相邻时，uniq 命令是不起作用的，即若文件内容为以下时，uniq 命令不起作用：

```powershell
$ cat testfile1      # 原有内容 
test 30  
Hello 95  
Linux 85 
test 30  
Hello 95  
Linux 85 
test 30  
Hello 95  
Linux 85 
```
这时我们就可以使用 sort：

```powershell
$ sort  testfile1 | uniq
Hello 95  
Linux 85 
test 30
```
统计各行在文件中出现的次数：

```powershell
$ sort testfile1 | uniq -c
   3 Hello 95  
   3 Linux 85 
   3 test 30
```
在文件中找出重复的行：
```powershell
$ sort testfile1 | uniq -d
Hello 95  
Linux 85 
test 30  
```

### 4、dirname
从文件名剥离非目录的后缀
dirname命令去除文件名中的非目录部分，仅显示与目录有关的内容。dirname命令读取指定路径名保留最后一个/及其后面的字符，删除其他部分，并写结果到标准输出。如果最后一个/后无字符，dirname 命令使用倒数第二个/，并忽略其后的所有字符。

```bash
# 如果最后一个文件是目录的情形
$ dirname /home/deng/share/
/home/deng

# 如果最后一个文件是普通文件情形
$ dirname /home/deng/scott_data.sql 
/home/deng

# 如果名字中没有包含/ 则输出 .
$ dirname dir
.

# 相对路径情形
$ dirname dir/a
dir

# 路径是根目录的情形
$ dirname /
/
$ dirname //
/
```

### 5、seq
用于产生从某个数到另外一个数之间的所有整数。
**语法**：
 - seq [选项]... 尾数 
 - seq [选项]... 首数 尾数 
 - seq [选项]... 首数 增量 尾数

**选项**：
 - -f, --format=格式 使用printf 样式的浮点格式
 - -s, --separator=字符串 使用指定字符串分隔数字（默认使用：\n）
 - -w, --equal-width 在列前添加0 使得宽度相同

实例：
-f选项：指定格式

```bash
# seq -f "%3g" 9 11
  9
 10
 11
```

%后面指定数字的位数 默认是%g，%3g那么数字位数不足部分是**空格**。

```bash
# seq -f "str%03g" 9 11
str009
str010
str011
```

这样的话数字位数不足部分是0，%前面制定字符串。

-w选项：指定输出数字同宽

```bash
# seq -w 98 101
098
099
100
101
```

不能和-f一起用，输出是同宽的。

-s选项：指定分隔符（默认是回车）

```bash
# seq -s" " -f"str%03g" 9 11
str009 str010 str011
```

指定/t做为分隔符号

```bash
# seq -s"`echo -e "/t"`" 9 11
9/t10/t11
```

指定 = 作为分隔符号：

```bash
# seq -s '=' 1 5
1=2=3=4=5
```

### 6、export
用于设置或显示环境变量。

在 shell 中执行程序时，shell 会提供一组环境变量。export 可新增，修改或删除环境变量，供后续执行的程序使用。export 的效力仅限于该次登陆操作。

语法 `export [-fnp][变量名称]=[变量设置值]`
参数说明：
 - -f 　代表[变量名称]中为函数名称。
 - -n 　删除指定的变量。变量实际上并未删除，只是不会输出到后续指令的执行环境中。
 -  -p 　列出所有的shell赋予程序的环境变量。

实例
列出当前所有的环境变量

```bash
# export -p //列出当前的环境变量值
export COLORFGBG='7;0'
export COLORTERM=truecolor
export COMMAND_MODE=unix2003
```
定义环境变量
```bash
# export MYENV //定义环境变量
# export MYENV=7 //定义环境变量并赋值
```
### 7、du
disk usage：用于显示目录或文件的大小。
du 会显示指定的目录或文件所占用的磁盘空间。
语法
```powershell
du [-abcDhHklmsSx][-L <符号连接>][-X <文件>][--block-size][--exclude=<目录或文件>][--max-depth=<目录层数>][--help][--version][目录或文件]
```
参数说明：
```powershell
-a或-all 显示目录中个别文件的大小。
-b或-bytes 显示目录或文件大小时，以byte为单位。
-c或--total 除了显示个别目录或文件的大小外，同时也显示所有目录或文件的总和。
-h或--human-readable 以K，M，G为单位，提高信息的可读性。
-X<文件>或--exclude-from=<文件> 在<文件>指定目录或文件。
```
显示目录或者文件所占空间:
```powershell
# du
4       ./scf/lib
4       ./scf/service/deploy/product
4       ./scf/service/deploy/info
12      ./scf/service/deploy
16      ./scf/service
4       ./scf/doc
4       ./scf/bin
32      ./scf
1288    .
```
只显示当前目录下面的子目录的目录大小和当前目录的总的大小，最下面的1288为当前目录的总大小

显示指定文件所占空间

```powershell
# du log2012.log 
300     log2012.log
```
方便阅读的格式显示test目录所占空间情况：
```powershell
# du -h test
4.0K    test/scf/lib
4.0K    test/scf/service/deploy/product
4.0K    test/scf/service/deploy/info
12K     test/scf/service/deploy
16K     test/scf/service
4.0K    test/scf/doc
4.0K    test/scf/bin
32K     test/scf
1.3M    test
```
### 8、sort 
用于将文本文件内容加以排序。
sort 可针对文本文件的内容，以行为单位来排序。

语法
```bash
sort [-bcdfimMnr][-o<输出文件>][-t<分隔字符>][+<起始栏位>-<结束栏位>][--help][--verison][文件][-k field1[,field2]]
```
参数说明：

```bash
-b 忽略每行前面开始出的空格字符。
-c 检查文件是否已经按照顺序排序。
-d 排序时，处理英文字母、数字及空格字符外，忽略其他的字符。
-f 排序时，将小写字母视为大写字母。
-i 排序时，除了040至176之间的ASCII字符外，忽略其他的字符。
-m 将几个排序好的文件进行合并。
-M 将前面3个字母依照月份的缩写进行排序。
-n 依照数值的大小排序。
-u 意味着是唯一的(unique)，输出的结果是去完重了的。
-o<输出文件> 将排序后的结果存入指定的文件。
-r 以相反的顺序来排序。
-t<分隔字符> 指定排序时所用的栏位分隔字符。
[-k field1[,field2]] 按指定的列进行排序。
```

在使用 sort 命令以默认的式对文件的行进行排序，使用的命令如下：
```bash
sort testfile 
```
sort 命令将以默认的方式将文本文件的第一列以 ASCII 码的次序排列，并将结果输出到标准输出。
### 9、let
let 命令是 BASH 中用于计算的工具，用于执行一个或多个表达式，变量计算中不需要加上 $ 来表示变量。如果表达式中包含了空格或其他特殊字符，则必须引起来。

语法格式
```python
let arg [arg ...]
```
参数说明：

 - arg：要执行的表达式

自加操作：let no++
自减操作：let no - -
简写形式 let no+=10，let no-=20，分别等同于 let no=no+10，let no=no-20。
```python
#!/bin/bash
let a=5+4
let b=9-3 
echo $a $b

9 6
```
### 10、wc
用于计算字数。

利用wc指令我们可以计算文件的Byte数、字数、或是列数，若不指定文件名称、或是所给予的文件名为"-"，则wc指令会从标准输入设备读取数据。

语法

```python
wc [-clw][--help][--version][文件...]
```

参数：

 - -c或--bytes或--chars 只显示Bytes数。
 - -l或--lines 显示行数。
 -  -w或--words 只显示字数。

在默认的情况下，wc将计算指定文件的行数、字数，以及字节数。使用的命令为：

```python
wc testfile 
```

先查看testfile文件的内容，可以看到：

```python
$ cat testfile  
Linux networks are becoming more and more common, but scurity is often an overlooked  
issue. Unfortunately, in today’s environment all networks are potential hacker targets,  
fro0m tp-secret military research networks to small home LANs.  
Linux Network Securty focuses on securing Linux in a networked environment, where the  
security of the entire network needs to be considered rather than just isolated machines.  
It uses a mix of theory and practicl techniques to teach administrators how to install and  
use security applications, as well as how the applcations work and why they are necesary. 
```

使用 wc统计，结果如下：

```python
$ wc testfile           # testfile文件的统计信息  
3 92 598 testfile       # testfile文件的行数为3、单词数92、字节数598 
```
如果想同时统计多个文件的信息，例如同时统计testfile、testfile_1、testfile_2，可使用如下命令：

```python
wc testfile testfile_1 testfile_2   #统计三个文件的信息 
```

输出结果如下：

```python
$ wc testfile testfile_1 testfile_2  #统计三个文件的信息  
3 92 598 testfile                    #第一个文件行数为3、单词数92、字节数598  
9 18 78 testfile_1                   #第二个文件的行数为9、单词数18、字节数78  
3 6 32 testfile_2                    #第三个文件的行数为3、单词数6、字节数32  
15 116 708 总用量                    #三个文件总共的行数为15、单词数116、字节数708 
```
### 11、timeout
timeout是一个命令行实用程序，它运行指定的命令，如果在给定的时间段后仍在运行，则终止该命令。timeout命令是GNU核心实用程序软件包的一部分，该软件包几乎安装在所有Linux发行版中.
语法格式：

```bash
timeout [OPTION] DURATION COMMAND [ARG]...
```

DURATION可以是正整数或浮点数，后跟可选的后缀：

 - s – 秒 (默认) 
 - m – 分钟 
 - h – 小时 
 - d – 天

如果不添加任何单位，默认是秒。如果DURATION为0，则关联的超时是禁用的。
## 十
### 1.ip a
查看所有的ip地址，参数 a,address,addr 都可以
### 2.ping
用于检测主机。

执行 ping 指令会使用 ICMP 传输协议，发出要求回应的信息，若远端主机的网络功能没有问题，就会回应该信息，因而得知该主机运作正常。
### 3.route
route命令用来显示并设置Linux内核中的网络路由表，route命令设置的路由主要是静态路由。要实现两个不同的子网之间的通信，需要一台连接两个网络的路由器，或者同时位于两个网络的网关来实现。

在Linux系统中设置路由通常是为了解决以下问题：该Linux系统在一个局域网中，局域网中有一个网关，能够让机器访问Internet，那么就需要将这台机器的ip地址设置为Linux机器的默认路由。要注意的是，直接在命令行下执行route命令来添加路由，不会永久保存，当网卡重启或者机器重启之后，该路由就失效了；可以在/etc/rc.local中添加route命令来保证该路由设置永久有效。

显示当前路由表(显示ip地址)     `route -n`
![在这里插入图片描述](/img/102cf26ac3e7086f3951b129c208eb1c.png)
Flags 含义
 - U 路由是活动的 
 - H 目标是个主机 
 - G 需要经过网关 
 - R 恢复动态路由产生的表项 
 - D 由路由的后台程序动态地安装 
 - M 由路由的后台程序修改 
 - ! 拒绝路由

