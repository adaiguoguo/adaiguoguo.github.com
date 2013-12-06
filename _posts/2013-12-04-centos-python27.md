---
layout: post
title: "Centos python2.7"
description: 升级centos中python2.7是个非常头痛的问题，因为yum源中只有2.6版本，所以必须下载编译才能OK。
category: sys
tags: [centos,python]
---
{% include JB/setup %}

---
升级centos中python2.7是个非常头痛的问题，因为yum源中只有2.6版本，所以必须下载编译才能OK。
其中涉及到python环境变量。
##运行脚本

代理请事先配置好，这里都是以只有sudo权限运行的

{% highlight html %}
sudo -E yum groupinstall "Development tools"
sudo -E yum install zlib-devel
sudo -E yum install bzip2-devel openssl-devel ncurses-devel
wget http://www.python.org/ftp/python/2.7.3/Python-2.7.3.tar.bz2
tar xf Python-2.7.3.tar.bz2 
cd Python-2.7.3
sudo ./configure --prefix=/usr/local
sudo make && sudo make altinstall
set -o vi
wget http://pypi.python.org/packages/source/d/distribute/distribute-0.6.27.tar.gz
tar xf distribute-0.6.49.tar.gz 
cd distribute-0.6.49
sudo -E python2.7 setup.py install
sudo -E /usr/local/bin/easy_install-2.7 virtualenv
{% endhighlight %}

easy_install-2.7无法运行的解决方案：

/usr/local/bin/easy_install-2.7中
{% highlight html %}
#!.
{% endhighlight %}

改为

{% highlight html %}
!/usr/local/bin/python2.7
{% endhighlight %}


##加入环境变量

* 1：安装完后python 的路径为
{% highlight html %}
/usr/local/bin/python2.7
{% endhighlight %}

我们在shell里面输入:

{% highlight html %}
/usr/local/bin/python2.7
{% endhighlight %}


显示的为

{% highlight html %}
Python 2.7.3 (default, Dec  4 2013, 17:28:23) 
[GCC 4.1.2 20080704 (Red Hat 4.1.2-52)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> 
{% endhighlight %}

说明路径没有错误

* 2：添加环境变量
找到.bashrc文件在后面添加上这几行代码就可以了

{% highlight html %}
PATH=$PATH:/usr/bin/python
export PATH
{% endhighlight %}

* 3：设置软连接地址

{% highlight html %}
cd /usr/bin
rm python
sudo ln -s /usr/local/bin/python2.7 /usr/bin/python
{% endhighlight %}

查看软连接对应的目录

{% highlight html %}
ls -al py*
{% endhighlight %}





---