---
layout: post
title: "修复jupyter notebook的问题"
date: 2017-07-26 17:41:00 +0800
comments: true
categories: 
- Machinelearning
---

前几天刚升级了Mac OS系统之后， 今天运行Anaconda中的jupyter notebook, 结果在控制台打印出来

```sh
execution error: “"http://localhost:8888/tree?token=.....
```

注：token后面东东我没写上去。

<!--more-->

平时都是直接点击

![](http://ww1.sinaimg.cn/large/62ca154dly1fhxfh76ibej207u0aut94.jpg)

图上的Launch按钮，会直接在浏览器中打开netbook来，  这次浏览器没有反应，

手动在浏览器中输入 http://localhost:8888

![](http://ww1.sinaimg.cn/large/62ca154dly1fhxfjel2i9j20yg09waa2.jpg)

提示让输入密码， 什么鬼，从来不知道这个密码， 尝试了一番密码后还是不行，  终于求救于Google大神

####解决方法

1、在jupyter的配置目录下

```sh
~/.jupyter
```

2、新建一个文件为： jupyter_notebook_config.py，  在这个文件中加入代码

```sh
c.NotebookApp.browser = u'Safari'
c.NotebookApp.token = ''
c.NotebookApp.password = ''
```

3、保存退出后， 执行命令

```sh
jupyter notebook
```

好了，又可以愉（虐）快（心）地玩耍机器学习了。