title: 58推荐系统学习
date: 2015-04-15 20:49:31
categories: Recommendation
tags:
---

## 1.推荐介绍
用户在某个场景下对某个商品或信息产生了某种行为，系统会对另一些商品或信息进行推荐。

## 2.推荐要素
1. 用户-user
2. 场景-scene
3. 商品或信息-item
4. 行为-action
5. 系统-recommendation-system
6. 推荐结果集合-recommendation-result / item-set <!--more-->

## 3.推荐概述

>- user-info：uid
- scene-info：entry、local、cateid
- item-info：jid
- action：post
- item-set：set<zid>

![](/img/Recommendation.png)

## 3.常用推荐方法

**协同过滤-CF**

- 协同过滤：collaborative filtering Recommendation
- 原理：用户的相似喜好进行推荐
- 举例：商家下载简历的推荐


|    | jid1 | jid2| jid3 | jid4 | jid5 | ... |jid1000W|
| -- | --- |-----|----|---|------|-----|---|
| user1  | yes | yes | yes |  |  | ...  | 
| user2  | yes | yes | yes | yes |  | ...|
| user3  |     |     |     |     | yes |...|
| user4  |     | yes |     | yes |  |...


**内容推荐**

- 内容推荐：content-based Recommendation
- 原理：抽取共有属性
- 举例：商家下载简历的推荐


步骤：

1. 历史行为收集
2. id详情查询
3. 共性内容挖掘（场景）
4. 推荐


注：  
**历史行为**  
jid1，download  
jid2，download  
zid1，post  

**详情**  
jid1（司机，北京，8000月薪，5年经验，NULL）  
jid2（司机，北京，NULL，2年经验，硕士研究生）  
zid1（司机，北京，NULL，3年经验，NULL）  

**共性**   
（司机，北京，NULL，2年经验，NULL）