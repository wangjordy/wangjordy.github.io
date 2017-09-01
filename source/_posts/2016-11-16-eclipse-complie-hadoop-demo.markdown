---
layout: post
title: "Ubuntu 安装Eclipse,并执行MapReduce的实例"
date: 2016-11-16 16:16:19 +0800
comments: true
categories: 
- Hadoop

---

在前几节中，我们主要介绍了在ubuntu环境上安装部署hadoop的环境， 本节将通过一个实例来介绍在eclipse下编写程序，并在hadoop下执行。

<!--more-->

###一、 ubuntu安装eclipse

方法一： 从ubuntu的软件中心

方法二： 对于网络不太好的用户，可以直接在eclipse官网上下载
下载eclipse的地址 : <http://www.eclipse.org/downloads/download.php?file=/technology/epp/downloads/release/luna/SR2/eclipse-jee-luna-SR2-linux-gtk-x86_64.tar.gz>  

下载好直接接下到 /opt目录下

```ruby
sudo tar zxvf eclipse-jee-luna-SR2-linux-gtk-x86_64.tar.gz -C /opt
```

###二、 eclipse安装Hadoop-Eclipse-Plugin

在配置之前请先参考之前的关于介绍伪分布式的教程安装Hadoop， 并启动hadoop服务。

在Eclipse上编译运行MapReduce程序需要安装Hadoop-Eclipse-Plugin，可以在Github上<https://github.com/winghc/hadoop2x-eclipse-plugin> 下载， 下载后将hadoop-eclipse-kepler-plugin-2.6.0.jar包赋值到Eclipse的安装目录plugins文件夹下，

如果是通过软件中心安装的eclipse应该是在/usr/lib/eclipse目录下

复制好jar包之后，重启eclipse即可，打开后，会看到

![](http://ww2.sinaimg.cn/large/62ca154dgw1f9u19kosp7j20dm03dq36.jpg)

在这个步骤请一定要先确保之前的hadoop服务已经开启，并且配置好了伪分布式的相关配置；

####1.  在eclipse的菜单栏选择Window->Preference

![](http://ww1.sinaimg.cn/large/62ca154dgw1f9u1j4zcu8j20ih0i176q.jpg)

注意上图步骤2中，要填写你自己的hadoop路径；

####2.  切换至Map/Reduce的开发模式视图
选择Window->Open Perspective -> Other 在弹出框中选择Map/Reduce选项；

####3.  建立Hadoop的连接
点击Eclipse右下角的Map/Reduce Locations控制面板， 在面板上点击右键选择New Hadoop Location
在弹出来的General选项面板中， General的设置要与Hadoop中的配置一致，  这里使用的伪分布式，填写localhost即可，
设置 fs.defaultFS 为 hdfs://localhost:9000， 最终配置如下图：

![](http://ww2.sinaimg.cn/large/62ca154dgw1f9u1qbk2k3j20jh0d9jtm.jpg)

Advanced parameters 选项面板是对Hadoop参数进行配置，目前还用不到，直接点击Finish即可；


###三、 在Eclipse中创建MapReduce的工程

点击File->New-> Project 选择Map/Reduce Project, 然后点击Next

![](http://ww4.sinaimg.cn/large/62ca154dgw1f9u1w4wqe2j20nh0hgmzk.jpg)

这里创建一个WordCount的项目名，点击Finish就创建完工程了；

在项目的src目录下创建一个类， Package为：  org.apache.hadoop.examples；在Name处填写 WordCount。

打开刚刚创建的WordCount.java文件， 将以下代码复制到该文件中：



```ruby
package org.apache.hadoop.examples;
 
import java.io.IOException;
import java.util.StringTokenizer;
 
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;
 
public class WordCount {
 
  public static class TokenizerMapper 
       extends Mapper<Object, Text, Text, IntWritable>{
 
    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();
 
    public void map(Object key, Text value, Context context
                    ) throws IOException, InterruptedException {
      StringTokenizer itr = new StringTokenizer(value.toString());
      while (itr.hasMoreTokens()) {
        word.set(itr.nextToken());
        context.write(word, one);
      }
    }
  }
 
  public static class IntSumReducer 
       extends Reducer<Text,IntWritable,Text,IntWritable> {
    private IntWritable result = new IntWritable();
 
    public void reduce(Text key, Iterable<IntWritable> values, 
                       Context context
                       ) throws IOException, InterruptedException {
      int sum = 0;
      for (IntWritable val : values) {
        sum += val.get();
      }
      result.set(sum);
      context.write(key, result);
    }
  }
 
  public static void main(String[] args) throws Exception {
    Configuration conf = new Configuration();
    String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
    if (otherArgs.length != 2) {
      System.err.println("Usage: wordcount <in> <out>");
      System.exit(2);
    }
    Job job = new Job(conf, "word count");
    job.setJarByClass(WordCount.class);
    job.setMapperClass(TokenizerMapper.class);
    job.setCombinerClass(IntSumReducer.class);
    job.setReducerClass(IntSumReducer.class);
    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(IntWritable.class);
    FileInputFormat.addInputPath(job, new Path(otherArgs[0]));
    FileOutputFormat.setOutputPath(job, new Path(otherArgs[1]));
    System.exit(job.waitForCompletion(true) ? 0 : 1);
  }
}
```


需资源文件copy到src目录下，如：
将/usr/local/hadoop/etc/hadoop的xml文件以及log4j.properties 复制到WordCount工程的src文件夹下


```ruby
cp /usr/local/hadoop/etc/hadoop/core-site.xml ~/workspace/WordCount/src
cp /usr/local/hadoop/etc/hadoop/hdfs-site.xml ~/workspace/WordCount/src
cp /usr/local/hadoop/etc/hadoop/log4j.properties ~/workspace/WordCount/src
```
复制完之后，在eclipse中选中WordCount工程名，右键选择Refresh，就可以看到刚刚添加的文件结构了，如图：

![](http://ww1.sinaimg.cn/large/62ca154dgw1f9u321t8ulj206904n3z0.jpg)

点击WordCount.java文件选择Run As -> Run Configurations, 在这里可以设置运行时的相关参数（如果Java Application下面没有 WordCount工程名，那么需要先双击 Java Application），在右侧窗体切换“Arguments”栏， 在 Program arguments 处填写 “input output” 就OK了，如图：

![](http://ww1.sinaimg.cn/large/62ca154dgw1f9u36yciw7j20po0khq78.jpg)
 
当然也可以直接在代码中设置好输入的参数， 可以在代码main()中的第二行，替换为


```ruby
// String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
String[] otherArgs=new String[]{"input","output"};  //直接设置输入参数input和output
```
设置好参数后，选择 Run As -> Run on Hadoop运行程序，在Console栏会看到输出的output的文件夹，如图：

![](http://ww3.sinaimg.cn/large/62ca154dgw1f9u3ay6sa7j20mu0b5tbt.jpg)

现在就可以使用Eclipse做MapReduce的程序开发了。
