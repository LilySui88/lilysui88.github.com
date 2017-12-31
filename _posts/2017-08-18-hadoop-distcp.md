---
layout: post
title: "通过hadoop distcp进行集群间数据迁移"
description: ""
category: tech
tags: [hadoop,distcp]
---
{% include JB/setup %}

# 问题描述
我所在的部门是BI，平时业务计算有两个Hadoop集群A和B。其中一个集群A因为大部分业务线计算都在上面，最近开始经常出问题，并且计算变慢。为了进行热备，决定把A集群的计算迁到B上一份，新抽取的数据可以在A和B上各自独立运行，但是历史数据没必要从头从MySQL中再抽一遍，即使可以这么做，也很耗费时间。所以最快的方式是把A的数据copy到B上一份。
# 解决方案
* Hadoop自带的集群间copy工具distcp

distcp（分布式拷贝）是用于大规模集群内部和集群之间拷贝的工具。 它使用Map/Reduce实现文件分发，错误处理和恢复，以及报告生成。 它把文件和目录的列表作为map任务的输入，每个任务会完成源列表中部分文件的拷贝。 由于使用了Map/Reduce方法，这个工具在语义和执行上都会有特殊的地方。 
* 格式

指定源hdfs和目的hdfs路径即可。

```
hadoop distcp hdfs://namenode01/user/hive/test.txt  hdfs://namenode02/user/hive/test.txt
```

* 参数

命令行输入

```
hadoop distcp
```

即得到如下参数的使用说明

```
usage: distcp OPTIONS [source_path...] <target_path>
              OPTIONS
 -append                Reuse existing data in target files and append new
                        data to them if possible
 -async                 Should distcp execution be blocking
 -atomic                Commit all changes or none
 -bandwidth <arg>       Specify bandwidth per map in MB
 -delete                Delete from target, files missing in source
 -f <arg>               List of files that need to be copied
 -filelimit <arg>       (Deprecated!) Limit number of files copied to <= n
 -i                     Ignore failures during copy
 -log <arg>             Folder on DFS where distcp execution logs are
                        saved
 -m <arg>               Max number of concurrent maps to use for copy
 -mapredSslConf <arg>   Configuration for ssl config file, to use with
                        hftps://
 -overwrite             Choose to overwrite target files unconditionally,
                        even if they exist.
 -p <arg>               preserve status (rbugpcax)(replication,
                        block-size, user, group, permission,
                        checksum-type, ACL, XATTR).  If -p is specified
                        with no <arg>, then preserves replication, block
                        size, user, group, permission and checksum
                        type.raw.* xattrs are preserved when both the
                        source and destination paths are in the
                        /.reserved/raw hierarchy (HDFS only). raw.*
                        xattrpreservation is independent of the -p flag.
                        Refer to the DistCp documentation for more
                        details.
 -sizelimit <arg>       (Deprecated!) Limit number of files copied to <= n
                        bytes
 -skipcrccheck          Whether to skip CRC checks between source and
                        target paths.
 -strategy <arg>        Copy strategy to use. Default is dividing work
                        based on file sizes
 -tmp <arg>             Intermediate work path to be used for atomic
                        commit
 -update                Update target, copying only missingfiles or
                        directories
```

# 应用实例

* 其他准备

对于在原始集群中已经建hive表的数据，通过以下命令拿到建表语句，然后在目标集群上执行

```
show create table xxx
```
* 脚本实例

脚本在目的集群client端机器执行，目标hdfs路径简写，具体copy.sh脚本内容如下:

```
#!/usr/bin/sh
if [ $# -eq 0 ];then
        DT=`date -d "-1 day" +"%Y-%m-%d"`
else
        DT=`date -d"$1" +"%Y-%m-%d"`
fi
mysqlTable=$2;
table_name=function_$mysqlTable;
```

# 按天增量拷贝，防止中间出错，并且方便按天进行数据验证
src_path=hdfs://namenode01/user/hive/warehouse/$table_name/dt=$DT
dest_path=/user/hive/warehouse/$table_name/dt=$DT

hadoop distcp  -D mapred.job.queue.name=compute_daily  $src_path $dest_path

# 给目标集群表加分区

```
hive -e"
use mbd;
alter table $table_name add partition(dt='$DT');
"
```

* 脚本调用

```
sh copy.sh 2017-08-18 pay
```

* 数据验证

脚本执行之后，还要进行数据验证，对于同样的sql，按天查看不同字段的去重计数，分别在两个集群上执行，即可充分验证，目标集群copy数据的准确性。