电商APP用户行为数据分析
========
本次分析基于阿里云天池数据平台——淘宝用户行为推荐数据。使用的EDA为eclipse。

数据集来源：https://tianchi.aliyun.com/dataset/dataDetail?dataId=649&userId=1


## 一、提出问题

1．	使用常见的分析指标，如PV,UV，分析用户从浏览到购买的转化情况；
2．	计算各个环节的用户流失情况，找出购买流程中可能存在的问题；
3．	从时间的角度刻画用户的使用习惯，制定营销策略；


## 二、数据清洗

### 2.1 导入数据
        #encoding:utf-8
        import pandas as pd
        import numpy as np
        import matplotlib.pyplot as plt
        plt.rcParams['font.sans-serif'] = ['SimHei']
        plt.rcParams['axes.unicode_minus'] = False
        TBData= pd.read_csv('tianchi_mobile_recommend_train_user.csv', header=0)
        
 查看数据集信息：
 
        print(TBData.info())

结果：
        

