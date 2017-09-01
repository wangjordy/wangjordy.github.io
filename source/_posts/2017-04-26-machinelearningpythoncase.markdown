---
layout: post
title: "机器学习---关于Matplotlib画图的使用"
date: 2017-04-26 14:27:58 +0800
comments: true
categories: 
-  Python
-  MachineLearning
---

上一节中，我们介绍了关于Python中一个很重要的库<font color=#008B8B>Pandas</font>， 今天我们继续学习关于<font color=#008B8B> Matplotlib </font>中如何画图的教程。

<!--more-->


##学习matplotlib的使用


```python
import pandas as pd

unrate = pandas.read_csv("xxx.csv")  //读取某一个csv文件

unrate['DATE'] = pd.to_datetime(unrate['DATE'])
print (unrate.head(12))       //打印出来是DataFrame（数据流）
```

上述csv表格中的数据， 我们希望通过matplotlib 用折线图的形式来展示出来


```python
import matplotlib.pyplot as plt

plt.plot(x, y)   //需要传入x轴的值和y轴的值
plt.show()       //通过这句话才能让视图显示出来
```

####例一  

```python
import matplotlib.pyplot as plt

first_twelve = unrate[0:12]
plt.plot(first_twelve['DATE'], first_twelve['VALUE'])   //需要传入x轴的值和y轴的值
plt.show()       //通过这句话才能让视图显示出来
```

如果横坐标的 属性的名字特别长（比如是“2017-04-21 8点”等）， 就需要将这个属性名做一个角度的旋转

```python
import matplotlib.pyplot as plt

first_twelve = unrate[0:12]
plt.plot(first_twelve['DATE'], first_twelve['VALUE'])   //需要传入x轴的值和y轴的值
plt.xticks(rotation=45)  //这里让它旋转45度，  如果完成让这个数字竖着表示就旋转90度
plt.show()       //通过这句话才能让视图显示出来
```

####例二  介绍matplotlib的使用

在一个区域里假如需要画多个图的时候， 就需要使用matplotlib库。

通过<font color=#008B8B>pyplot.add_subplot(4,2,x)</font>,  这个表示的是构造的图形为一个4行2列的图形 （按照属性：从左到右，从上到下的顺序），x表示的是在第几块画这个图形， 看下图就比较直白了


```python
import matplotlib.pyplot as plt

fig = plt.figure()      #表示我们默认指定一个默认画图的一个区间
ax1 = fig.add_subplot(2,2,1)   #第三个参数1表示的就是第一个子图
ax2 = fig.add_subplot(2,2,2)
ax3 = fig.add_subplot(2,2,4)   #第三个参数4表示的就是第四个子图

plt.show()
```

