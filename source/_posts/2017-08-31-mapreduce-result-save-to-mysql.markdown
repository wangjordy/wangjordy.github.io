---
layout: post
title: "MapReduce实战练习: MapReduce计算结果保存到Mysql数据库（三）"
date: 2017-08-31 10:25:04 +0800
comments: true
categories: 
- hadoop 
- hadoop code 
---

书接上文，上篇我们介绍了hadoop获取了排名前100的热搜词， 这里我们将介绍关于计算结果保存到Mysql数据库中。

<!--more-->

上一篇，我们计算出来排名前100的热搜词为：
![](http://ww1.sinaimg.cn/large/62ca154dly1fj2orrtyhgj20eu0h8mzp.jpg)

####新建数据库

#####1.新建sougou的数据库“sougouinfo”

```sh

create database sougouinfo;
use sougouinfo;

```
#####2.新建sougou的数据表“key_word”

```sh

CREATE TABLE `key_word` (  
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,  
  `word` varchar(255),  
  `total` bigint(20),  
  PRIMARY KEY (`id`)  
) ENGINE=InnoDB  DEFAULT CHARSET=utf8;

```

####相关的代码


源码


```java
package com.au.example;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.io.Writable;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.db.DBConfiguration;
import org.apache.hadoop.mapreduce.lib.db.DBOutputFormat;
import org.apache.hadoop.mapreduce.lib.db.DBWritable;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

public class SaveToMysql {

	public static class TblsWritable implements Writable, DBWritable {

		String tbl_name;
		int tbl_age;

		public TblsWritable() {

		}

		public TblsWritable(String name, int age) {
			this.tbl_name = name;
			this.tbl_age = age;
		}

		@Override
		public void readFields(ResultSet resultset) throws SQLException {

			this.tbl_name = resultset.getString(1);
			this.tbl_age = resultset.getInt(2);
		}

		@Override
		public void write(PreparedStatement statement) throws SQLException {

			statement.setString(1, this.tbl_name);
			statement.setInt(2, this.tbl_age);
		}

		@Override
		public void readFields(DataInput dataInput) throws IOException {

			this.tbl_name = dataInput.readUTF();
			this.tbl_age = dataInput.readInt();
		}

		@Override
		public void write(DataOutput dataOutput) throws IOException {

			dataOutput.writeUTF(tbl_name);
			dataOutput.writeInt(tbl_age);
		}

		public String toString() {
			return new String(this.tbl_name + " " + this.tbl_age);
		}

	}
	
	
	public static class StudentMapper extends Mapper <LongWritable, Text, LongWritable, Text>{
		
		protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
			context.write(key, value);
		}
	}
	
	
	public static class StudentReducer extends Reducer <LongWritable, Text, TblsWritable, TblsWritable> {
		
		protected void reduce(LongWritable key, Iterable<Text> values, Context context) throws IOException, InterruptedException{
			
			StringBuilder value = new StringBuilder();
			for (Text text : values) {
				value.append(text);
			}
			
			String [] studentArr = value.toString().split("\t");
			
			
			if (studentArr[0] != null) {
				String name = studentArr[1].trim();
				
				int age = 0;
				try {
					age = Integer.parseInt(studentArr[0].trim());
				}catch(NumberFormatException e) {
					
				}
				
				context.write(new TblsWritable(name, age), null);
			}
			
		}
		
	}
	

	public static void main(String[] args) throws Exception {

		Configuration conf = new Configuration();
		
		DBConfiguration.configureDB(conf, "com.mysql.jdbc.Driver", "jdbc:mysql://localhost:3306/sougouinfo?characterEncoding=utf8&amp;useSSL=false", "root", "root");
		
		//设置输入输出文件目录 
		String [] ioArgs = new String[] {"hdfs://localhost:9000/top_out"};
		
		String [] otherArgs = new GenericOptionsParser(conf, ioArgs).getRemainingArgs();
		
		if (otherArgs.length != 1) {
			System.err.println("Usage: <in><out>");
			System.exit(2);
		}
		
		//设置一个job 
		Job job = Job.getInstance(conf, "SaveResult");
		job.setJarByClass(SaveToMysql.class);
		
		// 输入路径
		FileInputFormat.addInputPath(job, new Path(otherArgs[0]));
		
		//Mapper  
		job.setMapperClass(StudentMapper.class);
		// Reducer
		job.setReducerClass(StudentReducer.class);
		
		// mapper输出格式
		job.setOutputKeyClass(LongWritable.class);
		job.setOutputValueClass(Text.class);
		
		// 输入格式，默认就是TextInputFormat  
		job.setOutputFormatClass(DBOutputFormat.class);
		
		// 输出到哪些表、字段 
		DBOutputFormat.setOutput(job, "key_word", "word", "total");  
		System.exit(job.waitForCompletion(true) ? 0 : 1);  
		
	}

}


```

在项目中添加mysql-connector-java-5.1.14-bin.jar


执行后的结果，如图：

![](http://ww1.sinaimg.cn/large/62ca154dly1fj2xjtdbw7j20eg0jvaej.jpg)



