---
title: "[FreeRTOS Learning] Task"
short: ""
date: 07.Oct.2016
tags:
    - c
    - freeRTOS
    - embedded
    - notes
---

>任务协程（Co-routines）

任务和协程使用不同的API，因此他们不能使用同一个队列或信号量传递数据。
协程仅用在资源非常少的微处理器中，现在一般很少使用。

>任务

* 概述
调度器主要的职责是，在任务切入切出时保存上下文环境（寄存器值、堆栈内容）。为了实现这点，每个任务都需要有自己的堆栈。当任务切出时，它的执行环境会被保存在该任务的堆栈中，这样当再次运行时，就能从堆栈中正确的恢复上次的运行环境。
* 特性
  + 简单
  + 没有使用限制
  + 支持完全抢占
  + 支持优先级
  + 每个任务都有自己的堆栈，消耗RAM较多
  + 如果使用抢占，必须小心的考虑可重入问题
* 状态
  + 运行 正在执行，此时占用CPU
  + 就绪 具备执行的能力等待被调度
  + 阻塞 等待某个时序或外部中断中断
  + 挂起 挂起状态的任务同样对调度器无效，必须调用api后才能进入就绪队列
![关系图](http://upload-images.jianshu.io/upload_images/3168440-2af09105667a9761.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 优先级
每个任务都需要被指定一个0->configMAX_PRIORITIES数值的优先级（在Cortex-M3中数值越大，优先级越低）

  >如果某[架构](http://lib.csdn.net/base/architecture)硬件支持CLZ（或类似）指令（计算前导零的数目，Cortex-M3是支持该指令的，从ARMv6T2才支持这个指令），并且打算在移植层使用这个特性来优化任务调度机制，需要有一些步骤，首先将FreeRTOSConfig.h中configUSE_PORT_OPTIMISED_TASK_SELECTION设置为1，并且最大优先级数目configMAX_PRIORITIES不能大于32。除此之外，configMAX_PRIORITIES可以设置为任意值，但是考虑到configMAX_PRIORITIES设置越大，RAM消耗也越大，一般设置为满足使用的最小值

  任何数量的任务可以共享同一个优先级。如果宏configUSE_TIME_SLICING未定义或着宏configUSE_TIME_SLICING定义为1，处于就绪态的多个相同优先级任务将会以时间片切换的方式共享处理器。
  任务函数如下
```
void vATaskFunction( voidvoid *pvParameters )  
{  
    for( ;; )  
    {  
        /*-- 应用程序代码放在这里. --*/  
    }  
   
    /* 任务不可以从这个函数返回或退出。在较新的FreeRTOS移植包中，如果 
    试图从一个任务中返回，将会调用configASSERT()（如果定义的话）。 
    如果一个任务确实要退出函数，那么这个任务应调用vTaskDelete(NULL) 
    函数，以便处理一些清理工作。*/  
    vTaskDelete( NULL );  
}  
```
***任务函数决不应该返回，因此通常任务函数都是一个死循环。***
* IDLE TASK 和 IDLE TASK HOOK
  * RTOS为了确保系统至少有一个任务在运行会自动创建一个具有最低优先级的空闲任务（idle task），除此之外空闲任务没有其他任何作用，因此合理的剥夺空闲任务的处理器时间是可取的。
 * 空闲任务钩子是一个函数，每一个空闲任务周期被调用一次。如果你想将任务程序功能运行在空闲优先级上，可以有两种选择：
   1. 在一个空闲任务钩子中实现这个功能：因为FreeRTOS必须至少有一个任务处于就绪或运行状态，因此钩子函数不可以调用可能引起空闲任务阻塞的API函数（比如vTaskDelay()或者带有超时事件的队列或信号量函数）。
   2. 创建一个具有空闲优先级的任务去实现这个功能：这是个更灵活的解决方案，但是会带来更多RAM开销。
      
      创建一个空闲钩子步骤如下：
      * 在FreeRTOSConfig.h头文件中设置configUSE_IDLE_HOOK为1；
     *  定义一个函数，名字和参数原型如下所示：

          ```
void vApplicationIdleHook( void ); 
          ```

         通常，**使用这个空闲钩子函数设置CPU进入低功耗模式**。
 

---

                                      2016.2.27

---