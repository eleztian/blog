---
title: "[FreeRTOS Learning] Kernel Controller"
short: ""
date: 12.Oct.2016
tags:
    - c
    - freeRTOS
    - embedded
    - notes
---

> 上下文切换
  
 taskYIELD 在中断服务程序中的等价版本为portYIELD_FROM_ISR，用于强制上下文切换的宏。对于Cortex-M3硬件，这个宏会引起PendSV中断。

>进入临界区

 taskENTER_CRITICAL：用于进入临界区的宏。在临界区中不会发生上下文切换。

对于Cortex-M3硬件，先禁止所有RTOS可屏蔽中断，这可以通过向basepri 寄存器写入configMAX_SYSCALL_INTERRUPT_PRIORITY来实现。basepri寄存器被设置成某个值后，所有优先级号大于等于此值的中断都被禁止，但若被设置为0，则不关闭任何中断，0为默认值。然后临界区嵌套计数器增1。

> 退出临界区

taskEXIT_CRITICAL：用于退出临界区的宏。

对于Cortex-M3硬件，先将临界区嵌套计数器减1，如果临界区计数器为零，则使能所有RTOS可屏蔽中断，这可以通过向basepri 寄存器写入0来实现。

> 禁止可屏蔽中断

 taskDISABLE_INTERRUPTS：禁止所有RTOS可屏蔽中断。在调用宏taskENTER_CRITICAL进入临界区时，也会间接调用该宏禁止所有RTOS可屏蔽中断。

>使能可屏蔽中断宏

taskENABLE_INTERRUPTS：使能所有RTOS可屏蔽中断。在调用宏taskEXIT_CRITICAL退出临界区时，也会间接调用该宏使能所有RTOS可屏蔽中断。

> 启动调度器

  ```
 void vTaskStartScheduler( void );
```
当调用vTaskStartScheduler()后，空闲任务被自动创建。如果configUSE_TIMERS被设置为1，定时器后台任务也会被创建。

如果vTaskStartScheduler()成功执行，则该函数不会返回，直到有任务调用了vTaskEndScheduler()。如果因为RAM不足而无法创建空闲任务，该函数也可能执行失败，并会立刻返回调用处。

> 停止调度器

```
void vTaskEndScheduler( void );
```
 仅用于x86硬件[架构](http://lib.csdn.net/base/architecture)中。停止RTOS内核系统节拍时钟。所有创建的任务自动删除并停止多任务调度。

> 挂起任务调度器

```
 void vTaskSuspendAll( void );
```
 挂起调度器，但不禁止中断。当调度器挂起时，不会进行上下文切换。调度器挂起后，正在执行的任务会一直继续执行，内核不再调度（意味着当前任务不会被切换出去），直到该任务调用了xTaskResumeAll ()函数。

内核调度器挂起期间，那些可以引起上下文切换的API函数（如vTaskDelayUntil()、xQueueSend()等）决不可使用。

> 恢复被挂起的调度器

```
BaseType_t xTaskResumeAll( void );
```
恢复因调用vTaskSuspendAll()函数而挂起的实时内核调度器。xTaskResumeAll()仅恢复调度器，它不会恢复那些被vTaskSuspend()函数挂起的任务。
```
voidvTask1( voidvoid * pvParameters )  
{  
    for( ;; )  
    {  
        /* 任务代码写在这里 */  
  
        /* ... */  
  
        /* 有些时候，某个任务希望可以连续长时间的运行，但这时不能使用taskENTER_CRITICAL ()/taskEXIT_CRITICAL ()的方法，这样会屏蔽掉中断，引起中断丢失，包括系统节拍时钟。可以使用vTaskSuspendAll ()停止RTOS内核调度：*/  
        xTaskSuspendAll ();  
  
        /* 执行操作代码放在这里。这样不用进入临界区就可以连续长时间执行了。在这期间，中断仍然会得到响应，RTOS内核系统节拍时钟也会继续保持运作 */  
  
        /* ... */  
  
        /* 操作结束，重新启动RTOS内核 。我们想强制进行一次上下文切换，但是如果恢复调度器的时候已经执行了上下文切换，再执行一次是没有意义的，因此会进行一次判断。*/  
        if( !xTaskResumeAll () )  
        {  
             taskYIELD ();  
        }  
    }  
}  
```
> 调整系统节拍

```
 void vTaskStepTick( TickType_txTicksToJump );
```
 如果RTOS使能tickless空闲功能，每当只有空闲任务被执行时，系统节拍时钟中断将会停止，微控制器进入低功耗模式。当微控制器退出低功耗后，系统节拍计数器必须被调整，将进入低功耗的时间弥补上。

 如果FreeRTOS移植文件中定义了宏portSUPPRESS_TICKS_AND_SLEEP()实体，则函数vTaskStepTick用于在这个宏portSUPPRESS_TICKS_AND_SLEEP()实体内部调整系统节拍计数器。函数vTaskStepTick是一个全局函数，所以也可以在宏portSUPPRESS_TICKS_AND_SLEEP()实体中重写该函数。

宏configUSE_TICKLESS_IDLE必须设置为1，此函数才有效。
```
/* 首先定义宏portSUPPRESS_TICKS_AND_SLEEP()。宏参数指定要进入低功耗（睡眠）的时间，单位是系统节拍周期。*/  
#define portSUPPRESS_TICKS_AND_SLEEP( xIdleTime ) vApplicationSleep( xIdleTime )  
   
/* 定义被宏portSUPPRESS_TICKS_AND_SLEEP()调用的函数 */  
void vApplicationSleep(TickType_t xExpectedIdleTime )  
{  
    unsigned long ulLowPowerTimeBeforeSleep,ulLowPowerTimeAfterSleep;  
   
    /* 从时钟源获取当前时间，当微控制器进入低功耗的时候，这个时钟源必须在运行 */  
    ulLowPowerTimeBeforeSleep =ulGetExternalTime();  
   
    /*停止系统节拍时钟中断。*/  
    prvStopTickInterruptTimer();  
   
    /* 配置一个中断，当指定的睡眠时间达到后，将处理器从低功耗中唤醒。这个中断源必须在微控制器进入低功耗时也可以工作。*/  
    vSetWakeTimeInterrupt( xExpectedIdleTime );  
   
    /*进入低功耗 */  
    prvSleep();  
   
    /* 确定微控制器进入低功耗模式持续的真正时间。因为其它中断也可能使得微处理器退出低功耗模式。注意：在调用宏portSUPPRESS_TICKS_AND_SLEEP()之前，调度器应该被挂起，portSUPPRESS_TICKS_AND_SLEEP()返回后，再将调度器恢复。因此，这个函数未完成前，不会执行其它任务。*/  
    ulLowPowerTimeAfterSleep =ulGetExternalTime();  
          
    /*调整内核系统节拍计数器。*/  
    vTaskStepTick( ulLowPowerTimeAfterSleep –ulLowPowerTimeBeforeSleep );  
   
    /*重新启动系统节拍时钟中断。*/  
    prvStartTickInterruptTimer();  
}  
```