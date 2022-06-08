---
title: "MySql如何使用索引（一）"
date: 2019-07-28T21:10:51+08:00
draft: false
categories: ["MySql"]
tags: ["MySql"]
---
> 我们都知道在 MySql 中使用索引可以提高查询效率,但有时候真正执行Sql查询的时候却没有按照我们的预想使用索引，而是全表扫描，导致有慢Sql影响了整个网站的效率，甚至导致网站崩溃，所以我们需要了解Mysql是如何选择使用索引的，以便建立合适的索引 （本文基于MySql5.7，InnoDB引擎）

## 前提：建立一张测试表
假设有一张用户表
```SQL
CREATE TABLE `test_user` (
  `user_id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(20) NOT NULL DEFAULT '',
  `birthday` date NOT NULL,
  `sex` tinyint(1) DEFAULT NULL,
  `height` int(11) NOT NULL,
  `weight` int(11) NOT NULL,
  PRIMARY KEY (`user_id`),
  KEY `idx_name_height_weight` (`name`,`height`,`weight`),
  KEY `idx_height` (`height`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
我们建立了name,height,weight 的联合索引，插入测试数据
```SQL
INSERT INTO `test_user` (`user_id`, `name`, `birthday`, `sex`, `height`, `weight`)
VALUES
	(1, 'Tony', '1991-01-01', 1, 176, 65),
	(2, 'Mary', '1989-12-19', 2, 160, 50),
	(3, 'Tom', '1996-05-29', 1, 180, 70),
	(4, 'Kiven', '1994-08-09', 1, 190, 80),
	(5, 'John', '1992-11-12', 1, 182, 75);
```

## 执行什么操作的时候Mysql会使用索引
### 查找与WHERE子句匹配的行

  ```SQL
  select * from test_user where name='mary'
  ```

  查看执行计划

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190310091420494.png)
    可见使用了idx_name_height_weight索引
### 从表中删除一行数据

```SQL
    DELETE from test_user where name='mary';
```
查看执行计划

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019031009144671.png)
### 查找索引列的MIN()或MAX()的值
```SQL
    SELECT MIN(height)  FROM `test_user`
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190310091522957.png)
    这个和之前的执行计划不太一样，Select tables optimized away（选择要优化的表）实际就是优化到不能再优化的意思，在这种情况下，MySql把每个Key的MIN() 和 MAX()值替换成一个常量，如果查到了这个常量就立即返回，然后看下面的例子，分别在索引列上和非索引列上使用MIN()函数的执行计划：
```SQL
    SELECT MIN(height)  from `test_user` where height>=176 AND height<=190;
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190310091554595.png)

```SQL
    SELECT MIN(height)  from test_user where sex=1;
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019031009164120.png)
    
### 在索引列上执行 SORT 或 ORDER BY 操作 

```SQL
    SELECT name,height from test_user ORDER BY `name` DESC;
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019031009165850.png)

注意SELECT的字段能匹配索引列，比如：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190310091712273.png)

将会出现Using filesort，Using filesort 是Mysql里一种速度比较慢的外部排序，应当尽量避免

> 本文讲述了MySql什么时候会使用索引，下章说明什么时候MySql不能使用索引，[点此查看](!https://blog.csdn.net/Magicio/article/details/88388012)

