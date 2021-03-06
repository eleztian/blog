---
title: "[FreeRTOS Learning] Kernel Configuration"
short: ""
date: 08.Oct.2016
tags:
    - c
    - freeRTOS
    - embedded
    - notes
---

FreeRTOS  的内核是高度可定制的，可以通过FreeRTOSConfig.h 配置，如果没有在配置文件中指定某个选项，那么RTOS内核会使用默认值。
如下配置文件：
```
#ifndef FREERTOS_CONFIG_H  
#define FREERTOS_CONFIG_H  
   
/*Here is a good place to include header files that are required across 
yourapplication. */  
#include "something.h"  
   
#define configUSE_PREEMPTION                    1  
#define configUSE_PORT_OPTIMISED_TASK_SELECTION 0  
#define configUSE_TICKLESS_IDLE                 0  
#define configCPU_CLOCK_HZ                      60000000  
#define configTICK_RATE_HZ                      250  
#define configMAX_PRIORITIES                    5  
#define configMINIMAL_STACK_SIZE                128  
#define configTOTAL_HEAP_SIZE                   10240  
#define configMAX_TASK_NAME_LEN                 16  
#define configUSE_16_BIT_TICKS                  0  
#define configIDLE_SHOULD_YIELD                 1  
#define configUSE_TASK_NOTIFICATIONS            1  
#define configUSE_MUTEXES                       0  
#define configUSE_RECURSIVE_MUTEXES             0  
#define configUSE_COUNTING_SEMAPHORES           0  
#define configUSE_ALTERNATIVE_API               0/* Deprecated! */  
#define configQUEUE_REGISTRY_SIZE               10  
#define configUSE_QUEUE_SETS                    0  
#define configUSE_TIME_SLICING                  0  
#define configUSE_NEWLIB_REENTRANT              0  
#define configENABLE_BACKWARD_COMPATIBILITY     0  
#define configNUM_THREAD_LOCAL_STORAGE_POINTERS 5  
   
/*Hook function related definitions. */  
#define configUSE_IDLE_HOOK                     0  
#define configUSE_TICK_HOOK                     0  
#define configCHECK_FOR_STACK_OVERFLOW          0  
#define configUSE_MALLOC_FAILED_HOOK            0  
   
/*Run time and task stats gathering related definitions. */  
#define configGENERATE_RUN_TIME_STATS           0  
#define configUSE_TRACE_FACILITY                0  
#define configUSE_STATS_FORMATTING_FUNCTIONS    0  
   
/*Co-routine related definitions. */  
#define configUSE_CO_ROUTINES                   0  
#define configMAX_CO_ROUTINE_PRIORITIES         1  
   
/*Software timer related definitions. */  
#define configUSE_TIMERS                        1  
#define configTIMER_TASK_PRIORITY               3  
#define configTIMER_QUEUE_LENGTH                10  
#define configTIMER_TASK_STACK_DEPTH            configMINIMAL_STACK_SIZE  
   
/*Interrupt nesting behaviour configuration. */  
#define configKERNEL_INTERRUPT_PRIORITY        [dependent of processor]  
#define configMAX_SYSCALL_INTERRUPT_PRIORITY   [dependent on processor and application]  
#define configMAX_API_CALL_INTERRUPT_PRIORITY  [dependent on processor and application]  
   
/*Define to trap errors during development. */  
#define configASSERT( ( x ) )     if( ( x ) == 0) vAssertCalled( __FILE__, __LINE__ )  
   
/*FreeRTOS MPU specific definitions. */  
#define configINCLUDE_APPLICATION_DEFINED_PRIVILEGED_FUNCTIONS 0  
   
/*Optional functions - most linkers will remove unused functions anyway. */  
#define INCLUDE_vTaskPrioritySet                1  
#define INCLUDE_uxTaskPriorityGet               1  
#define INCLUDE_vTaskDelete                     1  
#define INCLUDE_vTaskSuspend                    1  
#define INCLUDE_xResumeFromISR                  1  
#define INCLUDE_vTaskDelayUntil                 1  
#define INCLUDE_vTaskDelay                      1  
#define INCLUDE_xTaskGetSchedulerState          1  
#define INCLUDE_xTaskGetCurrentTaskHandle       1  
#define INCLUDE_uxTaskGetStackHighWaterMark     0  
#define INCLUDE_xTaskGetIdleTaskHandle          0  
#define INCLUDE_xTimerGetTimerDaemonTaskHandle  0  
#define INCLUDE_pcTaskGetTaskName               0  
#define INCLUDE_eTaskGetState                   0  
#define INCLUDE_xEventGroupSetBitFromISR        1  
#define INCLUDE_xTimerPendFunctionCall          0  
   
/* Aheader file that defines trace macro can be included here. */  
   
#end if/* FREERTOS_CONFIG_H*/
```
1.  configUSE_PREEMPTION(抢占)  
  * 1 ： 抢占式调度机，0 ： 表示协作式调度机（时间片）
  * 抢占式表示操作系统完全决定进程调度方案，操作系统可以剥夺耗时长的进程的时间片，提供给其它进程；
  * 协作式表示下一个进程被调度的前提是当前进程主动放弃时间片；
