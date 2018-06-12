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
*参考文献：[The Utility of Clustering in Prediction Tasks](https://arxiv.org/abs/1509.06163)*

#### ==简单==数据集测试
> 数据集 [kaggle-Titanic](https://www.kaggle.com/c/titanic/data)

先做[特征提取](https://www.kaggle.com/arthurtok/introduction-to-ensembling-stacking-in-python)，得到全部都是数值型变量，且++有解释性++的训练数据和测试数据。
	
![image](/image/Using Clustering in the Pre-processing Procedure/training set.PNG)

将training dataset用R2-Learn进行训练，并在delpoyment环节

