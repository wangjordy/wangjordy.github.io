---
layout: post
title: "Eclipse 实现关于MapReduce的实例： 排序、求平均数"
date: 2017-08-30 14:18:05 +0800
comments: true
categories:
- hadoop 
- hadoop code 
---


前端时间我们介绍过在eclipse里运行hadoop自带的关于wordcount的实例， 今天我们在原来的基础上继续介绍关于一些MapReduce的代码编程案例。

<!--more-->

在开始工程之前， 请先确保你的DFS Locations 已经连接到Hadoop， 如果没有配置的，可以出门左转反馈我之前的介绍

###在Eclipse中创建MapReduce项目

点击File菜单， 选择New->Project...，  在新打开的窗体里选择 Map/Reduce Project, 点击Next， 按照提示默认的配置，

新建工程名为<font color=#2F4F4F>WordCountDemo</font>， 创建好之后，点击Finish。


###在刚刚新建的工程里，新建 Sort.java 类

源码：


```java
package com.au.example;

import java.io.IOException;

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

public class Sort {

	// map将输入中的value化成IntWritable类型，作为输出的key
	public static class Map extends Mapper<Object, Text, IntWritable, IntWritable> {
		private static IntWritable data = new IntWritable();

		// 实现map函数
		public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
			String line = value.toString();
			data.set(Integer.parseInt(line));
			context.write(data, new IntWritable(1));
		}

	}

	// reduce将输入中的key复制到输出数据的key上，
	// 然后根据输入的value-list中元素的个数决定key的输出次数
	// 用全局linenum来代表key的位次
	public static class Reduce extends Reducer<IntWritable, IntWritable, IntWritable, IntWritable> {

		private static IntWritable linenum = new IntWritable(1);

		// 实现reduce函数
		public void reduce(IntWritable key, Iterable<IntWritable> values, Context context)
				throws IOException, InterruptedException {
			for (IntWritable val : values) {
				context.write(linenum, key);
				linenum = new IntWritable(linenum.get() + 1);
			}

		}

	}

	public static void main(String[] args) throws Exception {

		Configuration conf = new Configuration();
		String[] ioArgs = new String[] { "hdfs://localhost:9000/sort_in", "hdfs://localhost:9000/sort_out" };
		String[] otherArgs = new GenericOptionsParser(conf, ioArgs).getRemainingArgs();
		if (otherArgs.length != 2) {
			System.err.println("Usage: Data Sort <in> <out>");
			System.exit(2);
		}

		Job job = Job.getInstance(conf, "Data Sort");
		job.setJarByClass(Sort.class);

		// 设置Map和Reduce处理类
		job.setMapperClass(Map.class);
		job.setReducerClass(Reduce.class);

		// 设置输出类型
		job.setOutputKeyClass(IntWritable.class);
		job.setOutputValueClass(IntWritable.class);

		// 设置输入和输出目录
		FileInputFormat.addInputPath(job, new Path(otherArgs[0]));
		FileOutputFormat.setOutputPath(job, new Path(otherArgs[1]));
		System.exit(job.waitForCompletion(true) ? 0 : 1);

	}

}

```
创建好之后，我们需要几个输入源数据文件到hadoop的/sort_in文件夹中。

 <font color=#D19275>操作步骤：</font>
 
 * 在hadoop中新建/sort_in文件夹

 
 ```sh
hdfs dfs -mkdir /sort_in
 ```
 
 * 将源数据文件上传到/sort_in文件夹中
 
 这里输入文件为 file1.txt, file2.txt, file3.txt
 
 其中：
 
 file1.txt文件中的数据
 
 ```txt
2
32
654
32
15
756
65223
 ```
 
 file2.txt文件中的数据
 
 ```txt
5956
22
650
92
 ```
 
 file3.txt文件中的数据
 
 ```txt
26
54
6
 ```
<font color=red> 注意： 上述文件中的数据，不要有空格，否则会把数字解析为字符串， 影响程序的执行；</font>

上传命令为

 ```sh
hdfs dfs -put file1.txt /sort_in
hdfs dfs -put file2.txt /sort_in
hdfs dfs -put file3.txt /sort_in
 ```

