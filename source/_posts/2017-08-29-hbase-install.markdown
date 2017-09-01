---
layout: post
title: "HBase的安装教程"
date: 2017-08-29 14:25:16 +0800
comments: true
categories: 
- HBase
---

HBase是一个分布式、面向列存储的数据库， 我们知道HBase的设计与Google的BigTable一样， HBase是在HDFS之上提供了类似Bigtable的能力。

<font color=#D19275>HBase是建立在Hadoop文件系统上支持支持横向扩展的数据库， 它利用HDFS提供了容错的能力，人们可以直接或者通过HBase的存储HDFS数据。

使用HBase在HDFS读取消费/随机访问数据。 HDBase在HDFS基础上，提供了读写访问。
</font>

<!--more-->

###HBase的常用术语

 * <font color=#2F4F4F>行健Row key：</font> <font color=#008B8B>主见是用来检索记录的主键，访问HBase table中的行。</font>
 * <font color=#2F4F4F>列簇Column Family: </font> <font color=#008B8B>Table在水平方向有一个或多个的ColumnFamily， 而一个ColumnFamily是有任意多个Column组成。
   		Column Family支持动态扩展， 无需预先定义Column的数量和数据类型，所有Column均以二进制的格式存储。</font>
 * <font color=#2F4F4F>列Column</font> 
 * <font color=#2F4F4F>单元格Cell： </font> <font color=#008B8B>HBase中通过row和columns确定的一个存储单元为cell。</font>
 * <font color=#2F4F4F>版本version: </font> <font color=#008B8B>每个cell都保存同一份数据的多个版本。版本通过时间戳来索引。</font>


<font face="黑体" size=4>HBase数据结构图：</font>

![](http://ww1.sinaimg.cn/large/62ca154dly1fj0m0rgbzsj20kq08lmxu.jpg)
 
 
###HBase和HDFS 
 
HBase  | HDFS
------------- | -------------
HBase是建立在HDFS之上的数据库。  | HDFS是适于存储大容量文件的分布式文件系统。
HBase提供在较大的表快速查找  		| HDFS不支持快速单独记录查找。
它提供了数十亿条记录低延迟访问单个行记录（随机存取）。  | 它提供了高延迟批量处理;没有批处理概念。
HBase内部使用哈希表和提供随机接入，并且其存储索引，可将在HDFS文件中的数据进行快速查找。 | 它提供的数据只能顺序访问。



###HBase的使用场景

* 它用来当有需要写应用程序；
* HBase适用于我们需要提供快速、随机访问的数据；

目前在Facebook、Twitter、Yahoo等都在使用HBase。

###HBase的特点
* <font color=#228B22>HBase线性可扩展。</font>
* <font color=#228B22>它具有自动故障支持。</font>
* <font color=#228B22>它提供了一直的读取和写入。</font>
* <font color=#228B22>它集成了Hadoop，作为源和目的地。</font>
* <font color=#228B22>为客户端提供方便的API。</font>
* <font color=#228B22>它提供了跨集群数据复制。</font>


##HBase的安装教程

###官网下载

[下载地址](http://apache.fayea.com/hbase/1.3.1/).

目前最新版本是1.3.1，  不建议大家安装alpha版本；

###解压tar，并存放到指定目录下

```sh
mv hbase-1.3.1 /usr/local
cd /usr/local/
mv hbase-1.3.1 hbase
```
###配置环境变量


```sh
sudo vi /etc/profile

```
在文件中配置：

```sh
export HBASE_HOME =/usr/local/hbase
export PATH=$PATH:${HBASE_HOME}/bin 
```
退出保存后， 执行命令


```sh
source /etc/profile

```

###配置相关的文件

（1）进入hbase安装目录下，编辑conf目录下的 hbase-env.sh文件

```sh
cd conf
vi hbase-env.sh
```
编辑文件，在文件加入下面的环境变量配置

```sh
export JAVA_HOME=/usr/lib/jvm/java-8-oracle
export HBASE_CLASSPATH=/usr/local/hbase/conf
export HBASE_MANAGES_ZK=false #由HBase负责启动和关闭Zookeeper 

```
（2） 在conf目录下， 配置core-site.xml
注意配置文件中的fs.defaultFS要与hadoop里core-site.xml的fs.defaultFS配置一样的IP和端口


```xml
<configuration>
          <property>
             <name>hbase.root.dir</name>
             <value>hdfs://localhost:9000</value>
        </property>
        <property>
           <name>hbase.cluster.distributed</name>
           <value>true</value>
        </property>
        <property>
           <name>hbase.master</name>
           <value>localhost</value>
        </property>
        <property>
           <name>hbase.zookeeper.property.clientPort</name>
           <value>2181</value>
        </property>
        <property>
           <name>hbase.zookeeper.quorum</name>
           <value>localhost</value>
        </property>
        <property>
           <name>hbase.tmp.dir</name>
           <value>/usr/local/hbase/tmp</value>
        </property>
        <property>
           <name>hbase.zookeeper.property.dataDir</name>
           <value>/usr/local/zookeeper/data</value>       
        </property>

</configuration>

```

###启动HBase

在启动HBase之前， 请先启动Hadoop，然后启动Zookeeper，最后启动HBase

```sh
cd /usr/local/hbase/bin
start-hbase.sh
```

启动之后，输入jps，查看是否有对应的进程， 如图：

![](http://ww1.sinaimg.cn/large/62ca154dly1fj0nyc6j5cj20qn088gng.jpg)

###启动HBase shell

![](http://ww1.sinaimg.cn/large/62ca154dly1fj0obymserj20zp055gn6.jpg)

####创建一个member表

```sh
create 'member','member_id','address','info'
```
![](http://ww1.sinaimg.cn/large/62ca154dly1fj0oi1khdbj20g702i74h.jpg)

####显示表的内容

```sh
list
```
![](http://ww1.sinaimg.cn/large/62ca154dly1fj0oi9g3ddj206p030aa5.jpg)

####查看member表的结构

```sh
describe 'member'
```

####插入数据

```sh
put 'member','jordy','info:age','78'
```

####查询一条数据

```sh
get 'member','jordy'
```

![](http://ww1.sinaimg.cn/large/62ca154dly1fj0ol7jgaqj20n002cdg2.jpg)

####扫描获取全部内容

![](http://ww1.sinaimg.cn/large/62ca154dly1fj0olqh0d7j20q102aq37.jpg)
