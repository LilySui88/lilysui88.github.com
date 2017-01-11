---
layout: post
title: "Hive中with cube、with rollup、grouping sets用法"
description: ""
category: tech
tags: [hive]
---
{% include JB/setup %}

### 表结构

```
CREATE TABLE test (f1 string,  
                   f2 string,  
                   f3 string,  
                   cnt int) ROW FORMAT delimited FIELDS TERMINATED BY '\t' stored AS textfile;  
LOAD DATA LOCAL inpath '/data/logs/suiyingli/tmp/test.data' overwrite INTO TABLE test;  
```

### 原始数据

•A       A       B      1
•B       B       A      1
•A       A       A      2

### 1、with cube

#### 查询语句

```
SELECT f1,  
       f2,  
       f3,  
       sum(cnt),  
       GROUPING__ID,  
       rpad(reverse(bin(cast(GROUPING__ID AS bigint))),3,'0')  
FROM test  
GROUP BY f1,  
         f2,  
         f3 WITH CUBE;  
```

#### 结果

![with cube查询结果](http://upload-images.jianshu.io/upload_images/3367144-f1b11b559e732a3a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2、with rollup

#### 查询语句

```
SELECT f1,  
       f2,  
       f3,  
       sum(cnt),  
       GROUPING__ID,  
       rpad(reverse(bin(cast(GROUPING__ID AS bigint))),3,'0')  
FROM test  
GROUP BY f1,  
         f2,  
         f3 WITH ROLLUP;  
```

#### 结果

![with rollup查询结果](http://upload-images.jianshu.io/upload_images/3367144-f8b186483922ec4d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 3、grouping sets

#### 查询语句

```
SELECT f1,  
       f2,  
       f3,  
       sum(cnt),  
       GROUPING__ID,  
       rpad(reverse(bin(cast(GROUPING__ID AS bigint))),3,'0')  
FROM test  
GROUP BY f1,  
         f2,  
         f3  
GROUPING SETS((f1),(f1,f2))  
```

#### 结果

![grouping sets查询结果](http://upload-images.jianshu.io/upload_images/3367144-44b1da46d5efc81b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 总结
cube的分组组合最全，是各个维度值的笛卡尔（包含null）组合，
rollup的各维度组合应满足，前一维度为null后一位维度必须为null，前一维度取非null时，下一维度随意，
grouping sets则为自定义维度，根据需要分组即可。
ps:通过grouping sets的使用可以简化SQL，比group by单维度进行union性能更好。

