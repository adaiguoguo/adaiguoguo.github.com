---
layout: post
title: "Gitlab 新手使用教程"
description: Gitlab作为企业场景使用的指南
category: scm
tags: [git]
---
{% include JB/setup %}
---



##登录

![cardinal](/res/images/Gitlab_Guide/1.jpg)

只要是在cn1的账户，在Gitlab的web界面http://git.dev.sh.ctripcorp.com，输入账号与密码登陆
如果密码无法登录，请联系3333服务台查看下cn1账户是否被锁定了。

##安装最新版msysgit Git

* 这里提供给大家个更'cool'的安装方式使用请点击->[Chocolatey](http://chocolatey.org)

* 一般的下载安装方式：[msysgit Git for Windows](http://code.google.com/p/msysgit/downloads/list?q=full+installer+official+git+)


然后设置环境变量,右击计算机->属性->高级属性->高级->系统变量
![cardinal](/res/images/Gitlab_Guide/2.png)


## 增加SSH Key
如果你想进行代码的上传与下载等操作，需要你把自己的SSH Key导入到Gitlab里，方法如下:

打开Git-bash,输入
{% highlight html %}
ssh-keygen -t rsa -C"XXXXXX@ctrip.com"
{% endhighlight %}

`XXXXX请对应自己的工作邮箱`

用记事本打开`~\.ssh\id_rsa.pub`获取SSH Key。

点击my profile->ssh keys->add new 直接把内容保存再Key 即可，save。

![cardinal](/res/images/Gitlab_Guide/3.png)

## 创建仓库和提交代码
在任何系统下进行上传代码，先进行设置git global设置
{% highlight html %}
git config --global user.name "username"
git config --global user.email "mail address"
{% endhighlight %}
如果您是Git 新手可以使用：[Got 15 minutes and want to learn Git](http://try.github.io/levels/1/challenges/1)

## 设置头像
*http://cn.gravatar.com/ 注册一个帐号 
*将ctrip的邮箱填在 必要邮箱处
*选择一个头像上传
*成功, 回到http://git.dev.sh.ctripcorp.com 刷新页面

