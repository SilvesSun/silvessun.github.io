---
title: Defer, Panic, and Recove
date: 2018-05-25 16:07:05
tags:
- go关键字
- 翻译
categories:
- golang学习
---


本文译自(Defer, Panic, and Recove)[https://blog.golang.org/defer-panic-and-recover]

go既有常见的流程控制语句: if, for, switch, goto, 也有可用于goroutine的go语句. 下面介绍一些不那么常见的:defer, pannic和recover

## defer
defer后面的表达式将会被放进一个list中, list中保存的被调用的函数将会在其他函数返回后执行. defer通常用于简化各种清理操作.

我们以将一个文件中的内容复制到另一个文件中为例:

```go
func CopyFile(dstName, srcName string) (written int64, err error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }

    dst, err := os.Create(dstName)
    if err != nil {
        return
    }

    written, err = io.Copy(dst, src)
    dst.Close()
    src.Close()
    return
}
```

正常情况下, 上述代码能正常执行, 但是当 os.Create 发生错误时, 将会导致源文件未被关闭. 一个很简单的补救措施是在第二个return语句之前添加close操作. 但是在复杂的程序中这种bug未必能及时发现并处理. defer正是用于此.

```go
func CopyFile(dstName, srcName string) (written int64, err error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }
    defer src.Close()

    dst, err := os.Create(dstName)
    if err != nil {
        return
    }
    defer dst.Close()

    return io.Copy(dst, src)
}
```

defer允许打开文件后立即关闭, 而不用考虑函数中有多少return语句

defer语句的行为简单且可预测, 依据于三条规则

### defer表达式的值在defer表达式定义时就已明确
```go
func a() {
    i := 0
    defer fmt.Println(i)
    i++
    return
}
```

上述代码中, defer表达式中用到了i这个变量，i在初始化之后的值为0，接着程序执行到defer表达式这一行，表达式所用到的i的值就为0了，接着，表达式被放入list，等待在return的时候被调用。所以，后面尽管有一个i++语句，仍然不能改变表达式 fmt.Println(i)的结果. 最终程序打印的结果为0

### defer表达式的调用顺序是按照先进后出的方式

```go
func b() {
    for i := 0; i < 4; i++ {
        defer fmt.Print(i)
    }
}
```
打印结果为"3210"

### defer表达式中可以修改函数中的命名返回值

```go
func c() (i int) {
    defer func() { i++ }()
    return 1
}
```

上面这个例子中, 返回值变量为i, 虽然函数return值为1, 但是因为defer在return后调用, 将i值加一, 最终返回值为2.


## panic 和 recover
panic是一个內建函数, 可以中断函数流程, 并进入panic处理流程. 当函数F调用panic, 函数F的执行被中断, 但是F中的defer函数可以正常执行, 然后F返回到被调用处. 在被调用处继续panic流程, 直到程序奔溃时的所有goroutine返回. 

recover同样是个內建函数. 用于恢复panic流程且仅在defer中有效. 函数正常执行过程中调用recover会返回nil且没有其他任何效果. 如果当前goroutine处于panic流程中, 调用recover可以使程序恢复正常的执行.


```go
package main

import "fmt"

func main() {
    f()
    fmt.Println("Returned normally from f.")
}

func f() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered in f", r)
        }
    }()
    fmt.Println("Calling g.")
    g(0)
    fmt.Println("Returned normally from g.")
}

func g(i int) {
    if i > 3 {
        fmt.Println("Panicking!")
        panic(fmt.Sprintf("%v", i))
    }
    defer fmt.Println("Defer in g", i)
    fmt.Println("Printing in g", i)
    g(i + 1)
}

```

函数g当参数大于3时陷入panic, 否则i+1并递归调用自己. 函数f定义了一个defer函数, 调用recover. 程序输出如下:

```
Calling g.
Printing in g 0
Printing in g 1
Printing in g 2
Printing in g 3
Panicking!
Defer in g 3
Defer in g 2
Defer in g 1
Defer in g 0
Recovered in f 4
Returned normally from f.

```




