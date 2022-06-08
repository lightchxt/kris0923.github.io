---
title: "Nginx介绍"
date: 2019-09-01T23:46:27+08:00
draft: true
---

## Nginx是什么
nginx 是一个免费的，开源的，高性能HTTP服务器和反向代理，以及IMAP / POP3代理服务器。NGINX以其高性能，稳定性，丰富的功能集，简单的配置和低资源消耗而闻名。与传统服务器不同的是，Nginx不依赖线程来处理请求，它使用了更加可扩展的异步事件驱动的架构，因此与其他的服务器相比，Nginx可以使用很少的内存来同时处理上千个请求。

[netcraft](https://news.netcraft.com/archives/2019/08/15/august-2019-web-server-survey.html)调查显示，到2019年8月nginx拥有超过31％的计算机市场份额，仅落后于Apache5.39%

## Nginx可以做什么
#### 1 基本HTTP服务
- 静态文件处理（如HTML静态网页），处理索引文件以及支持自动个索引
- 打开并自行管理文件描述符缓存
- 提供反向代理服务，并可以使用缓存加速反向代理，同时完成简单的负载均衡及容错
- 提供缘层FastCGI服务缓存机制，加速访问，同时完成简单的负载均衡及容错
- 过滤器功能，Nginx基本过滤器有：gzip压缩、ranges支持、chunked支持、XSTL、SSI、以及图像缩放等
- 支持SSL
#### 2 高级HTTP服务
- 基于名字和IP的虚拟主机设置
- 支持HTTP/1.0中的KEEP-ALIVE模式和PipeLined模型连接
- 支持重新加载配置及升级时无需中断正在处理的请求
- 自定义日志格式、带缓存的日志写操作以及快速日志轮转
- 提供3XX～5XX错误代码重定向功能
- 支持HTTP DAV模块，从而为HTTP WEBDAV提供PUT、DELETE、MKCOL、COPY、以及MOVE方法
- 支持FLV流和MP4传输
- 支持网络监控，基于客户端IP地址和HTTP基本认证机制的访问控制，速率限制，来自同一IP的同时连接数或请求限制等
- 支持Perl语言
#### 邮件代理服务
- 使用外部HTTP认证服务器重定向 到用户的IMAP/POP3后端，并支持IMAP认证方式（LOGIN、AUTH LOGIN/PLAIN/CRAM-MD5）和POP3认证方式(USER/PASS、APOP、AUTH LOGIN/PLIAN/CRAM-MD5)
- 使用外部HTTP认证服务器认证用户后重定向到内部STMP后端，并支持 STMP认证方式
- 支持邮件代理服务下的安全套接层安全协议SSL
- 支持纯文本通信协议的扩展协议STARTTLS
## 优点
在Nginx之前市面上已经有Apache，tomcat，IIS等服务器
与这些Web服务器相比，Nginx具有高并发，低内存消耗的优势，Nginx在处理静态文件方面比其他服务器要优秀很多。

## Nginx的Rewrite功能
    地址重写与地址转发
在ngx_http_rewrite_module中
用法：
```
rewrite regex replacement [flag]
```
flag 为 last，break，redirect，permanent

～ 大小写敏感
～* 大小写不敏感

nginx 缓存

Proxy Store， Proxy Cache，基于memcache的缓存机制，
Squid联合，第三方模块 ncache

## Nginx 内存分配
当申请的内存大于4kb（一页内存的大小）时，该内存空间可用于分配的最大内存数据块不能超过4KB

Nginx服务器架构
