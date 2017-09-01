---
layout: post
title: "机器学习---入门（安装篇）"
date: 2017-03-25 17:31:30 +0800
comments: true
categories: 
-  Python
-  MachineLearning

---

之前一直在学习关于数据挖掘的理论知识， 今天开始接触一下关于它的一个分支---机器学习。

机器学习简单的来说就是通过历史数据，通过一些数据分析的技术，建立一个“数据模型”， 当有新的数据通过这个数据模型，可以进行相应的预测。

常用的学习主要分为三类：

 * 有监督学习：  分类、 回归
 * 无监督学习：  聚类
 * 半监督学习：  结合

<!--more-->

<font color=#006400>简单来说有监督学习和无监督学习的区别就是有没有Label的标记。
</font>

举例：

	比如我们有一堆的数据来表示来描述这个事物，用来表示“猫”
    又有另一堆的数据来表示来描述另一个事物，用来表示“狗”
    
 <font color=#FF8C00>这一类的数据，都有一个Label，表示“猫”或者表示“狗”， 我们称之为有监督的学习，
 无监督的学习，就是通过一堆的数据，进行分析，建立一个模型，但是不输出Label。</font>
    
	
机器学习常用的编程语言主要包括： Python、R语言、Java、C/C++ 等；
Python语言比较中立，并且更适合做产品， Python语言的内存管理也比较容易，交给自己的GC即可， 另外，它配套的类库也比较多。用的人数也更多，更
R语言更适合做数据模型，不太适合做产品。Java适合做产品，但是目前因配套的库还不多，编写起来就比较复杂。其他的语言目前相对人数就更少一些了。

这里我们就选用Python语言作为我们机器学习的编程语言。

闲话少说，我们从安装到编写“Hello World”程序开始。


###一、 Mac OS下安装python3

1、打开终端，安装homebrew（之前文章有介绍过，这里不在介绍它的安装方式）；

2、输入命令

```python
brew install python3

```
我们这里使用的是python3版本。

3、安装jupyter （jupyter是一个编辑器，可以在上填写公式、注释与代码）

```python
pip3 install jupyter 

```
安装成功之后，可以测试一下Juptyter的网页服务内容。

4、输入命令

```python
jupyter notebook  

```
这时，打开浏览器，输入地址： localhost:8888/tree

打开的网页中，默认在Files中，显示的是自己电脑根目录下的所有文件夹；

