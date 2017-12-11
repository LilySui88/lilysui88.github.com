---
layout: post
title: "Summary of February 2017"
description: ""
category: tech
tags: [work]
---
{% include JB/setup %}

### READING

* Head First 设计模式：完成50%。
内容：观察者模式、装饰者模式、工厂模式、单件模式、命令模式、适配器模式、外观模式
* 推荐系统实战：完成20%

### TOOLS

* mongoexport ，使用-h -d -c -f -q --csv -o等参数的用法

* Hadoop集群间copy数据，hadoop distcp -D mapred.job.queue.name=xxx_daily $src_path $dest_path
其中，src_path=hdfs://namenode01-xxx.xxx.hadoop/xxx/tablename/dt=$DT，dest_path=/xxx/tablename
最后，alter table tablename add partition (dt=$DT)

* 机器间同步数据
rsync -avzr /data/logs/${DT} root@xxx.xxx.xxx.xxx:/data/xxx_www/
或 
scp -i ~/.ssh/id_rsa_lily lilysui@xxx.xxx.xxx.xxx:/data/logs/bin/do_work.sh ~/Documents/iqiyi/iqiyi-shell/

* git命令解决冲突

* MapReduce几种OOM处理

### CODING
* 判断数独是否合法 http://www.lintcode.com/submission/8086177/
