---
title: PHP 知识总结（一）
draft: true
categories: ["php"]
tags: ["php"]
---
# 语法基础
## 变量类型
1. boolean
2. integet
3. float
    ```
    # 判断是否是一个数值类型
    bool is_nan(float $val)
    ```
4. string
```
    // 字符串转数值
    //如果 字符串中包含 '.', 'e'或者'E' 转换为float，其余转为 int
```
5. Array
    ```
    // 数组的key只能是int 或string
    合法的整型字符串会被转成int，例如："8"，"08"不会，因为08不是合法的十进制数
    浮点数也会被转换为整型，意味着其小数部分会被舍去。例如键名 8.7 实际会被储存为 8。
    布尔值也会被转换成整型。即键名 true 实际会被储存为 1 而键名 false 会被储存为 0。
    Null 会被转换为空字符串，即键名 null 实际会被储存为 ""。
    数组和对象不能被用为键名。坚持这么做会导致警告：Illegal offset type
    默认键： 当前最大整数索引值加1
    // 其他类型转为数组
    $a = array($a);
    or
    $a = (array) $a;
    如果一个 object 类型转换为 array，则结果为一个数组，其单元为该对象的属性。键名将为成员变量名，不过有几点例外：整数属性不可访问；私有变量前会加上类名作前缀；保护变量前会加上一个 '*' 做前缀。这些前缀的前后都各有一个 NULL 字符。这会导致一些不可预知的行为：
    class A {
        private $A; // This will become '\0A\0A'
    }

    class B extends A {
        private $A; // This will become '\0B\0A'
        public $AA; // This will become 'AA'
    }

    var_dump((array) new B());
    //上例会有两个键名为 'AA'，不过其中一个实际上是 '\0A\0A'
    ```
6. Object
7. Reource 资源类型
    ```
    资源 resource 是一种特殊变量，保存了到外部资源的一个引用
    可以用 is_resource()函数测定一个变量是否是资源，函数 get_resource_type()则返回该资源的类型
    ```
8. NULL
9. Callback/Callable
## 魔术方法
    http://php.net/manual/zh/language.oop5.magic.php#
    ```
    __construct()，
    __destruct()， 
    __call()， 
    __callStatic()， 
    __get()， 
    __set()， 
    __isset()， 
    __unset()， 
    __sleep()， 
    __wakeup()， 
    __toString()， 
    __invoke()， 
    __set_state()， 
    __clone() 和 
    __debugInfo() 
    ```

## 运算符
算数运算符

$a ** $b 表示已$a的$b次方（PHP5.6之后支持）, $a ** $b = exp($a*log($a));

赋值运算符=

一般情况下赋值运算是将原变量的值拷贝到新变量中，但有个例外，就是碰到对象*object*时是引用赋值，除非明确使用*clone*关键字