---
title: "《C程序设计语言学习笔记》（二）"
date: 2023-07-11T23:35:33+08:00
draft: true    
categories: ["C"]
tags: ["C", "编程语言"]
---

# 控制流
第三章介绍了C语言的控制流结构，包括if/else语句、switch语句、while循环和for循环等。

## if/else语句
if/else语句是C语言中最常用的选择结构。它可以根据某个条件来选择执行语句的分支。if/else语句的基本语法如下：

```
if (condition)
    statement1;
else
    statement2;
```

## switch语句
switch语句是另一种选择结构，它可以根据某个表达式的值选择执行语句的分支。switch语句的基本语法如下：

```
switch (expression)
{
    case const-express1:
        statement1;
        break;
    case const-express2:
        statement2;
        break;
    default:
        statement3;
        break;
}
```

## while循环与for循环
while循环和for循环都是C语言中的循环结构。while循环的基本语法如下：

```
while (condition)
    statement;
```

for循环的基本语法如下：

```
for (expr1; expr2; expr3)
    statement;
```

## 函数与程序结构
第四章介绍了C语言的函数和程序结构相关的知识。

## 外部变量（external）
在C语言中，变量可以定义为外部变量，它们永久存在，可以被多个函数共享。如果需要在多个函数之间共享数据，最方便的方法就是将这些数据定义为外部变量。

## 作用域规则
C语言使用作用域规则来确定变量的可见性。作用域是指可以访问变量的区域，C语言中有几种作用域，包括块作用域、函数作用域和文件作用域等。

## 头文件
头文件是一个包含函数原型、常量定义和符号定义等信息的文件，可以被多个源文件引用。C语言的标准头文件包含了许多常用的函数原型和定义，例如stdio.h、stdlib.h和math.h等。

## 静态变量
静态变量是指在函数内定义，但生命周期和程序运行时间一样长的变量。静态变量在函数调用之间会保留下它们的值，这使得它们非常有用，例如用于记录函数调用的次数。

## 寄存器变量
寄存器变量是指程序员请求编译器将变量存储在CPU中的寄存器中，从而提高程序的运行速度。但是，由于寄存器的数量有限，所以编译器可能会忽略程序员的请求，把变量存储在内存中。

## 程序块结构
C语言中的程序块是由一对花括号包围的一组语句。在程序块中，可以定义局部变量，这些变量只在程序块内可见。

## 初始化
变量初始化指在定义变量时为变量赋予一个初始值。在C语言中，变量可以通过等号右侧的表达式进行初始化。

## 递归
递归是指一个函数调用自己的过程。在C语言中，递归可以用于解决许多问题，但需要注意递归深度不能太大，否则可能会导致栈溢出等问题。

## C预处理器
C预处理器是一组特殊的指令，它们由#开头，并在编译之前对源代码进行转换。C预处理器包括文件包含、宏替换、条件包含等功能。

### 文件包含：#include
文件包含指令（#include）用于导入其他源文件中的代码。C语言的标准函数库通常是通过文件包含指令进行导入的。

### 宏替换：#define
宏指令（#define）用于定义宏。宏可以简化代码，例如可以定义宏来表示常量或函数。

### 条件包含：#if， #endif， #else， #elif， #ifdef， #ifndef
条件包含指令（#if， #endif， #else， #elif， #ifdef， #ifndef）用于根据某个条件编译源代码。这些指令可以帮助程序员增强代码的可移植性。

