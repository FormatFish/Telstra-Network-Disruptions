# 数据准备和探索
- 缺失值处理
- 异常值检查
- 为类型变量添加隐含变量，并将其数字化
- 数据透视表来对分表检测，并寻找相应的特征，包括在某个特征中训练数据占的比例，以及某个特征下多数的类别标签是什么（majority votes）
- 可视化数据分布，并对异常分布（某些值太大）进行log运算转换
- 将连续数据进行归类，比如location，先将其转换为value_counts ， 然后根据数量的多少设置阈值| 还有就是对于类别过多的特征，将数量少的归类到“其他”类别

# 使用XGBoost预训练，观察结果，并提交，决定是否基于此模型继续探索
# 特征工程
考虑时间因素，此处的特征工程主要基于由XGBoost得到的features importance中前n个影响最大的特征来继续探索
## 时间序列的特征提取
### lag features
```py
    pandas.DataFrame.shift
    pandas.DataFrame.rolling
    pandas.DataFrame.expanding
```
偏移原数据(severity type),并构建统计量
### pattern based on features(severity, log feature , event type)
此处将features以one-hot编码的形式，嵌入到一个pattern中，并将其作为新的特征

# Ensemble
## 过程
1. xgboost 在特征工程之后进行调参，调参完毕后开始使用stacking技术来对数据集进行处理
2. 基于上面得到的结果，再次使用xgboost进行训练，根据结果决定是否再调参，然后设置不同的seed，进行bagging
3. 训练随机森林（gini , entropy）和 Extra-tree (gini , entropy),基于原始的train和test集合，同时进行调参，使得结果最优。
4. 将得到的5个模型按照序列进行stacking，同时每个模型基于上一次得到的数据集进行训练。
5. 使用xgboost对得到的最后的数据集进行fit，同时根据结果决定调参
6. 然后为xgboost设置不同的seed，进行bagging
7. 最后根据LB score的反馈再次bagging


## stacking and bagging
构建多个模型，并对数据进行训练，同时进行调参
1. 随机森林
    - gini
    - entropy
2. Extra-Tree
    - gini
    - entropy
3. XGBoost
    - 采用多个不同的seed进行训练，得到多个不同的结果，然后使用bagging的原理将所有的结果整合起来
4. 逻辑回归

### stacking
1. 将训练数据分成两半
2. 使用模型利用训练集的一半来预测另一半
3. 将两份训练集反过来，用同一个模型再训练一次
4. 然后用同样的模型将整个训练集训练一次，同时预测测试集
5. 将以上三个模型的预测结果放在其对应的测试集中，然后得到一份新的训练集和新的测试集

按照上面得到的新的训练集和测试集，再用另外一个xgboost进行训练，从而提高了模型的精度

同时将随机森林和extra-tree按照上面的方法，将两个模型的结果也放在新的训练集和测试集上，从而又得到一份新的训练集和测试集，注意，这里的随机森林和extra-tree并不是并联的，而是串联的方式，extra-tree的训练集中已经有了随机森林的stack了

然后将得到的训练集和测试集再使用xgboost进行训练，继而进行调参，然后再进行一步bagging，得到最终结果。

## 调整XGBoost正则化相关参数
得到最后结果
## ensemble
然后根据LB score的反馈，对部分模型再进行bagging，得到最终结果

## 难点
- 调参
- 过拟合

## 结果
top 12% 119/975
![](https://ws2.sinaimg.cn/large/006tKfTcgy1fhu01ta7tij30qg0upq6a.jpg)
![](https://ws2.sinaimg.cn/large/006tKfTcgy1fhu02nltykj30qg0ritba.jpg)
