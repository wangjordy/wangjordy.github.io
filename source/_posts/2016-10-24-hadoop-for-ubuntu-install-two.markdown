---
layout: post
title: "Ubuntu 安装Hadoop2.7《二》单机伪分布配置"
date: 2016-10-24 17:27:00 +0800
comments: true
categories: 
- Hadoop
- BigData

---

上一节我们介绍了Hadoop的单机运行配置，这里我们继续采用单节点以伪分布式的方式来运行，Hadoop进程是以分离的Java进程来运行的，
节点即可以作为NameNode也可以作为DataNode， 同时，读取的是HDFS中的文件。

<!--more-->

###一、 配置Hadoop伪分布式环境

####1. 编辑hadoop-env.sh环境配置文件

执行的命令：

```ruby
sudo vim /usr/local/hadoop/etc/hadoop/hadoop-env.sh
```
打开后注释原来的JAVA_HOME,改为绝对路径，如图：


![](http://ww3.sinaimg.cn/large/62ca154dgw1f93marha6qj20dq0240t0.jpg)

注：这个文件中其他的配置暂时还不需要

####2. 编辑core-site.xml和hdfs-site.xml
这2个文件也都位于 /usr/local/hadoop/etc/hadoop/中，

修改core-site.xml，编辑文件，在configuration标签修改为：

```ruby
<configuration>
        <property>
             <name>hadoop.tmp.dir</name>
             <value>file:/usr/local/hadoop/tmp</value>
             <description>Abase for other temporary directories.</description>
        </property>
        <property>
             <name>fs.defaultFS</name>
             <value>hdfs://localhost:9000</value>
        </property>
</configuration>
```
同理修改hdfs-site.xml

```ruby
<configuration>
        <property>
             <name>dfs.replication</name>
             <value>1</value>
        </property>
        <property>
             <name>dfs.namenode.name.dir</name>
             <value>file:/usr/local/hadoop/tmp/dfs/name</value>
        </property>
        <property>
             <name>dfs.datanode.data.dir</name>
             <value>file:/usr/local/hadoop/tmp/dfs/data</value>
        </property>
</configuration>
```

解说： 由于Hadoop的运行方式都是由xml配置文件决定的，如果需要从伪分布式切换回非分布式模式，需要删除core-site.xml的配置项

另外，伪分布式虽然只需要配置`fs.defaultFS`和`dfs.replication`就可以运行了，不过如果没有配置`hadoop.tmp.dir`参数项的话，则默认使用的临时文件目录为`/tmp/hadoo-hadoop`, 然后这个目录在重启时有可能被系统清理掉，导致必须重新执行format才行，所以这里进行了设置，同时，指定了`dfs.namenode.name.dir` 和 `dfs.datanode.data.dir`。

配置完成之后，执行NameNode的格式化:

```ruby
./bin/hdfs namenode -format
```
注意： 我们当前是在/usr/local/hadoop/目录操作的命令， 成功的话会看到如下图的提示：

![](http://ww2.sinaimg.cn/large/62ca154dgw1f93mstw8ctj20v0053aef.jpg)

如果出现Error:JAVA_HOME is not set and could not be found. 就说明之前你的JAVA_HOME变量没配置对，

####3. 开启NameNode和DataNode守护进程

```ruby
./sbin/start-dfs.sh
```
执行命令，如果提示SSH输入yes即可，

![](http://ww2.sinaimg.cn/large/62ca154dgw1f93mwdfsjqj20ju04676q.jpg)

启动时可能会出现如下 WARN 提示：WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform… using builtin-java classes where applicable。该 WARN 提示可以忽略，并不会影响正常使用（该 WARN 可以通过编译 Hadoop 源码解决）。

启动完之后，输入命令

```ruby
jps
```
查看NameNode，DataNode SecondaryNameNode有没有启动 ，正常启动如图：

![](http://ww4.sinaimg.cn/large/62ca154dgw1f93mz4iwdvj20a102p0tb.jpg)

这时候，通过浏览器访问地址 http://localhost:50070 可以查看NameNode和DataNode的具体信息

![](http://ww1.sinaimg.cn/large/62ca154dgw1f93n0q5xcbj20wd0r1n3z.jpg)

###二、 运行Hadoop伪分布式实例
在上一节中，我们使用单机模式grep例子中读取的是本地的数据， 而伪分布式读取的则是HDFS上的数据。 要使用HDFS，首先需要再HDFS中创建用户目录

```ruby
./bin/hdfs dfs -mkdir -p /user/hadoop
```
接着将 ./etc/hadoop 中的 xml 文件作为输入文件复制到分布式文件系统中（也就是将 /usr/local/hadoop/etc/hadoop 上传到HDFS中的 /user/hadoop/input中）

```ruby
./bin/hdfs dfs -mkdir input
./bin/hdfs dfs -put ./etc/hadoop/*.xml input
```
复制完成后，可以通过如下命令查看文件列表：
```ruby
./bin/hdfs dfs -ls input
```
这时，我们的运行的文件与上一节中的运行文件是相同的， 区别仅是在HDFS中（单机的是本地input中）， 同时，我们也都输出到output文件夹中，

```ruby
./bin/hadoop jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar grep input output 'dfs[a-z.]+'
```

查看运行结果的命令

```ruby
./bin/hdfs dfs -cat output/*
```
运行的结果如下图：

![](http://ww4.sinaimg.cn/large/62ca154dgw1f93n8v1rcmj20gb03ldgq.jpg)

如果将HDFS中的结果，取到本地，执行命令

```ruby
./bin/hdfs dfs -get output ./output   将HDFS的outputcopy到本地的./output目录下，如果本地有output文件夹请先删除
cat ./output/*
```
如果想要删除HDFS中的output文件目录

```ruby
./bin/hdfs dfs -rm -r output
```
若要关闭 Hadoop，则运行

```ruby
./sbin/stop-dfs.sh
```
同理，如果下次运行hadoop，无需再次format NameNode了， 直接运行

```ruby
./sbin/start-dfs.sh
```
即可！