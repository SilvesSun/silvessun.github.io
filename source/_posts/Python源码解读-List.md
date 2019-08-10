---
title: Python源码解读_List
date: 2018-05-08 17:33:35
tags:
- 源码解读
- Dict
categories:
- Python
- 源码
---

<!-- TOC -->

- [1. PyListObject对象](#1-pylistobject对象)
- [2. PyListObject对象的创建与维护](#2-pylistobject对象的创建与维护)
    - [2.1. 创建对象](#21-创建对象)
    - [2.2. 设置元素](#22-设置元素)
    - [2.3. 插入元素](#23-插入元素)
    - [2.4. 删除元素](#24-删除元素)
- [3. PyListObject 对象缓冲池](#3-pylistobject-对象缓冲池)

<!-- /TOC -->

# 1. PyListObject对象

定义:

```c
typedef struct {
    PyObject_VAR_HEAD

    PyObject **ob_item;

    Py_ssize_t allocated;
} PyListObject;
```

问题:
>ob_size 和 allocated的联系

PyListObject 所采用的内存策略并不是需要多少内存才申请多大的内存, 在每一次需要申请内存的时候, PyListObject总会申请一大块内存, 这时申请的内存大小记录在 allocted中, 而实际使用的内存数量则记录在ob_size中.

# 2. PyListObject对象的创建与维护
## 2.1. 创建对象

- PYList_New
  这个函数接受一个size参数, 从而允许我们创建一个PyListObject对象的同时指定该列表初始的元素个数

```c
[listobject.c]

PyObject *
PyList_New(Py_ssize_t size)
{
	PyListObject *op;
	size_t nbytes;

	if (size < 0) {
		PyErr_BadInternalCall();
		return NULL;
	}

    //计算需要的内存数量
	nbytes = size * sizeof(PyObject *);
	/* Check for overflow */

    // 溢出检查
	if (nbytes / sizeof(PyObject *) != (size_t)size)
		return PyErr_NoMemory();
	if (num_free_lists) {
        // 缓冲池可用
		num_free_lists--;
		op = free_lists[num_free_lists];
		_Py_NewReference((PyObject *)op);
	} else {
		op = PyObject_GC_New(PyListObject, &PyList_Type);
		if (op == NULL)
			return NULL;
	}

    // 为PyListObject对象中维护的元素列表申请空间
	if (size <= 0)
		op->ob_item = NULL;
	else {
		op->ob_item = (PyObject **) PyMem_MALLOC(nbytes);
		if (op->ob_item == NULL) {
			Py_DECREF(op);
			return PyErr_NoMemory();
		}
		memset(op->ob_item, 0, nbytes);
	}
	op->ob_size = size;
	op->allocated = size;
	_PyObject_GC_TRACK(op);
	return (PyObject *) op;
}

```

Python的列表对象实际上是分为两部分的, 一是PyListObject对象本身, 二是PyListObject对象维护的元素列表. 这是两块分离的内存, 通过ob_item建立联系.

在Python2.5 中, 默认free_lists中最多会维护80个PyListObject对象

## 2.2. 设置元素

在第一个PyListObject创建的时候, 这时的num_free_lists为0, 这时会绕过对象缓冲池, 转而调用PyObject_GC_New在系统堆上创建一个新的PyListObject对象

当向list中添加元素时, 发生的情况是:

```c
int
PyList_SetItem(register PyObject *op, register Py_ssize_t i,
               register PyObject *newitem)
{
	register PyObject *olditem;
	register PyObject **p;

    // 类型检查
	if (!PyList_Check(op)) {
		Py_XDECREF(newitem);
		PyErr_BadInternalCall();
		return -1;
	}

    // 索引检查
	if (i < 0 || i >= ((PyListObject *)op) -> ob_size) {
		Py_XDECREF(newitem);
		PyErr_SetString(PyExc_IndexError,
				"list assignment index out of range");
		return -1;
	}
	p = ((PyListObject *)op) -> ob_item + i;
	olditem = *p;
	*p = newitem;
	Py_XDECREF(olditem);
	return 0;
}

```

设置元素时, 首先会进行类型检查, 然后进行索引的有效性检查, 然后将待加入的PyObject* 指针放到指定的位置, 然后调整引用计数, 将这个位置原来存放的对象的引用计数减1


## 2.3. 插入元素
```c
int
PyList_Insert(PyObject *op, Py_ssize_t where, PyObject *newitem)
{
	if (!PyList_Check(op)) {
		PyErr_BadInternalCall();
		return -1;
	}
	return ins1((PyListObject *)op, where, newitem);
}

static int
ins1(PyListObject *self, Py_ssize_t where, PyObject *v)
{
	Py_ssize_t i, n = self->ob_size;
	PyObject **items;
	if (v == NULL) {
		PyErr_BadInternalCall();
		return -1;
	}
	if (n == PY_SSIZE_T_MAX) {
		PyErr_SetString(PyExc_OverflowError,
			"cannot add more objects to list");
		return -1;
	}

    // 调用list_resize 保证有足够的内存
	if (list_resize(self, n+1) == -1)
		return -1;

	if (where < 0) {
		where += n;
		if (where < 0)
			where = 0;
	}
	if (where > n)
		where = n;
	items = self->ob_item;

    // 移动元素
	for (i = n; --i >= where; )
		items[i+1] = items[i];
	Py_INCREF(v);
	items[where] = v;
	return 0;
}

static int
list_resize(PyListObject *self, Py_ssize_t newsize)
{
	PyObject **items;
	size_t new_allocated;
	Py_ssize_t allocated = self->allocated;

    // [1]
	if (allocated >= newsize && newsize >= (allocated >> 1)) {
		assert(self->ob_item != NULL || newsize == 0);
		self->ob_size = newsize;
		return 0;
	}

	new_allocated = (newsize >> 3) + (newsize < 9 ? 3 : 6) + newsize;
	if (newsize == 0)
		new_allocated = 0;
	items = self->ob_item;
	if (new_allocated <= ((~(size_t)0) / sizeof(PyObject *)))
		PyMem_RESIZE(items, PyObject *, new_allocated);
	else
		items = NULL;
	if (items == NULL) {
		PyErr_NoMemory();
		return -1;
	}
	self->ob_item = items;
	self->ob_size = newsize;
	self->allocated = new_allocated;
	return 0;
}
```

- [1]如果 newsize < allocated && newsize > allocated / 2, 直接调整ob_size大小
- 其他情况直接调用 realloc

![像列表中插入元素](http://p3euxxfa8.bkt.clouddn.com/2018-05-08-16-59-07.png)

## 2.4. 删除元素

PyListObject 的删除操作时通过 listremove 操作完成的

```c
[listobject.c]

static PyObject *
listremove(PyListObject *self, PyObject *v)
{
	Py_ssize_t i;

	for (i = 0; i < self->ob_size; i++) {
		int cmp = PyObject_RichCompareBool(self->ob_item[i], v, Py_EQ);
		if (cmp > 0) {
			if (list_ass_slice(self, i, i+1,
					   (PyObject *)NULL) == 0)
				Py_RETURN_NONE;
			return NULL;
		}
		else if (cmp < 0)
			return NULL;
	}
	PyErr_SetString(PyExc_ValueError, "list.remove(x): x not in list");
	return NULL;
}
```

删除操作, Python遍历整个列表, 将待删除的元素和PyListObject中的每一个元素对比, 大于0表示列表中的某个元素与待删除的元素匹配. 发现匹配的元素将立即调用 list_ass_slice 函数

```c
int
list_ass_slice(PyListObject *a, Py_ssize_t ilow, Py_ssize_t ihigh, PyObject *v)

```

list_ass_slice 的完整功能为:
- a[ilow:ihigh] = v if v != NULL; 替换操作
- del a[ilow:ihigh] = v if v != NULL; 删除

最后一个参数的值决定了实用哪种操作

当进行删除操作时, 实际上是通过简单的移动内存来完成的, 所以调用remove操作时, 一定会导致内存移动操作的产生.

# 3. PyListObject 对象缓冲池

在PyListObject对象被销毁的时候, free_lists中将会获得被销毁的对象

```c
[listobject.c]

static void
list_dealloc(PyListObject *op)
{
	Py_ssize_t i;
	PyObject_GC_UnTrack(op);
	Py_TRASHCAN_SAFE_BEGIN(op)

	// 销毁list对象维护的元素列表
	if (op->ob_item != NULL) {

		i = op->ob_size;
		while (--i >= 0) {
			Py_XDECREF(op->ob_item[i]);
		}
		PyMem_FREE(op->ob_item);
	}

	// 释放PyListObject自身
	if (num_free_lists < MAXFREELISTS && PyList_CheckExact(op))
		free_lists[num_free_lists++] = op;
	else
		op->ob_type->tp_free((PyObject *)op);
	Py_TRASHCAN_SAFE_END(op)
}
```
