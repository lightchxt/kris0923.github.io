---
title: "【APUE】Unix体系结构"
date: 2022-04-28T12:50:33+08:00
draft: false
categories: ["读书笔记"]
tags: ["APUE", "Linux"]
---



Unix系统结构从内到外 内核->系统调用->(公用函数库、shell)->应用程序

一般用户信息在/etc/passwd 文件中，但macOS 的passwd文件中只有单用户模式的用户信息，普通登陆用户信息又Open Directory提供

fork 创建一个新进程，新进程是调用进程的一个副本，fork调用一次，返回两次，在父进程中返回子进程ID，在子进程中返回0

Unix系统中有两种时间 
- 日历时间 （早期的成为格里尼治标准时间）
- 进程时间（CPU时间）

# 系统调用和库函数
- 系统调用：各版本Unix实现的提供良好定义、数量有限、直接进入内核的入口点，成为系统调用
- 库函数：C语言实现的函数
> 库函数可以替换，系统调用不能替换

|用户进程|内核|
|---|---|
|应用程序->C库函数| 系统调用|

# POSIX
POSIX标准定义了7类系统实现的限制和常量，一些常量限制在limits.h（mac /usr/include/limits.h）文件中，sysconf，pathconf，fpathconf可以在运行时得到实际实现值
- 与文件和路径有关 pathconf，fpathconf
- 与文件和路径无关 sysconf

