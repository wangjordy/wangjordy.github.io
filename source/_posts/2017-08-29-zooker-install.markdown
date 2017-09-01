---
layout: post
title: "Zookeeper的安装教程"
date: 2017-08-29 11:25:16 +0800
comments: true
categories: 
- Zookeeper
---

Zookeeper是一个面向分布式系统的协调服务， 它本身也是一个分布式的应用程序， 用于管理大型主机。
在分布式环境中协调和管理服务是一个复杂的过程， Zookeeper通过其简单的架构和API解决了这个问题。

它的一个设计模型如下图：

<!--more-->

![](http://ww1.sinaimg.cn/large/62ca154dly1fj0fzsscwoj20fw04zdfp.jpg)

Zookeeper采用了简单的 客户端--服务器的模型， 在Client使用服务的节点（机器）， 在Server是提供服务的节点。

Zookeeper的服务器的组合成了一个Zookeeper集合体（ensemble）。  在任何给定的时间内， 一个Zookeeper客户端可以连接到一个
Zookeeper的服务器。 

每个客户端会定期发送ping到它连接的服务器，让服务器知道它还处在活动连接状态。  同时，客户端发送的ping连接也在确认服务器是否处在活跃连接状态，如果客户端在设定的时间内没有收到服务器的确认， 那么客户端会连接到 Zookeeper集合体的 另一台服务器上，

并且客户端的会话也会被透明的转移到新的服务器上。


##安装教程

###1、官方下载最新的Zookeeper

官网[下载地址](http://www.apache.org/dyn/closer.cgi/zookeeper/).
目前最新的包是  zookeeper-3.4.10.tar.gz

下载好之后，解压tar包

###2、将zookeeper放在指定的目录

因之前hadoop放在了/usr/local/目录下，  这里我们将zookeeper也放在/usr/local/目录下

```sh
mv zookeeper-3.4.10 /usr/local/
cd /usr/local/
mv zookeeper-3.4.10 zookeeper
```

###3、设置环境变量


```sh
sudo vi /etc/profile

```
在文件中配置：

```sh
export ZOOKEEPER_HOME=/usr/local/zookeeper
export PATH=$PATH:${ZOOKEEPER_HOME}/bin 
```
退出保存后， 执行命令


```sh
source /etc/profile

```
让配置生效

###4、修改zookeeper的配置文件


```sh
cd /usr/local/zookeeper/conf/
cp zoo_sample.cfg zoo.cfg
```
编辑zoo.cfg文件，在文件中加入


```sh
dataDir=/usr/local/zookeeper/zkdata
dataLogDir=/usr/local/zookeeper/zkdatalog
```

退出并保存文件， 接着文件中的文件夹

```sh
mkdir /usr/local/zookeeper/zkdata
mkdir /usr/local/zookeeper/zkdatalog
```
注意这2个文件要都存在，否则启动zkserver会失败。

###5、启动Zookeeper

```sh
cd /usr/local/zookeeper/bin
./zkServer.sh start 
```
启动成功后，如下图

![](http://ww1.sinaimg.cn/large/62ca154dly1fj0fzc8d5ij20w60dkdjj.jpg)

能看到有上图绿色框的进程，就说明你已经配置成功了。

###6、其他命令

```sh
./zkServer.sh stop    #关闭
./zkServer.sh restart  #重启
```

