---
title: 'MySql 修改表结构时 ALTER,MODIFY,CHANGE的区别'
date: 2019-07-17 23:09:55
draft: true
categories: ["MySql"]
tags: ["MySql"]
---
根据MySql文档，我们知道在修改表内某一列的属性的时候，MySql支持3中语法结构：

```
 ALTER [ONLINE|OFFLINE] [IGNORE] TABLE tbl_name  
	ALTER [COLUMN] col_name {SET DEFAULT literal | DROP DEFAULT}
```

```
ALTER [ONLINE|OFFLINE] [IGNORE] TABLE tbl_name  
	CHANGE [COLUMN] old_col_name new_col_name column_definition [FIRST|AFTER col_name]
```

```
ALTER [ONLINE|OFFLINE] [IGNORE] TABLE tbl_name 
	MODIFY [COLUMN] col_name column_definition [FIRST | AFTER col_name]
```
这里比较一下这三种语法的不同之处，以及什么情况下应该选用什么语法

|语法|功能|说明|
|---|---|---|
|ALTER|只能更改列的默认值||
|CHANGE|可以重命名列或者修改列的定义|标准SQL的扩展|
|MODIFY|可以更改列的定义，但不能更改列的名称|兼容Oracle的扩展|
通过文档介绍的功能，我们就基本能够判断处该使用使用哪种语法，CHANGE功能最强大，什么情况下都可以使用(达到预期的效果)。但是还有一个区别：
- ALTER 语法只是修改 .frm 文件，不会去更新表中的数据；
- MODIFY和CHANGE在更新表结构的时候重新插入表中的数据，因此比较耗费时间。

所以，当只需要修改某一列的默认值的时候，优先选择用ALTER，需要修改列的名称用CHANGE，只修改列的定义用MIODIFY
> 如果修改的列上有索引，修改完后最好重建一下索引