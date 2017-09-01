---
layout: post
title: "机器学习---关于常用的一些python库的介绍说明"
date: 2017-04-20 10:33:12 +0800
comments: true
categories: 
-  Python
-  MachineLearning

---

书接上文，上一讲我们主要介绍了，机器学习中关于Python的环境搭建和一些基础Python语法以及numpy库的一些简单的使用，今天我们继续再学习一些有关Python库的基础知识。

我们首先来看一下，机器学习中常用的另一个常见的库 <font color=#008B8B>Pandas</font>


<font color=#008B8B>Pandas</font>有2个独有的基本数据结构： Series 和 DataFrame， 这使得不论是读取还是写入等处理数据，使得我们用起来很方便， 下面用例子给大家介绍：

<!--more-->

####一、介绍Pandas的使用
例一：

```python
import pandas

food_info = pandas.read_csv("xxx.csv")  //读取某一个csv文件

print (type(food_info))       //打印出来是DataFrame（数据流）
print (food_info.dtypes)      //打印这个csv文件都有哪些列， 以及各列的数据类型
print (help(pandas.read_csv)) 

```

上述已经有介绍具体的方法代表的意思了， 这里不再过多描述。

我们通过上面实例知道了，food_info实际是一个DataFrame数据类型， 继续解说：

```python
food_info.head()   #表示的是csv的前几行的列表是什么样子的， 如果指定某几行的数据，直接在head()方法中，添加数字几就可以了
food_info.tail(3)  #表示的是csv的最后3行的内容
food_info.colums   #表示的是csv的每列的名字
food_info.shape    #表示的是csv的一共有多少行的数据， 并且有多少列

```

####关于取元素的值

```python
food_info.loc[1] 	     #表示的是csv的取文件中第一个数据（样本）
food_info.loc[3:6]       #表示的是csv的取文件中可以取某一个区间的数据（样本）
food_info.loc[2，5，10]   #表示的是csv的取文件中取某几个元素

food_info["NOB_NO"]      #表示的是csv的取某一列的所有数据，  “NOB_NO”表示的是在文件，那一列的名字

```

####关于取某些单位相同的值

假如第A列和C列的列名是，   A(g) 和 C(g)， 它们都是一个g为单位的数据，  加入我们提取cvs中，所有以g为单位的数据，


```python
col_names = food_info.columns.tolist() 	     #表示的是csv的取文件中所有列的名字

gram_colums = []

for c in col_names:                 #通过for循环获取所有以g结尾的列，并添加到gram_colums中
	 if c.endswith("(g)"):
	    gram_colums.append(c)
	    
gram_df = food_info[gram_colums]
print gram_df.head(3)      #打印gram_df它的前三行

```

####关于取某列并转换单位的功能

```python
div_100 = food_info["Iron_(mg)"] / 1000  #表示的是取Iron_(mg)列，将mg单位转换为g
print div_100

```

####添加新的特征（新增列）

```python
food_info["Iron_(g)"] = food_info["Iron_(mg)"] / 1000   #这里Iron_(g)是新造出来的一个特征（列）， 它是通过原有的特征Iron_(mg)经过变换后的出来的。

print (food_info.shape)
```


<font color=	#FF8C00>注意：这里新造的特征要保证它的维度与原来是能够对应的上的。</font>

####关于排序的操作

```python
food_info.sort_values("Sodium_(mg)", inplace=True)   #这个表的是，对Sodium_(mg)列进行按照从小到大排序， 
		                    #inplace=True表示这个我们构造出来一个新的DataFrame，不在原来的DataFrame上变换。
		                                       
print (food_info("Sodium_(mg)"))   #打印了新排序好的列


food_info.sort_values("Sodium_(mg)", inplace=True ascending=False)  表示按照从大到小的排序

print (food_info("Sodium_(mg)"))   #打印了新排序好的列
```

###二、关于Pandas的实际应用

例二：   根据泰坦尼克的数据进行分析，存活的人数与什么维度的数据有关


