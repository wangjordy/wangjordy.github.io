---
layout: post
title: "Ubuntu 安装Hadoop2.7《四》集群的安装配置"
date: 2016-10-26 15:08:00 +0800
comments: true
categories: 
- Hadoop
- BigData

---

在前3节我们分别介绍了单机和伪分布的方式教程， 这一节开始讲述如何配置Hadoop集群， 如果没看过前3节的，建议可以先看看之前的配置，为了方便理解，本教程使用的两个节点：

第一个作为Master节点， 内网的IP地址：<font color=#006400>10.168.13.166 </font>

第二个作为Slave节点，  内网的IP地址：<font color=#006400>10.168.13.167</font>

这个IP地址可以自行定义，常见的有192.168网段的IP地址， 只要2台主机可以互相Ping通即可；

<!--more-->

##操作流程

#####1. 选择一台主机作为Master节点
#####2. 在Master节点上新建Hadoop用户，安装SSH Server，安装Oracle的JDK1.7
#####3. 在Master节点上安装Hadoop配置
#####4. 在Slave节点上新建Hadoop用户，安装SSH Server，安装Oracle的JDK1.7
#####5. 将Master节点上配置好的Hadoop复制到Slave节点上
#####6. 在Master节点上开启Hadoop服务

本节不再介绍JDK和SSH的安装方式，可查看第一节的单机配置的内容

###一、 网络的配置

由于我使用的是VMware Fusion安装的虚拟机的方式， 需要先将系统的网络连接方式改为桥接(Bridge)模式，如图：

