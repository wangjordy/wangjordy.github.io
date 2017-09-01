---
layout: post
title: "Ubuntu 安装Hadoop2.7《一》单机配置"
date: 2016-10-24 17:27:00 +0800
comments: true
categories: 
- Hadoop
- BigData

---

###一、环境介绍
使用Ubuntu16.04， Hadoop2.7版本， JDK使用1.7版本 （这里可以选择自己的Hadoop2.x版本，或者JDK 1.x版本）

本教程是在VMware Fusion上安装的Ubuntu16.04桌面版系统，

<!--more-->

###二、 创建Hadoop用户

如果之前安装的Ubuntu系统不是hadoop用户， 需先添加一个hadoop的用户。

####1. 打开终端，添加账户

执行的命令：

```ruby
sudo useradd -m hadoop -s /bin/bash
```
这条命令创建了可以登陆hadoop账户，并且，使用 /bin/bash 作为shell。

####2. 修改hadoop账户的密码
执行的命令：

```ruby
sudo passwd hadoop
```

按照上面的提示，需要输入2次相同的密码。

####3. 设置hadoop账户为管理员权限

执行的命令：

```ruby
sudo adduser hadoop sudo
```
####4. 切换为hadoop账户
执行的命令：

```ruby
su hadoop
```
之后，输入hadoop的密码， 如果退出当前账号，输入 exit，  当然也可以选择注销当前账户，再重新选择hadoop登录

###三、 安装一些周边必备软件

推荐安装Vim、Ruby、SSH等

####1.安装Vim
执行的命令：

```ruby
sudo apt-get install vim
```

####2.安装SSH
执行的命令：

```ruby
sudo apt-get install openssh-server
```
安装完成后，使用以下命令登录本机
执行的命令：

```ruby
ssh localhost
```
之后， 弹出的提示信息输入yes，  之后就可以看到我们使用了ssh登录了当前系统，
如果退出， 执行exit命令

当然，这样的登录需要每次都输入密码， 接下来我们配置SSH无密码登录方式

```ruby
ssh localhost
```
####3.设置SSH免密码登录

执行命令前请先退出刚刚ssh localhost登录的方式，

执行的命令：

```ruby
cd ~/.ssh/
ssh-keygen -t rsa         使用RSA方式加密， 生成ssh的秘钥
cat ./id_rsa.pub >> ./authorized_keys   将公钥copy到authorized_keys文件中，相当于一种自身的授权
```
之前再次输入  
```ruby
ssh localhost
```
就可以直接登录了

###四、 安装Oracle的JDK

安装JDK可以有2种方式安装、 一个是通过ppa（源）方式安装， 另一种是通过下载安装包配置环境安装；

这里为了方便起见，就采用ppa的方式安装，

####1.添加ppa源

```ruby
sudo add-apt-repository ppa:webupd8team/java

sudo apt-get update
```
####2.安装oracle-java-installer
这里安装的是JDK1.7
```ruby
sudo apt-get install oracle-java7-installer
```
如果有想安装JDK1.8，也可以使用命令
```ruby
sudo apt-get install oracle-java8-installer
```
安装完成之后， 测试JDK
```ruby
java -version
```

如果是下载JDK的方式，配置环境变量的方式：

```ruby
sudo apt-get install oracle-java8-installer
```
一般配置一个全局配置文件  /etc/bash.bashrc

![](http://ww3.sinaimg.cn/large/62ca154dgw1f93jjfdgpuj20dc02ejs4.jpg)


```ruby
source /etc/bash.bashrc
```
###五、 安装Hadoop2.7

Hadoop2 下载地址：http://mirror.bit.edu.cn/apache/hadoop/common/

选择hadoop-2.7.0.tar.gz

下载好之后

####1.解压缩到/usr/local目录下

```ruby
sudo tar -zxf ~/Donwload/hadoop-2.7.0.tar.gz -C /usr/local    解压到/usr/local中
cd /usr/local/
sudo mv ./hadoop-2.7.0/ ./hadoop            将文件夹名改为hadoop
sudo chown -R hadoop ./hadoop       修改文件权限
```
####2.Hadoop的单机配置

Hadoop默认模式为非分布式模式，不需要进行其他的配置就可以直接运行，（非分布式也就是Java的单进程，方便开发时调试）

测试hadoop是否可用

```ruby
cd /usr/local/hadoop
mkdir ./input
cp ./etc/hadoop/*.xml ./input   将配置文件作为输入文件
./bin/hadoop jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar grep ./input ./output 'dfs[a-z.]+'
cat ./output/*  查看运行结果
```

执行成功后如下所示，输出了作业的相关信息，输出的结果是符合正则的单词 dfsadmin 出现了1次

如图：

![](http://ww3.sinaimg.cn/large/62ca154dgw1f93jkm824nj20co04dq3x.jpg)

最后， 由于Hadoop 默认不会覆盖结果文件，因此再次运行上面实例会提示出错，需要先将 ./output 删除。

```ruby
rm -r ./output
```

 
 