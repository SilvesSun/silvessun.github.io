---
title: LinkedList
date: 2018-02-02 17:52:54
tags:
- Java
- 集合框架
categories:
- Java
---

LinkedList的底层实现是双向链表, 无容量限制, 但是双向链表需要更多的空间, 同时需要额外的链表指针操作. LinkedList为在列表的开头和结尾获取, 移除以及列表中插入提供了统一的命名

## 结构
![LinkedList](http://p3euxxfa8.bkt.clouddn.com/linkedList.png)

## 用途
基于LinkedList的结构, 可以将其作为栈, 队列和双端队列来使用

## 源码阅读
### LinkedList基本内容
```Java
//  LinkedList继承自AbstractSequentialList, 实现了List<E>, Deque<E>, Cloneable, java.io.Serializable接口

public class LinkedList<E>   extends AbstractSequentialList<E> implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
    //  元素个数
    transient int size = 0;

    // 首尾节点
    transient Node<E> first;
    transient Node<E> last;

    // 默认的构造方法, 生成一个空的链表
    public LinkedList() {}

    // 根据集合中的元素生成链表
    public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);
    }

    //  在链表头插入节点
    // Node的定义    Node(Node<E> prev, E element, Node<E> next)
    private void linkFirst(E e) {
        final Node<E> f = first; // f指向第一个节点
        final Node<E> newNode = new Node<>(null, e, f); //要插入的节点, 新节点的前指针为空, 后指针指向第一个节点, 值为e
        first = newNode; // 将first指向新构建的节点, 之前节点的信息保存在f中.

        // 判断之前链表是否为空, 为空的话就插入过后只有一个元素, 所以讲后指针指向新节点, 反之将旧节点的前指针指向新节点
        if (f == null)
            last = newNode;
        else
            f.prev = newNode;

        // 链表长度加1, 修改次数加1
        size++;
        modCount++;
    }

    // 尾部添加元素, 基本分析类似添加到头部
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }

    // 在一个节点之前插入新节点
    void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        final Node<E> pred = succ.prev; // 记录当前节点的上一节点
        final Node<E> newNode = new Node<>(pred, e, succ); // 构建一个新节点, 新节点的前指针指向当前节点的上一节点, 后指针指向当前节点
        succ.prev = newNode;  // 当前节点的前节点指向新节点

        // 判断当前节点的上一节点是否为空, 为空说明当前节点为头节点.
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        size++;
        modCount++;
    }

    // 删除头节点, 并返回头节点的值
    private E unlinkFirst(Node<E> f) {
        // assert f == first && f != null;
        final E element = f.item;
        final Node<E> next = f.next;
        f.item = null;
        f.next = null; // help GC
        first = next; // 新头部指向第二个元素
        if (next == null)  // 如果next为空, 说明只有一个节点, 删除后为空链表
            last = null;
        else
            next.prev = null;

        // 长度-1, 修改次数加1 返回头元素值.
        size--;
        modCount++;
        return element;
    }

    // 删除尾节点, 返回尾节点的值
    private E unlinkLast(Node<E> l) {
        // assert l == last && l != null;
        final E element = l.item;
        final Node<E> prev = l.prev;
        l.item = null;
        l.prev = null; // help GC
        last = prev;
        if (prev == null)
            first = null;
        else
            prev.next = null;
        size--;
        modCount++;
        return element;
    }

    // 删除一个非空节点x
    E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        // 如果上一节点为空, 说明当前节点为头节点, 将下一节点设置为头节点
        if (prev == null) {
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }
        // 尾节点的情况
        if (next == null) {
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }

        x.item = null;
        size--;
        modCount++;
        return element;
    }

    // 获取头/尾节点值
    public E getFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return f.item;
    }

    public E getLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return l.item;
    }

    // 可用的删除首尾元素的方法, 本质上分别调用的是unlinkFirst, unlinkLast方法
    public E removeFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return unlinkFirst(f);
    }

    public E removeLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return unlinkLast(l);
    }

    // 添加首尾元素
    public void addFirst(E e) {
        linkFirst(e);
    }

    public void addLast(E e) {
        linkLast(e);
    }

    // 是否包含元素o
    public boolean contains(Object o) {
        return indexOf(o) >= 0;
    }

    public int size() {
        return size;
    }

    // 尾部添加元素
    public boolean add(E e) {
        linkLast(e);
        return true;
    }

    // 删除元素. 这个方法是删除LinkedList中所有的值为o的元素
    public boolean remove(Object o) {
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }

    // 将c中所有的元素添加到LinkedList中
    public boolean addAll(Collection<? extends E> c) {
        return addAll(size, c); // 在尾部添加
    }

    // 在指定位置添加c中的所有元素
    public boolean addAll(int index, Collection<? extends E> c) {
        checkPositionIndex(index);  // 首先确定index未超界

        Object[] a = c.toArray();  // 将集合转为数组
        int numNew = a.length;
        if (numNew == 0)
            return false;

        Node<E> pred, succ; // 声明当前上一节点和当前节点

        if (index == size) {
            succ = null;
            pred = last;
        } else {
            succ = node(index);
            pred = succ.prev;
        }


        //for循环结束后，a里面的元素都添加到当前链表里面，后向添加
        for (Object o : a) {

            // SuppressWarnings抑制编辑器会产生的警告
            @SuppressWarnings("unchecked") E e = (E) o;
            Node<E> newNode = new Node<>(pred, e, null);
            if (pred == null)
                first = newNode;
            else
                pred.next = newNode;
            pred = newNode;
        }

        if (succ == null) {
            last = pred;
        } else {
            pred.next = succ;
            succ.prev = pred;
        }

        size += numNew; // 长度设为原长度加c的长度
        modCount++; // 修改次数加1
        return true;
    }

    // 清除链表里面的所有元素
    public void clear() {
        // Clearing all of the links between nodes is "unnecessary", but:
        // - helps a generational GC if the discarded nodes inhabit
        //   more than one generation
        // - is sure to free memory even if there is a reachable Iterator
        for (Node<E> x = first; x != null; ) {
            Node<E> next = x.next;
            x.item = null; //释放值结点，便于GC回收
            x.next = null;
            x.prev = null;
            x = next;
        }
        first = last = null;
        size = 0;
        modCount++;
    }

    // 获得index处的值
    public E get(int index) {
        checkElementIndex(index);
        return node(index).item;
    }

    // 设置指定位置的值
    public E set(int index, E element) {
        checkElementIndex(index);
        Node<E> x = node(index);
        E oldVal = x.item;
        x.item = element;
        return oldVal;
    }

    // 指定位置添加一个元素
    public void add(int index, E element) {
        checkPositionIndex(index);

        if (index == size)
            linkLast(element);
        else
            linkBefore(element, node(index));
    }

    // 移除指定位置的值
    public E remove(int index) {
        checkElementIndex(index);
        return unlink(node(index));
    }

    private boolean isElementIndex(int index) {
        return index >= 0 && index < size;
    }

    private boolean isPositionIndex(int index) {
        return index >= 0 && index <= size;
    }

    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size;
    }

    private void checkElementIndex(int index) {
        if (!isElementIndex(index))
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    private void checkPositionIndex(int index) {
        if (!isPositionIndex(index))
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    // 定位index位置的元素
    Node<E> node(int index) {
        // assert isElementIndex(index);
        // index小于size的一半时，从头向后找
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            // index大于size的一半时，从尾向前找
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }

    //定位元素，首次出现的元素的值为o的结点序号
    public int indexOf(Object o) {
        int index = 0;
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null)
                    return index;
                index++;
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item))
                    return index;
                index++;
            }
        }
        return -1;
    }

    // 最后一次出现o的位置
    public int lastIndexOf(Object o) {
        int index = size;
        if (o == null) {
            for (Node<E> x = last; x != null; x = x.prev) {
                index--;
                if (x.item == null)
                    return index;
            }
        } else {
            for (Node<E> x = last; x != null; x = x.prev) {
                index--;
                if (o.equals(x.item))
                    return index;
            }
        }
        return -1;
    }
}
```

### LinkedList作为队列使用时的相关源码
```java
// 返回第一个元素的值
public E peek() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
    }

// 返回第一个节点
public E element() {
        return getFirst();
    }

// 弹出第一个节点
public E poll() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }

// 删除第一个节点
public E remove() {
        return removeFirst();
    }

// 添加元素到尾节点
public boolean offer(E e) {
        return add(e);
    }

```

### LinkedList的双端队列操作
```java
// 首/尾插入元素
public boolean offerFirst(E e) {
        addFirst(e);
        return true;
    }

public boolean offerLast(E e) {
        addLast(e);
        return true;
    }

// 获取首尾的值
public E peekFirst() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
     }

public E peekLast() {
     final Node<E> l = last;
     return (l == null) ? null : l.item;
 }

// 删除首尾元素
 public E pollFirst() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }

public E pollLast() {
        final Node<E> l = last;
        return (l == null) ? null : unlinkLast(l);
    }

// 添加头部节点
public void push(E e) {
        addFirst(e);
    }

//弹出第一个结点
public E pop() {
    return removeFirst();
}

// 删除值为o的结点
public boolean removeFirstOccurrence(Object o) {
        return remove(o);
    }

public boolean removeLastOccurrence(Object o) {
        if (o == null) {
            for (Node<E> x = last; x != null; x = x.prev) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = last; x != null; x = x.prev) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
```
