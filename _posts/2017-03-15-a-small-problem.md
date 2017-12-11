---
layout: post
title: "一个小问题~"
description: ""
category: tech
tags: [hive,SQL]
---
{% include JB/setup %}
### 问题描述
一个小伙伴提的问题，假设我有一张表：send_user,recv_user, send_times，分别代表：发送人、接收人、消息条数
我想算出一个人发送多少条消息，接受多少条消息，该怎么算啊？
### 希望得到的数据形式

```
user,send_times,recv_times
lily,100,11
someone,1000,100
```

### 解决

```
select xname,sum(send_times) as send_times,sum(recv_times) as recv_times
from
(
	select case when base.send_user is not null then base.send_user else base.recv_user end as xname,
	sum(case when base.send_user is not null then base.times else 0 end) as send_times,
	sum(case when base.send_user is null then base.times else 0 end) as recv_times
from(
	select send_user,recv_user, sum(recv_times) as times from tablename
	group by send_user,recv_user 
	grouping sets ((send_user),(recv_user))
	) base
)temp
group by xname
```

### 原理
最内层的grouping sets的查询结果数据有如下格式，并且，第一列和第二列互斥，也就是说，第一列有值，第二列就没有值，第一列没有有值第二列就有值

```
send_user,recv_user, times
lily,null,100
null,lily,11
someone,null,1000
null,someone,100
```

由于两个列互斥，可以用case when来把前两列的值转到一列下，再通过互斥条件生成两个列，分别代表发送数和接收数，起到了行转列的作用，这时数据变成如下格式

```
xname, send_times, recv_times
lily,0,100
lily,11,0
someone,1000,0
someone,0,100
```

最外层再group by xname，来合并同名，分别sum后两列，由于上一步用0替代了互斥时不存在的值，所以sum(0)不会对值有影响，得到最后结果

```
xname, send_times, recv_times
lily,100,11
someone,1000,100
```
