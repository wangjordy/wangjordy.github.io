---
layout: post
title: "ios关于Core SpotLight的使用"
date: 2017-06-09 16:30:19 +0800
comments: true
categories: 
- ios
---

SpotLight已经推出来很久了， 从ios9苹果推出了一些新的API之后，我们可以将App的部分内容通过建立索引，然后在Spotlight上搜索结果， 并且完全可以根据你的需求来设定跳转的页面， 以及Spotlight页面显示的内容设置 。

<!--more-->

为了给小白同学科普一下SpotLight的作用，SpotLight简单的说就是在ios系统的（桌面滑到最左端的搜索页面），在搜索栏中输入一些信息，
我们希望输入的这些信息，可以点击后进入到我们APP对应的页面中。

可以参考国外[关于SpotLight的英文版 for swift的介绍](http://www.appcoda.com/core-spotlight-framework/)的使用

我们这里使用的代码是Objective-C语言来介绍说明，  这些借用上面blog中的图片，来综合说明一下：

###下图是我们的App界面， 点击列表的cell（电影信息），可以进入该电影的详情页
![](https://www.appcoda.com.tw/wp-content/uploads/2015/12/t46_1_app_working-compressor.gif)

###退出APP后，在手机的搜索页面中输入一个电影名

![](http://www.appcoda.com/wp-content/uploads/2015/12/t46_2_final_sample-compressor.gif)

点击搜索出来的内容，直接进入电影的详情页


好了，接下来我们来一下实现的教程吧

##开始教程

####1、项目中加入CoreSpotlight.framework

####2、创建索引--所需要的元数据内容

![](http://ww1.sinaimg.cn/large/62ca154dly1fgf1yor0vtj20p40ekn0r.jpg)

上图中我标记出来的这几个信息内容都是可以自定义的。

 * 标题对应的属性:  <font color=#FF7F50>attributeset.title</font>
 * 简介对应的属性:  <font color=#FF7F50>attributeset.contentDescription</font>
 * 图片对应的属性:  <font color=#FF7F50>attributest.thumbnailData或thumbnailURL</font>
 
如果默认不填写图片的属性值，则直接显示的是APP的icon

#####（1）创建一个元数据的属性内容

```obj-c
CSSearchableItemAttributeSet *attributeSet = [[CSSearchableItemAttributeSet alloc] initWithItemContentType:@"contact"];
attributeSet.title = @"标题";
attributeSet.contentDescription = @"内容";
attributeSet.keywords = @[@"关键字1", @"关键字2"];
attributeSet.thumbnailData = UIImagePNGRepresentation([UIImage imageNamed:@"缩略图"]);
```
#####（2）创建一个元数据的对象，将这些属性需添加到这个对象中，并设置UniqueIdentifier值


```obj-c
CSSearchableItem *searchableItem = [[CSSearchableItem alloc] initWithUniqueIdentifier:@"carid" domainIdentifier:@"com.car" attributeSet:attributeSet];
```
这里的`initWithUniqueIdentifier`是我们将来点击搜索结果后能获取的到值，一般这里填写我们的关键数据（<font color=#DC143C>比如车的id值，注意是值，不是carid这个名字，并且这个值一定要是唯一的</font>），  
`domainIdentifier`一个可选的标识符，用来表示item的域

#####（3） 也可以设置一个指定的过期日期， 一般默认的过期时间是一个月

```obj-c
searchableItem.expirationDate = [NSDate dateWithTimeIntervalSinceNow:3600];
```

#####（4）如果有一组这样车的信息，可以先将创建好的CSSearchableItem的对象保存到一个数组中

```obj-c
NSMutableArray *carArray = [[NSMutableArray alloc] initWithObjects: searchableItem,nil];
```
这里根据实际需求，可以创建多个对象。

#####（5）将创建的item信息，添加到CoreSpotlight的索引对象中

```obj-c
[[CSSearchableIndex defaultSearchableIndex] indexSearchableItems:carArray completionHandler:^(NSError * _Nullable error) {
	if(error) {
		NSLog(@"%@", [error localizedDescription]);
	}
}];

```
####3、创建好索引之后，在AppDelegate文件中配置点击搜索结果的处理

```obj-c
- (BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity restorationHandler:(void(^)(NSArray * __nullable restorableObjects))restorationHandler {
    UIViewController *rootViewController = self.window.rootViewController;
    if (rootViewController && [rootViewController isKindOfClass:[ViewController class]]) {
        ViewController *viewController = (ViewController *)rootViewController;
        [viewController showCarDetailViewController:userActivity];
    }
    return YES;
}
```
在ViewController的showCarDetailViewController方法中， 先获取userActivity的值

```obj-c
NSString *spotlightIdentifier = userActivity.userInfo[@"kCSSearchableItemActivityIdentifier"];
```
比如，这些携带的是车的id， 可以通过id信息，请求服务器端接口，获取整个页面的详细信息；


