---
layout: post
title: "Css markdown"
description: 有一个居中图片问题困扰了我好久，解决了写个文档分享下
category: coolskill
tags: [css]
---
{% include JB/setup %}
---
近日需要在个人网站上添加图片，位置需要居中。markdown语言没有原生的解决方法。[How to Align Images in Markdown](http://www.denizoguz.com/2013/08/07/how-to-align-images-in-markdown/)中给出了一种解决方法：

{% highlight html %}
<div style="text-align:center" markdown="1">
![Alt Text](/path/to/image "Caption")
</div>
{% endhighlight %}

不过我觉得这种方法不够优雅，没有采用。

[Using Markdown, how do I center an image and its caption?](http://stackoverflow.com/questions/3912694/using-markdown-how-do-i-center-an-image-and-its-caption/15185481#15185481)里讨论了这个问题。由于我自己使用的解释器是[kramdown](http://kramdown.rubyforge.org/syntax.html#block-ials)，因此采纳了Steven Penny的如下的解决方案。

在当前撰写的markdown文件里，紧挨着在想要居中的元素之后添加`{:.center}`：

{% highlight html %}
Hello there!

![cardinal](/img/2012/cardinal.jpg)
{:.center}
This is an image

Hi!
{% endhighlight %}

在css文件中（我这里的文件是`assets/themes/twitter/css/style.css`），需要添加`center`的定义：

{% highlight css %}
.center {
  text-align: center;
  margin: 0 auto;
}
{% endhighlight %}

这样就搞定啦！
---