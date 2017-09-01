---
layout: post
title: "Hive for Ubuntu 安装"
date: 2017-07-25 10:00:36 +0800
comments: true
categories: 
- Hive
---

本期给大家介绍一下在Ubuntu系统下安装Hive

##Hive的安装环境

 * Ubuntu16.04
 * Hadoop2.7
 * JDK 1.7
 * Hive 2.3
 * Mysql

<!--more-->
<font color=#FF7F50>注：在安装Hive之前，请先将上述的软件安装完成</font>
 
###一、Mysql中创建hive账号
 
 ```shell
 $mysql -u root -p
 mysql>grant all privileges on *.* to hive@"%" identified by "hive" with grant option;
 mysql>flush privileges;
 ```
###二、安装Hive的详细步骤

####1、安装Hive到 /usr/local/hive环境下

 ```shell
sudo tar xvfz apache-hive-2.3.0-bin.tar.gz 
sudo cp -R apache-hive-2.3.0-bin /usr/local/hive
sudo chmod -R 775 /usr/local/hive/
sudo chown -R hadoop:hadoop /usr/local/hive

 ```
 
####2、修改/etc/profile加入HIVE_HOME的变量
 
 ```sh
export HIVE_HOME=/usr/local/hive
export PATH=$PATH:$HIVE_HOME/bin
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:/usr/local/hive/lib
$source /etc/profile
 ```
 
####3、修改hive/conf的文件配置
 
 ```sh
cp hive-env.sh.template hive-env.sh
cp hive-default.xml.template hive-site.xml

 ```
修改hive-env.sh文件， 加入代码：

 ```sh
HADOOP_HOME=/usr/local/hadoop
 ```
<font color=#FF7F50>注： 要配置你自己的hadoop的路径</font>

修改hive-site.xml文件，制定MySQL数据库的驱动、数据库名、用户名和密码，修改的内容如下：



 ```txt
<property>
  <name>javax.jdo.option.ConnectionURL</name>
  <value>jdbc:mysql://localhost:3306/hive?createDatabaseIfNotExist=true;u‌​seSSL=false</value>
  <description>JDBC connect string for a JDBC metastore</description>
</property>
<property>
  <name>javax.jdo.option.ConnectionDriverName</name>
  <value>com.mysql.jdbc.Driver</value>
  <description>Driver class name for a JDBC metastore</description>
</property>
<property>
  <name>javax.jdo.option.ConnectionUserName</name>
  <value>hive</value>
  <description>username to use against metastore database</description>
</property>
<property>
  <name>javax.jdo.option.ConnectionPassword</name>
  <value>hive</value>
  <description>password to use against metastore database</description>
</property>

注释：
javax.jdo.option.ConnectionURL参数指定的是Hive连接数据库的连接字符串；
javax.jdo.option.ConnectionDriverName参数指定的是驱动的类入口名称；
javax.jdo.option.ConnectionUserName参数指定了数据库的用户名；
javax.jdo.option.ConnectionPassword参数指定了数据库的密码。

 ```
同样还在这个文件中，配置缓存目录

 ```sh
<property> 
 <name>hive.exec.local.scratchdir</name>
 <value>/home/hadoop/iotmp</value>
 <description>Local scratch space for Hive jobs</description>
 </property>
 <property>
 <name>hive.downloaded.resources.dir</name>
 <value>/home/hadoop/iotmp</value>
 <description>Temporary local directory for added resources in the remote file system.</description>
 </property>
 ```
 
 设置为配置之后， 需要创建这个缓存目录，并修改其文件权限
 
 ```sh
mkdir -p /home/hadoop/iotmp 
chmod -R 775 /home/hadoop/iotmp
 ```

###三、修改hive/bin下的hive-config.sh文件

 ```sh
export JAVA_HOME=/usr/lib/jvm
export HADOOP_HOME=/usr/local/hadoop
export HIVE_HOME=/usr/local/hive
 ```

<font color=#FF7F50>注： 要配置你自己的java和hadoop的路径</font>
 
###四、下载mysql-connector-java-5.1.42-bin.jar包放到Hive的lib目录下
 
###五、在HDFS创建/tmp和/user/hive/warehouse并设置其权限

 ```sh
hadoop fs -mkdir /tmp
hadoop fs -mkdir -p /user/hive/warehouse
hadoop fs -chmod g+w /tmp
hadoop fs -chmod g+w /user/hive/warehouse
 ```
 
###六、初始化Meta数据库


 ```sh
 cd /usr/local/hive/lib
 schematool -initSchema -dbType mysql
 ```
 
 运行后显示
 ![](http://ww1.sinaimg.cn/large/62ca154dly1fhvyjrvsz8j20ti0ad77x.jpg)
 
 最后显示`schemaTool completed`提示语表示安装成功。
 
####1、测试hive shell
 
  
 ```sh
hive
show databases；
show tables;
 ```
 
####2、例：创建table的方式
 
  ```sh
 CREATE TABLE invites (foo INT, bar STRING) PARTITIONED BY (ds STRING);
  ```
  
如图：
  
  ![](http://ww1.sinaimg.cn/large/62ca154dly1fhvyomdqzvj20jf08eaay.jpg)
  
####3、创建一个 t_hive 的table，并插入数据

```sh
CREATE TABLE t_hive (a int, b int, c int) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t';

insert into t_hive values (3,4,5);

```

####4、查看表数据

```sh
select * from t_hive;

```

####5、查看表数据

```sh
desc t_hive;

```

####6、增加一个column

```sh
ALTER TABLE t_hive ADD COLUMNS (new_col String);
desc t_hive;  #再查看一下表结构
```
####7、删除表

```sh
DROP TABLE t_hadoop;
```
####8、重命名表

```sh
ALTER TABLE t_hive RENAME TO t_hadoop;
```

####9、创建视图

```sh
CREATE VIEW IF NOT EXISTS t_hive (a int, b int, c int) AS SELECT a,b,c FROM t_hive;
```

####10、删除视图

```sh
DROP VIEW t_hive;
```

####11、创建函数

```sh
CREATE TEMPORARY FIUNCTION aFunc as class_name; 
```
####12、删除函数

```sh
DROP TEMPORARY FIUNCTION aFunc; 
```

####13、显示分区

```sh
SHOW PARTITIONS t_hive; 
```

####14、添加分区

```sh
ALTER TABLE t_hive ADD PARTITION(ds = '2013-05-07',country = 'china') LOCATION '/usr/test/data/test.txt';   
```
####15、删除分区

```sh
ALTER TABLE t_hive DROP PARTITION(ds = '2013-05-07',country = 'china'); 
```
