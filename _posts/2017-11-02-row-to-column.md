---
layout: post
title: "HIVE: lateral view explode & json_turpe 实现 json数组行转列&字段拆分"
description: ""
category: tech
tags: [hive,json]
---
{% include JB/setup %}

# 问题描述
有时候因为业务的需要，有些字段不但是json格式，并且还是个json数组，比如下表 pay_infos:

| pay_id        | infos           | 
| ------------- |:-------------:| 
| 1111      | [{"uid":123,"terminalFrom":0,"couponBatchId":1410115799,"cost":5},{"uid":123,"terminalFrom":0,"couponBatchId":1410116199,,"cost":7,}] | 
| 1112      | [{"uid":124,"terminalFrom":1,"couponBatchId":1410115799,"cost":20}] | 

为了统计的需要，需要解析出每个json的具体key和value，比如上表，我需要求出，所有couponBatchId分类的cost的值。

# 解决办法
* explode

对于infos字段，首先应该解决数组的行转列问题，这个交给explode函数

```
select 
  explode(split(substring(infos,3,length(coupon_enties)-4),'\\},\\{'))  as entity
from pay_infos
```
通过explode一次处理infos按照单个json的字符串转换成多行，如下

| entity | 
| ------------- |
|  "uid":123,"terminalFrom":0,"couponBatchId":1410115799,"cost":5 |
| "uid":123,"terminalFrom":0,"couponBatchId":1410116199,,"cost":7 | 
| "uid":124,"terminalFrom":1,"couponBatchId":1410115799,"cost":20| 

如果需要保留其他字段信息，则使用lateral view

```
select pay_id,concat('{',entity,'}') as coupons
from
(       select * 
       from pay_infos
) a 
lateral view explode(split(substring(infos,3,length(infos)-4),'\\},\\{')) b as entity
```

结果如下：

| pay_id        | coupons | 
| ------------- |:-------------:| 
| 1111      | "uid":123,"terminalFrom":0,"couponBatchId":1410115799,"cost":5 | 
| 1111      | "uid":123,"terminalFrom":0,"couponBatchId":1410116199,,"cost":7 | 
| 1112      | "uid":124,"terminalFrom":1,"couponBatchId":1410115799,"cost":20 | 

* json_turpe

对于key:value格式的信息，可以通过拼成完成的json之后使用，json_turpe，也就是行转列，比如上面输出的表称为source，栗子SQL如下：

```
select info.*
from 
(
  select concat('{',entity,'}') as coupons
  from source
  lateral view json_tuple(coupons,'couponBatchId','terminalFrom','cost','uid') info 
  as couponBatchId,terminalFrom,cost,uid
```

结果如下：

| couponBatchId | terminalFrom | cost | uid | 
| ------------- | :------------:|:-------------:|:-------------:| 
| 1410115799    | 0 | 5 | 123 | 
| 1410116199    | 0 | 7 | 123 | 
| 1410115799    | 1 | 20 | 124 | 

# 完整SQL
```
select info.*
from 
(
  (select 
  pay_id,concat('{',entity,'}') as coupons
  from
  (       
    select *  from  pay_infos
  ) a 
  lateral view explode(split(substring(infos,3,length(infos)-4),'\\},\\{')) b as entity) source
  lateral view json_tuple(coupons,'couponBatchId','terminalFrom','cost','uid') info 
  as couponBatchId,terminalFrom,cost,uid
```

# 说明
* concat用于补出完整json
* split时，分隔符需要转义，如果sql写在" "内(如：shell脚本里调用hive -e "$sql"的情形)，则需要4个' \ '转义特殊字符，即：```split(infos,'\\\\},\\\\{')```
