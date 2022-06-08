---
title: PHP 内核分析笔记（二）多进程与多线程SAPI生命周期
draft: true
categories: ["php"]
tags: ["php"]
---
### 多进程SAPI生命周期
以Apache为例，PHP编译为apache的一个模块来处理php请求，Apache启动后会fork多个进程，每个进程拥有独立的内存空间，单独处理php请求，所以每个子进程都是完整的生命周期
![img](http://www.php-internals.com/images/book/chapt02/02-01-02-multiprocess-life-cycle.png)

### 多线程的SAPI生命周期
同一组线程只有一个模块初始化和销毁的过程
![img](http://www.php-internals.com/images/book/chapt02/02-01-013-multithreaded-lift-cycle.png)