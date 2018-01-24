---
title: "[NSQ Learning]  CAS And ABA"
short: "Learning NSQ about Compare And Swap."
date: 24.Jan.2018
tags:
    - golang
    - server
    - nsq
    - notes
---
# CAS (Compare And Swap)

```go
func CompareAndSwapInt32(addr *int32, old, new int32) (swapped bool)
```

原子操作：比较 \*addr 和 old 是否相等，相等则使用new替换\*addr的值。

CAS 和使用锁不同的是，锁是假设存在其他地方在改变它，那么把它放到临界区保护起来。而CAS则是假设没有其他地方在改变它，那么一旦确认没有，则赋值。

CAS操作的优势是，可以在不形成临界区和创建互斥量的情况下完成并发安全的值替换操作。这可以大大的减少同步对程序性能的损耗。当然，CAS操作也有劣势。在被操作值被频繁变更的情况下，CAS操作并不那么容易成功。有些时候，我们可能不得不利用for循环以进行多次尝试。示例如下：

```go
var value int32
func addValue(delta int32) {
    for {
        v := value
        if atomic.CompareAndSwapInt32(&value, v, (v + delta)) {
            break
        }
    }
}

```

读取value的值的操作并不是并发安全的。在该读取操作被进行的过程中，其它的对此值的读写操作是可以被同时进行的。

```go
var value int64
func addValue(delta int64) {
    for {
        v := atomic.LoadInt64(&value)
        if atomic.CompareAndSwapInt64(&value, v, (v + delta)) {
            break
        }
    }
}
```

当想并发安全的更新一些类型（int32、int64、uint32、uint64、uintptr和unsafe.Pointer类型）的值的时候，我们总是应该优先选择CAS操作。

善用原子操作。因为它比锁更加简练和高效。不过，由于原子操作自身的限制，锁依然常用且重要。

# ABA

如果另一个线程修改V值假设原来是A，先修改成B，再修改回成A。当前线程的CAS操作无法分辨当前V值是否发生过变化。

ABA问题是无锁结构实现中常见的一种问题，可基本表述为：

- 进程P1读取了一个数值A
- P1被挂起(时间片耗尽、中断等)，进程P2开始执行
- P2修改数值A为数值B，然后又修改回A
- P1被唤醒，比较后发现数值A没有变化，程序继续执行
