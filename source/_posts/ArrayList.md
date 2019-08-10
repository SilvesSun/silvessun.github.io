---
title: ArrayList
date: 2018-01-30 11:39:43
tags:
- Java
- 集合框架
categories:
- Java
---

# 结构图
![constructOfArrayList](img/markdown-img-paste-20180125152600647.png)

可以看到, ArrayList继承自AbstractList, AbstractList继承自AbstractCollection. 主要实现了Collection, List, Cloneable, Serializable, RandomAccess接口.

# 实现

## 基础属性
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    private static final long serialVersionUID = 8683452581122892189L;

    private static final int DEFAULT_CAPACITY = 10;

     private static final Object[] EMPTY_ELEMENTDATA = {};

    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    transient Object[] elementData;

    private int size;

    //...省略
}
```

## 构造方法
```java
public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // defend against c.toArray (incorrectly) not returning Object[]
            // (see e.g. https://bugs.openjdk.java.net/browse/JDK-6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }


```

## 常见方法
### 容量可变的实现

```Java

public void ensureCapacity(int minCapacity) {
        if (minCapacity > elementData.length
            && !(elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
                 && minCapacity <= DEFAULT_CAPACITY)) {
            modCount++;
            grow(minCapacity);
        }
    }

private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

private Object[] grow(int minCapacity) {
        return elementData = Arrays.copyOf(elementData,
                                           newCapacity(minCapacity));
    }

private Object[] grow() {
    return grow(size + 1);
}

private int newCapacity(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity <= 0) {
            if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
                return Math.max(DEFAULT_CAPACITY, minCapacity);
            if (minCapacity < 0) // overflow
                throw new OutOfMemoryError();
            return minCapacity;
        }
        return (newCapacity - MAX_ARRAY_SIZE <= 0)
            ? newCapacity
            : hugeCapacity(minCapacity);
    }

```

第一个方法说明如果长度大于默认的长度, 则扩容. newCapacity说明了扩容的逻辑.
- int newCapacity = oldCapacity + (oldCapacity >> 1); 扩容的大小为3/2的原数组长度
- 如果 newCapacity比 minCapacity 小, 则返回 minCapacity; 如果 newCapacity比最大的数组容量大, 则使用 hugeCapacity.



### add

-  在末尾添加

```java
public boolean add(E e) {
        modCount++;
        add(e, elementData, size);
        return true;
    }
```

 调用add(e, elementData, size)插入到末尾

- 添加指定元素到指定的位置上

```java
public void add(int index, E element) {
        rangeCheckForAdd(index);
        modCount++;
        final int s;
        Object[] elementData;
        if ((s = size) == (elementData = this.elementData).length)
            elementData = grow();
        System.arraycopy(elementData, index,
                         elementData, index + 1,
                         s - index);
        elementData[index] = element;
        size = s + 1;
    }
```
首先校验下标, 当超出默认的size时进行扩容, 然后将index后面的元素全部向后移位, 将要插入的元素赋值, 并将size增加.

### 遍历
主要有三种方法, for, foreach以及迭代器
