分析此代码有如下值得学习的地方：
- 【时间序列】原始数据是时间序列类型的，应该不随便打乱这种顺序。
  - 拆分训练、验证、测试集的时候不应该打乱后拆，而应该按时间拆分。
  - 得到计数count特征的时候，也应该反映时间性：比如可以统计一些feature在不同时间段（小时、天）的计数；即使在同一个时间段内，可以得到总计数，但也可以“针对单个样本得到截止该样本时间点的该时间段内的计数”（script/addc.py中就是这样处理的）
- 【数据拆分】可以把原始数据按某些维度拆成多份，每份训练单独一个model，最后merge(这里有按app_id/site_id做拆分，排第一的方案，也是这样！)
- 【低频、低频特征】同一个categorical特征下的低频0/1特征（出现次数小于某个阈值比如100或10），可以统一归为一个特征（类似于词表外所有词统一为 UNKNOW_TOKEN）。但是还可以更细化：对低频特征取值，按出现次数分为多类，假如高低频阈值是100，那么频数低于100的特征取值，可以按计数分为99类。这样，one-hot后，这类特征就是占据100 bits，而不是1个。这样应该能更好刻画数据。另外，必要时也可以所有高频归为同一个值，低频不变。
- 用 FTRL 模型，则做特征组合
- 用 FM/FMM, 不做特征组合
- GBDT 特征（塞给他非categorical特征，即使count值特征也算real值的，可以不用归一化直接用GBDT），可以直接和其他特征一起，给下一步用
- 数据分析：应该学会怎样分析数据。
  - 前三名方案，都发现了C14， C17很重要；前两名方案都发现了C21很重要。（看kaggle论坛，第一名方案说是用排除法多次训练把不重要的特征去掉；3rd方案是通过分析得出C14，C17背后的实际含义。不管怎样，他们都能找出！）
  - 前几名都发现了按 site_id/app_id 拆分数据。1st 与 3rd 在拆分上简直如出一辙。都是拆分时 site_id/site_domain/site_category 与 app_id/app_domain/app_category 这两组只取一组
  - user_id: 1st 与 3rd 在确认user_id 怎么选取上也完全一样
- 【特征 hash trick】塞给 FM/FMM/LR(FTRL)模型的特征数据，可以是特征hash后的。这样不管原始有多少特征，总数量是可以得到控制的。


以下是原始 README.md
------------
Random Walker's solution for Avazu Click-Through rate prediction

The introduction of our approach could be found in doc.pdf.

System Requirement
------------------
- 64-bit Unix-like os
- Python 2.7
- g++
- pypy
- sklearn
- at least 20GB memory and 50GB disk space

To reproduce our submission:
-------------------
- Download tha data("train" and "test") to this folder.
- Change directory to script:
	cd script
- Run the code:
	./run.sh
