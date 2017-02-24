---
layout: post
title: "保障Hadoop集群间shell脚本依赖关系的办法"
description: ""
category: tech
tags: [shell,hadoop]
---
{% include JB/setup %}

### 问题描述

做数据开发的同学都需要在Hadoop集群客户端上通过crontab配置shell脚本的调度（有调度系统的另说啦），这些脚本可能在一台机器上，也可能不是，相互之间有数据上的依赖关系，通常我们会根据各个脚本的运行时长，设置调度时间，考虑好时间间隔。

正常情况下，统计数据第二天都会按时出现在业务系统的报表中。但有时候可能出现其中某一个脚本因为无法预期的原因没有正常执行，那么后续的脚本肯定也不会有结果。

### 思考
出了问题就要补数据，手动的一个个的起脚本就很low了。如果可以做到前面的脚本出问题，后续脚本等待而不结束任务，当前面脚本恢复执行结果以后，后续脚本自动运行，那就方便多了。

### 方案
来了iqiyi以后，发现这里同事的做法是，通过在每个脚本最后创建Hadoop hdfs的文件标识任务正常结束，其他需要依赖的别人数据的脚本，在脚本最开始每间隔固定时间判断依赖的Hadoop hdfs文件是否生成，如果没生成就一直等待，直到前置数据完成了当前的脚本向下运行。
#### 举个栗子
* 前置脚本例子

```sh
#!/bin/sh

if [ $# -eq 0 ];
then
        DT=`date -d"1 days ago" +'%F'`
else
        DT=$1
fi

###############################
## 这里写自己任务的逻辑代码
###############################

if [ $? == 0 ];#判断之前的任务有没有正常结束，如果正常结束就创建Hadoop hdfs的文件
then
        if ! hadoop fs -test -e  "/data/sqoopdata/$DT/xxx.done"
        then
                hadoop fs -touchz  "/data/sqoopdata/$DT/xxx.done"
        fi
else
        exit 1
fi
```

* 后续脚本列子

```sh
#!/bin/sh
###############################
##需要依赖别的脚本产生的数据，
##所以得判断该脚本的任务是否正常执行了
###############################
if [ $# -eq 0 ];
then
        DT=`date -d"1 days ago" +'%F'`
else
        DT=$1
fi

while :
do
        if !  hadoop fs -test -e "/data/sqoopdata/$DT/xxx.done"
        then
                echo "xxx data of $DT  doesn't exist"
                sleep 60
        else
                break
        fi

done
```

### 结语
在没有调度系统的时候，上述的方法会起到一定脚本依赖关系的维护作用，补数据的时候也省事一些，只要把出问题的脚本重跑就可以了。
缺点，脚本多了以后，肯定乱，需要对业务比较清楚才知道各个阶段生成了哪些数据。不过，只要是每个脚本都规范的写，把依赖的Hadoop hdfs文件抽出来，单独建一个公共的脚本，以备各个业务脚本引用，逻辑还是可以很清楚的。