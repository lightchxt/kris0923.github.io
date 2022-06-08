---
title: PHP5与PHP7的zval比较
date: 2019-07-17 12:16:53
draft: false
categories: ["php"]
tags: ["php"]
---
## PHP5中zval定义
```C
struct _zval struct {
    zvalue_value value; // 16
    zend_unit refcount_gc; // 4字节 引用计数
    zend_uchar type;        // 1 
    zend_uchar is_ref__gc; // 1 是否为引用类型
}

typedef union _avalue_value {
    long lval; // 8
    double dval; // 8
    struct {
        char *val;
        int len;
    } str; // 16
    HashTable *ht; // HashTable 数组 8
    zend_object_value obj; // 12
    zend_ast * ast; // 8
} zvalue_value;
```

为了解决*循环引用问题*，PHP5.3之后通过重写分配zval的宏，对zval进行扩充，新的分配方式：
```
#undef ALLOC_ZVAL
#define ALLOC_ZVAL (z)
    do {
        (z) = (zval*) emalloc (sizeof(zval_gc_info));
        GC_ZVAL_INT (z);
    } while(0)

// _zval_gc_info 结构如下：
typedef struct _zval_gc_info {
    zval z;
    union {
        gc_root_buffer *buffer;
        struct _zval_gc_info *next;
    }
}
```

除此之外，在开启Zend内存池的情况下，zval_gc_info 在内存池中分配，内存池会为每个zval_gc_info额外申请一个大小为16字节的zend_mm_block结构体，用来存放内存相关信息，zend_mm_block结构如下：
```
typedef struct _zend_mm_block_info {
    size_t _size;
    size_t _prev;
} zend_mm_block_info;
typedef struct _zend_mm_block {
    zend_mm_block_info info;
} zend_mm_block;
```
最终一个变量在PHP5中占内存大小为48个字节（48个字节其实有很多浪费），占用情况如图：

> 1.zend_mm_block 是啥
2.为什么会有循环引用的问题

## PHP7的zval
```
typedef union _zend_value {
	zend_long         lval;				/* long value */
	double            dval;				/* double value */
	zend_refcounted  *counted; 			// 引用计数
	zend_string      *str;
	zend_array       *arr;
	zend_object      *obj;
	zend_resource    *res;
	zend_reference   *ref;
	zend_ast_ref     *ast;
	zval             *zv;
	void             *ptr;
	zend_class_entry *ce;
	zend_function    *func;
	struct {
		uint32_t w1;
		uint32_t w2;
	} ww;
} zend_value;

struct _zval_struct {
	zend_value        value;			/* value */
	union {
		struct {
			ZEND_ENDIAN_LOHI_4(
				zend_uchar    type,			/* active type 记录变量类型*/
				zend_uchar    type_flags,
				zend_uchar    const_flags,
				zend_uchar    reserved)	    /* call info for EX(This) */
		} v; /* 4字节 */
		uint32_t type_info; // 4字节 其实type_info 就是v 中4个char的组合
	} u1;
	union {
		uint32_t     next;                 /* hash collision chain */
		uint32_t     cache_slot;           /* literal cache slot */
		uint32_t     lineno;               /* line number (for ast nodes) */
		uint32_t     num_args;             /* arguments number for EX(This) */
		uint32_t     fe_pos;               /* foreach position */
		uint32_t     fe_iter_idx;          /* foreach iterator index */
		uint32_t     access_flags;         /* class constant access flags */
		uint32_t     property_guard;       /* single property guard 单一属性保护*/
		uint32_t     extra;                /* not further specified 保留字段，暂无意义*/
	} u2;
};


```
PHP7的zval中value占8字节，u1占4字节，u2为辅助字段，占4字节，这样zval一共占16字节，即使没有u2，在内存对齐的情况下，zval也要占16字节，用u2记录了一些特殊信息，并不会浪费内存，反而是对内存的充分利用

PHP7 Z_VAL内存占用图
![php7_z_val](https://github.com/kris0923/kris0923.github.io/blob/master/images/php_z_val.png)