---
layout: post
title: "Ubuntu 安装Hadoop2.7《三》YARN的启动配置"
date: 2016-10-25 10:11:00 +0800
comments: true
categories: 
- Hadoop
- BigData

---

###一、简介

我们Hadoop早期的版本是JobTracker和TraskTracker，那么在2.0版本之后，使用新的MapReduce框架，加入（YARN---Yet Another Resource Negotiator）, YARN是运行在MapReduce之上，负责资源的管理和任务调度，从MapReduce分离出来的YARN,提供了高可用性、高扩展性。

在上一节中我们通过伪分布式只是启动了MapReduce， 现在我们通过启动YARN，来看看YARN的配置吧。

<!--more-->

###二、配置


####1. 首先重命名配置文件mapred-site.xml.template

```ruby
mv ./etc/hadoop/mapred-site.xml.template ./etc/hadoop/mapred-site.xml
```
####2. 修改配置文件mapred-site.xml

```ruby
gedit ./etc/hadoop/mapred-site.xml
```
编辑这个文件内容如下

```ruby
<configuration>
        <property>
             <name>mapreduce.framework.name</name>
             <value>yarn</value>
        </property>
</configuration>
```

####3. 修改配置文件yarn-site.xml

```ruby
<configuration>
        <property>
             <name>yarn.nodemanager.aux-services</name>
             <value>mapreduce_shuffle</value>
            </property>
</configuration>
```
####4. 启动YARN
启动YARN之前， 先输入命令jps查看HDFS是否已经启动（可以通过`./sbin/start-dfs.sh`启动）

```ruby
./sbin/start-yarn.sh 
```

启动完yarn之后， 输入命令jps，查看进程，如图：

![](http://ww1.sinaimg.cn/large/62ca154dgw1f94ay7q3nlj20dr03iq3m.jpg)

可以看到多了ResourceManager和NodeManager 两个进程

####5. 启动历史的服务器，方便web中查看任务运行情况
```ruby
./sbin/mr-jobhistory-daemon.sh start historyserver
```
如图：

![](http://ww4.sinaimg.cn/large/62ca154dgw1f94bc99sdwj20ok014wf7.jpg)

启动完之后， 通过浏览器，访问http://localhost:8088/cluster  如图：

![](http://ww3.sinaimg.cn/large/62ca154dgw1f94b1duyyoj20yo0c4gql.jpg)

在不启用YARN时，是“mapred.LocalJobRunner”执行任务， 而启动YARN之后，是“mapred.YARNRunner”在执行任务。

在YARN主要是为了资源管理与任务调度， 如果是在单机上执行不但体现不出来价值，反而会因为多加的新的YARN进程，让程序执行稍慢一些，


<font color=#DC143C>注意： 如果不启动YARN的配置，需要将mapred-site.xml 重命名，改成 mapred-site.xml.template。</font>

####6. 关闭YARN

```ruby
./sbin/stop-yarn.sh
./sbin/mr-jobhistory-daemon.sh stop historyserver
```



