---
layout: post
title: "ios中使用FBMemoryProfiler工具"
date: 2017-05-22 14:58:46 +0800
comments: true
categories: 
- ios
- facebook
- Objective-C
---

Facebook最早开源过`FBRetainCycleDetector`, 这已经是2016年4月推出`FBMemoryProfiler`的前身了， 最新的`FBMemoryProfiler`提供了很多的交互界面和更多一些的功能。

<!--more-->


在ios项目中集成`FBMemoryProfiler`的工具有2种:

* <font color=#008B8B size=4>CocoaPods </font>
* <font color=#008B8B size=4>Carthage</font>


由于最早在开发中， `CocoaPods`自身有很多的缺陷， 另外`CocoaPods`也会改变之前工程的结构方式，所以我个人一项不太喜欢用这个工具。


使用`Carthage`会帮助我们下载并编译好framework供我们使用，相当于一个去中心化的`Cocoa`依赖管理工具， 当然目前它是从ios8之后开始支持的， 如果你的工程还需要支持更低版本，还是绕路用`CocoaPods`吧，  或者可以`Carthage`制作好的framework添加到工程里使用也行。


建议使用`HomeBrew`的方式下载安装<font color=#008B8B size=4>Carthage</font>；

如果`Homebrew`不会安装，可以自行翻看我早期的blog，或者自行google；


##安装方法：

```sh
brew install carthag
```


安装好之后，我们就开始来将facebook的FBMemoryProfiler工具集成到我们的项目中，

####一、新建好自己的项目工程

####二、进入项目工程所在的文件夹，然后新建空文件Cartfile

```sh
touch Cartfile
```
####三、使用xcode打开这个文件

```sh
open -a Xcode Cartfile
```

####四、在打开的文件中输入

```ruby
github "facebook/FBMemoryProfiler"
```
添加完成之后，保存并退出。

<font color=#FF7F50> 注意： 如果想要添加其他的类库， 就可以在这个文件中另起一行，输入其他的url地址 </font>

####四、在终端执行命令

```sh
carthage update --configuration Debug
```

执行成功后，就会看到在工程根目录下多了2个文件和一个文件夹

![](http://ww1.sinaimg.cn/large/62ca154dly1ffu67rb5pxj20bi03mq36.jpg)

并且在Carthage文件夹里会有Build和Checkouts2个文件夹， 其中Build里面会保存我们编译好的framework，如图：

![](http://ww1.sinaimg.cn/large/62ca154dly1ffu69catrfj20yc0e00vo.jpg)

####五、打开工程，将上述framework添加到项目中

点击`项目名称`-> `target` -> `Gerneral`，在最底部找到`Linked Frameworks and Libraries`，

然后将上述的3个framework拖到此区域内。

![](http://ww1.sinaimg.cn/large/62ca154dly1ffu6bpjurtj20ve03p3yx.jpg)

####六、在xcode的菜单上，选择“Build Phases”，并添加一个新的“Run Script”，并添加命令

```sh
/usr/local/bin/carthage copy-frameworks
```
<font color=#FF7F50> 注意：如果之前已经创建过Run script，也可以新建一个新Run script </font>

然后点击`Input Files`下面的+号为每一个framework添加条目：

```sh
$(SRCROOT)/Carthage/Build/iOS/FBAllocationTracker.framework;
$(SRCROOT)/Carthage/Build/iOS/FBMemoryProfiler.framework;
$(SRCROOT)/Carthage/Build/iOS/FBRetainCycleDetector.framework
```

参考下图：

![](http://ww1.sinaimg.cn/large/62ca154dly1ffu6go2a3xj21om0zmk02.jpg)

以上是我们需要配置的信息，接下来，我们在工程中添加必要的代码

####1、在`main.m`文件中添加`FBRetainCycleDetector`的hook，同时也要开启`FBAllocationTracker`的生成追踪：

```obj-c
#import <UIKit/UIKit.h>
#import "AppDelegate.h"
#if DEBUG
#import <FBAllocationTracker/FBAllocationTrackerManager.h>
#import <FBRetainCycleDetector/FBRetainCycleDetector.h>
#endif

int main(int argc, char * argv[]) {
    @autoreleasepool {
#if DEBUG
        [FBAssociationManager hook];
        [[FBAllocationTrackerManager sharedManager] startTrackingAllocations];
        [[FBAllocationTrackerManager sharedManager] enableGenerations];
#endif
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}

```

<font color=#FF7F50> 注意： 我们只在Debug模式下才开启`FBMemoryProfiler`</font>

####2、在`AppDelegate.m`中添加代码：

```obj-c
#import "AppDelegate.h"
#if DEBUG
#import <FBMemoryProfiler/FBMemoryProfiler.h>
#import <FBRetainCycleDetector/FBRetainCycleDetector.h>
#endif
@interface AppDelegate ()
{
#if DEBUG
    FBMemoryProfiler *memoryProfiler;
#endif
}
@end

@implementation AppDelegate

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
#if DEBUG
    memoryProfiler = [FBMemoryProfiler new];
    [memoryProfiler enable];
#endif
    return YES;
}

```

最后编译运行，就会看到在项目中有一个悬浮窗，标记着我们当前APP的运行的内存打开，点开之后，可以查看是否有循环引用等功能。

