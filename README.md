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

运行结果：

![](https://github.com/PaulxzHuang/Dataanalysis/blob/main/image/1.png?raw=true)

该数据集共有12256906条数据。为了便于分析，导入数据仅取其中200万条。此外，为了提高数据集的可读性对列名进行重命名：

        TBData=pd.read_csv('tianchi_mobile_recommend_train_user.csv',header=0,names=['用户ID','商品ID','行为类型','地理位置','商品种类ID','时间'],nrows=2000000)

输出TBData.info()：

![](https://github.com/PaulxzHuang/Dataanalysis/blob/main/image/2.png?raw=true)

### 2.2 缺失值及重复值处理

首先统计缺失值：

        print(TBData.apply(lambda x: sum(x.isnull())))

得到统计结果：

![](https://github.com/PaulxzHuang/Dataanalysis/blob/main/image/3.png?raw=true)

可见缺失值仅存在于地理位置列，本次分析从时间角度出发，不考虑地理位置的因素，因此此处直接将地理位置列与重复值一并删除。

        TBData=TBData.drop('地理位置',axis=1)
        TBData=TBData.drop_duplicates()

### 2.3 转化时间戳
时间列目前的类型为str字符串类型，为了后续基于时间的分析，必须将时间列更改为时间戳类型，并将日期和小时分开新建两列：

        TBData['时间']=pd.to_datetime(TBData['时间'])
        TBData['日期']=pd.to_datetime(TBData['时间']).dt.date
        TBData['日期']=pd.to_datetime(TBData['日期'])
        TBData['小时']=pd.to_datetime(TBData['时间']).dt.hour
        TBData['小时']=TBData['小时'].astype(int)

### 2.4 行为类型修改
根据官方提供的说明，行为类型列数字1、2、3、4分别代表浏览、收藏、加购物车及购买四种用户行为，为了提高数据的可读性此处将数字替换为文本。

        TBData['行为类型']=TBData['行为类型'].replace(1,'浏览').replace(2,'收藏').replace(3,'购物车').replace(4,'购买')

修改后的数据信息：

![](https://github.com/PaulxzHuang/Dataanalysis/blob/main/image/4.png?raw=true)

数据清洗完成，下一步开始数据分析工作。

## 三、数据分析
### 3.1 独立访客(UV)购买转化率
计算各个行为类型的独立访客数量：

        UV= TBData['用户ID'].drop_duplicates().count()
        CartUV=TBData[TBData['行为类型']=='购物车']['用户ID'].drop_duplicates().count()
        ColUV=TBData[TBData['行为类型']=='收藏']['用户ID'].drop_duplicates().count()
        BuyUV=TBData[TBData['行为类型']=='购买']['用户ID'].drop_duplicates().count()

绘制条形图：

        xlabel_list=['TotalUV','CartUV','ColUV','BuyUV']
        ylabel_list=[UV,CartUV,ColUV,BuyUV]
        plt.title('各环节UV数量')
        ax=plt.bar(xlabel_list,ylabel_list,facecolor='g',alpha=0.8,width=0.6)
        for i in range(len(xlabel_list)):
                plt.text(xlabel_list[i],ylabel_list[i],ylabel_list[i],ha='center',va='bottom')
        plt.ylabel('人数')
        plt.xlabel('行为类型')
        plt.show()

![](https://github.com/PaulxzHuang/Dataanalysis/blob/main/image/5.png?raw=true)

有购买行为的用户数占浏览用户总数：BuyUV/TotalUV=6044/9969=60.6%

### 3.2 页面访问量(PV)购买转换率
计算总的访问量及用户购买次数：

        TotalPV=TBData[TBData['行为类型']=='浏览']['用户ID'].count()
        Buycounts=TBData[TBData['行为类型']=='购买']['用户ID'].count()

得到总的访问量：TotalPV=1636876
用户购买次数：Buycounts=18925
转换率为Buycounts/TotalPV=1.15%
平均86次点击才会产生一次购买行为。

尽管付费用户转化率较高，达到60%以上，但是用户购买行为发生所需要的浏览次数同样过高，可能的原因是：

1、推送的商品与用户的期望匹配值不高，用户需要大量的点击才能找到需要的商品；
2、由于重复的推送过多，导致某些用户重复点击了某件商品，致使总访问量虚高。

### 3.3 日PV和日UV 
计算日均页面浏览次数和日均独立访客数量：
        
        Dailypv=TBData.groupby('日期')['用户ID'].count()
        Dailyuv=TBData.drop_duplicates(subset=['日期','用户ID']).groupby('日期')['用户ID'].count()
        PV_UV_daily=pd.concat([Dailypv,Dailyuv],axis=1)
        PV_UV_daily.columns=['PV','UV']
        
绘制PV和UV的曲线图，观察变化趋势：

        plt.subplot(211)
        plt.plot(Dailypv,color='r')
        plt.title('日PV')
        plt.subplot(212)
        plt.plot(Dailyuv,color='b')
        plt.title('日UV')
        plt.show()

![](https://github.com/PaulxzHuang/Dataanalysis/blob/main/image/6.png?raw=true)

PV和UV的变化大致上是同步的，且日PV和日UV都在双十二购物节达到峰值，其余时间数量的变化较不明显。

### 3.4 一周内用户购买量的变化
用户在一周内的购物情况是值得研究的，为了避免购物节的影响，此处取11月24日到30日的数据进行绘图。

        DatabyDay = TBData[['日期','行为类型']] 
        DatabyDay=DatabyDay.set_index('日期')
        Oneweek=DatabyDay[DatabyDay['行为类型']=='购买']['2014-11-24':'2014-11-30'].groupby('日期').count()
        plt.plot(Oneweek)
        plt.xticks(Oneweek.index,['星期一','星期二','星期三','星期四','星期五','星期六','星期天'])
        plt.show()

得到一周内购买次数变化曲线：
        
![](https://github.com/PaulxzHuang/Dataanalysis/blob/main/image/7.png?raw=true)

用户在周四的购买量是最高的，而在周五周六达到低谷。
用户趋向于周四购物的原因可能是：

1、考虑到运送时间和收件的方便程度，周四下单的快递到达目的地的时间恰好是周六左右，双休日便于接收。

2、周五周六用户的社交活动可能较多，社交活动与网购的时间产生冲突。

为了验证以上猜测，进一步对网购的具体时间进行分析。

### 3.5 用户一天中浏览及下单的时段分析
先将每小时的PV和UV取出，构成列表：

        Hourlypv=TBData.groupby('小时')['用户ID'].count()
        Hourlyuv=TBData.drop_duplicates(subset=['小时','用户ID']).groupby('小时')['用户ID'].count()
        PV_UV_hourly=pd.concat([Hourlypv,Hourlyuv],axis=1)
        PV_UV_hourly.columns=['每小时PV','每小时UV']

绘制曲线：
        plt.figure(figsize=(16,9))
        P1,=plt.plot(PV_UV_hourly['每小时PV'],label='每小时页面浏览量')
        plt.ylabel('每小时浏览总量')
        plt.xlabel('时刻')
        plt.twinx()
        P2,=plt.plot(PV_UV_hourly['每小时UV'],label='每小时浏览用户数',color='y')
        plt.ylabel('每小时浏览用户数')
        plt.legend(handles=[P1,P2],labels=['每小时浏览总量','每小时浏览用户数'])
        plt.xticks(range(0,24))
        plt.show()

![](https://github.com/PaulxzHuang/Dataanalysis/blob/main/image/8.png?raw=true)

由曲线可见，每小时PV和UV大致成正相关，都在18时开始明显上升，并在21时左右达到峰值，之后逐步下降，在凌晨4时达到谷底。

## 四、结论
客户购物的转化率很高，但总浏览量是总购物次数的86倍，无效的浏览量过多，应当提高用户的搜索体验，精确找到目标商品，具体可以通过以下方式：

1、优化广告推送机制，使其更加匹配用户的搜索习惯；

2、过多的无效浏览量可能产生于商品的详情信息不明确，应当减少浮夸的文字及配图等次要信息，强调商品的关键参数；

3、部分商家会使用在标题及缩略图上对商品进行虚假标价的方式营销，在进入详情页面后用户会发现实际价格要高于营销的价格，平台应当加强对商家虚假营销行为的管理。

结合一周内的客户购物数量变化曲线，大多数用户习惯于下班后浏览购物网站，由于周五下班后及周六客户的社交等活动较多，与网购时间发生冲突，周五周六的购物行为会减少。综合以上考量，广告投入在一周中以周三四为最佳，最好避开周五周六。在一天中，最佳的广告投放时间是用户下班后的晚间时段。


