---
title: "MySql如何使用索引（二）"
date: 2019-07-28T21:11:18+08:00
draft: true
categories: ["MySql"]
tags: ["MySql"]
---
> [上篇](!https://blog.csdn.net/Magicio/article/details/88374896)介绍了MySql什么时候会尝试使用索引，本文介绍一下我了解的不会使用索引的情况, 仍然使用上次建立好的表

### 1. where 子句中like 使用了前缀通配符 %keyword
```SQL
 select * from test_user where name like "%Kiven%";
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190311093338834.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hZ2ljaW8=,size_16,color_FFFFFF,t_70)
### 2.  使用>, >=, <,<=, !=,NOT IN 范围查找或否定查找,且范围查找时选择的边界查找条件范围过大
```SQL
select * from test_user where height>=180
# 不会使用索引
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019031109335347.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hZ2ljaW8=,size_16,color_FFFFFF,t_70)
```SQL
select * from test_user where height>=190
# 会使用索引
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019031109342113.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hZ2ljaW8=,size_16,color_FFFFFF,t_70)
### 3. 数据类型隐式转换
例如：name字段为varchar类型的
```SQL
select * from test_user where name=1
```
将不能使用索引，而
```SQL
select * from test_user where name='1'
```
可以使用索引

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190311093435141.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hZ2ljaW8=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190311093445291.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hZ2ljaW8=,size_16,color_FFFFFF,t_70)

原因是当不同的字段类型比较时，MySql会做引式类型转换，而 int类型的1可能等于 '01', '0001'或者 '01.e1'

### 4. 不符合最左匹配原则（联合索引）
我们建立的索引顺序是
```SQL
KEY `idx_name_height_weight` (`name`,`height`,`weight`)
```
所以使用的时候where子句也不能跳过前一个联合索引列
```SQL
# 比如直接联合索引的最后一列是不支持的
select * from test_user where weight=65
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190311093459583.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hZ2ljaW8=,size_16,color_FFFFFF,t_70)
```SQL
# 而使用全部索引列做查询条件是可以的
select * from test_user where weight=65 AND name='Tom' AND `height`=160
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190311093510277.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hZ2ljaW8=,size_16,color_FFFFFF,t_70)

### 6. 在索引列上进行运算
```SQL
select * from test_user where `height`+10=160
#是不会使用索引的，可以写成
select * from test_user where `height`=160-10
# 就能够使用索引了
```

 > 在索引使用方面MySql本身帮我们做了很多优化，有时候不一定会按照我们的想法去使用索引，接下来还需要探索
