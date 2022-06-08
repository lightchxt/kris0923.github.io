---
title: GO语言语法基础
draft: false
date: 2018-012-17 12:18:45
categories: ["go"]
tags: ["go"]
---
# 基础语法
1. i++, i-- 在go语言中是语句，而不像其他语言一样是表达式，所以， j = i++ 在go语言里面是不合法的,并且只支持后缀， --i 是不合法的
2. for 是go里面唯一的循环语句       
    传统的while循环写成     
    for condition {
    
    }
3. 函数 
    
    函数的类型，称为函数签名，函数的实参都是按值传递的
4. 文件结束标示
    
    ```
    err = errors.New("EOF")
    in := bufio.NewReader(os.stdin)
    for {
        r,_,err = in.ReadRune()
        if err == io.EOF {
            break // 文本结束标记
        }
        if err != nil {
            return fmt.Errorf("read failed: $v",err)
        }
        // 处理逻辑
    }
    ```
    go 文本结束标记不怎么友好，为什么要放到error信息中
5. 数组 
    Go 语言中的数组是一种 **值类型**（不像 C/C++ 中是指向首元素的指针），所以可以通过 new() 来创建： var arr1 = new([5]int)。

    那么这种方式和 var arr2 [5]int 的区别是什么呢？arr1 的类型是 *[5]int，而 arr2的类型是 [5]int。

    这样的结果就是当把一个数组赋值给另一个时，需要在做一次数组内存的拷贝操作。例如：

    arr2 := *arr1
    arr2[2] = 100
    这样两个数组就有了不同的值，在赋值后修改 arr2 不会对 arr1 生效。
    
    ==切片是引用类型，所以不能用指针指向切片==    
    
6. 参数传递  
    go中所有的东西都是按值传递的，每次调用函数时，传入的数据都会被复制。对于具有值接收者的方法，在调用该方法时将复制该值。因为所有的参数都是通过值传递的，这就可以解释为什么 *Cat 的方法不能被 Cat 类型的值调用了。任何一个  Cat 类型的值可能会有很多 *Cat 类型的指针指向它，如果我们尝试通过 Cat 类型的值来调用 *Cat 的方法，根本就不知道对应的是哪个指针。
# go 并发编程
    1. goroutine 和通道
        
        程序启动时，只有一个 goroutine 来调用 main 函数，称为主 goroutine，新的 goroutine 通过 go 语句创建
，就是在普通的函数或方法前加上 go 关键字，当 main 函数返回时，所有的 goroutine 都将暴力终止。  
    2. 缓冲通道
    ```
    func mirroredQuery() string{
        responses := make(chan string,3)
        go func() {responses <- request("asia.gopl.io")}()
        go func() {responses <- request("europe.gopl.io")}()
        go func() {responses <- request("americas.gopl.io")}()
        return <-responses //return the questest response
    }
    ```
    主goroutine只接收第一个返回的，或者极端情况下，两个较慢的服务器还没有返回响应，就返回了结果
    3. goroutine 泄漏   
        如果使用无缓冲通道，两个较慢的通道goroutine将被卡住（因为他们发送响应结果到通道没有goroutine来接收），这种情况叫goroutine泄漏。不要通过共享内存来通信，应该通过通信来共享内存
        