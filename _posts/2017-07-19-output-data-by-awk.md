---
layout: post
title: "活用awk出数据"
description: ""
category: tech
tags: [shell,awk]
---
{% include JB/setup %}

### 问题表述
最近业务线故障，一些用户受到了影响。产品经理急切需要知道，这次事故是否导致用户流失，换句话希望看看这些用户近一个星期的购物留存。所以产品经理给数据工程师提供了一个大概有1.4万的受影响用户id的文本文件accidentIds.txt，希望通过购买记录表（order_table）知道从事故发生当天开始，7天内这部分人群的再次消费人数和消费金额。数据按天给出。

- accidentIds.txt数据形式。

```
123
234
345
```
- order_table表结构

| create_time  | user_id  | purchase |
| :------------ |:---------------:| -----:|
| 2017-07-10 10:20:29      | 123 | 90 |
| 2017-07-10 23:23:23      | 123 | 30 |
| 2017-07-11 13:23:23      | 345 | 60 |
| 2017-07-12 03:23:23      | 234 | 130 |

### 分析
用户购买记录属于业务表，通常会从mysql抽到hive当中来进行统计计算。而需求方提供的却是一个文本文件，完成这个需求，其实最笨的一个做法都是在hive里建一个临时表，然后把文件load进去，连表查询就行了。但是这样做，真的很麻烦，出数据的效率也不高。
这时我们应该想起Linux的一个高效工具——awk。
如果我们是从hive中把按天、用户id、够买金额总和的形式把数据导出到服务器，然后再awk计算，就会方便会多，出数据简直是秒级别。
### 具体步骤
假设当前工作目录，既是最后输出目录。且已经把accidentIds.txt拷贝到当前目录。接下来：

- 把数据导出到buylist.txt


```sh
hive -e " 
use database_a;
select substr(create_time,1,10) as dt,user_id,sum(purchase) 
from order_table 
group by substr(create_time,1,10),user_id
">buylist.txt
```

- awk处理


```sh
awk 'NR==FNR{ 
   //把accidentIds.txt的id存到uidDict数组中，且下标用id标识
  uidDict[$1]=$1; }
NR>FNR{
  //对于buylist.txt中的记录，$2表示user_id只要在uidDict中出现，表示有够买
if(uidDict[$2]!=""){
  //$1是够买日期
  day[$1]=$1;
  //userCount是按日期计数数组，表示够买过的人数
  userCount[$1]++;
  //$3表示某个人某天的够买金额总和,purchase是按日期累加够买金额
  purchase[$1]=purchase[$1]+$3;}} 
END{ 
  //按天输出
for (i in day) {
  print day[i],userCount[i],purchase[i]
}}' accidentIds.txt buylist.txt
```

### 输出

```
2017-07-10 123 120
2017-07-11 345 60
2017-07-12 234 130
```
### 结语
对于1.4万条数据的accidentIds.txt和38万条数据的buylist.txt，awk命令的计算结果简直是秒出。
而用hive连表，首先导表，再连表查询，怎么着也得折腾个半个小时。
所以用好awk真心省时省力啊！！！