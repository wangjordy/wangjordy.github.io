---
layout: post
title: "Swift基础语法(一)"
date: 2015-04-10 17:55:42 +0800
author: "Jordy"
comments: true
categories: 
- swift
---

Swift语言是苹果公司在2014年WWDC上公开的新编程语言，Swift语言有可能就是用于替代Objective-C语言成为IOS的首选语言。

###Swift的特点
 * 易上手， 相比Objective-C，swift语法简单，入门的门槛较低
 * 兼容性， swift无缝链接Cocoa和Cocoa Touch，现在Apple的框架接口中，已经提供了大量的swift版本，还有一部分没有提供的接口，也可以通过调用Objective-C的接口来实现；
 * 运行效率， swift语言所有的的代码都是使用LLVM编译为机器语言，克服了Objective与C语言兼容性的问题， 根据苹果提供的数据，swift语言编写的程序运行效率，比Python快3.9倍，比Objective-C快1.4倍。 （备注：当然测试的运行效率，与硬件设备、还有执行什么样的任务都有很大关系，具体的效率还是自己编写相同的程序测试一下吧！）
 * 运行时（Runtime） 这点Objective-C也有，不再多述；
 * 可混编，swift语言将声明和实现都放在一个单元文件中，不用单独编写头文件和实现文件， swift还支持可以Objective-C混编于同一个工程中；


