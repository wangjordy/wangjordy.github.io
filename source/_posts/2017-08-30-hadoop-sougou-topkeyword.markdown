---
layout: post
title: "MapReduce实战练习: 使用搜狗实验数据对搜词热度排序（二）"
date: 2017-08-30 18:04:26 +0800
comments: true
categories: 
- hadoop 
- hadoop code 
---

我们接着上一篇《[MapReduce实战练习: 使用搜狗实验数据对数据去重](http://wangjordy.github.io/blog/2017/08/30/hadoop-sougou-search-data-example/)》 接着讲解关于对keyword（热搜词）的top100计算

<!--more-->

###第一步，我们得到每个词的搜索次数

我们将上篇项目中的输出文件，作为本篇的输入源

![](http://ww1.sinaimg.cn/large/62ca154dly1fj2ok1rnzyj20kc0m243c.jpg)

#####计算每个关键词出现的次数，新建 KeyWordCount.java

源码


```java

package com.au.example;

import java.io.IOException;  
import java.util.StringTokenizer;  
  
import org.apache.hadoop.conf.Configuration;  
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

public class KeyWordCount {

	
	public static class Map extends Mapper<LongWritable, Text, Text, IntWritable> {  
        private final static IntWritable one = new IntWritable(1);  
          
        @Override  
        public void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {  
            // 将输入的纯文本文件的数据转化成String  
            String line = value.toString();  
            // 将输入的数据首先按行进行分割  
            StringTokenizer tokenizerArticle = new StringTokenizer(line, "\n");  
            // 分别对每一行进行处理  
            while (tokenizerArticle.hasMoreElements()) {  
                // 每行按空格划分  
                StringTokenizer tokenizerLine = new StringTokenizer(tokenizerArticle.nextToken());  
                String c1 = tokenizerLine.nextToken();//  
                String c2 = tokenizerLine.nextToken();// 关键词  
                String c3 = tokenizerLine.nextToken();// 关键词  
                Text newline = new Text(c3);  
                context.write(newline, one);  
            }  
        }  
    }
	
	
	public static class Reduce extends Reducer<Text, IntWritable, Text, IntWritable> {  
        private IntWritable result =new IntWritable();  
          
        // 实现reduce函数  
        @Override  
        public void reduce(Text key, Iterable<IntWritable> values,Context context) throws IOException, InterruptedException {  
            int count = 0;  
             for(IntWritable val:values){  
                  count += val.get();  
             }  
            result.set(count);  
            context.write(key, result);  
        }  
    }  
	
	
	
	
	
	
	public static void main(String[] args) throws Exception {
		Configuration conf = new Configuration();  
        //设置输入输出文件目录  
        String[] ioArgs = new String[] { "hdfs://localhost:9000/clean_same_out", "hdfs://localhost:9000/Key_out" };  
        String[] otherArgs = new GenericOptionsParser(conf, ioArgs).getRemainingArgs();  
        if (otherArgs.length != 2) {  
            System.err.println("Usage:  <in> <out>");  
            System.exit(2);  
        }  
        //设置一个job  
        Job job = Job.getInstance(conf, "key Word count");  
        job.setJarByClass(KeyWordCount.class);  
          
        // 设置Map、Combine和Reduce处理类  
        job.setMapperClass(KeyWordCount.Map.class);  
        job.setCombinerClass(KeyWordCount.Reduce.class);  
        job.setReducerClass(KeyWordCount.Reduce.class);  
          
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

输出的数据文件，如下图：

![](http://ww1.sinaimg.cn/large/62ca154dly1fj2oozw7lgj20k30iyn04.jpg)



###第二步，根据统计每一词的count，按照做从小到大的排序， 前100的热搜词


#####计算每个关键词出现的次数，新建 KeyWordCount.java

源码


```java
package com.autohome.example;

import java.io.IOException;
import java.util.TreeMap;

import org.apache.hadoop.conf.Configuration;
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

public class TopKeyword {

	public static final int K = 100;

	public static class KMap extends Mapper<LongWritable, Text, IntWritable, Text> {  
        
        TreeMap<Integer, String> map = new TreeMap<Integer, String>();   
          
        @Override  
        public void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {  
            String line = value.toString();  
            if(line.trim().length() > 0 && line.indexOf("\t") != -1) {  
                String[] arr = line.split("\t", 2);  
                String name = arr[0];  
                Integer num = Integer.parseInt(arr[1]);  
                map.put(num, name);  
                if(map.size() > K) {  
                    map.remove(map.firstKey());  
                }  
            }  
        }  
        
        @Override  
        protected void cleanup(Mapper<LongWritable, Text, IntWritable, Text>.Context context) throws IOException, InterruptedException {  
            for(Integer num : map.keySet()) {  
                context.write(new IntWritable(num), new Text(map.get(num)));  
            }  
        }  
        
    }  
        
	
	public static class KReduce extends Reducer<IntWritable, Text, IntWritable, Text> {  
        TreeMap<Integer, String> map = new TreeMap<Integer, String>();  
          
        @Override  
        public void reduce(IntWritable key, Iterable<Text> values, Context context) throws IOException, InterruptedException {  
            map.put(key.get(), values.iterator().next().toString());  
            if(map.size() > K) {  
                map.remove(map.firstKey());  
            }  
        }  
  
        @Override  
        protected void cleanup(Reducer<IntWritable, Text, IntWritable, Text>.Context context) throws IOException, InterruptedException {  
            for(Integer num : map.keySet()) {  
                context.write(new IntWritable(num), new Text(map.get(num)));  
            }  
        }  
    }  
	
	
	
	
	public static void main(String[] args) throws Exception {
		
		Configuration conf = new Configuration();  
        //设置输入输出文件目录  
        String[] ioArgs = new String[] { "hdfs://localhost:9000/Key_out", "hdfs://localhost:9000/top_out" };  
        String[] otherArgs = new GenericOptionsParser(conf, ioArgs).getRemainingArgs();  
        if (otherArgs.length != 2) {  
            System.err.println("Usage:  <in> <out>");  
            System.exit(2);  
        }  
        //设置一个job  
        Job job = Job.getInstance(conf, "top Keyword");  
          
        job.setJarByClass(TopKeyword.class);  
          
        // 设置Map、Combine和Reduce处理类  
        job.setMapperClass(KMap.class);  
        job.setCombinerClass(KReduce.class);  
        job.setReducerClass(KReduce.class);  
          
        // 设置输出类型  
        job.setOutputKeyClass(IntWritable.class);  
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

输出结果为：

![](http://ww1.sinaimg.cn/large/62ca154dly1fj2orrtyhgj20eu0h8mzp.jpg)

我们可以看到前面是搜索的次数， 后面是搜索的关键词，  按照搜索的热度，取最高的100名， 按从小到大的顺序。

