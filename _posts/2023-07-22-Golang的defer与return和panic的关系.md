---
title: "return/defer/panic"
layout: post
date: 2023-07-22
image: /assets/images/markdown.jpg
headerImage: false
tag:
- 笔记
category: golang
---

# return的实现逻辑

+ 第一步给返回值赋值(若是有名返回值直接赋值，匿名返回值 则 先声明再 赋值) ;
+ 第二步调用RET返回指令并传入返回值，RET会检查是否存在defer语句，若存 在就先逆序插播 defer语句 ;
+ 最后 RET 携带返回值退出函数 。
+ return 不是一个原子操作，函数返回值与 RET 返回值并不一定一致

## defer、 return、返回值三者顺序

+ return 最先给返回值赋值；
+ 接着 defer 开始执行一些收尾工作；
+ 最后 RET 指令携带返回值退出函数。

# defer 内容

+ defer 后面必须是函数调用语句，defer a = 1 + 1 或者 defer a++ 都是错误的；
+ defer 后面的语句在函数执行结束的时候才会被调用，顺序是后入先出；
+ defer 后面的语句参数是实时解析的；

## defer执行时修改变量

defer函数定义时，对外部变量的引用方式有两种，分别是函数参数以及作为闭包引用。

+ 在作为函数参数的时候，在defer定义时就把值传递给defer，并被缓存起来。（实时解析）
+ 如果是作为闭包引用，则会在defer真正调用的时候，根据整个上下文去确定当前的值。

闭包

```golang
func f() (result int) {
    defer func() {
        result++
    }()
    return 0
}
```
return xxx这一条语句并不是一条原子指令  
含有defer函数的外层函数，返回的过程是这样的：

+ 先给返回值赋值
+ 调用defer函数 
+ 返回到更上一级调用函数中

```
返回值 = xxx

调用defer函数(这里可能会有修改返回值的操作)

return 返回值

```

示例

```
func f() (result int) {
    defer func() {
        result++
    }()
    return 0
}

可改写为：
func f() (result int) {
    result = 0
    //在return之前，执行defer函数
    func() {
        result++
    }()
    return
}

返回值：1
```

```
func f() (r int) {
    t := 5
    defer func() {
        t = t + 5
    }()
    return t
}

可改写为：
func f() (r int) {
    t := 5
    //赋值
    r = t
    //在return之前，执行defer函数，defer函数没有对返回值r进行修改，只是修改了变量t
    func() {
        t = t + 5
    }()
    return
}

返回值：5

```

```
func f() (r int) {
    defer func(r int) {
        r = r + 5
    }(r)
    return 1
}

func f() (r int) {
    //给返回值赋值
    r = 1
    /**
     * 这里修改的r是函数形参的值，是外部传进来的
     * func(r int){}里边r的作用域只该func内，修改该值不会改变func外的r值
     */
    func(r int) {
        r = r + 5
    }(r)
    return
}

返回值：1
```

```
func main() {
    i := 0
    defer fmt.Println("a:", i)
    //传参数，不会改变i的值（因为传的是值，不是引用），如上边的例3
    defer func(i int) {
        fmt.Println("b:", i)
    }(i)
    //闭包调用，捕获同作用域下的i进行计算
    defer func() {
        fmt.Println("c:", i)
    }()
    i++
}

返回值：（实时传参）
c: 1
b: 0
a: 0
```

## defer配合recover

+ recover(异常捕获)可以让程序在引发panic的时候不会崩溃退出。
+ 在引发panic的时候，panic会停掉当前正在执行的程序，但是，在这之前，它会有序的执行完当前goroutine的defer列表中的语句。
+ 所以我们通常在defer里面挂一个recover，防止程序直接挂掉，类似于try...catch，但绝对不能像try...catch这样使用，因为panic的作用不是为了抓异常。recover函数只在defer的上下文中才有效，如果直接调用recover，会返回nil。
+ Panic是一个内置的函数: 停止当前控制流, 然后开始panicking. 当F函数调用panic,F函数将停止执行后续的普通语句, 但是之前的defered函数调用仍然被正常执行, 然后再返回到F的调用者. 对于F函数的调用者, F 的行为和直接调用panic函数类似. 以上的处理流程会一直沿着调用栈回朔, 直到 当前的goroutine返回引起程序崩溃! Panics可以通过直接调用panic方式触发, 也可以由某些运行时 错误触发, 例如: 数组的越界访问.
+ Recover也是一个内置函数: 用于从panicking恢复.Recover和defer配合使用会非常有用. 对于一个普通的执行流程, 调用recover将返回nil, 也没有任何效果。但如果当前goroutine处于panicking状态,recover调用会捕获触发panic时的参数, 并且恢复到正常的执行流程.

总结：recover可以捕获panic，一般在同一个函数里在defer里进行recover捕获，如果出现panic，则直接停止往下执行，调用本函数里的defer，如果本函数里的defer没有recover捕获，则会一直向上直到调用处，在这个过程中由于没有捕获，则每个调用函数后面的语句都不会执行，如果调用处最终捕获，则可以收到错误，但是调用处往下依旧不能执行，所以如果想要所有函数的正常执行，出现panic错误不退出的话，则在每个函数或者协程的defer 里加一个recover捕获异常。

```
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
```
输出

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

+ Recovered in f 4 的4是产生panic时的调用参数，即g(4)。
+ 注意没有 Returned normally from g. 这一句，panic会波及调用者，使得调用者也停止执行后续语句。
+ panic传递g(4) --> g(3) --> g(2) --> g(1) --> g(0) --> f()，所以f函数最后一行的打印不执行。f中的recover使得panic不再向上传递，所以f的调用者main函数正常退出。
+ 如果我们从函数f中移除deferred语句,panic在扩散到goroutine栈顶前将不会被捕获, 最终会引起 程序崩溃. 下面是修改后的输出结果

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
panic: 4

panic PC=0x2a9cd8
[stack trace omitted]
```



