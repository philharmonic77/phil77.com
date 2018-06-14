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
一般用于分析人员本身具有行业知识，认为聚类的处理是必要的情况下。且数据集太小时，慎用。

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

在此例中， 使用先聚类在分类的方法未见明显作用，原因可能是数据量太小，聚类之后进一步缩小了样本量，导致模型过拟合。


类别|k=1 | k=2 | k=3
---|---|---|---
训练集AUC|0.837 |0.874 |0.918
训练集accuracy|0.852|0.889|0.928
kaggle accuracy|0.780|0.770|0.713

此外，一个自然的想法是，数据集本身是否存在一定的聚集模式。这里使用manifold learning将原始数据中的X值降维可视化，得到下图：

![image](/image/Using Clustering in the Pre-processing Procedure/t-SNE of Titanic clean.png)