[下载泰坦尼克的数据](https://www.kaggle.com/c/titanic/data)

####关于查看缺失值的操作

```python
import pandas as pd
import numpy as np

titanic_survival = pd.read_csv("~/Downloads/Titanic/train.csv")

titanic_survival.head()


age = titanic_survival["Age"]

age_is_null = pd.isnull(age)  #这里就是一个True或False的list

age_null_true = age[age_is_null]  #这里就是只保留True的值（将一个list放在一个矩阵当中，会保留下来True值，False值会被过滤掉）

age_null_count = len(age_null_true) #查看一共有多少个缺失值
print age_null_count

```

截图：

![](http://ww1.sinaimg.cn/large/0061Ec5yly1ff9gf56ityj30rr094q4y.jpg)

<font color=#5F9EA0>我们在实际应用中，实际需要考虑一些特征（列），缺失值的多少来判定这个特征是否还有作用，  对于如果缺失值不多的话， 需要做一个缺失值的填充的操作。</font>

例如： Age列存在一些数据是NaN的缺失值。

####关于mean()的使用

```python
good_ages = titanic_survival["Age"][age_is_null == False]  #获取所有有年龄的值的数据， 组合成一个list
correct_means_age = sum(good_ages) / len(good_ages)       #计算这些有年龄值的一个均值（也就是平均年龄）
print correct_means_age


###########内置函数##########
correct_mean_age = titanic_survival["Age"].mean()
print (correct_mean_age)      #这里得到的值与上述correct_means_age的值是一样的， 说明pandas其实是帮我们过滤了缺失值之后的一个计算

```
<font color=#5F9EA0>pandas通过mean()帮助我们将缺失值去掉。</font>

继续， 假如我们计算每一个舱位的票价


```python
passenger_classes = [1,2,3]  //有3种类别的舱位
fares_by_class = {}          //获取每一个舱位的平均票价

for this_class in passenger_classes:
    pclass_rows = titanic_survival[titanic_survival["Pclass"] == this_class]  #获取当前是某一个类别的所有数据（比方是1等舱的数据）
    pclass_fares = pclass_rows["Fare"]     #获取上述的数据中，再获取得到该舱位的所有票价（比方获取1等舱的所有票价）
    
    fare_for_class = pclass_fares.mean()    #过滤缺失的数据,并计算出当前舱位的平均价
    fares_by_class[this_class] = fare_for_class   #将本舱位的票价添加到fares_by_class中

print fares_by_class

```

####关于2个特别之间的一个关系-----掌握pivot_table() “数据透视表”的使用

比如我们刚刚计算了1，2，3等舱的每个舱平均票价，  接下来，我们来介绍关于每一个舱中，平均获救的概率。

那么，我们以Pclass（船舱）为基准（为键）， 计算survived的均值，

```python
passenger_survival = titanic_survival.pivot_table(index="Pclass", values="Survived", aggfunc=np.mean)
print passenger_survival

```

同理，我们计算不同的船舱的，来计算年龄的均值


```python
passenger_age = titanic_survival.pivot_table(index="Pclass", values="Age")
print passenger_age

```
这里，同样是以Pclass为基准， 计算每个Pclass中的平均年龄。 这里没有加入上述参数<font color=#5F9EA0>aggfunc=np.mean</font>， 这是因为默认<font color=#5F9EA0>titanic_survival.pivot_table</font>计算的就是均值的意思。


####关于pivot_table其他的应用

我们这里假如通过不同的登船地点， 计算登船地点的总票价和总获救人数。


```python
port_stats = titanic_survival.pivot_table(index="Embarked", vlaues=["Fare", "Survived"], aggfunc=np.sum)
print port_stats

```
Embarked表示的是登船的地点字段；
Fare表示登船的票价字段；
Survived表示获救的人数字段；

这里是以“Embarked”为基准， 通过登船的地址 字段， 来查看，不同地点的总票价，  以及不同地点登船获救的人数统计。 通过的函数是<font color=#5F9EA0>np.sum</font>

####关于主动扔掉缺失值的操作

在实际中，有一些有缺失值的样本，我们需要主动的舍弃掉， 那么通过方法如下：


```python
drop_na_colums = titanic_survival.dropna(axis=1)
print drop_na_colums

```
这里表示把有缺失值的列都告诉你，  如果我们不是想扔掉全部的缺失值， 而是只关注某些维度的缺失值的话， 可以通过以下来做：

```python
new_titanic_survival = titanic_survival.dropna(axis=0, subset=["Age","Sex"])
print new_titanic_survival

```

这个实例就是表示，我们把Age和Sex中含有缺失值的样本数据给主动的drop掉。 axis=1表示全局维度， axis=0表示局部维度。

####准确获取数据的操作

比如，我们要定位第83号某列的数据

```python
row_index_83_age = titanic_survival.loc[83,"Age"]
row_index_766_pclass = titanic_survival.loc[766, "Pclass"]
print (row_index_83_age)
print (row_index_766_pclass)

```

我们定位了，第83关于Age的数据和第766关于Pclass的数据


####根据某一些进行排序，排序后重新设置index


```python
new_titanic_survival = titanic_survival.sort_values("Age", ascending=False)
print new_titanic_survival

titanic_reindexed = new_titanic_survival.reset_index(drop=True)
print titanic_reindexed


```

我们知道上述第一行是通过Age对titanic_survival进行重新排序后，得到新的DataFrame，也就是new_titanic_survival。
此时我们打印new_titanic_survival的index还是原来排序前的index值， 如果我们想改变这个index为我们排序后重新设置的index，
需要进行 <font color=#5F9EA0> reset_index(drop=True)</font> 的操作。

这里再打印titanic_reindexed，则为一个从0开始一个index排序的新DataFrame


####三、 关于Pandas其他的操作

####自定义操作

例一、 我们通过以下的内容自定义一个操作


```python
def hundredth_row(column):
    hundredth_item = column.loc[99]
    return hundredth_item

hundredth_row = titanic_survival.apply(hundredth_row)
print hundredth_row


```

通过apply() 声明一个自定义函数操作， 上面的函数比较简单就是返回第99号样本的元素。

同理， 我们将船舱的值1，2，3改为字符串 First Class，Second Class的形式返回。

```python
def which_class(row):
    pclass = row['Pclass']
    
    if pd.isnull(pclass):
        return "Unknown"
    elif pclass == 1:
        return "First Class"
    elif pclass == 2:
        return "Second Class"
    elif pclass == 3:
        return "Third Class"   

classes = titanic_survival.apply(which_class, axis=1)
print (classes)

```


