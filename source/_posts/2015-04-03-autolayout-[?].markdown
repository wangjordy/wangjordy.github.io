---
layout: post
title: "AutoLayout教程 (一)"
author: "Jordy"
date: 2015-04-03 13:41:04 +0800
comments: true
categories: 
- ios
- Objective-C
---

在iphone6， iphone6 plus出现之前，iphone设备的宽度是320，高度稍有不同， 一般我们简单的做法是使用AutoResizing
自动拉伸高度即可：

如图：

![AutoResizing](http://ww4.sinaimg.cn/large/62ca154djw1eqslg6yyt4j20im07r75q.jpg)

```
 •	UIViewAutoresizingNone view的frame不会随superview的改变而改变
 •	UIViewAutoresizingFlexibleLeftMargin 自动调整view与superview左边的距离保证右边距离不变
 •	UIViewAutoresizingFlexibleWidth 自动调整view的宽，保证与superView的左右边距不变
 •	UIViewAutoresizingFlexibleRightMargin 自动调整view与superview右边的距离保证左边距不变
 •	UIViewAutoresizingFlexibleTopMargin 自动调整view与superview顶部的距离保证底部距离不变
 •	UIViewAutoresizingFlexibleHeight 自动调整view的高，保证与superView的顶部和底部距离不变
 •	UIViewAutoresizingFlexibleBottomMargin 自动调整view与superview底部部的距离保证顶部距离不变

```
由于AutoResizing很难能做视图与视图之间的间距，比如下图：

![AutoResizing](http://ww4.sinaimg.cn/large/62ca154djw1eqsllh0pkvj20an0gv0tk.jpg)

我们想约束左侧红色View与右侧橘黄色View保持一个固定像素（比方说是10px，RedView与OrangeView随屏幕的宽度不同而不同），
这样使用使用AutoResizing显然就不合适了。

接下来我们开始正式进入AutoLayout：

首先，打开我们的Main.stroyboard文件， 先介绍一下常用的视图窗：

在右下角的位置，
第一个icon按钮，打开后的功能描述
![AutoResizing](http://ww1.sinaimg.cn/large/62ca154djw1eqslpysiljj20p00d2ae7.jpg)


第二个icon按钮，打开后的功能描述
![AutoResizing](http://ww1.sinaimg.cn/large/62ca154djw1eqslsg9q8hj20p00eagol.jpg)

第三个icon按钮是用于刷新我们的视图，第4个icon我们暂时用不到，所以这里就不在切图描述了

##AutoLayout的基本使用

1、拖拽一个UIView到ViewController的视图中， 如下样式： （要求，这个view居顶部10px，居左20px，宽度200px，高度100px）

![AutoResizing](http://ww2.sinaimg.cn/large/62ca154djw1er0b25kgmzj20gm0hmt8s.jpg)
添加约束的方式：
![AutoResizing](http://ww4.sinaimg.cn/large/62ca154djw1er0b3bfy9gj207e0a6wfe.jpg)

上边方格内，红色“I”选中表示添加了这个约束，中间的width和height前面的方格打钩表示添加了宽度和高度的约束；
最后点击底部的“Add 4 Constraints”按钮， 点击之后的效果，如下图：

备注： Update Frames 选择“Items of New Constraints” 表示新添加的约束，会自动刷新当前屏幕上view显示的样式和位置；

添加之后的样式，如下图：
![AutoResizing](http://ww4.sinaimg.cn/large/62ca154djw1er0ba7ng21j20gt0hoglr.jpg)

出现了蓝色的边线表示那条约束显示是正确的， 如果，我们只添加了约束，没有设置Update Frames的选项，显示如下图：

![AutoResizing](http://ww4.sinaimg.cn/large/62ca154djw1er0euq1pnfj208u09vjrj.jpg)

图中虚线表示的是添加完约束后，应该显示样子（只是样子还没有在屏幕上刷新而已！），刷新的方式，可以点击（右下角的第三个图标）

![AutoResizing](http://ww1.sinaimg.cn/large/62ca154djw1er0f2j3rjzj208q06kt99.jpg)

或者是快捷键，刷新样式即可！

并且点击右侧菜单栏（尺子的icon）
![AutoResizing](http://ww4.sinaimg.cn/large/62ca154djw1er0f57tgufj206w0dhab0.jpg)
可以看到对应的约束。

如果我们添加的约束不正确，则会出现以下状况（例如，我把刚刚View的宽度约束删除后），如下图：

![AutoResizing](http://ww3.sinaimg.cn/large/62ca154djw1er0f8u38pgj20r10900uk.jpg)

当然，点击这个这个红色箭头，里面会告诉你缺失的约束是什么， 或者建议是什么。
如果我们重复添加约束，或者约束过度，这里也会提示为黄色的警告约束。

如果我们想设置一个宽度与屏幕宽度成一定比例的UIView(比方说这个view的宽度是屏幕的一半)，并且高度与宽度也是一个等比例的关系（比方说宽高比是2：1）时， 

第一步：先添加以下约束（同时选中这个view与superview）：如下图：

![AutoResizing](http://ww2.sinaimg.cn/large/62ca154djw1er0fivcsbkj207d0a10th.jpg)

第二步：修改宽度的比例：

![AutoResizing](http://ww3.sinaimg.cn/large/62ca154djw1er0fsu2c13j207f07z0ta.jpg)

设置宽度的比例为0.5，（如果我们设置宽度比一半多，还可以修改上边的constant常数即可）；

这里FisrtItem是superview，SecondItem是我们的View， 那么Multiplier参数自然是2，表示superview是view的2倍
关于FistItem与SecondItem会在讲解代码部分详细提到。

同理我们修改Ratio为2：1即可

此时，我们只给这个view添加了width和height的约束，还缺少x，y的坐标约束，比方设置这个view，与superview居中显示，那么添加
约束，如：

![AutoResizing](http://ww4.sinaimg.cn/large/62ca154djw1er0fwu388bj208c0810tf.jpg)

如果我们的视图是居中偏左，或者偏右（或者偏上、偏下），都可以修改绿色框内，添加偏移量即可！

以上是关于UIView添加约束的实现， 同理，自己可以拖拽一个UILabel、UIButton添加约束试试
（这里需要说明的一点，由于UILabel、UIButton可以根据title来自适应宽度、高度，也就是内置的contentsize）所以只需要添加UILabel、UIButton的x，y坐标即可！

##Size Classes的简介

早期我们创建一个项目选择Devices为Universal时， 默认项目中会有一个“_iphone.xib”, “_ipad.xib”（后期加入storyboard后，是建2个storyboard）

随着iphone6，iphone6 plus 以及Apple watch的出现， 苹果也是为了将这些视图统一起来，于是ios8中加入Size Classes的概念，

Size Classes就是对设备尺寸的一个抽象概念， 现在的通常的设备都可以简单看成为： 普通（Regular）、 紧密（Compact）和任意（Any），
于是这样就会出现 3*3的一个矩阵形式（一共是9种模式），如下图：


![AutoResizing](http://ww1.sinaimg.cn/large/62ca154djw1er0gsjucltj20ld0acq3r.jpg)

现在默认新建的项目都已经开启了Size Classes和AutoLayout功能了，默认我们创建的是在w:Any h:Any

![AutoResizing](http://ww3.sinaimg.cn/large/62ca154djw1er0gwd1xmhj20ir0n8wf0.jpg)

点击红色区域，就会弹出一个9宫格，鼠标在9宫格里移动，会看到不同的宽度和高度的选择，也就是（Regular、Compact、Any）
例如，你要适配所有iphone的设备，可以选中，

![AutoResizing](http://ww4.sinaimg.cn/large/62ca154djw1er0h0mgf02j206j08hglq.jpg)

由于ipad模式下，宽度和高度都是Regular模式，所以，选择宽高都是Regular或者Any都行。

Any表示所有设备下都会install到， 比如，我们选择了iphone模式下开发（即W：Compact，H:Regular），此时又想添加其他的模式也能显示此视图，只需要在下图点击左侧的加号，添加W：H：的模式，并选中installed。

![AutoResizing](http://ww2.sinaimg.cn/large/62ca154djw1er0h3e04j7j20780aeaar.jpg)


关于AutoLayout的代码介绍和高级用法，我将在之后推出！
