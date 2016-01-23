---
layout: post
title: "Java程序捕获kill信号"
description: ""
category: tech
tags: [java]
---
{% include JB/setup %}

#源程序


{% highlight java%}
import sun.misc.*;

/**
 * @Description 在java主程序退出前，可以做点事
 * @Author LilySui
 * @Date 2016年1月23日
 */

@SuppressWarnings("restriction")
public class TestKillJava implements SignalHandler{

	/**
	 * @param args
	 */
	public static void main(String[] args)  throws InterruptedException {
		TestKillJava testSignalHandler = new TestKillJava();  
        Signal.handle(new Signal("TERM"), testSignalHandler); //相当于kill -15 
        //Signal.handle(new Signal("USR1"), testSignalHandler);  测试时，抛出该信号被占用的异常
        Signal.handle(new Signal("USR2"), testSignalHandler);  //相当于kill -12
        for (;;) {  
            Thread.sleep(3000);  
            System.out.println("running......");  
        }  
	}

	/**
	 * 回调函数
	 */
    @Override  
    public void handle(Signal signalName) {  
        signalCallback(signalName);  
    }  
    
    private void signalCallback(Signal sn) {  
        System.out.println(sn.getName() + "is recevied."); 
        //do something you want to do before being killed
        System.exit(-1);
    }  
}
	
{% endhighlight %}

#信号说明

在Linux下支持的信号（具体信号kill -l命令查看）：
SEGV, ILL, FPE, BUS, SYS, CPU, FSZ, ABRT, INT, TERM, HUP, USR1, USR2, QUIT, BREAK, TRAP, PIPE
在Windows下支持的信号：
SEGV, ILL, FPE, ABRT, INT, TERM, BREAK

#异常
运行中可能会抛出异常：
java.lang.IllegalArgumentException: Signal already used by VM: USR1
这是因为某些信号可能已经被JVM占用，可以考虑用其它信号代替

#kill小技巧
直接关闭指定Java程序的命令

{% highlight  bash linenos %}
ps -ef |grep JavaProgramName |awk '{print $2}'| grep -v grep |xargs kill -15
{% endhighlight %}