2. configUSE_PORT_OPTIMISED_TASK_SELECTION

  * 1 ： 特定于硬件的方法， 0 ： 通用方法
+ configUSE_IDLE_HOOK
  * RTOS 开始工作后，为保证至少又一个任务在运行，空闲任务会被制动创建，对于已经被删除的任务，空闲任务会释放分配给它的堆栈空间，因此，在应用中应该注意，使用vTaskDelete()函数时要确保空闲任务获得一定的处理器时间。除此之外，空闲任务没有其它特殊功能，因此可以任意的剥夺空闲任务的处理器时间
  * 0 忽略空闲钩子。
  * 1 使用空闲钩子。
+ configUSE_TICK_HOOK
  * 0 保持系统节拍（tick）中断一直运行
  * 1 使能低功耗tickless模式
+ configCPU_CLOCK_HZ
  * cpu内核时钟频率	
+ configTICK_RATE_HZ	
  * TickType_t （uint32_t）
  * RTOS x系统时钟中断频率， 每次中断RTOS都会进行任务调度。
  * 统节拍中断用来测量时间，因此，越高的测量频率意味着可测到越高的分辨率时间。但是，高的系统节拍中断频率也意味着RTOS内核占用更多的CPU时间，因此会降低效率。RTOS演示例程都是使用系统节拍中断频率为1000HZ，这是为了测试RTOS内核，比实际使用的要高。（实际使用时不用这么高的系统节拍中断频率）多个任务可以共享一个优先级，RTOS调度器为相同优先级的任务分享CPU时间，在每一个RTOS 系统节拍中断到来时进行任务切换。高的系统节拍中断频率会降低分配给每一个任务的“时间片”持续时间。
+ configMAX_PRIORITIES		( 5 )
  * 应用程序有效的优先级数目
  * 在RTOS内核中，每个有效优先级都会消耗一定量的RAM，因此这个值不要超过你的应用实际需要的优先级数目.
+ configMINIMAL_STACK_SIZE	( ( unsigned short ) 128 )
  * 空闲任务使用的最小 堆栈空间
  * 值应该大于或等于128，堆栈大小不是以字节为单位而是以字为单位的，比如在32位架构下，栈大小为100表示栈内存占用400字节的空间
+ configTOTAL_HEAP_SIZE		( ( size_t ) ( 17 * 1024 ) )
  *   RTOS内核总计可用的有效的RAM大小
+ configMAX_TASK_NAME_LEN		( 16 )
  * 任务描述信息的字符串的最大长度
+ configUSE_TRACE_FACILITY	0
  * 设置成1表示启动可视化跟踪调试，会激活一些附加的结构体成员和函数。
+ configUSE_16_BIT_TICKS		0
  * 定义系统节拍计数器的数据类型（portTickType），为1表示使用 uint16_t 的数据类型，否则使用 uint32_t 的数据类型。
+ configIDLE_SHOULD_YIELD		1
  * 0 将阻止空闲任务为用户任务让出CPU
  * 1 当有用户任务时及时让出cpu

+ configUSE_CO_ROUTINES 		0
  * 0 不使用协程
  * 1 使用协程
  * 协程：（Co-routines）主要用于资源发非常受限的嵌入式系统（RAM非常少），通常不会用于32位微处理器。在当前嵌入式硬件环境下，不建议使用协程，FreeRTOS的开发者早已经停止开发协程。
+ configMAX_CO_ROUTINE_PRIORITIES ( 2 )
  * 应用程序协程（Co-routines）的有效优先级数目，任何数目的协程都可以共享一个优先级。使用协程可以单独的分配给任务优先级。见configMAX_PRIORITIES。


   以“INCLUDE”起始的宏允许用户不编译那些应用程序不需要的实时内核组件（函数），这可以确保在你的嵌入式系统中RTOS占用最少的ROM和RAM。

1. INCLUDE_vTaskPrioritySet		1
+ INCLUDE_uxTaskPriorityGet		1
+ INCLUDE_vTaskDelete				1
+ INCLUDE_vTaskCleanUpResources	0
+ INCLUDE_vTaskSuspend			1
+ INCLUDE_vTaskDelayUntil			1
+ INCLUDE_vTaskDelay				1

***Cortex-M中断优先级数值越大代表的优先级反而越小***
下面是关于这两个宏的理解：
+ configKERNEL_INTERRUPT_PRIORITY 		255
+ configLIBRARY_KERNEL_INTERRUPT_PRIORITY	15


----

**感谢[作者](http://my.csdn.net/zhzht19861011)的资料整理**