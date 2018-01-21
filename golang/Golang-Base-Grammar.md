---
title: [Golang Learning]  The Base Grammar
short: "Golang base grammer, Zhe different with other program grammers., And the points of easy error."
date: 16.Dec.2017
tags:
    - c
    - golang
    - server
    - notes
---

- 引用类型: map, slice, chan, interface, func
- 非引用类型: ...
- 在golang里面所有变量在定义时如果没有初始化都都会被初始化为零值。
    - 数值类型为 0 ，
    - 布尔类型为 false ，
    - 字符串为 "" （空字符串）
    - pointers、slices、 maps、 channel、 functions、 interfaces 为 nil
    
    nil并不是Go的关键字之一，你甚至可以自己去改变nil的值：
    ```go
    var nil = errors.New("xxxx")
    ```
    但是这样做有点蠢。详细可以看这里 [youtube](https://www.youtube.com/watch?v=ynoY2xz-F8s)
- slice
    ```go
    var slice = []int{1, 2, 3, 4}
    ns = slice[1:2]
    nns := slice[1:2:2]
    fmt.Println(len(slice), len(ns), len(nns)) // 4 1 1
    fmt.Println(cap(slice), cap(ns), cap(nns)) // 4 3 1
    ```
    - 对于slice[i:j:k] len = j - i,  cap = k - i 
    - slice {pointer, len, cap}, 各个切片在底层共享内存。在64bit里，一个slice的大小为24(3*8)字节。
    - range slice 范围有len限制，slice的索引由cap限定。
- interface
    ```go
    interface{ 
        itable -> {类型， 方法集}
        value-pointer -> {values...}
    }
    ```
    在64bit里， interface的大小为16(2*8)字节

    | Value | Methods recevier | 
    | ------ | ------ | 
    | T | (t T) | 
    | *T| (t T) or (t *T) | 

    如果使用值作为接收者，那么只能通过值调用，反之，如果使用指针作为接收者，那么可以通过指针或值调用。
    
    或者换个角度来看：

    | Methods recevier | Value | 
    | ------ | ------ | 
    | (t T) | T or *T | 
    | (t *T) | *T | 
    
    如果使用指针接收者来实现一个接口，那么只有指向那个类型的指针才能够实现对应的接口。

    why?

    因为编译器并不是总能自动获得一个值的地址。如下：
    ```go
    package main 

    import "fmt" 

    // duration 是一个基于 int 类型的类型 
    type duration int 

    // 使用更可读的方式格式化 duration 值 
    func (d *duration) pretty() string { 
        return fmt.Sprintf("Duration: %d", *d) 
    } 

    // main 是应用程序的入口 
    func main() { 
        duration(42).pretty() 

        // ./listing46.go:17: 不能通过指针调用 duration(42)的方法 
        // ./listing46.go:17: 不能获取 duration(42)的地址 
    } 
    ```
    而我们在通过值调用指针作为接收者的方法时，编译器会自动帮我们进行转换。但是这仅仅是一个语法糖。

- channel




