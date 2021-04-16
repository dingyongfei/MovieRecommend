# MovieRecommend
This is a movie recommend system project.



项目数据流顺序:
在后端代码中做日志埋点, 日志中的评分数据格式为UID|MID|SCORE|TIMESTAMP -> 通过flume从业务系统文件夹下的一个access.log文件中采集日志后, 输出到Kafka Stream的”log” topic中 -> 通过Kafka Stream的处理后, 将评分数据输出到”recommender” topic中 -> Spark Streaming订阅Kafka Stream的”recommender” topic, 再进行实时的流式处理.

项目的几个功能模块包括:
1. 基于统计的推荐模块 (Spark sql)
2. 基于LFM [隐语义模型] (用到了ALS [最小交替二乘法]算法训练模型) 的离线推荐模块. 并且对模型进行了评估 (评估的标准是计算均方根误差RMSE)，选取了最优参数 (包括: rank特征矩阵维度, iterations迭代次数, lambda正则化系数). ---> 注, 在该模块中, 用ALS算法训练出的模型会产生出一个副产品 --- 电影的相似度矩阵! 该矩阵用于了后面的实时推荐模块
3. 基于Spark Streaming的实时推荐模块. 在该模块中使用到了上个离线推荐模块中的电影相似度矩阵, 同时我们使用了一个自定义的模型来做最终的实时推荐. 该自定义模型是:


 ![image](https://user-images.githubusercontent.com/34892973/114959129-010fe000-9e97-11eb-83a1-b4c439ca779c.png)

 
(在Redis集群中存储了每一个用户最近对电影的K次评分, 实时算法可以快速获取 [通过lrange命令获取].)

4. 基于内容的推荐模块 (基于TF-IDF). 即先通过电影表Movie中的genres字段提取出电影的原始特征矩阵 (是一个个稀疏的特征向量), 再用TF-IDF从内容信息中提取出电影特征向量. 后面的业务代码就和实时推荐模块一模一样了……

注, 所有的数据最终都写入到了MongoDB数据库中.
