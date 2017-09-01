---
layout: post
title: "Hive的基础知识总结"
date: 2017-07-25 18:03:34 +0800
comments: true
categories: 
- Hive
---

上一节我们介绍了Hive的安装配置， 本节我们将简单介绍一下Hive的一些基础知识。

##Hive关于内部表、外部表、分区和分桶概念

####一、内部表

一般说Hive的表通常指的就是内部表。

Hive的表在HDFS中都有相应的目录来存储表的数据，目录的路径，也是我们上节在<font color=#FF7F50>hive-site.xml</font>文件
中关于`hive.metastore.warehouse.dir`的上配置， 默认我们放在`/user/hive/warehouse`路径下。
<!--more-->
比如我们有一个表<font color=#0099ff>`t_hive`</font>， 那么在HDFS中会创建 `/user/hive/warehouse/t_hive`目录。 <font color=#0099ff>`t_hive`</font>表中所有的数据也就存在这个目录中。

####二、外部表

Hive的外部表与内部表很相似，只是在建表时可以指定加载的HDFS目录， 也可以不指定（根据需要时再加载）。

如果外部表使用Hive命令删除表分区，对应的HDFS文件是不会被删除的， 外部表比较灵活，既可以关联HDFS文件也可以关联HBSE表。

####三、分区

在Hive中表的每一个分区对应表内的相应的目录，所有分区的数据都存储在对应的目录中， 比如<font color=#0099ff>`t_hive`</font>表中的column `a`就对应有a的分区。

####四、桶

桶的概念主要为了并行， 它是指，对指定的column计算hash值， 根据hash值来切分数据， 每一个桶对应一个文件， 比方<font color=#0099ff>`t_hive`</font>的a column分散到10个桶中， 首先对id列的值进行hash计算，根据hash值将数据数据分散到不同的桶中。  （一般情况下，建议分桶设置不要太大）


##Hive中文件存储常见的几种格式

####一、textfile

textfile是默认格式，建表时如果不指定的话，默认为textfile， 它的存储方式是 <font color=#008B8B>行存储</font>

优点： 可以直接读取。

缺点： 磁盘开销比较大，数据解析开销大， 压缩的text文件，hive无法进行合并和拆分

####二、sequencefile

sequencefile是二进制的文件， 它是以<key,value>的形式存储， 它的存储方式是 <font color=#008B8B>行存储</font>

优点：可以分割，压缩， 全表查询效率高

缺点：存储空间消耗最大

一般选择block压缩，文件和Hadoop api中的mapfile是相互兼容的。EQUENCEFILE将数据以<key,value>的形式序列化到文件中。序列化和反序列化使用Hadoop 的标准的Writable 接口实现。key为空，用value 存放实际的值， 这样可以避免map 阶段的排序过程。三种压缩选择：NONE, RECORD, BLOCK。 Record压缩率低，一般建议使用BLOCK压缩。使用时设置参数，

```mysql
SET hive.exec.compress.output=true;  
SET io.seqfile.compression.type=BLOCK; -- NONE/RECORD/BLOCK  
create table test2(str STRING)  STORED AS SEQUENCEFILE;
```
####三、rcfile
rcfile是采用<font color=#008B8B>行列存储相结合的方式</font> 

首先，将数据按行分块，保证同一个record在同一个块上，表面读一个record需要读取多个block。

其次，块数据列式存储，有利于数据压缩和快速的列存取，

优点： 压缩快， 快速列存取， 读记录尽量涉及到的block最少 ，读取需要的列只需要读取每个row group 的头部定义。

缺点：读取全量数据的操作 性能可能比sequencefile没有明显的优势。但是如果指定一列的话，效率最高

####四、orc

orc将数据<font color=#008B8B>按行分块，每块按照列存储</font>

效率比rcfile高,是rcfile的改良版本。

优点： 压缩快 快速列存取

####五、自定义格式

用户可以通过 inputformat和outputformat来自定义输入输出的格式。


###总结：

textfile 存储空间消耗比较大，并且压缩的text 无法分割和合并 查询的效率最低,可以直接存储，加载数据的速度最高

sequencefile 存储空间消耗最大,压缩的文件可以分割和合并 查询效率高，需要通过text文件转化来加载

rcfile 存储空间最小，查询的效率最高 ，需要通过text文件转化来加载，加载的速度最低