![](http://ww3.sinaimg.cn/large/62ca154dgw1f95p9i6jqnj20b7045gmb.jpg)

注意：  这一步很重要， 否则可能多节点之间不能ping通，

<font color=#1E90FF size=4>在ubuntu下设置网络的方式：</font>

####1. 打开系统的设置找到Network(网络配置)

如图： 点击Option按钮， 选择IPv4 Settings 修改网络配置为手动设置IP地址， 并添加新的IP、子网掩码、网关、DNS
![](http://ww1.sinaimg.cn/large/62ca154dgw1f95ph30vzgj20ne0lgdkk.jpg)

####2. 检查网络配置
重启系统或者重启网络之后，通过`ifconfig` 查看是否ip地址设置正确， 并且2个节点都可以互相ping通

###二、 设置Hostname

如果之前配置过Hadoop， 先关闭Hadoop服务器，`/usr/local/hadoop/sbin/stop-dfs.sh`， 再继续后面的集群配置，
为了便于在使用上对节点的区分， 我们先修改节点的主机名

```ruby
sudo vim /etc/hostname
```
直接将里面的ubuntu改为Master （如果是Slave节点， 改为Slave1）

接着修改Hosts文件（做IP地址与节点名的映射）
通过

```ruby
sudo vim /etc/hosts
```
在hosts文件中添加

```ruby
10.168.13.166   Master
10.168.13.167   Slave1
```

<font color=#DC143C>注意： 打开的hosts文件中 （127.0.0.1 localhost  这里不能直接修改为 127.0.0.1 Master），</font>  修改后hosts配置文件，如下图：

![](http://ww3.sinaimg.cn/large/62ca154dgw1f95puqpgz2j20en05b3z1.jpg)

<font color=	#DAA520>另外注意的点时，再设置完hostname，必须要重启系统后，才能生效， 所以这里执行到这里一定要重启一下系统，
上述修改Master和Slave节点的配置一样 </font>

配置好上述的配置之后， 可以通过命令

```ruby
ping Master -c 5
ping Slave1 -c 5
```
检测这2个节点是否可以ping通。 如果没有问题，那我们继续。

###三、 设置SSH无密码登录节点

在之前的小节中也有介绍， 这里的操作是要让Master节点可以无密码的方式登录到各个Slave节点上。

<font color=	#20B2AA>以下操作在Master节点上：</font>

####1. 生成ssh秘钥

如果之前有生成过秘钥的，需要先删除之前生成的秘钥文件，因为修改hostname之后， 之前的秘钥就失效了，命令：


```ruby
cd ~/.ssh
rm ./id_rsa*
ssh-keygen -t rsa
cat ./id_rsa.pub >> ./authorized_keys
```
设置好之后， 通过执行`ssh Master`验证一下（可能需要输入 yes，成功后执行 exit 返回原来的终端，第二次再重试登录，则不再需要输入密码， 最后exit退出)。

####2. 将Master的公钥分发给Slave1节点

```ruby
scp ~/.ssh/id_rsa.pub hadoop@Slave1:/home/hadoop/
```
scp完成之后，如下图：

![](http://ww3.sinaimg.cn/large/62ca154dgw1f95qe58kpcj21ed02uabf.jpg)


<font color=	#20B2AA>切换到Slave1节点上操作：</font>

####3. 将刚刚Master节点上的公钥添加授权

```ruby
mkdir ~/.ssh
cat ~/id_rsa.pub >> ~/.ssh/authorized_keys
rm ~/id_rsa.pub 
```
如果还有其他的Slave节点， 同样的方式，也需要将Master的公钥下发给Slave节点并做这一步的操作。

<font color=	#20B2AA>切换到Master节点上操作：</font>
####4. 在Master节点上检验是否可以无密码登录

```ruby
ssh Slave1
```
如图：
 
![](http://ww3.sinaimg.cn/large/62ca154dgw1f95ql5r6bsj20hj05z765.jpg)

表示登录成功。

###四、 配置PATH变量

为了方便在任意目录下都能直接使用hadoop、hdfs这样的命令， 需要在Master节点进行配置，

####1. 编辑.bashrc文件

```ruby
 vim ~/.bashrc
```
打开文件后，在底部加入内容：

```ruby
export PATH=$PATH:/usr/local/hadoop/bin:/usr/local/hadoop/sbin
```
保存并退出。

####2. 设置.bashrc文件生效
```ruby
 source ~/.bashrc
```
###五、 配置集群环境（Hadoop的环境）

<font color=	#20B2AA>切换到Master节点上操作：</font>

修改Hadoop集群/分布式模式，需要修改/usr/local/hadoop/etc/hadoop中的5个配置文件：
<font color=#228B22> slaves、core-site.xml、hdfs-site.xml、mapred-site.xml、yarn-site.xml </font>

####1. 设置文件slaves

编辑slaves文件中内容，将原来的localhost删除， 只添加一行内容 Slave1

####2. 设置文件core-site.xml 改为以下的内容
```ruby
<configuration>
        <property>
                <name>fs.defaultFS</name>
                <value>hdfs://Master:9000</value>
        </property>
        <property>
                <name>hadoop.tmp.dir</name>
                <value>file:/usr/local/hadoop/tmp</value>
                <description>Abase for other temporary directories.</description>
        </property>
</configuration>
```
####3. 设置文件 hdfs-site.xml 改为以下的内容
```ruby
 <configuration>
        <property>
                <name>dfs.namenode.secondary.http-address</name>
                <value>Master:50090</value>
        </property>
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
<font color=#DAA520>上述配置项dfs.replication 一般设为 3，但我们只有一个 Slave 节点，所以 dfs.replication 的值还是设为 1 </font>
####4. 设置文件mapred-site.xml（如果没有该文件，需先重命名mapred-site.xml.template为mapred-site.xml） 改为以下的内容
```ruby
<configuration>
        <property>
                <name>mapreduce.framework.name</name>
                <value>yarn</value>
        </property>
        <property>
                <name>mapreduce.jobhistory.address</name>
                <value>Master:10020</value>
        </property>
        <property>
                <name>mapreduce.jobhistory.webapp.address</name>
                <value>Master:19888</value>
        </property>
</configuration>
```
####5. 设置文件yarn-site.xml 改为以下的内容
```ruby
<configuration>
        <property>
                <name>yarn.resourcemanager.hostname</name>
                <value>Master</value>
        </property>
        <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
        </property>
</configuration>
```

配置好之后， 将Master的/usr/local/Hadoop文件夹打包复制到其他Slave节点上（如果之前有运行过单机或伪分布式，需要先删除之前的临时文件）， 在Master节点上继续执行一下命令：

```ruby
cd /usr/local
sudo rm -r ./hadoop/tmp     # 删除 Hadoop 临时文件
sudo rm -r ./hadoop/logs/*   # 删除日志文件
tar -zcf ~/hadoop.master.tar.gz ./hadoop   # 先压缩再复制
cd ~
scp ./hadoop.master.tar.gz Slave1:/home/hadoop
```
<font color=	#20B2AA>切换到Slave1节点上操作：</font>

```ruby
sudo rm -r /usr/local/hadoop   删除之前安装过旧的hadoop
sudo tar -zxf ~/hadoop.master.tar.gz -C /usr/local   解压所到/usr/local目录下
sudo chown -R hadoop /usr/local/hadoop        设置权限
```
同理， 如果还有其他的Slave节点， 需要重复以上的步骤。


###六、 验收集群环境

配置完以上的所有环境之后， 我们进行启动Hadoop服务器，进行验收环境配置是否成功。

<font color=	#20B2AA>切换到Master节点上操作：</font>
####1. NameNode 的格式化

```ruby
hdfs namenode -format
```
因修改了Hadoop的配置文件，所以需要先格式化， 之后再重启Hadoop服务，则不用再执行这一步。

####2. 启动Hadoop服务器

```ruby
start-dfs.sh
start-yarn.sh
mr-jobhistory-daemon.sh start historyserver
```

通过`jps`命令，可以查看各个节点启动的进程， 如果启动没问题的话， 在Master节点可以看到的进程，如图：

![](http://ww4.sinaimg.cn/large/62ca154dgw1f95re5g10uj207e02yjrr.jpg)

在Slave1节点上可以看到的进程，如图：

![](http://ww1.sinaimg.cn/large/62ca154dgw1f95rfzw89vj2084020wem.jpg)

上述2个节点，如果缺少任何一个进程都表示配置出错了。

当然，在Master节点上也可以通过命令 `hdfs dfsadmin -report` 查看DataNode是否正常启动，如果 Live datanodes 不为 0 ，则说明集群启动成功。  例如：我们只配置了一个Datanodes， 应与下图一致：

![](http://ww1.sinaimg.cn/large/62ca154dgw1f95rjqjls9j20db03bt92.jpg)

也可以通过Web页面查看NameNode和DataNode的详细信息， 访问地址：  http://master:50070/


####3. 通过实例测试环境是否可用

<font color=	#20B2AA>在Master节点上操作：</font>

首先， 在HDFS上创建用户目录：

```ruby
hdfs dfs -mkdir -p /user/hadoop
```
其次，将/usr/local/hadoop/etc/hadoop 中的配置文件作为输入文件复制到分布式文件系统中：

```ruby
hdfs dfs -mkdir input
hdfs dfs -put /usr/local/hadoop/etc/hadoop/*.xml input
```
接着， 我们查看web页面中DataNode占用的大小是否有改变， 正常如下图：

![](http://ww2.sinaimg.cn/large/62ca154dgw1f95rpfl5qdj20h808v0t7.jpg)

最后，我们执行一个MapReduce的作业

```ruby
hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar grep input output 'dfs[a-z.]+'
```
这些步骤与之前做伪分布式类型， 会显示出Job的进度， 在虚拟机上运行会比较慢， 如果你运行的job一直卡住不动，有可能是你自己机器或者给虚机分配的内存不足， 建议增加内存，或者修改YARN的内存配置来解决。

好了， 如果执行正确的话， 应该如下图：


![](http://ww4.sinaimg.cn/large/62ca154dgw1f95ru5tr03j20h80370ta.jpg)

同样，我们也可以查看web页面，访问 http://master:8088/cluster查看任务的进度， 在页面中，点击 “Tracking UI” 中History，
就可以看到任务的运行信息了， 如图：

![](http://ww2.sinaimg.cn/large/62ca154dgw1f95rvwfxhej20h809jab8.jpg)

当然我们也可以通过控制台查看，执行完后结果：

![](http://ww3.sinaimg.cn/large/62ca154dgw1f95rwz8ogsj20fq02uwep.jpg)


####4. 关闭Hadoop的操作

如果想要关闭Hadoop的话， 在Master节点上执行

```ruby
stop-yarn.sh
stop-dfs.sh
mr-jobhistory-daemon.sh stop historyserver
```
以上就是所有Hadoop的集群搭建了。
