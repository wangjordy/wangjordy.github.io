---
layout: post
title: "机器学习---关于前面学习几个库的综合使用"
date: 2017-04-27 19:11:17 +0800
comments: true
categories: 
-  Python
-  MachineLearning
---

前面我们花了3节的时间介绍了关于机器学习常用的<font color=#008B8B>numpy、pandas、matplotlib</font>库。 今天我们综合介绍一下它们的联合使用。

<!--more-->

##关于信用卡欺诈的案例分析

我们今天读取一个信用卡的统计信息， 来分析样本的信息。

下面的代码中， ‘Class’列的数据信息， 0表示正常，1表示异常， 那么会表现出来这个样本数据是极度的不均衡的数据（均衡的数据，比方一些网站信息统计男女比率等）

```python
import pandas as pd

unrate = pandas.read_csv("xxx.csv")  //读取某一个csv文件

unrate['DATE'] = pd.to_datetime(unrate['DATE'])
print (unrate.head(12))       //打印出来是DataFrame（数据流）

data = pd.read.csv("creaditcard.csv")  #读取信用卡的记录
data.head()

#对Class列的数据进行排序， 那组数据多就排到最前面
count_classes = pd.value_counts(data['Class'], sort=True).sort_index()
#直接用pandas来画图  bar：柱状图
count_classes.plot(kind='bar')   
plt.title("Fraud class histogram") #设置一个标题
plt.xlabel("Class")    #设置横纵坐标
plt.ylable("Frequency")

```

通过上图我们可以得到一个柱状图，显示出现正常数据和异常数据之前的分布关系。
由于这种极度不均衡的数据，假如一共1000条的数据， 其中0表示的数据有990条， 1表示的数据有10条， 那么在设置训练集中，很容易建立出来一个模型， 要么这个‘class’都是0， 要么这个‘class’的精度是99%， 但在实际中缺没有任何实际的意义。

那么，对于这种情况这里可以用到的常规方案：
 
 *  <font color=#0000FF>下采样方案</font> （让两端的数据一样的少， 比如在990条中选出来10条0的数据， 再与1的数据10条组合成训练集， 再训练模型）
 *  <font color=#0000FF>过采样方案</font>（让两端的数据一样的多， 设置将1的条数，变换出来990条， 让1的数据与0的数据一样的多， 再训练模型）

当然还有其他的方案，比方按照比率关系来采样。

###预处理的操作

####1、标准化操作（归一化操作）

一些特征数据的值差异比较大，  比如：Amount字段， 有的数据是140.3， 有的数据只有2.34， 那么我们通过以下代码操作：
 
```python
from sklearn.pregrocessing import StandardSCaler

#这里我们将原始数据Amount列中的数据，通过StandardSCaler类将Amount里面所有数据转化到-1到1期间的一个值
#并将转化后的值重新赋值给normAmount列
data['normAmount'] = StandardScaler().fit_transform(data['Amount'].reshape(-1,1))
#我们将原始不再使用的列去掉
data = data.drop(['Time', 'Amount'], axis=1)
data.head() #最后查看一下数据转化后的信息

```

将里面一个列的数据，转到到一个区间上。 并删除之前不用关注或者不再使用的列。

<font color=#FF7F50>重点： 标准化的操作很重要，可以消除数据本身单位造成的影响。</font>   

例如:

一组数据中有A,B列，  A列中的数据都是0-1之间的数据， B列中的数据是500-1000的数据。
如果我们不做标准化的处理， 那么根据线性方程计算公式

<font color=#006400>y = w1 * x1 + w2 * x2 </font>

(假如不考了w1，w2权重的情况)，  x1代表A列数据特征， x2代表B列数据特征；

那么由于A,B列的数据值本身的取值范围就不同，  假如在w1=w2的情况， x2如果不做标准化的处理，那么计算出来的结果，会使得这个y更收x2的影响，  所以我们为了保证两者的取值范围相同，就需要先使用这种标准化的预处理的操作。



####2、下采样方案

首先将所有class=0的数据，和class=1的数据分开。

```python
#x,y 代表特征和label分开
x = data.ix[:, data.columns != 'Class']
y = data.ix[:, data.columns == 'Class']

#统计class==1的个数
number_records_fraud = len(data[]data.Class==1)


#再把所有class等于1的index值都找出来
fraud_indices = np.array(data[data.Class ==1].index)
#接着把所有class等于0的index值也找出来
normal_indices = np.array(data[data.Class ==0].index)

```

接着，我们在所有class=0的数据，随机找出来n条数据， （n的值与class=1的所有数据的个数相等）


```python

#在class=0的所有数据集中，随机选出来与class=1的个数相同的 数据来， 
random_normal_indices = np.random.choice(normal_indices, number_records_fraud, replace=False)
random_normal_indices = np.array(random_normal_indices) #最后将选出来的数据 组合为一个numpy的array格式的数据

```

最后，我们将class=1和class=0的数据，组合在一起（形成一个新的训练集）


```python
#组合成一个新的训练集
under_sample_indices = np.concatenate([fraud_indices, random_normal_indices])

#获取到这批新的数据集合
under_sample_data = data.iloc[under_sample_indices,:]

#这里X和Y的值  都是under_sample的结构（通过下采样的方式得到了）
X_undersample = under_sample_data.ix[:, under_sample_data.columns != 'Class']
Y_undersample = under_sample_data.ix[:, under_sample_data.columns == 'Class']

```

最后获取了下采样后的class=0和class=1的数据集了

####3、对采样的数据进行分割

这里我们使用的一个新的库 sklearn,  比如我们的数据一共有100个， 我们可以通过sklearn，将其中3/4的数据当做训练集， 将1/4的数据当做测试集， 当然这个分割为几份是我们自定义的。



```python
from sklearn.cross_validation import train_testsplit

#这个是对所有的数据进行一个拆分
#这个方法就是需要对我们的X,Y进行划分，  并且设置test_size=0.3 设置了测试集占30%， random_state 0表示指定了随机的种子来验证
X_train, X_test, Y_train, Y_test = train_test_split(X,Y, test_size=0.3, random_state=0)

print ("X的训练集是多少：", len(X_train))
print ("X的测试集是多少：", len(X_test))

#这里对处理了下采样的数据，进行一个数据拆分
X_train_undersample, X_test_undersample, Y_train_undersample, Y_test_undersample = train_test_split(X_undersample,Y_undersample, test_size=0.3, random_state=0)

print ("下采样后的X的训练集是多少：", len(X_train_undersample))
print ("下采样后的X的测试集是多少：", len(X_test_undersample))
```

