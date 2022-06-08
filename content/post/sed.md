---
title: "Linux命令之sed"
date: 2019-07-28T21:09:34+08:00
draft: true
categories: ["Linux"]
tags: ["Linux"]
---


## sed 流处理编辑器
- 行处理一次只处理一行数据 （sed处理文件内容的核心思想）
- 不改变文件内容（除非重定向）

##### 命令行格式
	sed [option] 'command' file(s)
##### 脚本格式
	sed -f scriptfile file(s)

##### sed 命令
###### p 打印命令

 	-n 只打印匹配的行
```
定位一行 
	sed -n '2p' test.txt
定位多行
	sed -n '2,5p' test.txt
	// (2和5也可以用正则代替)
定位反向选择
	sed -n '2,5!p' test.txt // 第2-5行不被选择
定位间隔几行
	sed -n '2~2p' test.txt // 间隔输出 
```
###### 行命令 
- a （新增行）/ i（插入行）
- c（替代行）
- d（删除行）

```
linux 用法
sed -n "2a text2++" test.txt
sed -n "2,5a text2++" test.txt
```

```
mac os 用法
sed -n "2a \
text2++
" test.txt

sed -n "2,5a \
text2++
" test.txt
```
###### 替换命令 s - sed命令的核心
基本命令
```
sed 's/search/replace' filename （每行替换一次）
```
全局替换
```
sed 's/search/replace/g' filename 
```
eg:获取本机IP地址
```
ifconfig en0 | sed -n '/inet /p' | sed 's/inet //' | sed 's/netmask.*$//'
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190310100020804.png)
#### sed高级命令
>  假设有如下文本 test.txt

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190310093056404.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hZ2ljaW8=,size_16,color_FFFFFF,t_70)

- {} : 多个sed命令，用;分开
```
# 删除1，2行，并将5替换为12
sed "{1,2d;s/5/12/}" test.txt
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190310200450522.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hZ2ljaW8=,size_16,color_FFFFFF,t_70)
- n 读取下一个输入行，用下一个命令处理
```
隔行输出 与 sed -n '2~2p' test.txt 效果相同
sed -n "{n;p}" // 打印偶数行
sed -n "{p;n}" // 打印奇数行
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190310200006195.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hZ2ljaW8=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190310200036784.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hZ2ljaW8=,size_16,color_FFFFFF,t_70)
- & 替换固定字符串
```
 sed 's/[a-z]\+/&-/' test.txt  
 // &符号代替了前面的正则表达式匹配到的内容
 ```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190310202023188.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hZ2ljaW8=,size_16,color_FFFFFF,t_70)
 - u/l/U/l 大小写转换
 	-  u 首字母大些
	- l 首字母小写
	- U 单词大写
	- L  单词小写
 ```
 sed 's/[a-z]\+/\u&/' test.txt
 ```
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/2019031020241712.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hZ2ljaW8=,size_16,color_FFFFFF,t_70)
- () 获取匹配的内容
```
# 获取文件中的所有小写字符
sed 's/\([a-z]\+\).*$/\1/' test.txt
# 获取本机ip
ifconfig en0 | sed -n '/inet /p' | sed 's/inet \([0-9.]\+\) .*$/\1/'
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190310203530323.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hZ2ljaW8=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190310204355128.png)

- r 复制指定文件到匹配行
- w 复制匹配行到指定文件
> 再添加一个测试文件123.txt

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190310211331445.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hZ2ljaW8=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190310211435499.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hZ2ljaW8=,size_16,color_FFFFFF,t_70)
```
把test.txt文件内容写到123.txt中，如果需要指定行，在w前加行号
```
 **attention： w 会改写文件，使用是应慎重**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190310211824916.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hZ2ljaW8=,size_16,color_FFFFFF,t_70)

- q 退出sed

第5行后退出sed
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190310212414444.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hZ2ljaW8=,size_16,color_FFFFFF,t_70)


**sed 还有一个重要的选项 -i: 直接修改读取的文件内容，而不是输出到终端**
```
sed -i 's/5/13/' test.txt
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190310213643546.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hZ2ljaW8=,size_16,color_FFFFFF,t_70)
> 更多用法请查看 [gnu sed 官方文档](!https://www.gnu.org/software/sed/)