上传完之后，运行Sort.java， <font color=red> 注意在工程的src目录下添加hadoop中的log4j.properties,否则在java的控制台不会输出执行的日志信息</font>

执行完后，可以在DFS Locations里看到， 点击Refresh看到输入文件夹和输出文件夹的相关文件
![](http://ww1.sinaimg.cn/large/62ca154dly1fj1qye1qjjj20a804f74k.jpg)

并且打开文件看上述的结果为：

![](http://ww1.sinaimg.cn/large/62ca154dly1fj1r127ofaj205s070aa6.jpg)



###关于去重的实例： 新建 DiffData.java 类

源码


```java
package com.au.example;

import java.io.IOException;  

import org.apache.hadoop.conf.Configuration;  
import org.apache.hadoop.fs.Path;  
import org.apache.hadoop.io.IntWritable;  
import org.apache.hadoop.io.Text;  
import org.apache.hadoop.mapred.JobConf;  
import org.apache.hadoop.mapreduce.Job;  
import org.apache.hadoop.mapreduce.Mapper;  
import org.apache.hadoop.mapreduce.Reducer;  
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;  
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;  
import org.apache.hadoop.util.GenericOptionsParser;  

public class DiffData {

	// map将输入中的value复制到输出数据的key上，并直接输出  
    public static class Map extends Mapper<Object, Text, Text, Text> {  
        private static Text line = new Text();// 每行数据  
  
        // 实现map函数  
        public void map(Object key, Text value, Context context)  
                throws IOException, InterruptedException {  
            line = value;  
            context.write(line, new Text(""));  
        }  
  
    }  
	
 // reduce将输入中的key复制到输出数据的key上，并直接输出  
    public static class Reduce extends Reducer<Text, Text, Text, Text> {  
        // 实现reduce函数  
        public void reduce(Text key, Iterable<Text> values, Context context)  
                throws IOException, InterruptedException {  
            context.write(key, new Text(""));  
        }  
    }  

	public static void main(String[] args) throws Exception {
		
		JobConf conf = new JobConf(DiffData.class);  
        conf.setJobName("WordCount");  
        conf.addResource("classpath:/hadoop2/core-site.xml");  
        conf.addResource("classpath:/hadoop2/hdfs-site.xml");  
        conf.addResource("classpath:/hadoop2/mapred-site.xml");  
        conf.addResource("classpath:/hadoop2/yarn-site.xml");  
  
        String[] ioArgs = new String[] { "hdfs://localhost:9000/inputchar", "hdfs://localhost:9000/outputchar" };  
        String[] otherArgs = new GenericOptionsParser(conf, ioArgs).getRemainingArgs();  
        if (otherArgs.length != 2) {  
            System.err.println("Usage: Data Deduplication <in> <out>");  
            System.exit(2);  
        }  
  
        Job job =  Job.getInstance(conf, "Data Deduplication");  
        job.setJarByClass(DiffData.class);  
  
        // 设置Map、Combine和Reduce处理类  
        job.setMapperClass(Map.class);  
        job.setCombinerClass(Reduce.class);  
        job.setReducerClass(Reduce.class);  
  
        // 设置输出类型  
        job.setOutputKeyClass(Text.class);  
        job.setOutputValueClass(Text.class);  
  
        // 设置输入和输出目录  
        FileInputFormat.addInputPath(job, new Path(otherArgs[0]));  
        FileOutputFormat.setOutputPath(job, new Path(otherArgs[1]));  
        System.exit(job.waitForCompletion(true) ? 0 : 1);  

	}

}


```

* 同理，我们按上上述步骤在hadoop新建/inputchar文件夹；
* 上传的数据源文件为file1.txt, file2.txt;

其中，file1.txt文件内容为：

 ```txt
2012-3-1 a
2012-3-2 b
2012-3-3 c
2012-3-4 d
2012-3-5 a
2012-3-6 b
2012-3-7 c
2012-3-3 c
 ```
其中，file2.txt文件内容为：

 ```txt
2012-3-1 b
2012-3-2 a
2012-3-3 b
2012-3-4 d
2012-3-5 a
2012-3-6 c
2012-3-7 d
2012-3-3 c
 ```
* 最后执行的结果为：

![](http://ww1.sinaimg.cn/large/62ca154dly1fj1r7s39q9j204a05lt8s.jpg)

###关于求平均数的实例： 新建 Average.java 类

源码


```java
package com.autohome.example;

import java.io.IOException;  
import java.util.Iterator;  
import java.util.StringTokenizer;  
  
import org.apache.hadoop.conf.Configuration;  
import org.apache.hadoop.fs.FileSystem;  
import org.apache.hadoop.fs.Path;  
import org.apache.hadoop.io.IntWritable;  
import org.apache.hadoop.io.LongWritable;  
import org.apache.hadoop.io.Text;  
import org.apache.hadoop.mapreduce.Job;  
import org.apache.hadoop.mapreduce.Mapper;  
import org.apache.hadoop.mapreduce.Reducer;  
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;  
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;  
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;  
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;  
import org.apache.hadoop.util.GenericOptionsParser; 

public class Average {

	
	public static class Map extends Mapper<LongWritable, Text, Text, IntWritable> {  
        // 实现map函数  
        public void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {  
            // 将输入的纯文本文件的数据转化成String  
            String line = value.toString();  
            // 将输入的数据首先按行进行分割  
            StringTokenizer tokenizerArticle = new StringTokenizer(line, "\n");  
            // 分别对每一行进行处理  
            while (tokenizerArticle.hasMoreElements()) {  
                // 每行按空格划分  
                StringTokenizer tokenizerLine = new StringTokenizer(tokenizerArticle.nextToken());  
                String strName = tokenizerLine.nextToken();// 学生姓名部分  
                String strScore = tokenizerLine.nextToken();// 成绩部分  
                Text name = new Text(strName);  
                int scoreInt = Integer.parseInt(strScore);  
                // 输出姓名和成绩  
                context.write(name, new IntWritable(scoreInt));  
            }  
        }  
  
    }  
	
	public static class Reduce extends Reducer<Text, IntWritable, Text, IntWritable> {  
        // 实现reduce函数  
        public void reduce(Text key, Iterable<IntWritable> values,Context context) throws IOException, InterruptedException {  
            int sum = 0;  
            int count = 0;  
            Iterator<IntWritable> iterator = values.iterator();  
            while (iterator.hasNext()) {  
                sum += iterator.next().get();// 计算总分  
                count++;// 统计总的科目数  
            }  
            int average = (int) sum / count;// 计算平均成绩  
            context.write(key, new IntWritable(average));  
        }  
    }  
	
	
	
	public static void main(String[] args) throws Exception{
		
		Configuration conf = new Configuration();  
        //设置输入输出文件目录  
        String[] ioArgs = new String[] { "hdfs://localhost:9000/average_in", "hdfs://localhost:9000/average_out" };  
        String[] otherArgs = new GenericOptionsParser(conf, ioArgs).getRemainingArgs();  
        if (otherArgs.length != 2) {  
            System.err.println("Usage: Score Average <in> <out>");  
            System.exit(2);  
        }  
        //设置一个job  
        Job job = Job.getInstance(conf, "Score Average");  
          
        job.setJarByClass(Average.class);  
          
        // 设置Map、Combine和Reduce处理类  
        job.setMapperClass(Map.class);  
        job.setCombinerClass(Reduce.class);  
        job.setReducerClass(Reduce.class);  
          
        // 设置输出类型  
        job.setOutputKeyClass(Text.class);  
        job.setOutputValueClass(IntWritable.class);  
          
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

* 同理，我们按上上述步骤在hadoop新建/average_in文件夹；
* 上传的数据源文件为file1.txt, file2.txt, file3.txt;

其中，file1.txt文件内容为：

 ```txt
张三    88
李四    99
王五    66
赵六    77
 ```
其中，file2.txt文件内容为：

 ```txt
张三    78
李四    89
王五    96
赵六    67
 ```
 其中，file3.txt文件内容为：

 ```txt
张三    80
李四    82
王五    84
赵六    86
 ```
 * 最后执行的结果为：

 ![](http://ww1.sinaimg.cn/large/62ca154dly1fj1rcqetv3j203a03hweg.jpg)