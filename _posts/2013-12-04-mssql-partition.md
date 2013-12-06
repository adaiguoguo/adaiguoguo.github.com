---
layout: post
title: "Mssql Partition"
description: 数据库中某个表中的数据很多。很多是什么概念？一万条？两万条？还是十万条、一百万条？这个，我觉得是仁者见仁、智者见智的问题。当然数据表中的数据多到查询时明显感觉到数据很慢了，那么，你就可以考虑使用分区表了。
category: sys
tags: [Mssql]
---
{% include JB/setup %}
---

##分区表简介
分区表可以从物理上将一个大表分成几个小表，但是从逻辑上来看，还是一个大表。
但分区表创建时就需要定义完整。
##分区表的步骤

* 1、定义分区函数

* 2、定义分区Schema

* 3、定义分区表

![cardinal](/res/images/Partition/1.jpg)
###haha
##创建数据文件组与文件

1、找到分区表所在数据库，右键单击，在弹出的菜单里选择“属性”。然后选择“Filegroup”选项，再单击下面的“add”按钮，如下图所示：

![cardinal](/res/images/Partition/2.jpg)

2、选择“文件”选项，然后添加几个文件。在添加文件的时候要注意以下两点：

* 1、不要忘记将不同的文件放在文件组中。当然一个文件组中也可以包含多个不同的文件。

* 2、如果可以的话，将不同的文件放在不同的硬盘分区里，最好是放在不同的独立硬盘里。要知道IQ的速度往往是影响SQL Server运行速度的重要条件之一。将不同的文件放在不同的硬盘上，可以加快SQL Server的运行速度。

![cardinal](/res/images/Partition/3.jpg)

##创建分区函数
创建分区函数的目的是告诉SQL Server以什么方式对分区表进行分区。这次例子是将SID分为3个区，假设划分为小于20、大于等于20小于30、大于等于30

{% highlight html %}
create partition function partscher(int)
as range right
for values (20,30)
{% endhighlight %}

##创建分区Schema
分区方案的作用是将分区函数生成的分区映射到文件组中去。分区函数的作用是告诉SQLServer，如何将数据进行分区，而分区方案的作用则是告诉SQL Server将已分区的数据放在哪个文件组中

{% highlight html %}
CREATE PARTITION SCHEME partschSale
AS PARTITION partscher
TO (  
      filegroup1,  
      filegroup2,  
      filegroup1  )
{% endhighlight %}

##创建分区表

{% highlight html %}
CREATE TABLE [dbo].[sensitive](
	[sid] [int] IDENTITY(1,1) NOT FOR REPLICATION NOT NULL,
	[dbName] [varchar](50) NOT NULL,
	[tbName] [varchar](50) NOT NULL,
	[dealMethod] [varchar](500) NOT NULL,
	[columnName] [varchar](50) NOT NULL,
	[enable] [int] NOT NULL,
) ON partschSale([sid])
{% endhighlight %}

* 1、CREATE TABLE 意思是创建一个数据表。
* 2、Sale为数据表名。
* 3、()中为表中的字段，这里的内容和创建普通数据表没有什么区别，惟一需要注意的是不能再创建聚集索引了。道理很简单，聚集索引可以将记录在物理上顺序存储的，而分区表是将数据分别存储在不同的表中，这两个概念是冲突的，所以，在创建分区表的时候就不能再创建聚集索引了。
* 4、ON partschSale()说明使用名为partschSale的分区方案。
* 5、partschSale()括号中为用于分区条件的字段是SaleTime。

##查询分区的数据分布:

{% highlight html %}
select convert(varchar(50), ps.name) as partition_scheme,
p.partition_number,
convert(varchar(10), ds2.name) as filegroup,
convert(varchar(19), isnull(v.value, ''), 120) as range_boundary,
str(p.rows, 9) as rows
from sys.indexes i
join sys.partition_schemes ps on i.data_space_id = ps.data_space_id
join sys.destination_data_spaces dds
on ps.data_space_id = dds.partition_scheme_id
join sys.data_spaces ds2 on dds.data_space_id = ds2.data_space_id
join sys.partitions p on dds.destination_id = p.partition_number
and p.object_id = i.object_id and p.index_id = i.index_id
join sys.partition_functions pf on ps.function_id = pf.function_id
LEFT JOIN sys.Partition_Range_values v on pf.function_id = v.function_id
and v.boundary_id = p.partition_number - pf.boundary_value_on_right
WHERE i.object_id = object_id('sensitive')
and i.index_id in (0, 1)
order by p.partition_number
{% endhighlight %}

![cardinal](/res/images/Partition/4.jpg)

##查询数据

小于20为1
大于等于20小于30为2
大于等于30为3

查询大于30的数据：
{% highlight html %}
select * from sensitive where $PARTITION.partscher(sid)=3 
{% endhighlight %}

参考资料：

[SQL Server 2008中的分区表](http://blog.sina.com.cn/s/blog_681cd80d0100ih60.html)

[理解SQL SERVER中的分区表](http://www.cnblogs.com/sienpower/archive/2011/12/31/2308741.html)



---
<div class="alert alert-block alert-warn form-inline" style="text-align:center; vertical-align:middle; font-size: 16px; font-weight:300;">To be continue!</div>