![](http://ww1.sinaimg.cn/large/0061Ec5yly1ff06kr0mx0j30po0n4mze.jpg)

我们通过上图可以清晰的看到， 原本的第三个区域，应该放的子图是空着，  我们ax3这个是直接放到第4个子图的位置上了。

##画折线图

####例三  给figure带入参数的方式

在<font color=#008B8B>plt.figure() </font>给它指定一个画图的大小区域，  比如<font color=#008B8B>plt.figure(figsize=(3,3)) </font>其第一个参数3表示的是宽度是3， 第二个参数3表示的是高度3，  如果想变宽就把第一个参数设置的大一些，同理设置第二个参数。

```python
import matplotlib.pyplot as plt
import numpy as np

fig = plt.figure(figsize=(3,3))
ax1 = fig.add_subplot(2,1,1)   #画出第一个子图
ax2 = fig.add_subplot(2,1,2)   #画出第二个子图

ax1.plot(np.random.randint(1,5,5), np.arange(5))  #这里在第一个图上画一组随机值
ax2.plot(np.arange(10)*3, np.arange(10))         #这里在第二个图上画第二组的值

plt.show()
```
详细见下图：

![](http://ww1.sinaimg.cn/large/0061Ec5yly1ff073oaqkjj30r20mu0vg.jpg)

####如果再同一个图中，画2条线的操作：

这里画2条折线，  折线的数据是从上述的csv文件中读取的。

```python
import matplotlib.pyplot as plt

unrate['MONTH'] = unrate['DATE'].dt.month

fig = plt.figure(figsize=(6,3))

plt.plot(unrate[0:12]['MONTH'], unrate[0:12]['VALUE'], c='red')    #这里画一条红线
plt.plot(unrate[12:24]['MONTH'], unrate[12:24]['VALUE'], c='blue') #这里画一条蓝线

plt.show()
```

如果在图形上加入label来表示每个不同颜色的先表达的是什么的实例：

```python
import matplotlib.pyplot as plt

fig = plt.figure(figsize=(10,6))

colors = ['red','blue','greeen','orange','black']  #声明5种颜色的线

for i in range(5):
    start_index = i * 2
    end_index = (i+1) * 12
    subset = unrate[start_index:end_index]
    label = str(1948 + i)   
    plt.plot(subset['MONTH'], subset['VALUE'], c=colors[i], label=label)
    
plt.legend(loc='best')   #让label框显示在合适的位置， 这里也可以选择其他的参数（根据打印help(plt.legend)即可查看）

plt.xlabel('Month, Integer')     #添加表示x轴表达的含义
plt.ylabel('UNemployment Rate, Percent') #添加表示y轴表达的含义
plt.show()                  
```


##柱状图

调用柱状图的方法是通过 bar(x,value, width) , x表示的在x轴的间距， value表示柱子的高度（真实值）， width表示的柱子的宽度。 

```python

import pandas as pd
from numpy import arange

bar_heights = [2.3, 3, 5.5, 4.2, 1.5]
print (bar_heights)

bar_positions = arange(5) + 0.75
print (bar_positions)

fig, ax = plt.subplots()
ax.bar(bar_positions, bar_heights, 0.3)  #这里的0.3表达的是柱子的宽度， 

plt.show()                 
```
![](http://ww1.sinaimg.cn/large/0061Ec5yly1ff0aekb7lhj30yg0ten0m.jpg)

###横向的柱状图

将上面代码中的<font color=#008B8B>ax.bar()</font>,这个函数， 改为<font color=#008B8B>ax.barh()</font>， 这个图形就变为横向表示的主状态了

![](http://ww1.sinaimg.cn/large/0061Ec5yly1ff1dpe8ra7j311g0k8jt1.jpg)

如果想要去掉图像上的“小齿” （比如1的位置多出来的一点点线头）的方式， 

```python

axtick_params(bottom="off", top="off", left="off", right="off")

```
##散点图

如果csv数据中的值， x，y轴表示的都有意义， 那么在二维坐标轴画出来的都是离散的点， 这种图形的代码， 通过调用
<font color=#008B8B>ax.scatter()</font>函数来实现，

```python

import pandas as pd
from numpy import arange

fix,ax = plt.subplots()
ax.scatter(normal_reviews['Fandango_Ratingvalue'], normal_reviews['RT_user_norm']) 
ax.set_xlabel('Fandango')           #给X轴加上注释
ax.set_ylabel('Rotten Tomatoes')    #给X轴加上注释

plt.show()                  
```

##简化图形的做法

###柱状图
如果我们的x轴上每一个值都画一个柱状图（那么看起来会特别的乱）， 比方1-2区间内，有1.1，1.2……1.9.20， 我们可能想关注1-1.5之间有多少个值， 1.5-2之间有多少值，  这样的需求。  可以通过以下<font color=#008B8B>ax.hist()</font>函数来实现.

```python

import pandas as pd
from numpy import arange

fix,ax = plt.subplots()
ax.hist(norm_reviews['Fandango_Ratingvalue'])  #这里不指定的bins的话（区间个数）， 系统会默认给我们指定一个（通常是10个格子）
#当然也可以通过加入bins参数，来认为指定bins个数
#ax.hist(norm_reviews['Fandango_Ratingvalue'], bins=20)  #这里我们制定20个格子。

#如果我们想看某一个区间内的个数， 假如有1-10之间都有值， 但是我们想关注4-5区间内的一个分布情况， 可以通过以下
#ax.hist(norm_reviews['Fandango_Ratingvalue'], range=(4,5), bins=20)

plt.show()                  
```


###折线图
我们也可以在subplot折线图， 同样做指定一个区间的值，  

```python
#其他的代码不再写了
ax.set_ylim(0,50) #这里指定了y轴0-50的区间，  当然也可以制定x轴的区间

```

###盒图（四分图）

通过的函数是<font color=#008B8B>ax.boxplot()</font> 来实现一个盒图

```python

fix,ax = plt.subplots()
ax.boxplot(norm_reviews['RT_user_norm'])  #通过将RT_user_norm中的值进行画一个盒图
ax.set_xticklabels(['Rotten Tomatoes'])  #制定x轴的名字
ax.set_ylim(0,5)  #设置y轴的一个区间
plt.show()

```

如果想把多个特征的盒图都画到同一个图上

```python

num_cols = ['RT_user_norm', 'Metacritic_user_nom', 'IMDB_norm', 'Fandango_Ratingvalue']
fix,ax = plt.subplots()
ax.boxplot(norm_reviews[norm_reviews[num_cols].values)  #通过将RT_user_norm中的值进行画一个盒图
ax.set_xticklabels(num_cols, rotation=90)  #制定x轴的名字,并旋转x轴上每个特征名字的角度
ax.set_ylim(0,5)  #设置y轴的一个区间
plt.show()

```

##其他细节

####画折线图，使用自定义的颜色，

```python

cb_dark_blue = (0/255, 107/255, 164/255)
ax1.plot(np.random.randint(1,5,5), np.arange(5), c=cb_dark_blue)  #这里的c的值就可以写为我们自定义的色值

```

####在图上，在特定的值添加文字描述

```python
ax.text(x,y,'文字描述') #x和y就是x轴和y轴的值，  第三个参数就是这个值要添加的文字描述了
```



