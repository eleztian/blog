---
title: "[Golang Learning]  Panic Recover And Defer"
short: ""
date: 17.Jan.2018
tags:
    - c
    - golang
    - server
    - notes
---
# panic recover and defer

panic-defer(recover): panic 抛出异常， recover捕获异常， recover在defer中使用， 但是recover之后程序逻辑并不会在panic处继续。panic 程序停止运行进入defer中。

如果满足以下任一条件，recover的返回值为nil：
1. panic参数是nil
2. 当前goroutine没有产生panic
3. recover不是由延迟函数直接调用。


## panic只能向上传播，沿着函数调用堆栈反向。panic从不通过深入到函数调用中传播。

```go
func main() { // calling level 0
	defer func() { //d1
		fmt.Println("now level 0")
		if x := recover(); x != nil {
			fmt.Println("recover panic")
		}
	}()
	defer func() { //d2
		defer func() { d21
			fmt.Println("now level 1")
		}()
		defer func() { d22
			if x := recover(); x != nil {
				fmt.Println("recover panic")
			}
		}()
	}()

	func(){ //p
		defer func() {
			fmt.Println("now level 1")
		}()
		panic(1) 
	}()

}
//output:
now level 1
now level 1
now level 0
recover panic
```
```
// 调用栈
p*                              d22          
d2                              d21(p*)      d21
d1			-->     d1        -->d1*
goroutine			goroutine    goroutine*
```

## panic级别意味着panic传播到函数调用的哪一层级、因为panic只能向上传递、所以panic等级永远不加增加、只会减小、在goroutine中，当前panic的水平永远不会大于goroutine的执行水平。

```go
func main(){
    defer fmt.Println("program will not crash")
    defer func(){
        fmt.Println(recover())// 3
    }()
    defer fmt.Println("now, panic 3 suppresses panic 2")
    defer panic(3)
    defer fmt.Println("now, panic 2 suppresses panic 1")
    defer panic(2)
    panic(1)

}
// Outputs:
// now, panic 2 suppresses panic 1
// now, panic 3 suppresses panic 2
// 3
// program will not crash
```

- 在同一层级上、新的panics会压制原有panics
- 在goroutine中最多只有一个活动的panic

## 多个主动panic在一个Goroutine中的共存
```go
func main(){// callnig level 0
    defer fmt.Println("program will crash, for panic 3 is stll active")
    defer func(){// calling level 1
        defer func(){// calling level 2
            fmt.Println(recover())// 6
        }()// the level of panic 3 is 0.// the level of panic 6 is 1.
        defer fmt.Println("now, there are two active panics: 3 and 6")
        defer panic(6)// will suppress panic 5
        defer panic(5)// will suppress panic 4
        panic(4)// will not suppress panic 3, for they have differrent levels 
        // the  level of panic 3 is 0.// the level of panic 4 is 1.
    }()
    defer fmt.Println("now, only panic 3 is active")
    defer panic(3)// will suppress panic 2
    defer panic(2)// will suppress panic 1
    panic(1)
}
// Output:
// now, only panic 3 is active
// now, there are two active
// panics: 3 and 6
// 6
// program will crash, for panic 3 is stll active
// panic: 1
// panic: 2
// panic: 3
```

## 低级的panics将会被首先捕获
```go
func main(){
    defer func(){
        defer func(){
            fmt.Println("panic",recover(),"is recovered")// panic 2 is recovered
        }()
        defer fmt.Println("panic",recover(),"is recovered")// panic 1 is recovered
        defer fmt.Println("now, two active panics coexist")
        panic(2)
    }()
    panic(1)
}
// Output:
// now, two active panics coexist
// panic 1 is recovered
// panic 2 is recovered
```


原文链接：http://www.tapirgames.com/blog/golang-panic-recover-mechanism