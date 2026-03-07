---
title: aws s3命令与hdfs dfs命令_hadoop fs命令（笔记）
date: 2026-01-01 08:04:00
tags:
  - "AWS"
  - "S3"
  - "HDFS"
  - "Hadoop"
  - "命令"
categories:
  - "数据科学"
---

﻿[https://github.com/QInzhengk/Math-Model-and-Machine-Learning](https://github.com/QInzhengk/Math-Model-and-Machine-Learning)
@[TOC](aws s3命令与hdfs dfs/hadoop fs命令)
## 一、aws s3命令 
### 1、查看目录下所有文件夹(head查看前10个)：

```bash
aws s3 ls s3://me-l/qinz/sis1219.csv
aws s3 ls s3://dict-fse/2011/12/19/06/|head
```
查看文件夹大小：

```bash
aws s3 ls  s3://ane-l/202221/ --s --h
# 输出结果
Total Objects: 1001
   Total Size: 226.0 GiB
```

### 2、复制文件到s3（后面是复制后的名称）：

```bash
aws s3 cp *** s3://bucket-name/
aws s3 cp s3://maie-l/qnz/us_pv_aa119.csv evet129.csv （路径）
aws s3 cp s3://dicmin_cl/9-06-00-01-8cf2935e8b.gz test.gz
```

### 3、复制文件夹：

```bash
aws s3 cp s3://bucket-name/example s3://my-bucket/
```

### 4、使用 echo 将文本“hello world”流式传输到 s3://bucket-name/filename.txt 文件：

```bash
echo "hello world" | aws s3 cp - s3://bucket-name/filename.txt
```

### 5、将 s3://bucket-name/filename.txt 文件流式传输到 stdout，并将内容输出到控制台：

```bash
aws s3 cp s3://bucket-name/filename.txt -
```

### 6、将 s3://bucket-name/pre 的内容流式传输到 stdout，使用 bzip2 命令压缩文件，并将名为 key.bz2 的新压缩文件上传到 s3://bucket-nam：

```bash
aws s3 cp s3://bucket-name/pre - | bzip2 --best | aws s3 cp - s3://bucket-name/key.bz2
```

### 7、同步文件到s3：
（sync 命令同步一个存储桶与一个目录中的内容，或者同步两个存储桶中的内容。通常，s3 sync 在源和目标之间复制缺失或过时的文件或对象）

```bash
aws s3 sync *** s3://my-bucket/***/
aws s3 sync s3://mng/log_eer/126 cli_g_226 --quiet
```
--quiet代表不显示指定命令执行的操作（不输出过程）
### 8、删除S3上文件：

```bash
aws s3 rm s3://my-bucket/***
```

### 9、删除S3上文件夹：

```bash
aws s3 rm s3://my-bucket/*** —recursive
```

### 10、移动S3上文件夹：（移动example中所有对象到my-bucket/）

```bash
aws s3 mv s3://bucket-name/example s3://my-bucket/
```

### 11、移动文件：

```bash
aws s3 mv filename.txt s3://bucket-name
```

### 12、转移s3某一个目录下所有.jpg文件到本地目录./aa：

```bash
aws s3 mv s3://bucket-name/*** ./aa —exclude ‘*’ —include ‘*.jpg’ —recursive
```

### 13、从s3上拉取文件夹到本地文件夹./aa：

```bash
s3cmd get s3://bucket-name/***/ ./aa —recursive
```

### 14、创建存储桶：

```bash
aws s3 mb s3://bucket-name
```

### 15、查看存储桶：

```bash
aws s3 ls s3://bucket-name
```

### 16、删除存储桶：

```bash
aws s3 rb s3://bucket-name
```

## 二、hdfs dfs命令/hadoop fs命令

```bash
hadoop fs 具体命令  或者  hdfs dfs 具体命令：两个是完全相同的。
```
### 0、命令大全

```bash
hadoop fs 或 hdfs dfs
```
通过-help 得到命令用法

```bash
hadoop fs -help mkdir
```

### 1、-mkdir 创建目录 

```bash
Usage：hdfs dfs -mkdir [-p] < paths> 
```

选项：-p 很像Unix mkdir -p，沿路径创建父目录。
### 2、-ls 查看目录下内容，包括文件名，权限，所有者，大小和修改时间 Usage：hdfs dfs -ls [-R] < args> 
选项：-R 递归地显示子目录下的内容
### 3、-put 将本地文件或目录上传到HDFS中的路径 

```bash
Usage：hdfs dfs -put < localsrc> … < dst>
```

### 4、-get 将文件或目录从HDFS中的路径拷贝到本地文件路径 

```bash
Usage：hdfs dfs -get [-ignoreCrc] [-crc] < src> < localdst> 
```

选项： -ignoreCrc选项复制CRC校验失败的文件。 -crc选项复制文件和CRC。
### 5、-du 显示给定目录中包含的文件和目录的大小或文件的长度，用字节大小表示，文件名用完整的HDFS协议前缀表示，以防它只是一个文件。

```bash
Usage：hdfs dfs -du [-s] [-h] URI [URI …] 
```

选项： -s选项将显示文件长度的汇总摘要，而不是单个文件。 -h选项将以“人类可读”的方式格式化文件大小（例如64.0m而不是67108864）
### 6、-dus 显示文件长度的摘要。 

```bash
Usage：hdfs dfs -dus < args> 
```

注意：不推荐使用此命令。而是使用hdfs dfs -du -s。
### 7、-mv 在HDFS文件系统中，将文件或目录从HDFS的源路径移动到目标路径。不允许跨文件系统移动文件。 

```bash
Usage: hdfs dfs -mv URI [URI …] < dest>
```

### 8、-cp 在HDFS文件系统中，将文件或目录复制到目标路径下 

```bash
Usage：hdfs dfs -cp [-f] [-p | -p [topax] ] URI [ URI …] < dest> 
```

选项： -f选项覆盖已经存在的目标。 -p选项将保留文件属性[topx]（时间戳，所有权，权限，ACL，XAttr）。如果指定了-p且没有arg，则保留时间戳，所有权和权限。如果指定了-pa，则还保留权限，因为ACL是一组超级权限。确定是否保留原始命名空间扩展属性与-p标志无关。
### 9、-copyFromLocal 从本地复制文件到hdfs文件系统（与-put命令相似） 

```bash
Usage: hdfs dfs -copyFromLocal < localsrc> URI 
```

选项： 如果目标已存在，则-f选项将覆盖目标。
### 10、-copyToLocal 复制hdfs文件系统中的文件到本地 （与-get命令相似） 

```bash
Usage: hdfs dfs -copyToLocal [-ignorecrc] [-crc] URI < localdst>
```

### 11、-rm 删除一个文件或目录 

```bash
Usage：hdfs dfs -rm [-f] [-r|-R] [-skipTrash] URI [URI …] 
```

选项： 如果文件不存在，-f选项将不显示诊断消息或修改退出状态以反映错误。 -R选项以递归方式删除目录及其下的任何内容。 -r选项等效于-R。 -skipTrash选项将绕过垃圾桶（如果已启用），并立即删除指定的文件。当需要从超配额目录中删除文件时，这非常有用。
### 12、-cat 显示文件内容到标准输出上。 

```bash
Usage：hdfs dfs -cat URI [URI …]
```

### 13、-text 获取源文件并以文本格式输出文件。允许的格式为zip和TextRecordInputStream。

```bash
Usage: hdfs dfs -text 
```

### 14、-touchz 创建一个零长度的文件。 

```bash
Usage：hdfs dfs -touchz URI [URI …]
```

### 15、-stat 显示文件所占块数(%b)，文件名(%n)，块大小(%n)，复制数(%r)，修改时间(%y%Y)。 

```bash
Usage：hdfs dfs -stat URI [URI …]
```

### 16、-tail 显示文件的最后1kb内容到标准输出 

```bash
Usage：hdfs dfs -tail [-f] URI 
```

选项： -f选项将在文件增长时输出附加数据，如在Unix中一样。
### 17、-count 统计与指定文件模式匹配的路径下的目录，文件和字节数 

```bash
Usage: hdfs dfs -count [-q] [-h] < paths>
```

### 18、-getmerge 将源目录和目标文件作为输入，并将src中的文件连接到目标本地文件（把两个文件的内容合并起来） 

```bash
Usage：hdfs dfs -getmerge < src> < localdst> [addnl] 
```

注：合并后的文件位于当前目录，不在hdfs中，是本地文件
### 19、-grep 从hdfs上过滤包含某个字符的行内容 

```bash
Usage：hdfs dfs -cat < srcpath> | grep 过滤字段
```
### 20、-moveFromLocal 从本地剪切粘贴到 HDFS 指定目录

```bash
dfs -moveFromLocal <src> <dst>
```

### 21、-appendToFile 追加一个文件到已经存在的文件末尾

hadoop shell官网：[https://hadoop.apache.org/docs/r1.0.4/cn/hdfs_shell.html](https://hadoop.apache.org/docs/r1.0.4/cn/hdfs_shell.html)
