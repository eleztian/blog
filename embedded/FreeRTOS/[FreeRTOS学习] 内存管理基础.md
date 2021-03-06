---
title: "[FreeRTOS Learning] The Base Memory Management"
short: ""
date: 10.Oct.2016
tags:
    - c
    - freeRTOS
    - embedded
    - notes
---

*FreeRTOS的内存管理较为只有，它提供了多套管理法案有简单的有复杂的，它还允许用户同时使用两种管理方案，甚至允许你自己设计内存管理方案。*
1. heap_1.c
        当需要分配RAM时，这个内存分配方案只是简单的将一个大数组细分出一个子集来。
        大数组的容量大小通过FreeRTOSConfig.h文件中的configTOTAL_HEAP_SIZE宏来设置
  * 用于从不会删除任务、队列、信号量、互斥量等的应用程序（实际上大多数使用FreeRTOS的应用程序都符合这个条件）
  * 执行时间是确定的并且不会产生内存碎片
  * 实现和分配过程非常简单，需要的内存是从一个静态数组中分配的，意味着这种内存分配通常只是适用于那些不进行动态内存分配的应用。
2. heap_2.c
       和方案1不同，这个方案使用一个最佳匹配算法，它允许释放之前分配的内存块。
       它不会把相邻的空闲块合成一个更大的块（换句话说，这会造成内存碎片）
  * 可以用于重复的分配和删除具有相同堆栈空间的任务、队列、信号量、互斥量等等，并且不考虑内存碎片的应用程序。
  * 不能用在分配和释放随机字节堆栈空间的应用程序
    * 如果一个应用程序动态的创建和删除任务，并且分配给任务的堆栈空间总是同样大小，那么大多数情况下heap_2.c是可以使用的。但是，如果分配给任务的堆栈不总是相等，那么释放的有效内存可能碎片化，形成很多小的内存块。最后会因为没有足够大的连续堆栈空间而造成内存分配失败。在这种情况下，heap_4.c是一个很好的选择。
    * 如果一个应用程序动态的创建和删除队列，并且在每种情况下队列存储区域（队列存储区域指队列项数目乘以每个队列长度）都是同样的，那么大多数情况下heap_2.c可以使用。但是，如果队列存储区在每种情况下并不总是相等，那么释放的有效内存可能碎片化，形成很多小的内存块。最后会因为没有足够大的连续堆栈空间而造成内存分配失败。在这种情况下，heap_4.c是一个很好的选择。
    * 应用程序直接调用pvPortMalloc() 和 vPortFree()函数，而不仅是通过FreeRTOS API间接调用。
  * 如果你的应用程序中的队列、任务、信号量、互斥量等等处在一个不可预料的顺序，则可能会导致内存碎片问题，虽然这是小概率事件，但必须牢记。
  * 不具有确定性，但是它比标准库中的malloc函数具有高得多的效率。
* heap_3.c
      heap_3.c简单的包装了标准库中的malloc()和free()函数，包装后的malloc()和free()函数具备线程保护。
  * 需要链接器设置一个堆栈，并且编译器库提供malloc()和free()函数。
  * 不具有确定性
  * 可能明显的增大RTOS内核的代码大小
      ***注：使用heap_3时，FreeRTOSConfig.h文件中的configTOTAL_HEAP_SIZE宏定义没有作用。***
* heap_4.c
      使用一个最佳匹配算法。
      一个合并算法,它会将相邻的空闲内存块合并成一个更大的块。
  * 可用于重复分配、删除任务、队列、信号量、互斥量等等的应用程序。
  * 可以用于分配和释放随机字节内存的情况，并不像heap_2.c那样产生严重碎片。
  * 不具有确定性，但是它比标准库中的malloc函数具有高得多的效率。
* heap_5.c
      实现了heap_4.c中的合并算法，并且允许堆栈跨越多个非连续的内存区。

 Heap_5通过调用vPortDefineHeapRegions()函数实现初始化，在该函数执行完成前不允许使用内存分配和释放。创建RTOS对象（任务、队列、信号量等等）会隐含的调用pvPortMalloc()，因此必须注意：使用heap_5创建任何对象前，要先执行vPortDefineHeapRegions()函数。
 vPortDefineHeapRegions()函数只需要单个参数。该参数是一个HeapRegion_t结构体类型数组。HeapRegion_t在portable.h中定义，如下所示：

  ```
typedef struct HeapRegion    
{    
    /* 用于内存堆的内存块起始地址*/    
    uint8_t *pucStartAddress;    
    
    /* 内存块大小 */    
    size_t xSizeInBytes;    
} HeapRegion_t;  
  ```
 这个数组必须使用一个NULL指针和0字节元素作为结束，起始地址必须从小到大排列。下面的代码段提供一个例子。MSVCWin32模拟器演示例程使用了heap_5，因此可以当做一个参考例程。
  
    ```
    /* 在内存中为内存堆分配两个内存块.第一个内存块0x10000字节,起始地址为0x80000000,  
    第二个内存块0xa0000字节,起始地址为0x90000000.起始地址为0x80000000的内存块的  
    起始地址更低,因此放到了数组的第一个位置.*/    
    const HeapRegion_t xHeapRegions[] =    
    {    
        { ( uint8_t * ) 0x80000000UL, 0x10000 },    
        { ( uint8_t * ) 0x90000000UL, 0xa0000 },    
        { NULL, 0 } /* 数组结尾. */    
    };    
        
    /* 向函数vPortDefineHeapRegions()传递数组参数. */    
    vPortDefineHeapRegions( xHeapRegions );   
    ```