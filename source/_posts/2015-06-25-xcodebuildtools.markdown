---
layout: post
title: "xcode自动打包与shell脚本"
date: 2015-06-25 10:41:04 +0800
comments: true
categories:
- ios
- Objective-C
- Xcode
- shell
---

在xcode5以后，一般在选择自动打包时， 经常会提示选择证书情况， 如果想要绕过选择证书，可以通过xcodebuild命令实现
在持续集成中：
* 第一步，使用命令行打ipa包；
* 第二步，可以使用svn或git工具 hook实现自动发布；

本篇只说明一下如果使用命令行以及shell来打ipa包

###项目工程为*.xcodeproj

对于这种工程来说， 只需要在当前工程目录下，运行命令xcodebuild就可以了， 这时，xcodebuild命令运行之后，会在项目目录下新建一个build文件夹，在build/Release-iphoneos/xxx.app 这个目录下会生成一个与Target相同名的.app文件。

执行的命令：

```ruby

xcodebuild

```

接下来， 将xxx.app文件使用命令生成ipa包

执行的命令：

```ruby

xcrun -sdk iphoneos PackageApplication -v build/Release-iphoneos/xxx.app -o build/Release-iphoneos/xxx.ipa

```

解释：

- -v参数之后的路径是刚刚使用xcodebuild命令生成的xxx.app的路径；
- -o参数之后的路径是存放.ipa包的路径；
(备注，xxx是你起的ipa包名)

###项目工程为*.xcworkspace

对于像使用了CocoaPods工具之类的工程来说， 在第一步使用xcodebuild命令时，需要加几个参数，

执行的命令：

```ruby

xcodebuild -workspace xxx.xcworkspace -scheme xxx -sdk iphoneos -configuration Release -derivedDataPath build

```

解释：

- -workspace 之后的参数 xxx.xcworkspace表示需要打包的项目名；
- -scheme 后边的xxx表示选择的scheme是什么（对于多个target来说）
如果，不知道scheme都包括什么，可以运行命令

```ruby

xcodebuild -list

```

查看， 项目中都包括哪些target，选择合适的target（一般scheme后的名称与项目名相同）
- -sdk表示编辑后的包为iphone device使用，而非Simulator使用
- -configuration后面的Release表示打的包为Release版本， 当然也可以选择Debug版本；
- -derivedDataPath表示的生成的.app文件的路径

最后生成的ipa包的命令与上述打.xcodeproj工程文件的命令相同


###关于使用shell脚本的编写

1.对于*.xcodeproj来说， 代码如下：

```ruby

#! /bin/bash


echo "准备开始打ipa包...................."

#工程环境路径
workspace_path=/Users/jordy/Desktop/testbao
#项目名称
project_name=TestProject

#build的路径
build_path=$workspace_path/$project_name

echo "第一步，进入项目工程文件: $build_path"

cd $build_path

echo "第二步，执行build clean命令"

xcodebuild clean

echo "第三步，执行编译生成.app命令"

xcodebuild

echo "在项目工程文件内生成一个build子目录，里面有${project_name}.App程序"

echo "第四步, 导出ipa包"

#.app生成后的路径
app_name_path=$build_path/build/Release-iphoneos/${project_name}.app
#.ipa生成后的路径
ipa_name_path=$build_path/build/Release-iphoneos/${project_name}.ipa

#生成ipa包
xcrun -sdk iphoneos PackageApplication -v $app_name_path -o $ipa_name_path

echo "制作ipa包完成......................."


```

2.对于*.xcworkspace来说， 代码如下：

```ruby

#! /bin/bash


echo "准备开始打ipa包...................."

#工程环境路径
workspace_path=/Users/xingchaowang/Desktop/testbao
#项目名称
project_name=TestProject

#build的路径
build_path=$workspace_path/$project_name

echo "第一步，进入项目工程文件: $build_path"

cd $build_path

echo "第二步，执行build clean命令"

xcodebuild clean

echo "第三步，执行编译生成.app命令"

xcodebuild -workspace $project_name.xcworkspace -scheme $project_name -sdk iphoneos -configuration Release -derivedDataPath build

echo "在项目工程文件内生成一个build子目录，里面有${project_name}.App程序"

echo "第四步, 导出ipa包"

#.app生成后的路径
app_name_path=$build_path/build/Build/Products/Release-iphoneos/${project_name}.app
#.ipa生成后的路径
ipa_name_path=$build_path/build/Build/Products/Release-iphoneos/${project_name}.ipa

#生成ipa包
xcrun -sdk iphoneos PackageApplication -v $app_name_path -o $ipa_name_path

echo "制作ipa包完成......................."

```

上述文件中都有备注说明这里就不在重复解释了， 如果你粘贴使用的话，记得修改最上面的worspace_path和project_name的为你自己的工程路径与工程名，另外

```ruby

xcodebuild clean

```
表示在打包前，先clean一下工程。

