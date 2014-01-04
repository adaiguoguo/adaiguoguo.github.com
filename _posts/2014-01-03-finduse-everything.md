---
layout: post
title: "如何把Mac OS X用的更炫-顺畅操作篇"
category: 
tags: [coolskill]
description: |
  Mac OS X的常用快捷键、自带搜索神器 Spotlight 及自定义工作流 Alfred
---
{% include JB/setup %}


记住并使用快捷键会明显提高操作，熟练掌握快捷键是需要成本的，需要一定的记忆。
我就在这里介绍些常用的。

## 通用快捷键
Command+Table               从左到右的程序切换

Command+Shift+Table         从右往左的程序切换

Command+~				    同一程序多窗口的切换

Command+F 					调出程序的查找功能

Command+C/V/X               复制/粘贴/剪切

Command+N 					新建程序窗口

Command+Q					彻底退出程序

Command+L 					快速返回浏览器的地址栏

Command+'+/-'				放大或者缩小字体	

Command+Space				切换输入法

Command+Shift+F         	窗口全屏化及还原

## 搜索神器Spotlight

这是Mac OS X上系统自带搜索软件，说来惭愧，我也是最近才用起来。被微软的搜索坑伤了就把这个强大功能给忘了。

Control+Space				呼出 Spotlight
###  1. Spotlight 注释功能定位文件 

OS X 的文件系统提供了 Spotlight 注释功能，可以帮助用户更有针对性的定位文件。选中一个文件或文件夹， command+ I 打开简介，在 Spotlight 注释功能中加入自己特定的关键词。关掉简介窗口，呼出Spotlight并输入刚才的关键词，可以准确定位到相关的文件或文件夹。

###  2. Spotlight 检索的高级技巧 

通过 文件类型搜索文件，搜索格式是： 

kind: 文件类型搜索关键字

举例： kind: app—— 搜索应用程序 

kind: bookmark—— 搜索书签和历史纪录 

kind: word—— 搜索 word 

kind: pages—— 搜索 pages 

kind: key—— 搜索 keynote

###  3. 重建 Spotlight 索引

 Spotlight 机制，可以瞬间找到你想要的文件，只要你记得这个文件的一点蛛丝马迹。但是 Spotlight 也有出问题的时候，就是它的索引文件出事了，比如查找非常慢，这时候只需要三步：
{% highlight html %}
sudo   mdutil   -i   off   #该 命令用来关闭索引 
sudo   mdutil   -E    #该 命令用来删除索引 
sudo   mdutil   -i   on    #该 命令用来重建索引
{% endhighlight %}

## Alfred find everything

和Spotlight直接搜索各种类型不同，Alfred要多敲一个空格或者单引号来指示它搜索文件名。实际使用中，我觉得这样更方便和快捷，alfred几乎是秒出结果，而spotlight在迅速反馈结果的同时，有个逐步增加的过程。

详细教程参见Weiphone 
[键盘流——Alfred全攻略](http://bbs.weiphone.com/forum.php?mod=viewthread&tid=6398178)

Alfred 本身是免费的；上面多数功能是需要powerpack的，入门价15欧，终身升级价30欧，也有所谓家庭装，自己按照需求选吧。有资金的话还请支持正版。