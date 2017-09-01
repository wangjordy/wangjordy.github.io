---
layout: post
title: "AutoLayout教程 (二)"
date: 2015-04-10 15:06:35 +0800
comments: true
categories: 
- ios
- Objective-C
---

上一讲在[AutoLayout教程 (一)](http://wangjordy.github.io/blog/2015/04/03/autolayout-%5B%3F%5D/)中，我们简单介绍了一下AutoLayout的基本用法还有Size Classes的一些概念，

今天我们主要从代码的角度，来解说一下AutoLayout以及它里面的属性说明：

先看一段代码：

```ruby
NSLayoutConstraint *constraint = [NSLayoutConstraint constraintWithItem:view1
                                                            attribute:NSLayoutAttributeTop
                                                            relatedBy:NSLayoutRelationEqual
                                                               toItem:self.view
                                                            attribute:NSLayoutAttributeTop
                                                           multiplier:1.0f
                                                             constant:20.0f];
[self.view addConstraint:constraint];

```
通过下图的公式来看：
![haroopad icon](http://ww3.sinaimg.cn/large/62ca154djw1er0hv5k5a1j20fr08adhc.jpg)
A表示我们要添加约束到那个视图上（对于上述代码A=view1）， view1下面的NSLayoutAttributeTop表示
A的顶部约束相对于B视图（toItem： 相对于上述代码B=self.view）的NSLayoutAttributeTop属性（顶部），
距离是20px。m是一个等比例系数，constant表示常数。

我们如果使用代码来添加视图的约束，需要先给视图设置translatesAutoresizingMaskIntoConstraints为NO，
代码为：

```ruby
view1.translatesAutoresizingMaskIntoConstraints = NO;
```
如果我们不添加这句话，AutoLayout是不会起作用的，而使用xib或者StoryBoard默认拖拽上去的视图，这个属性已经自动设置为NO

其他的属性添加方式，这里就不再一一介绍了，

如果我们使用代码同样的方式，创建一个UIView到superview上，需要创建4个NSLayoutConstraint，并且代码行数也很多，
正对这种情况，苹果还推荐使用VFL(Visual format Language)


![haroopad icon](http://ww3.sinaimg.cn/large/62ca154djw1er0i9wcsxgj20gh0353yq.jpg)

解释说明一下：

![haroopad icon](http://ww2.sinaimg.cn/large/62ca154djw1er0iboes1yj20b60bmmyq.jpg)

关于上图format中需要用到的参数解释，如下图：


![haroopad icon](http://ww1.sinaimg.cn/large/62ca154djw1er0idyjowcj20fj094my6.jpg)

具体解释一下：比方说我们要创建一个视图为：（备注，下面的几个图是从其他博客里摘选的）

![haroopad icon](http://ww1.sinaimg.cn/large/62ca154djw1er0ifjlljwj20d404i3yn.jpg)

format参数为： H:|-[view]-|

![haroopad icon](http://ww1.sinaimg.cn/large/62ca154djw1er0ihqazfjj20i707i3yv.jpg)


第一个约束的format参数（横向）为：H:|-[view]-|
第二个约束的format参数（竖向）为：V:|-15-[view(==30)]

![haroopad icon](http://ww2.sinaimg.cn/large/62ca154djw1er0ijztv32j20hh07kgm4.jpg)

这里只列举一下format参数（横向）为：H:|-[view1]-[view2]-[view3(>=50)]-|

具体使用代码的方式，如下图：

![haroopad icon](http://ww3.sinaimg.cn/large/62ca154djw1er0imctrxoj20ou05uaav.jpg)


说明：默认视图距离父视图的边距为20px， 视图与视图之间的间距是8px。 这里也特别说明一下上一节没提到一点，


![haroopad icon](http://ww2.sinaimg.cn/large/62ca154djw1er0ir677c5j206r05eaab.jpg)

如果我们选中了Spacing to nearest neighbor下面的 Constrain to margins 这个框，
就表示系统默认会给view1相对于superview的间距是16（如果我们的在上面的框中添加了左侧的约束是0的情况下），
如果不理解，请看下图，


![haroopad icon](http://ww2.sinaimg.cn/large/62ca154djw1er0iuyu4ppj20zh0bq75g.jpg)

###动态添加删除约束
如果我们要添加一个约束到superview上，可以通过
```ruby
[superview addConstraint: constraint];

```
或者添加一组约束的方式：
```ruby
[superview addConstraints: constraints];
```
如果要删除一个约束，可以通过
removeConstraint方法

最后记得在更改约束的地方调用setNeedsUpdateConstraints即可。

备注说明一点：

layoutIfNeed和setNeedsLayout方法的区别：

其实这2个方法，比较类似于我们之前接触过UIView的重绘调用的
displayIfNeed 和 setNeedsDisplay

其中，setNeedsLayout表示立即执行， 而layoutIfNeed表示等UIKit空闲的时候才会执行。


###AutoLayout的动画

动画一般有2种方式来实现：

 * 移除旧的约束，添加新的约束；
 * 动态修改约束的constant常数；

通常我们使用第二种偏多一些，一般在xib中，将对应的约束声明一个IBOutlet NSLayoutConstraint，然后在需要添加动画的地方，修改
constant值，代码如下：


```ruby
[UIView animateWithDuration:0.3f animations:^{
        [self.commentViewHeightLayoutConstraint setConstant:300];
        [self.view layoutIfNeeded];
    } completion:^(BOOL finished) {

    }];

```
注意：使用NSLayoutConstraint的方式，与frame的方式做动画不同的地方，是需要调用layoutIfNeeded，否则这个动画不会生效，而是直接会设置commentViewHeightLayoutConstraint的constant值为300。


###IOS6与IOS7，8的兼容问题

尽管现在ios6在市场占的份额越来越低了，基本新软件的最低适配都是ios7开始的，但是有一些公司还是要兼容IOS6版本。
对于AutoLayout之前，我们的做法

![haroopad icon](http://ww2.sinaimg.cn/large/62ca154djw1er0jv4j26xj206u0b0q3h.jpg)

动态这个deltas值即可， 如果使用AutoLayout则不在有这种偏移量了，那么如何适配ios6，ios7关于屏幕状态的20px呢？

我这里推荐2种做法，仅供大家参考：

 * 设置一个顶部视图TopView，其他的视图如果与TopView，Y一样，则添加约束时，使用Align方式，那么在代码里，判断如果是ios7以上，动态修改TopView的NSLayoutConstraint的constant值，这样只需要修改一个视图的约束即可，其他视图会自动改变；
 * 使用Storyboard方式开发，添加约束时，将视图的TopLayoutConstraint与Top Layout Guide相关联。

如下图：

![haroopad icon](http://ww1.sinaimg.cn/large/62ca154djw1er0k3q5zjoj206z06omxn.jpg)

不过第二种方式的缺点就是只能在StoryBoard中创建视图了， 在Xib中由于没有Top Layout Guide,
当然我的其他同事，还有对于这种UIViewController的view布局，都是使用代码方式实现，对于像里面使用自定义的UITableViewCell则使用Xib方式。
总之，大家找到一个适合的方式就可以了，不必纠结于此！


接下来，我们将介绍一些AutoLayout的对于UIScrollView，UITableViewCell动态高度等一些相对高级的用法，尽情期待吧！









