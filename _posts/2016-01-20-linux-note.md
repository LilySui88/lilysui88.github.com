---
layout: post
title: "Linux命令笔记"
description: ""
category: tech
tags: [linux]
---
{% include JB/setup %}


* 查看系统磁盘空间，查看系统是否有挂载的网络系统

{% highlight bash%}
df -lh
{% endhighlight %}

* 查看当前路径文件夹占用空间

{% highlight bash%}
du -sh ./*
{% endhighlight %}

* 查看内存使用情况

{% highlight bash%}
free
{% endhighlight %}

* 查看进程资源占用情况，既负载

{% highlight bash%}
top
{% endhighlight %}

* 查看指定进程的资源占用情况

{% highlight bash%}
ps aux | grep keyword
或
ps -ef |grep
{% endhighlight %}

* 查看网卡信息

{% highlight bash%}
ifconfig
{% endhighlight %}

* 查看当前系统每1s cpu空闲状态

{% highlight bash%}
vmstat 1
{% endhighlight %}

* 后台运行命令

{% highlight bash%}
nohup python *.py &
{% endhighlight %}

* 查看CPU or 内存配置

{% highlight bash%}
cat /proc/cpuinfo #cpu配置
cat /proc/meminfo #内存配置
{% endhighlight %}

* crontab配置

{% highlight bash%}
crontab -l -u username #列出
crontab -e -u username #编辑
{% endhighlight %}

* 文件空白行处理

{% highlight bash%}
grep ^$ -n filename #文件空行行号
sed -i '/^$/d' filename #删除文件空白行
{% endhighlight %}

* 查看本机端口占用

{% highlight bash%}
netstat -ntpl
{% endhighlight %}
