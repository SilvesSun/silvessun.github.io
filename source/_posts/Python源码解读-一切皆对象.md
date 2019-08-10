---
title: Python源码解读(一切皆对象)
date: 2018-08-07 10:30:36
tags:
categories:
---

在Python中, 人们常说一切皆对象. 不论是基本的数据类型如int, str, 还是Python中的一等公民函数, 或者是面向对象编程中的类, 在Python中都是一个个的对象. 那么什么是对象? 为什么说Python中一切都是对象? 对象的实现原理是什么? 我想这三个问题或多或少都曾在每一个接触过Python的人的心里思考过. 前段时间也阅读了![Python源码剖析](https://book.douban.com/subject/3117898/)一书中相关内容, 这里想把自己的一些理解记录下来.

什么是对象? 对于人来说, 对象是一个形象的可以感知的词. 比如说我们说一只笔, 这里笔就是一个对象; 一本书, 书也是一个对象等等. 但是对于计算机来说, 事情可能并没有这么直观. 在计算机的世界里, 一切都是字节, 人们将对象的概念抽象出来, 反应到计算机的世界里就是一个个不同的字节. 所以不同的对象, 对于计算机来说也就是在内存中占用的不同的位置. 所以对于计算机来说, 对象可创建, 可销毁. 

谈到对象这个概念, 还想多说一句*鸭子类型*(duck type)

> 当看到一只鸟走起来像鸭子、游泳起来像鸭子、叫起来也像鸭子，那么这只鸟就可以被称为鸭子

我们知道, 鸭子类型是动态类型的一种风格. 描述一个对象, 不像静态语言一样是继承自特定的类, 实现特定的接口; 而是说由当前方法和属性的集合来决定.

不同的语言描述这块内存的方式也不一样. 就对象来说, c中可以用struct这样的结构体来模拟. 这块内存可能是连续的, 也可能是离散的, 但是这些都不是重点, 重点是我们可以用结构体这样的一个概念来代替这块内存, 也就是这个对象. 我们知道, Python的底层实现语言是c, 这也就意味着Python中的对象其本质也就是用一个结构体描述的一块内存空间. 

## 一切皆对象
当我们说Python中一切皆对象的时候, 背后的意思是在Python中, 或者说在Python的底层实现中, 有着一个同样的东西用来标识这是一个对象. 比如说整数对象的结构体定义:

```C
typedef struct {
    PyObject_HEAD
    long ob_ival;
} PyIntObject;
```

或者说列表
```c
typedef struct {
    PyObject_VAR_HEAD

    PyObject **ob_item;

    Py_ssize_t allocated;
} PyListObject;
```

可以看到, 不同的对象结构体中都有相同的东西. 直观来看就是PyObject这个结构体了. 可以说, Python对象的基础就是PyObject. PyObject这个结构体的本身是对一个宏变量的引用, 这个宏变量就是PyObject_HEAD.

```c
[object.h]

#ifdef Py_TRACE_REFS
/* Define pointers to support a doubly-linked list of all live heap objects. */
#define _PyObject_HEAD_EXTRA            \
    struct _object *_ob_next;           \
    struct _object *_ob_prev;

#define _PyObject_EXTRA_INIT 0, 0,
#else
#define _PyObject_HEAD_EXTRA
#define _PyObject_EXTRA_INIT
#endif

/* PyObject_HEAD defines the initial segment of every PyObject. */
#define PyObject_HEAD                   \
    _PyObject_HEAD_EXTRA                \
    Py_ssize_t ob_refcnt;               \
    struct _typeobject *ob_type;

```

如果不考虑Py_TRACE_REFS时, PyObject的定义可以简化为

```c
typedef struct _object {
    int ob_refcnt;
    struct _typeobject *ob_type;
}
```

这里, 整型变量 ob_refcnt 与Python的内存管理机制有关, 它实现了基于引用计数的垃圾收集机制;ob_type是一个指向_typeobject结构体的指针, 这个结构体用来指定一个对象类型的类型对象.

所以说, Python对象的核心机制就是引用计数和类型信息. 在PyObject中定义的类容将出现在每一个Python对象所占有的内存的开始的字节中, 同时对于任何一个特定的对象, 还有一些额外的信息来表明这些对象. 比如说整数对象定义中的长整型ob_ival.

## 可变对象
在上面考虑整型对象和list对象时, 有一个明显的区别就是对于列表来说, 其定义中开头为 PyObject_VAR_HEAD. PyObject_HEAD和 PyObject_VAR_HEAD 分别对应于P定长对象和变长对象. 很明显, 定长对象是定义后其分配的内存大小就保持不变, 而变长对象就存在变的可能. 比如说list中添加元素, 删除元素等. PyObject_VAR_HEAD 这个对象描述了变长对象相关的信息. 其中 ob_size 描述了实际对象中保存的元素数量. 

```c
#define PyObject_VAR_HEAD               \
    PyObject_HEAD                       \
    Py_ssize_t ob_size; /* Number of items in variable part */
#define Py_INVALID_SIZE (Py_ssize_t)-1

/* Nothing is actually declared to be a PyObject, but every pointer to
 * a Python object can be cast to a PyObject*.  This is inheritance built
 * by hand.  Similarly every pointer to a variable-size Python object can,
 * in addition, be cast to PyVarObject*.
 */
 typedef struct {
     PyObject_VAR_HEAD
 } PyVarObject;

```

## 类型对象

在上面我们说过, 包含在每一个对象内存开始字节的相关类容描述了对象共有的信息. 但是具体到每一个对象, 在创建对象的时候, 还需要知道对象的大小, 对象的类型等元信息, 这些信息伴随着对象的整个生命周期. 这些相关的信息在类型对象中描述(\_typeobject)

```c
typedef struct _typeobject {
    PyObject_VAR_HEAD
    const char *tp_name;
    Py_ssize_t tp_basicsize, tp_itemsize;

    destructor tp_dealloc;
    printfunc tp_print;
    ...
} PyTypeObject

```

在 \_typeobject 中定义的信息包括

- 类型名 tp_name, 主要是Python内部以及调试的时候使用
- 创建该类型对象时分配的内存空间大小的信息, 即 tp_basicsize, tp_itemsize
- 该对象相关联的操作信息
- 类型的类型信息