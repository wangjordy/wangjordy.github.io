---
layout: post
title: "ios使用Facebook的Infer工具"
date: 2017-05-23 16:41:51 +0800
comments: true
categories: 
- ios
- facebook
- Objective-C
---

上一节我们提到了Facebook的一个关于检测内存泄漏的重要工具是`FBRetainCycleDetector`， 今天我们继续介绍Facebook的另一个重要工具  <font color=#008B8B size=4>Infter</font>  <br/>

<font color=#008B8B size=4>Infter</font>是一个静态分析工具， 它可以分析的编程语言有： Objective-C， Java还有C，

通过它对代码的静态分析，来报告潜在的问题。

<!--more-->

![](http://ww1.sinaimg.cn/large/62ca154dly1ffveblwpmzj20pv0c9mxe.jpg)

关于Infer的介绍可以参考官网 [Infer官网](https://infer.liaohuqiu.net/)


##Infer的优势

 * 静态扫描效率很高
 * 支持增量分析和全量分析
 * 也可以分解分析局部范围，再整合为统一结果输出
 * 在ios和C中，可以帮助我们排查内存泄露

###Infer捕获的ios中的bug类型

 * 循环引用
 * 参数不为null的校验
 * Ivar不为null的校验

###Infer安装教程


安装Infer的方法常用的有2种：

第一种， 通过[Infer github代码](https://github.com/facebook/infer) ，下载里面的release的最新版本，然后安装命令方式安装；

第二种，是我比较推荐的方式，通过homebrew的方式安装；


这里重点介绍第二种方式:

###安装infer

```sh
brew install infer
```
安装好之后，可以通过`info --version`校验是否安装成功；

###配置到环境变量中

```sh
echo "export PATH=\"\$PATH:pwd/infer/infer/bin\"" \ >> ~/.bash_profile &&source ~/.bash_profile 
```

###安装好Infer工具之后， 下载Demo代码

facebook已经为我们提供好了测试的Demo， 下载地址[github代码](https://github.com/facebook/infer)

```sh
git clone https://github.com/facebook/infer.git
```

下载好之后，进入工程的examples文件夹中， 

###输入以下命令

```sh
infer -- clang -c Hello.m
```
会提示给我们Hello.m这个文件中的那些语法有问题，  详细如下图：

![](http://ww1.sinaimg.cn/large/62ca154dly1ffvf4p24yzj20se0eyjtf.jpg)
 
我们可以看到在代码中 `return  hello->s`;  执行这行代码之前，hello已经设置为nil了

修改的方法设置改为 

```obj-c
return hello.s;
```

同时代码中也强调声明s变量时，应该用strong或者weak的方式， 因为默认修饰关键字是assign

我们还需要修改为：

```obj-c
@property (nonatomic, strong) NSString* s;
```
###对工程的分析

进入上述demo中的ios_hello文件夹中， 里面有HelloWorldApp，然后编译执行

```sh
cd ios_hello
infer -- xcodebuild -target HelloWorldApp -configuration Debug -sdk iphonesimulator
```
如果此时，终端提示如下错误：

![](http://ww1.sinaimg.cn/large/62ca154dly1ffvfheamfsj20nb042wf2.jpg)

提示没有按照xcpretty， 可通过以下命令安装

```sh
gem install xcpretty
```
安装好之后，重新执行上一条命令，就会在终端提示出这个工程中分析的结果，如图：

![](http://ww1.sinaimg.cn/large/62ca154dly1ffvfk3m100j20um0qsjwa.jpg)

在一堆错误的最底部会有一个综合整理报告： 如图

![](http://ww1.sinaimg.cn/large/62ca154dly1ffvfl7w8xlj20c505amxg.jpg)

此时，我们可以根据上述提示的问题， 找到具体的行去做具体的修复。 但是，且慢着急修复bug， 我们还需要接着这个先讲下去。

* 我们在控制台已经看到了这些错误信息，但其实这个看起来还是不太方便， 其实我们发现在原工程中多了`build`和`infer-out`2个文件夹

  ![](http://ww1.sinaimg.cn/large/62ca154dly1ffvfpbjiilj20bg066aao.jpg)
  
* build文件夹： <font color=#FF7F50>捕获阶段</font>    Infer 捕获编译命令(上面介绍的编译器命令)，将文件翻译成 Infer 内部的中间语言

* infer-out文件夹： <font color=#FF7F50>分析阶段</font>   Infer将分析bugs结果输出到不同格式文件中，如csv、txt、json 方便对分析结果进行加工分析

好了，介绍完这些后， 我们还没有修复一个这个工程里的bug；

接着我们再重复执行命令:

```sh
infer -- xcodebuild -target HelloWorldApp -configuration Debug -sdk iphonesimulator
```
这时候，我们发现控制台居然不报错了，如图：

![](http://ww1.sinaimg.cn/large/62ca154dly1ffvfui48j3j20on07x3zn.jpg)

这是因为默认infer工具，采用的是增量更新，对于之前已经提示出错的信息，则不再报告了

###增量更新->非增量更新

执行命令

```sh
xcodebuild -target HelloWorldApp -configuration Debug -sdk iphonesimulator clean
```
然后再执行之前命令，就可以看到错误信息了。

好了， 以上就是关于facebook的infer工具的简单介绍， 详细还是看看上面写的文档吧。
