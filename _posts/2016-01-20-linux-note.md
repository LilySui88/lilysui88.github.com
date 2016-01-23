---
layout: post
title: "Linux命令笔记"
description: ""
category: tech
tags: [linux]
---
{% include JB/setup %}


* 查看系统磁盘空间，查看系统是否有挂载的网络系统

        df -lh
    

* 查看当前路径文件夹占用空间

		du -sh ./*

* 查看内存使用情况

		free

* 查看进程资源占用情况，既负载

		top

* 查看指定进程的资源占用情况

		ps aux | grep keyword
		或
		ps -ef |grep

* 查看网卡信息

		ifconfig

* 查看当前系统每1s cpu空闲状态

		vmstat 1

* 后台运行命令

		nohup python *.py &

* 查看CPU or 内存配置

		cat /proc/cpuinfo #cpu配置
		cat /proc/meminfo #内存配置

* crontab配置

		crontab -l -u username #列出
		crontab -e -u username #编辑

* 文件空白行处理

		grep ^$ -n filename #文件空行行号
		sed -i '/^$/d' filename #删除文件空白行

* 查看本机端口占用

		netstat -ntpl

