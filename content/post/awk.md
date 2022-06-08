---
title: "Linux 命令之 Awk"
date: 2019-07-28T21:08:44+08:00
draft: true
categories: ["Linux"]
tags: ["Linux"]
---
## awk 文本处理工具
- 可编程，功能强大
- 可以用来统计，制表等功能

###### 处理方式
* 一次处理一行内容
* 可以对每行进行切片处理
###### 格式
- 命令行格式
	```
	awk [options] 'command' file(s)
	```
- 脚本格式
	```
	awk -f script_file file(s)
	```

##### 内置参数
- $0 当前整行
- $1 每行第一个字段
- $2 每行第二个字段
- NR 行号
- NF 字符总数
- FILENAME 文件名
- ARGC 命令行参数个数
- ARGV 命令行参数数组
- 

-F 指定分割符，默认空格
```
awk -F ':' '{print $1,$3, NR}'  test.txt  // 以: 分割

awk -F ':' '{if ($3>100) print "This Msg"}' test.txt
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190313221813746.png)![在这里插入图片描述](https://img-blog.csdnimg.cn/20190313221921397.png)
正则匹配 ～ ， 不匹配(！～)
打印第一个字符以m开头的
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190313223242892.png)
- BEGIN 行处理前执行
- END 行处理完成后执行
eg：文件夹大小求和
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190313224452488.png)

> AWK 是一种程序设计语言，语法和C语言相似，把AWK命令写到脚本文件中可以完成更复杂的文本处理