![](http://ww1.sinaimg.cn/large/62ca154dly1fe15y2o0b0j21ts0ki0v9.jpg)

接着点击右上角 “New--Python3”， 此时会自动跳转到一个新的网页，

如图：

![](http://ww1.sinaimg.cn/large/62ca154dly1fe15xmejn1j21u60gmmzf.jpg)

我们可以在 In []:  地方，输入


```python
print （“Hello world”）
```
程序立刻会输出我们需要打印的信息。

以上的方式可以做python自带的一些库的操作， 但是如果想要使用机器学习中的库，就需要安装Anaconda工具了。

###安装Anaconda

Anaconda是一个用于科学计算的Python发行版， 它提供了包管理与环境管理的功能， 方便解决Python多个版本并存，切换以及第三方包安装的问题。

Anaconda利用命令<font color=#DC143C> conda </font>来进行package和environment的管理，并且提供了Python相关的工具。

可以直接在官网上下载自己系统的安装包 https://www.continuum.io/downloads

选择自己合适的Python版本， 建议选择3.6以上的版本。

下载好安装包之后，直接安装即可。

如果需要使用conda命令，  需要进行配置

```python
vi ~/.zshrc
```
在文件中，加入anaconda的PATH路径

```python
export PATH="$PATH:$HOME/.rvm/bin:/anaconda/bin" # Add RVM to PATH for scripting
```
保存并退出后，  执行命令

```python
source ~/.zshrc
```
让此文件生效。  验证是否生效的方式，直接输入命令<font color=#DC143C> conda </font> 查看是否输出，如下图

![](http://ww1.sinaimg.cn/large/0061Ec5yly1feswtd2emjj30i90p6jvt.jpg)

####安装好Anaconda后，我们打开工具Anaconda-Navigator， 界面如下图：

![](http://ww1.sinaimg.cn/large/0061Ec5yly1ff060rlo2xj314x0k8n0l.jpg)

点击第一个jupyter栏， 就会默认打开一个网页（与上述jupyter notebook 命令相同，但这个打开的页面，会带入很多常用的第三方工具库）。

###Python的基础入门

因为Python语言与JavaScript语言比较接近，使用起来很方便， 不用提前提前声明变量的类型。 

#####声明 Int 或者 String类型

![](http://ww1.sinaimg.cn/large/0061Ec5yly1fel7pgn3zpj30ak0460sr.jpg)

#####判断某一个变量的类型方法

![](http://ww1.sinaimg.cn/large/0061Ec5yly1fel7rvtt9yj306z07ht8w.jpg)

#####声明一个数组，添加新的元素，取值等

![](http://ww1.sinaimg.cn/large/0061Ec5yly1fel7tdzf9qj30ft0813z2.jpg)

<font color=#DC143C>注意：上面的months[1:3]这一行的index比较特殊， 我们知道数组的index是从第0位开始取值的， [1:3]中的1表示的从第1个位置取值， 这里的3表示的是第3个位置，但是不包括3个位置的元素。 </font>

#####声明一个数组，对所有的元素进行遍历

![](http://ww1.sinaimg.cn/large/0061Ec5yly1fel84mc99xj30cr05v74o.jpg)

#####声明一个字典的2种方式

![](http://ww1.sinaimg.cn/large/0061Ec5yly1fel86kaccdj309w0a0gm8.jpg)

是不是感觉以上的内容还是特别容易理解呢，这里不再过多介绍了，介绍我们看一个机器学习中经常使用到的类库 <font color=	#008B8B>numpy</font>，  <font color=		#D2691E>numpy是Python对数据进行分析和矩阵中使用的一个非常关键的库</font>


#####例如，声明一个一维矩阵和一个三维矩阵

![](http://ww1.sinaimg.cn/large/0061Ec5yly1fel8dm4dlkj30fg08it9c.jpg)

#####查看矩阵的行列数为

![](http://ww1.sinaimg.cn/large/0061Ec5yly1fepqmscd0mj30qo0iqgni.jpg)

可以看到vector是一个1行4列的矩阵，  而matrix是一个2行4列的矩阵 （使用的函数就是<font color=#008B8B>shape</font>）

#####假如设置不同类型的存储

![](http://ww1.sinaimg.cn/large/0061Ec5yly1fepqrz5fwhj30m808eq3i.jpg)

<font color=#FF8C00>当numpy array中的最后一个元素为string，其余元素是int类型时， numpy会强制自动将它的元素统一转换类型，

这点与Python中的list不同的， Python中的list是元素之间的存储类型，是不影响其他元素的 </font>

#####矩阵的元素取值

![](http://ww1.sinaimg.cn/large/0061Ec5yly1fepr15g5svj30hg0o8wfx.jpg)

<font color=#008B8B>matrix[:,1] </font>表示的我要取矩阵中的第一列的所有的数据 （从第0列开始取）

<font color=#008B8B>matrix[:,0:2] </font>表示的我要取矩阵中的第零列和第一列的数据 （从第0列开始取）

同理，假如取某一行的数据

<font color=#008B8B>matrix[1,:] </font> 表示的我要取矩阵中的第一行的数据 （从第0行开始取）

#####判断矩阵中所有元素的值是否等于某一个数值

![](http://ww1.sinaimg.cn/large/0061Ec5yly1feprhohpa1j30r40h2gn1.jpg)

最终会打印出一个bool类型组成的新矩阵

#####判断矩阵中所有元素的值等于某一个数组的取值方式

![](http://ww1.sinaimg.cn/large/0061Ec5yly1feproc0x4fj30im0dkdgz.jpg)

equal_to_ten为一个bool类型组成的数组，再带入到vector数组中，则只会保留为true的元素；


#####将一个数据类型转为另一个数据类型 （String 转 float）

![](http://ww1.sinaimg.cn/large/0061Ec5yly1feps0awfnuj30ig0f0t9z.jpg)


#####将矩阵中按行求和

![](http://ww1.sinaimg.cn/large/0061Ec5yly1fepsbb9bw7j30eg0bc754.jpg)

#####获取一维数组转换为多维矩阵

![](http://ww1.sinaimg.cn/large/0061Ec5yly1fepsgewer0j30oi0byt9s.jpg)

<font color=#008B8B>np.arange(15) </font> 表示从0开始取15个数，

<font color=#008B8B>np.arange(15).reshape(3,5) </font> 表示将取出来的15个数，划分为一个3行5列的矩阵

另外， <font color=#008B8B>a.dtype.name</font> 表示的这个矩阵中的元素是什么类型的（这里是int32）
      <font color=#008B8B>a.size</font> 表示这个矩阵中的元素个数  （这里是15）

      
#####定义一个3*4的初始化矩阵

![](http://ww1.sinaimg.cn/large/0061Ec5yly1fepsp960jgj30fu08kmxk.jpg)

表示得到了一个3行4列的一个空矩阵

#####同理，定义一个2*3*4结构并且初始化为1的元素的矩阵

![](http://ww1.sinaimg.cn/large/0061Ec5yly1fepsrh1zsvj30j40cagmg.jpg)

#####定义一个等差数列

![](http://ww1.sinaimg.cn/large/0061Ec5yly1fepsvhtynzj30ps0bsq40.jpg)

比如上边第2行表示，从10-30中取值（不包含30）， 每次累加值（差值）为5

下面那个例子是从0-2中取值（不包含2）， 每次累加值（差值）为0.3  （小数也是可以这样定义）

#####矩阵与矩阵的运算

![](http://ww1.sinaimg.cn/large/0061Ec5yly1fept86lofkj30j60to76h.jpg)


上图中， A*B表示的是矩阵之间的内积的运算

比如A*B结果矩阵中的第0行第0列的2， 

是由A矩阵中第0行第0列的1 乘以 B矩阵中第0行第0列的2， 得到的结果2，放到结果矩阵中的第0行第0列
同理，A矩阵中第0行第1列的1 乘以 B矩阵中第0行第1列的0 ， 得到的结果0，放到结果矩阵中的第0行第1列

<font color=#DC143C>A.dot(B) 与 np.dot(A,B)的计算结果相同 </font>

计算的方式为：

比如A.dot(B)结果矩阵中的第0行第0列的5，
是由A矩阵中第0行第0列的1 乘以 B矩阵中第0行第0列的2得到结果2， 再加上A矩阵中第0行第1列的1 乘以B矩阵第1行第0列第3得到结果3，
然后再把2+3=5 这个值放到结果矩阵的第0行第0列中， （即 1x2+1x3=5）

同理A矩阵中第0行第0列的1 乘以 B矩阵中第0行第1列的2得到结果0， 再加上A矩阵中第0行第1列的1 乘以B矩阵第1行第1列4得到结果4，
然后再把0+4=4 这个值放到结果矩阵的第0行第1列中， 即 0x0+1x4=4）

其实就是A与B矩阵的乘法运算

#####矩阵的exp和sqrt的运算

![](http://ww1.sinaimg.cn/large/0061Ec5yly1fepu97hnskj30kw0bct9q.jpg)

上图，指定了一个数组， 通过np.exp(B), 计算数组的e的多次次幂，
通过np.sqrt(B),计算数组的根号运算


####其他
<font color=#008B8B>ravel()</font>相当于将一个多维矩阵，变换为了一个一维的矩阵；

<font color=#008B8B>shape</font>也可以变换矩阵， a.shape=（6，2）表示将原来的矩阵变换为一个6*2的矩阵；

<font color=#008B8B>vstack</font>表示将2个矩阵进行竖着组合为一个矩阵；

<font color=#008B8B>hstack</font>表示将2个矩阵进行横着组合为一个矩阵；

<font color=#008B8B>hsplit</font>表示将1个矩阵进行横着切分为多个矩阵，  比如np.hsplit(a,3)表示切分为3个矩阵；

当然也可以指定位置切分np.hsplit(a,(3,4))表示从第3列切一次，在第4列又切一次， 最终切分为了3个矩阵；

<font color=#008B8B>vsplit</font>表示将1个矩阵进行竖着切分为多个矩阵， 比如np.vsplit(a,3)是表示切分为3个竖的矩阵；

###内存管理

![](http://ww1.sinaimg.cn/large/0061Ec5yly1fepuqtu4onj30ec0gsdh1.jpg)

通过这个实例，我们也能清楚的知道，a和b指向的是同一块内存；  id(a) 表示a的内存地址；


![](http://ww1.sinaimg.cn/large/0061Ec5yly1fepuve6bb4j30ia0miabu.jpg)

通过这个实例，我们知道 c = a.view（）， 表示的c对a是一个浅复制， 他们的内存地址不同，只是c在创建是，与a的值相同。

当然，也可以通过copy的方式进行浅复制

![](http://ww1.sinaimg.cn/large/0061Ec5yly1fepuz8dm97j30hg0i6t9w.jpg)

也是只是值的复制，a和d的内存地址不同。


今天先介绍到这里，接下来我还会持续更新此次内容，敬请期待。

