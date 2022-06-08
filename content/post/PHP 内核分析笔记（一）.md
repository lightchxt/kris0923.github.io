---
title: PHP 内核分析笔记（一）单进程SAPI生命周期
draft: false
categories: ["php"]
tags: ["php"]
---
<inernal class="png">http://www.php-internals.com/images/book/chapt02/02-00-php-</inernal>
### 单进程SAPI生命周期
``` flow 
:start
    php -f test.php
        call each extension`s MINIT (模块初始化阶段，调用所有模块的MINIT函数)
            Request test.php
                call each extension`s RINIT (调用所有模块的RINIT函数)
                    Execute test.php
                call each extension`s RSHUTDOWM
            Finsh cleaning up after test.php
        call each extension`s MSHUTDOWM （Web服务器退出或者命令行脚本执行完毕退出）
    terminate test.php
:end
```

### 启动 模块初始化阶段
 * 初始化若干全局变量
 * 初始化若干常量，这里的常量是PHP自己的一些常量，如PHP_VERSION
 * 初始化Zend引擎和核心组件
 * 解析php.ini
 * 全局操作函数的初始化
 * 初始化静态构建的模块和共享模块(MINIT)
 * 禁用函数和类 php_disable_functions, 其禁用的过程是调用zend_disable_function函数将指定的函数名从CG(function_table)函数表中删除,php_disable_classes,将指定的类名从CG(class_table)类表中删除
 
### ACTIVATION
* 激活Zend引擎
* 激活SAPI
* 环境初始化
  这里的环境初始化是指在用户空间中需要用到的一些环境变量初始化，这里的环境包括服务器环境、请求数据环境等。 实际到我们用到的变量，就是$_POST、$_GET、$_COOKIE、$_SERVER、$_ENV、$_FILES。 和sapi_module.default_post_reader一样，sapi_module.treat_data的值也是在模块初始化时， 通过php_startup_sapi_content_types函数注册了默认数据处理函数为main/php_variables.c文件中php_default_treat_data函数
* 模块请求初始化 (Call each extension's RINIT)
### 运行
* php_execute_script 函数包含了运行PHP脚本的全部过程
### DEACTIVATION
    PHP关闭请求的过程是一个若干个关闭操作的集合，这个集合存在于php_request_shutdown函数中
    1. 调用所有通过register_shutdown_function()注册的函数
    2. 执行所有可用的__destruct函数
    3. 将所有的输出刷出去
    4. 发送HTTP应答头
    5. 遍历每个模块的关闭请求方法 call each extension`s RSHUTDOWM
    6. 销毁全局变量表（PG(http_globals)）的变量
    7. 通过zend_deactivate函数，关闭词法分析器、语法分析器和中间代码执行器
    8. 调用每个扩展的post-RSHUTDOWN函数
    9. 关闭SAPI，通过sapi_deactivate销毁SG(sapi_headers)、SG(request_info)等的内容
    10. 关闭流的包装器、关闭流的过滤器
    11. 关闭内存管理
    12. 重新设置最大执行时间
### 结束
* flush
* 关闭Zend引擎
* 在关闭所有的模块后，PHP继续销毁全局函数表，销毁全局类表、销售全局变量表等。 通过zend_shutdown_extensions遍历zend_extensions所有元素，调用每个扩展的shutdown函数