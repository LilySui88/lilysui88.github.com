---
layout: post
title: "用jekyll搭建github免费博客"
description: ""
category: fun
tags: [github,jekyll]
---
{% include JB/setup %}
###1、注册[github](https://github.com/)，
建议：最好用Gmail,因为国内邮箱在找回密码时根本收不到邮件，😢（前车之鉴）

###2、本地安装[git](http://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
安装后，在[github](https://github.com/)的settings里设置ssh公钥，否则没权限提交本地代码到github。

附：[git命令大全](http://www.cnblogs.com/mengdd/p/4153773.html)

###3、本地安装jekyll（默认你已经有了ruby环境）
mac用gem方式安装，为了加快安装速度，需要先修改gem源，在终端输入：
    $ gem sources --add https://ruby.taobao.org/ --remove https://rubygems.org/
    $ gem sources -l
    *** CURRENT SOURCES ***

    https://ruby.taobao.org
    # 请确保只有 ruby.taobao.org
    $ sudo gem install jekyll #加sudo避免无写权限

###4、在[github](https://github.com/)上创建repository名为USERNAME.github.com（USERNAME是注册时申请的github用户名）

###5、下载Jekyll-Bootstrap博客模板到本地（USERNAME同上）
本地，从终端进入想要安装模板的目录，输入：
    git clone https://github.com/plusjade/jekyll-bootstrap.git USERNAME.github.com #把Jekyll-Bootstrap下载到USERNAME.github.com
    cd USERNAME.github.com
    git remote set-url origin git@github.com:USERNAME/USERNAME.github.com.git #把本地代码关联USERNAME.github.com这个repository
    git push origin master
等几分钟后，就可以在http://USERNAME.github.io下看到博客模板最开始的样子

###6、本地运行jekyll预览博客
同样替换USERNAME成你自己的github用户名，执行以下命令:
    $ cd USERNAME.github.com 
    $ jekyll serve
访问http://localhost:4000/，本地博客运行起来了，退出服务的话Ctrl+C

###7、本地建一个帖子页（post）
在USERNAME.github.com目录下，输入如下命令：
    $ rake post title="Hello World"
jekyll服务在检测到变化后自动重启，刷新http://localhost:4000/，会看到名为Hello World的帖子
###8、把修改的代码提交到github
本地运行没问题后，通过git命令提交
    $ git add .
    $ git commit -m "Add new content"
    $ git push origin master
###至此通过jekyll-bootstrap模板建博客已经完成，更改主题和具体改页面内容需要进一步学习
###更改主题
**[jekyll博客其他的主题预览](http://themes.jekyllbootstrap.com/)
下载the-program主题到本地的命令：
    $ rake theme:install git="https://github.com/jekyllbootstrap/theme-the-program.git"
在提示是否马上切换主题时选择y/n

另外如果，多个主题已经下载到本地可以通过命令切换:
    $ rake theme:switch name="the-program"
###参考文章
**[jekyll官网](http://jekyllbootstrap.com/usage/jekyll-quick-start.html)
**[非常棒的指导文章](http://blog.csdn.net/on_1y/article/details/19259435#t8)


