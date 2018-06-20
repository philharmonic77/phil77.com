---
title: "聚类分析在有监督学习里的应用"
date: 2018-06-12
categories: ["R2.ai"]
tags: ["Clustering"]
author: "JIN Shuhan"

mathjax: true

menu:
  main:
    parent: "docs"
    weight: 1
---

### 先聚类再分类
一般用于分析人员本身具有行业知识，认为聚类的处理是必要的情况下，因为一般来说有监督学习利用了标签信息，效果是优于无监督学习的。

该方法训练很耗时，且数据集太小时容易产生过拟合，慎用。另外，在自动化流程中，需要谨慎控制每一类的样本量。

此文档中，聚类方法均使用kmeans，后续可拓展。

*参考文献：[The Utility of Clustering in Prediction Tasks](https://arxiv.org/abs/1509.06163)*

#### 简单数据集测试
> 数据集 [kaggle-Titanic](https://www.kaggle.com/c/titanic/data)

先做[特征提取](https://www.kaggle.com/arthurtok/introduction-to-ensembling-stacking-in-python)，得到全部都是数值型变量，且有解释性的训练数据和测试数据。
	
![image](/image/Using Clustering in the Pre-processing Procedure/training set.PNG)

##### 1. 直接采用分类模型

将training dataset用[R2-Learn](http://stage20.newa-tech.com:22000/modeling/169/2)进行训练，选择推荐的ensemble模型，
并在[deployment环节](http://stage20.newa-tech.com:22000/deploy/169)使用test dataset，得到y值的预测之后，上传到kaggle进行评分。

采用auto-sklearn 0.3.0以默认参数训练5min，同样使用kaggle评分。结果汇总如下：


 类别|R2-Learn | AutoML 0.3.0 
---|---|---
训练时间|5 min |5 min
训练集AUC|0.87 |0.84
kaggle accuracy|0.732 | 0.775

##### 2. 先对数据集进行聚类，再对各个类进行训练

总体思路见下图：

![image](/image/Using Clustering in the Pre-processing Procedure/screen shot of method.PNG)

###### 2.1 不使用ensemble的情况

k依次取1,2,3,..., 每一轮计算中，先将training data标准化，再聚成k类，然后分别使用auto-sklearn 0.3.0/random forest训练，最终汇集预测值，然后计算AUC。完成全部计算后，对比不同k值对预测的影响。找到预测效果较好的设定，并尝试解释聚类的合理性。

此例中，样本量较小，所以至多考虑3类。在样本量较大时，可以加入k值的自动选择机制。

在此例中， 使用先聚类在分类的方法~~未见明显作用~~，原因可能是数据量太小，聚类之后进一步缩小了样本量，导致模型**过拟合**。


类别|k=1 | k=2 | k=3
---|---|---|---
轮廓系数（+）| - |0.285|0.307
训练集AUC|0.837 |0.874 |0.918
训练集accuracy|0.852|0.889|0.928
kaggle accuracy|0.780|0.770|0.713

此外，一个自然的想法是，数据集本身是否存在一定的聚集模式。这里使用manifold learning将原始数据中的X值降维可视化，并使用轮廓系数最大情形下的聚类标签着色，得到下图。可见，该数据集并不具有天然可分性。

![image](/image/Using Clustering in the Pre-processing Procedure/t-SNE of Titanic clean.png)

##### 2.2 更换数据集

于是，我们改换数据量更大的数据进行实验,或者有明显聚集性的数据继续实验。

###### 2.2.1

数据集：[default of credit card clients](http://archive.ics.uci.edu/ml/datasets/default+of+credit+card+clients)， 3w行，25列。

训练集，测试集划分：70%， 30%。
使用k-means进行聚类，在k=2时，轮廓系数达到最高。降维可视化后的数据，仍是不具有明显聚集性的数据。

![image](/image/Using Clustering in the Pre-processing Procedure/credit_cluster.png)

训练时长统一为500s。结果并未显示聚类对效果有改进，相反时间耗费大大增加。


类别|k=1 | k=2 | k=3| k=4
---|---|---|---|---
轮廓系数（+）| - |0.355|0.188|0.165
训练集AUC|0.837|0.880|0.770|0.864
训练集accuracy|0.827|0.864|0.837|0.880
测试集AUC|0.784|0.752|0.702|0.713
测试集accuracy|0.822|0.819|0.824|0.818

###### 2.2.2

数据集：[Abalone](https://archive.ics.uci.edu/ml/machine-learning-databases/abalone/)，4177行，9列。

训练集，测试集划分：70%， 30%

$$\displaystyle\frac{x+y}{y+z}$$

$\displaystyle\frac{x+y}{y+z}$


