---
layout: post
title: "Xcode7 打包静态.a文件或framework产生大量警告"
date: 2016-04-05 17:39:00 +0800
comments: true
categories: 
- ios
- Objective-C

---

在升级使用Xcode7工具之后， 之前我们集成第三方的framework或者静态.a文件， 编译会报大量的警告， 具体如下:

![](http://ww2.sinaimg.cn/large/62ca154dgw1f93jlm5vkcj20ws03edkg.jpg)

`warning:  /var/folders/    .pcm: No such file or directory`


<!--more-->

第三方的framework中的类文件， 会被警告找不到pcm文件或者文件夹

解决的方法， 打开第三方的framework的源码，在Build Settings中，设置

```
* Precompile Prefix (GCC_PRECOMPILE_PREFIX_HEADER) = NO

* Debug Information Format (DEBUG_INFORMATION_FORMAT) = DWARF with dSYM

* Enable Modules (C and Objective-C) (CLANG_ENABLE_MODULES) = NO

```



如果对这三个属性不明白， 可以参考下图：

![](http://ww1.sinaimg.cn/large/62ca154dgw1f93jlxidudj20ei02jjrh.jpg)



设置好以上3个属性后， 重新编译制作新的framework后， 再集成到自己的项目中，验收是否还有以上的警告！






