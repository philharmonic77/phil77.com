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
# 欢迎使用马克飞象

@(web links from PC)[马克飞象, 帮助, Markdown]

**马克飞象**是一款专为印象笔记（Evernote）打造的Markdown编辑器，通过精心的设计与技术实现，配合印象笔记强大的存储和同步功能，带来前所未有的书写体验。特点概述：
 
- **功能丰富** ：支持高亮代码块、*LaTeX* 公式、流程图，本地图片以及附件上传，甚至截图粘贴，工作学习好帮手；
- **得心应手** ：简洁高效的编辑器，提供[桌面客户端][1]以及[离线Chrome App][2]，支持移动端 Web；
- **深度整合** ：支持选择笔记本和添加标签，支持从印象笔记跳转编辑，轻松管理。

-------------------

[TOC]

## Markdown简介

> Markdown 是一种轻量级标记语言，它允许人们使用易读易写的纯文本格式编写文档，然后转换成格式丰富的HTML页面。    —— [维基百科](https://zh.wikipedia.org/wiki/Markdown)

正如您在阅读的这份文档，它使用简单的符号标识不同的标题，将某些文字标记为**粗体**或者*斜体*，创建一个[链接](http://www.example.com)或一个脚注[^demo]。下面列举了几个高级功能，更多语法请按`Ctrl + /`查看帮助。 

### 代码块
``` python
@requires_authorization
def somefunc(param1='', param2=0):
    '''A docstring'''
    if param1 > param2: # interesting
        print 'Greater'
    return (param2 - param1 + 1) or None
class SomeClass:
    pass
>>> message = '''interpreter
... prompt'''
```
### LaTeX 公式

可以创建行内公式，例如 $\Gamma(n) = (n-1)!\quad\forall n\in\mathbb N$。或者块级公式：

$$	x = \dfrac{-b \pm \sqrt{b^2 - 4ac}}{2a} $$

### 表格
| Item      |    Value | Qty  |
| :-------- | --------:| :--: |
| Computer  | 1600 USD |  5   |
| Phone     |   12 USD |  12  |
| Pipe      |    1 USD | 234  |

### 流程图
```flow
st=>start: Start
e=>end
op=>operation: My Operation
cond=>condition: Yes or No?

st->op->cond
cond(yes)->e
cond(no)->op
```

以及时序图:

```sequence
Alice->Bob: Hello Bob, how are you?
Note right of Bob: Bob thinks
Bob-->Alice: I am good thanks!
```

> **提示：**想了解更多，请查看**流程图**[语法][3]以及**时序图**[语法][4]。

### 复选框

使用 `- [ ]` 和 `- [x]` 语法可以创建复选框，实现 todo-list 等功能。例如：

- [x] 已完成事项
- [ ] 待办事项1
- [ ] 待办事项2

> **注意：**目前支持尚不完全，在印象笔记中勾选复选框是无效、不能同步的，所以必须在**马克飞象**中修改 Markdown 原文才可生效。下个版本将会全面支持。


## 印象笔记相关

### 笔记本和标签
**马克飞象**增加了`@(笔记本)[标签A|标签B]`语法, 以选择笔记本和添加标签。 **绑定账号后**， 输入`(`自动会出现笔记本列表，请从中选择。

### 笔记标题
**马克飞象**会自动使用文档内出现的第一个标题作为笔记标题。例如本文，就是第一行的 `欢迎使用马克飞象`。

### 快捷编辑
保存在印象笔记中的笔记，右上角会有一个红色的编辑按钮，点击后会回到**马克飞象**中打开并编辑该笔记。
>**注意：**目前用户在印象笔记中单方面做的任何修改，马克飞象是无法自动感知和更新的。所以请务必回到马克飞象编辑。

### 数据同步
**马克飞象**通过**将Markdown原文以隐藏内容保存在笔记中**的精妙设计，实现了对Markdown的存储和再次编辑。既解决了其他产品只是单向导出HTML的单薄，又规避了服务端存储Markdown带来的隐私安全问题。这样，服务端仅作为对印象笔记 API调用和数据转换之用。

 >**隐私声明：用户所有的笔记数据，均保存在印象笔记中。马克飞象不存储用户的任何笔记数据。**

### 离线存储
**马克飞象**使用浏览器离线存储将内容实时保存在本地，不必担心网络断掉或浏览器崩溃。为了节省空间和避免冲突，已同步至印象笔记并且不再修改的笔记将删除部分本地缓存，不过依然可以随时通过`文档管理`打开。

> **注意：**虽然浏览器存储大部分时候都比较可靠，但印象笔记作为专业云存储，更值得信赖。以防万一，**请务必经常及时同步到印象笔记**。

## 编辑器相关
### 设置
右侧系统菜单（快捷键`Ctrl + M`）的`设置`中，提供了界面字体、字号、自定义CSS、vim/emacs 键盘模式等高级选项。

### 快捷键

帮助    `Ctrl + /`
同步文档    `Ctrl + S`
创建文档    `Ctrl + Alt + N`
最大化编辑器    `Ctrl + Enter`
预览文档 `Ctrl + Alt + Enter`
文档管理    `Ctrl + O`
系统菜单    `Ctrl + M` 

加粗    `Ctrl + B`
插入图片    `Ctrl + G`
插入链接    `Ctrl + L`
提升标题    `Ctrl + H`

## 关于收费

**马克飞象**为新用户提供 10 天的试用期，试用期过后需要[续费](maxiang.info/vip.html)才能继续使用。未购买或者未及时续费，将不能同步新的笔记。之前保存过的笔记依然可以编辑。


## 反馈与建议
- 微博：[@马克飞象](http://weibo.com/u/2788354117)，[@GGock](http://weibo.com/ggock "开发者个人账号")
- 邮箱：<hustgock@gmail.com>

---------
感谢阅读这份帮助文档。请点击右上角，绑定印象笔记账号，开启全新的记录与分享体验吧。




[^demo]: 这是一个示例脚注。请查阅 [MultiMarkdown 文档](https://github.com/fletcher/MultiMarkdown/wiki/MultiMarkdown-Syntax-Guide#footnotes) 关于脚注的说明。 **限制：** 印象笔记的笔记内容使用 [ENML][5] 格式，基于 HTML，但是不支持某些标签和属性，例如id，这就导致`脚注`和`TOC`无法正常点击。


  [1]: http://maxiang.info/client_zh
  [2]: https://chrome.google.com/webstore/detail/kidnkfckhbdkfgbicccmdggmpgogehop
  [3]: http://adrai.github.io/flowchart.js/
  [4]: http://bramp.github.io/js-sequence-diagrams/
  [5]: https://dev.yinxiang.com/doc/articles/enml.php

