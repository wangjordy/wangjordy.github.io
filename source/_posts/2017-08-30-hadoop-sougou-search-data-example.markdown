---
layout: post
title: "MapReduce实战练习: 使用搜狗实验数据对数据去重（一）"
date: 2017-08-30 16:51:55 +0800
comments: true
categories: 
- hadoop 
- hadoop code 
---

上一篇我们使用自己创作的数据文件，对文件进行了去重、排序、求平均数的操作， 今天我们通过搜狗提供的数据来做一个去重的操作。

<!--more-->

搜狗实验室 [数据来源] (http://www.sogou.com/labs/resource/q.php)

这里我使用的是精简版的（一天的数据）


下载好的之后, 默认sougou的文件编码是gbk格式的， 所以在linux下打开是乱码， 所以需要先将文件转化为utf-8格式的。

转化命令：


 ```sh
cat SogouQ.reduced | iconv -f gbk -t utf-8 -c | > corpus.txt
 ```
其中SogouQ.reduced是我们刚下载的sougou的文件，   转化后的文件是corpus.txt

转化好的文件的打开之后的数据格式：

访问时间\t 用户ID\t[查询词]\t该URL在返回结果中的排名\t用户点击的顺序号\t用户点击的URL

对于用户ID是根据用户使用浏览器访问Cookie信息自动赋的值。

样例：

![](http://ww1.sinaimg.cn/large/62ca154dly1fj2obgdgadj20ui0gsjzu.jpg)

数据去重： 每一个用户ID对于同一个搜索的关键词可能会点击点击几次，  我们的需求是对同一个用户ID的同一个关键词重复的条数去重。


###新建MapReduce工程

#####1. 新建MapReduce工程， SougouDemo

#####2. 将hadoop中的log4j.properties文件放到工程的src文件夹下

#####3. 新建CleanSameData.java类

源码：


```java
package com.au.example;
import java.io.IOException;  
import java.util.StringTokenizer;  
  
import org.apache.hadoop.conf.Configuration;  
import org.apache.hadoop.fs.Path;  
import org.apache.hadoop.io.Text;  
import org.apache.hadoop.mapreduce.Job;  
import org.apache.hadoop.mapreduce.Mapper;  
import org.apache.hadoop.mapreduce.Reducer;  
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;  
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;  
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;  
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;  
import org.apache.hadoop.util.GenericOptionsParser;  

public class CleanSameData {

	
	 // map将输入中的value复制到输出数据的key上，并直接输出  
    public static class Map extends Mapper<Object, Text, Text, Text> {  
  
        // 实现map函数  
        @Override  
        public void map(Object key, Text value, Context context) throws IOException, InterruptedException {  
            // 将输入的纯文本文件的数据转化成String  
            String line = value.toString();  
            // 将输入的数据首先按行进行分割  
            StringTokenizer tokenizerArticle = new StringTokenizer(line, "\n");  
            // 分别对每一行进行处理  
            while (tokenizerArticle.hasMoreElements()) {  
                // 每行按空格划分  
                StringTokenizer tokenizerLine = new StringTokenizer(tokenizerArticle.nextToken());  
                String c1 = tokenizerLine.nextToken();// time 
                String c2 = tokenizerLine.nextToken();//用户id  
                String c3 = tokenizerLine.nextToken(); //查询词
                c3 = c3.substring(1, c3.length() - 1);  
                Text newline = new Text(c1 + "    " + c2 + "    " +c3);  
                context.write(newline, new Text(""));  
            }  
  
        }  
  
    }  
	
 	 // reduce将输入中的key复制到输出数据的key上，并直接输出  
    public static class Reduce extends Reducer<Text, Text, Text, Text> {  
        // 实现reduce函数  
        @Override  
        public void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {  
            context.write(key, new Text(""));  
        }  
    }  
    
    
    
    
	public static void main(String[] args) throws Exception {
		
		Configuration conf = new Configuration();  
        // 设置输入输出文件目录  
        String[] ioArgs = new String[] { "hdfs://localhost:9000/data_in", "hdfs://localhost:9000/clean_same_out" };  
        String[] otherArgs = new GenericOptionsParser(conf, ioArgs).getRemainingArgs();  
        if (otherArgs.length != 2) {  
            System.err.println("Usage:  <in> <out>");  
            System.exit(2);  
        }  
        // 设置一个job  
        Job job = Job.getInstance(conf, "clean same data");  
        job.setJarByClass(CleanSameData.class);  
  
        // 设置Map、Combine和Reduce处理类  
        job.setMapperClass(Map.class);  
        job.setCombinerClass(Reduce.class);  
        job.setReducerClass(Reduce.class);  
  
        // 设置输出类型  
        job.setOutputKeyClass(Text.class);  
        job.setOutputValueClass(Text.class);  
  
        // 将输入的数据集分割成小数据块splites，提供一个RecordReder的实现  
        job.setInputFormatClass(TextInputFormat.class);  
  
        // 提供一个RecordWriter的实现，负责数据输出  
        job.setOutputFormatClass(TextOutputFormat.class);  
  
        // 设置输入和输出目录  
        FileInputFormat.addInputPath(job, new Path(otherArgs[0]));  
        FileOutputFormat.setOutputPath(job, new Path(otherArgs[1]));  
        System.exit(job.waitForCompletion(true) ? 0 : 1);  

	}

}

```

#####4. 在hadoop中，新建data_in文件夹,并将下载搜狗数据解压后的文件，上传到hadoop中

 ```sh
hdfs dfs -mkdir /data_in
hdfs dfs -put corpus.txt /data_in
 ```

####5. 执行程序后， 查看结果

![](http://ww1.sinaimg.cn/large/62ca154dly1fj2ok1rnzyj20kc0m243c.jpg)

