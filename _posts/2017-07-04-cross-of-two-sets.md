---
layout: post
title: "两个数据文件求交集、差集"
description: ""
category: tech
tags: [shell,awk]
---
{% include JB/setup %}

### 问题描述

对于两个同处在一个集群的hive表，进行文件或者数据的对比是非常方便的，直接写个关联查询就可以搞定。
然而有时候，一个表在MySql里，一个在Hive里，或者是两个表在不同的MySql库下，而我们对两份数据只做临时的比对，把数据抽到集群再进行计算，显然有些麻烦。
通常的做法是，我们把两个不同源的数据抽出需要的字段，导出到两个文本文件里，然后通过shell命令或者excel等工具处理一下。
* comm
* grep
* awk
* vlookup函数

### 例子
aaa.txt文本

```
aaa
bbb
ccc
ddd
eee
111
222
```

bbb.txt

```
bbb
ccc
aaa
hhh
ttt
jjj
```
求aaa.txt和bbb.txt的差集

### comm命令

comm命令可以用于两个文件之间的比较，它有一些选项可以用来调整输出，以便执行交集、求差、以及差集操作。 
* 交集：打印出两个文件所共有的行。 
* 求差：打印出指定文件所包含的且不相同的行。 
* 差集：打印出包含在一个文件中，但不包含在其他指定文件中的行。

  ![comm帮助文档](http://upload-images.jianshu.io/upload_images/3367144-530d059e1d806d4e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 说明
默认输出为三列，第一列为是A-B，第二列B-A，第三列为A交B。如果两个文件不是排序的，中间会提示文件未被排序。可以把两个文件排序和出重处理，然后再求交集。

```
$comm aaa.txt bbb.txt     
aaa
                bbb
                ccc
comm: file 2 is not in sorted order
        aaa
ddd
eee
comm: file 1 is not in sorted order
111
222
        hhh
        ttt
        jjj
```

* 先排序计算

```
$comm <(sort aaa.txt ) <(sort bbb.txt )
111
222
                aaa
                bbb
                ccc
ddd
eee
        hhh
        jjj
        ttt
```

* 排序去重在计算

```
$comm <(sort aaa.txt|uniq ) <(sort bbb.txt|uniq )
111
222
                aaa
                bbb
                ccc
ddd
eee
        hhh
        jjj
        ttt
```

* 求交集

```
$comm -12 <(sort aaa.txt|uniq ) <(sort bbb.txt|uniq )
aaa
bbb
ccc
```

* 求差

  原始的求差结果如下，可以通过删除'\t'的方式合成一列

```
$comm -3 <(sort aaa.txt|uniq ) <(sort bbb.txt|uniq )  
111
222
ddd
eee
        hhh
        jjj
        ttt
```

```
$comm <(sort aaa.txt) <( sort bbb.txt) -3 | sed 's/^\t//'
111
222
ddd
eee
hhh
jjj
ttt
```

* 求差集

通过删除不需要的列，可以得到aaa.txt和bbb.txt的差集
aaa.txt的差集

```
$comm <(sort aaa.txt) <( sort bbb.txt) -23
111
222
ddd
eee
```
bbb.txt的差集

```
$comm <(sort aaa.txt) <( sort bbb.txt) -13 
hhh
jjj
ttt
```

### grep
  
  grep是很常用的搜索文本内容的命令，也可以实现求交集、差集功能

* 求交集 

```
$grep -F -f aaa.txt bbb.txt
bbb
ccc
aaa
```
* 排序去重

```
$grep -F -f aaa.txt bbb.txt  | sort | uniq
aaa
bbb
ccc
```
* 差集

```
$grep -F -v -f aaa.txt bbb.txt | sort | uniq
hhh
jjj
ttt
```

```
$grep -F -v -f bbb.txt aaa.txt| sort | uniq        
111
222
ddd
eee
```
第一行结果为bbb.txt去除aaa.txt，；第二行为aaa.txt去除bbb.txt。注意顺序很重要！

### awk

* 交集

```
$ awk 'NR==FNR{ a[$1]=a[$1]+1} NR>FNR{ if(a[$1]>=1 &&b[$1]<1){ print $1;b[$1]=b[$1]+1}}' aaa.txt bbb.txt 
bbb
ccc
aaa
```

NR表示已经处理的行数，FNR表示当前文件处理的行数。所以当NR==FNR时，处理的是aaa.txt，NR>FNR时，处理的是bbb.txt，在处理aaa.txt时，把a数组记录不同字符串个数，且起到去重作用。在处理bbb.txt时，判断a数组中是否含当前字符串，并且在本文件中出现的次数小于1，同样也是起到了去重的作用。

* 差集

```
$awk 'NR==FNR{ a[$1]=$1 } NR>FNR{ if(a[$1] == ""){ print $1}}' aaa.txt bbb.txt 
hhh
ttt
jjj
```

```
 $awk 'NR==FNR{ a[$1]=$1 } NR>FNR{ if(a[$1] == ""){ print $1}}' bbb.txt aaa.txt 
ddd
eee
111
222
```

### Excel的VLOOKUP函数
实践一下，也比较好用，具体使用网上教程很多，不再累述！
