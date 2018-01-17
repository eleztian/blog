# golang memory map

```go
package main

import (
	"fmt"
	"reflect"
)

type MyData struct {
	aByte   byte // 1
	aShort  int16  // 2
	anInt32 int32  // 4
	aSlice  []byte // 24
}

func main() {
	data := MyData{
		aByte:   0x1,
		aShort:  0x0203,
		anInt32: 0x04050607,
		aSlice:  []byte{
			0x08, 0x09, 0x0a,
		},
	}
	typ := reflect.TypeOf(data)
	fmt.Println(typ.Size())
	n := typ.NumField()
	for i := 0; i < n; i++ {
		field := typ.Field(i)
		fmt.Printf("%s at offset %v, size=%d, align=%d\n",
			field.Type, field.Offset, field.Type.Size(),
			field.Type.Align())
	}
	dataBytes := (*[32]byte)(unsafe.Pointer(&data))
	fmt.Printf("Bytes are %#v\n", dataBytes)

}
// Output:
// uint8 at offset 0, size=1, align=1
// int16 at offset 2, size=2, align=2
// int32 at offset 4, size=4, align=4
// []uint8 at offset 8, size=24, align=8
// Bytes are &[32]uint8{0x1, 0x0, 0x3, 0x2, 0x7, 0x6, 0x5, 0x4, 0x88, 0x40, 0x4, 0x42, 0xc0, 0x0, 0x0, 0x0, 0x3, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x3, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0}
```

- 存储方式：小端
- 字节码解析：
    - 0x1, 0x0, : byte 0x1  后面是一位对其
    - 0x3, 0x2, ：uint16 = 0x0203
    - 0x7, 0x6, 0x5, 0x4, : uint16=0x04050607
    - slice
        - 0x88, 0x40, 0x4, 0x42, 0xc0, 0x0, 0x0, 0x0, : uintptr
        - 0x3, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, : int = 3 
        - 0x3, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0 : int = 3

slice的底层结构为：

```go
type SliceHeader struct {
        Data uintptr // 8
        Len  int // 8
        Cap  int // 8
}
```
当将

```go
type MyData struct {
	aByte   byte // 1
	aShort  int16  // 2
	anInt32 int32  // 4
	aSlice  []byte // 24
}
```
改为

```go
type MyData struct {
    aByte   byte // 1
    anInt32 int32  // 4
	aShort  int16  // 2
	aSlice  []byte // 24
}
// output:
// uint8 at offset 0, size=1, align=1
// int32 at offset 4, size=4, align=4
// int16 at offset 8, size=2, align=2
// []uint8 at offset 16, size=24, align=8
// Bytes are &[32]uint8{0x1, 0x0, 0x0, 0x0, 0x7, 0x6, 0x5, 0x4, 0x3, 0x2, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x88, 0x40, 0x4, 0x42, 0xc0, 0x0, 0x0, 0x0, 0x3, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0}
// Slice data is &[3]uint8{0x8, 0x9, 0xa}
```

- struct 每一元素都有一个对齐方式。按对应的类型进行对齐。

## 解析string

```go
type StringHeader struct {
    Data uintptr
    Len  int
}
func main(){
str := "hello"
    str2 := str[2:]
    fmt.Println(str2)
    strb := (*[16]byte)(unsafe.Pointer(&str))
    fmt.Println(strb)
    strb2 := (*[16]byte)(unsafe.Pointer(&str2))
    fmt.Println(strb2)
}
Output:
// &[137 139 76 0 0 0 0 0 5 0 0 0 0 0 0 0]
// &[139 139 76 0 0 0 0 0 3 0 0 0 0 0 0 0]
```
- 切片[]不是生成新的字符串而是在原有的基础上进行偏移。

## 解析map

```go
#define BUCKETSIZE
// 每个bucket中存放最多8个key/value对, 如果多于8个，那么会申请一个新的bucket，并将它与之前的bucket链起来。
struct Hmap
{
    uint8   B;    // 可以容纳2^B个项
    uint16  bucketsize;   // 每个桶的大小

    byte    *buckets;     // 2^B个Buckets的数组
    byte    *oldbuckets;  // 前一个buckets，只有当正在扩容时才不为空
};
struct Bucket
{
    uint8  tophash[BUCKETSIZE]; // hash值的高8位....低位从bucket的array定位到bucket
    Bucket *overflow;           // 溢出桶链表，如果有
    byte   data[1];             // BUCKETSIZE keys followed by BUCKETSIZE values
};

#define LOAD 6.5 // 扩容因子平衡点
```

- Hash表以空间换时间，扩容后大小为原来的两倍2^(B+1).Hash 表扩容后需要重新计算Hash值，然后将原来的位置移动到新的位置（删除原来的，添加新的，这里的删除并不会真正的删除，而是添加一个已删除的标记）

## nil

- 读或者写一个nil的channel的操作会永远阻塞。
- 读一个关闭的channel会立刻返回一个channel元素类型的零值。
- 写一个关闭的channel会导致panic。

## defer return

一般retrun: 返回值赋值->返回被调处

过程：返回值赋值->defer->返回被调处

## interface

>依赖于接口而不是实现，优先使用组合而不是继承，这是程序抽象的基本原则。

### 定义

Interface 其实就是一个如下结构体：

```go
struct Eface // 空接口
{
    Type*    type; // 类型信息，用于实现反射
    void*    data; // 具体数据
};
struct Iface // 带方法的接口
{
    Itab*    tab;
    void*    data;
};
struct Type
{
    uintptr size;
    uint32 hash;
    uint8 _unused;
    uint8 align;
    uint8 fieldAlign;
    uint8 kind;
    Alg *alg; // 函数指针数组
    void *gc; // 用于垃圾回收
    String *string;
    UncommonType *x;
    Type *ptrto;
};
struct    Itab
{
    InterfaceType*    inter;
    Type*    type;
    Itab*    link;
    int32    bad;
    int32    unused;
    void    (*fun[])(void); // 具体类型中实现的方法
};

```

### 具体类型想接口赋值

在**编译过程中**会检测具体类型是否实现了这个接口的所有函数

```go
struct Method // 方法的具体实现信息
{
    String *name;
    String *pkgPath;
    Type    *mtyp;
    Type *typ;
    void (*ifn)(void);
    void (*tfn)(void);
};
struct IMethod // 接口中方法的定义
{
    String *name;
    String *pkgPath;
    Type *type;
};
```

检测是查找IMethod中方法是否在mthod中实现，如果实现了则复制到Itab的func那张表中。

## 方法调用

>对象的方法调用相当于普通函数调用的一个语法糖衣

当一个类型被匿名嵌入结构体时，它的方法表会被拷贝到嵌入结构体的Type的方法表中。这个过程也是在编译时就可以完成的。

对组合对象的方法调用同样也仅仅是普通函数调用的语法糖衣。

## 内存模型

>在一个groutine中对变量进行读操作能够侦测到在其他goroutine中对该变量的写操作"的条件.

// TODO:

在golang里面没有真正意义的引用类型，golang里面的参数传递都是值传递，我们所说的应用类型仅仅是因为它的底层实现里面是一个指针。在传递的时候我们拷贝了它底层的地址到一个新的结构中。