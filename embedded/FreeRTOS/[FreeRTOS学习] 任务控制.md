---
title: "[FreeRTOS Learning] Task Control"
short: ""
date: 06.Oct.2016
tags:
    - c
    - freeRTOS
    - embedded
    - notes
---

>创建任务

1. 任务函数
```
void ATaskFunction( void *pvParameters );
```
***FreeRTOS 任务不允许以任何方式从实现函数中返回——它们绝不能有一条”return”语句，也不能执行到函数末尾。如果一个任务不再需要，可以显式地将其删除***

 ```
void ATaskFunction( void *pvParameters )
{
/* 可以像普通函数一样定义变量。用这个函数创建的每个任务实例都有一个属于自己的iVarialbleExample变
量。但如果iVariableExample被定义为static，这一点则不成立 – 这种情况下只存在一个变量，所有的任务实
例将会共享这个变量。 */
int iVariableExample = 0;
/* 任务通常实现在一个死循环中。 */
for( ;; )
{
/* 完成任务功能的代码将放在这里。 */
}
/* 如果任务的具体实现会跳出上面的死循环，则此任务必须在函数运行完之前删除。传入NULL参数表示删除
的是当前任务 */
vTaskDelete( NULL );
}
```
一个任务函数可以创建若干个函数，每一个函数独立运行，拥有属于自己的堆栈空间。

2. 创建任务函数，删除任务函数
```
voidvTaskDelete( TaskHandle_t xTask );
 xTaskCreate();
portBASE_TYPE xTaskCreate( pdTASK_CODE pvTaskCode,
                              const signed portCHAR * const pcName,
                              unsigned portSHORT usStackDepth,
                              void *pvParameters,
                              unsigned portBASE_TYPE uxPriority,
                              xTaskHandle *pxCreatedTask 
                             );
```
具体参数含义 见API文档
Demo

```
void vTask1( void *pvParameters )
{
      const char *pcTaskName = "Task 1 is running\r\n";
      volatile unsigned long ul;
      /* 和大多数任务一样，该任务处于一个死循环中。 */
      for( ;; )
      {
            /* Print out the name of this task. */
            vPrintString( pcTaskName );
            /* 延迟，以产生一个周期 */
            for( ul = 0; ul < mainDELAY_LOOP_COUNT; ul++ )
            {
                  /* 这个空循环是最原始的延迟实现方式。在循环中不做任何事情。后面的示例程序将采用
                delay/sleep函数代替这个原始空循环。 */
            }
      }
  }
程序清单4 例1中的第一个任务实现代码
void vTask2( void *pvParameters )
{
      const char *pcTaskName = "Task 2 is running\r\n";
      volatile unsigned long ul;
      /* 和大多数任务一样，该任务处于一个死循环中。 */
      for( ;; )
      {
            /* Print out the name of this task. */
            vPrintString( pcTaskName );
            /* 延迟，以产生一个周期 */
            for( ul = 0; ul < mainDELAY_LOOP_COUNT; ul++ )
            {
                  /* 这个空循环是最原始的延迟实现方式。在循环中不做任何事情。后面的示例程序将采用
                  delay/sleep函数代替这个原始空循环。 */
            }
      }
}
int main( void )
{
      /* 创建第一个任务。 需要说明的是一个实用的应用程序中应当检测函      数xTaskCreate()的返回值，以确保任务创建成功。 */
      xTaskCreate( vTask1, /* 指向任务函数的指针 */
                    "Task 1", /* 任务的文本名字，只会在调试中用到 */
                    1000, /* 栈深度 – 大多数小型微控制器会使用的值会比此值小得多 */
                    NULL, /* 没有任务参数 */
                    1, /* 此任务运行在优先级1上. */
                    NULL ); /* 不会用到任务句柄 */
      /* Create the other task in exactly the same way and at the same priority. */
      xTaskCreate( vTask2, "Task 2", 1000, NULL, 1, NULL );
      /* 启动调度器，任务开始执行 */
      vTaskStartScheduler();
      /* 如果一切正常， main()函数不应该会执行到这里。但如果执行到这里，很可能是内存堆空间不足导致空闲
任务无法创建。第五章有讲述更多关于内存管理方面的信息 */
      for( ;; );
}
```

3. 优先级
  * vTaskPrioritySet()  修改优先级
  * 任意数量的任务可以共享同一个优先级——以保证最大设计弹性
  * 有效的优先级号范围从 0 到(configMAX_PRIORITES – 1)
  * 被选中的优先级上具有不止一个任务，调度器会让这些任务轮流执行，每个任务都执行一个”时间片”
  * 时间片的长度通过心跳中断的频率进行设定，configTICK_RATE_HZ 进行配置（如，设为100(HZ)，则时间片长度为 10ms）
  * 常量 portTICK_RATE_MS 用于将以心跳为单位的时间值转化
为以毫秒为单位的时间值
* 相关API
  + 相对延时
```
    void vTaskDelay( portTickTypexTicksToDelay )
```
    常量portTICK_RATE_MS 用来辅助计算真实时间，此值是系统节拍时钟中断的周期，单位是毫秒。在文件FreeRTOSConfig.h中，宏INCLUDE_vTaskDelay 必须设置成1，此函数才能有效。

    比如vTaskDelay(100)，那么从调用vTaskDelay()后，任务进入阻塞状态，经过100个系统时钟节拍周期，任务解除阻塞
***调用vTaskDelay()到任务解除阻塞的时间不总是固定的并且该任务下一次调用vTaskDelay()函数的时间也不总是固定的（两次执行同一任务的时间间隔本身就不固定，中断或高优先级任务抢占也可能会改变每一次执行时间）。***
  + 绝对延时
    ```
    void vTaskDelayUntil( TickType_t *pxPreviousWakeTime,
                              const TickType_txTimeIncrement );
    ```
    * INCLUDE_vTaskDelayUntil 必须设置成1，此函数才有效
    * 这个函数不同于vTaskDelay()函数的一个重要之处在于：vTaskDelay()指定的延时时间是从调用vTaskDelay()之后（执行完该函数）开始算起的，但是vTaskDelayUntil()指定的延时时间是一个绝对时间。

  + 获取任务优先级
       ```
      UBaseType_t uxTaskPriorityGet(TaskHandle_t xTask );
      ```
      * INCLUDE_vTaskPriorityGet必须设置成1
      * xTask：任务句柄。NULL表示获取当前任务的优先级

  + 设置任务优先级
      ```
 void vTaskPrioritySet( TaskHandle_txTask,
                           UBaseType_tuxNewPriority );
```
    * INCLUDE_vTaskPrioritySet 必须设置成1

  + 任务挂起
    ```
  void vTaskSuspend( TaskHandle_txTaskToSuspend );
    ```
    * 被挂起的任务绝不会得到处理器时间
    * INCLUDE_vTaskSuspend必须设置成1
  + 恢复挂起任务
    ```
 void vTaskResume( TaskHandle_txTaskToResume );
```
    * INCLUDE_vTaskSuspend必须置1
    * 通过调用一次或多次vTaskSuspend()挂起的任务，可以调用一次vTaskResume ()函数来再次恢复运行
  + 恢复挂起的任务（在中断服务函数中使用）
      ```
    BaseType_t xTaskResumeFromISR(TaskHandle_t xTaskToResume );
    ```
    * 用于恢复一个挂起的任务，用在ISR中
    *  INCLUDE_xTaskResumeFromISR 必须设置成1
    * xTaskResumeFromISR()不可用于任务和中断间的同步，如果中断恰巧在任务被挂起之前到达，这就会导致一次中断丢失（任务还没有挂起，调用xTaskResumeFromISR()函数是没有意义的，只能等下一次中断）。这种情况下，可以使用信号量作为同步机制。
    ```
xTaskHandlexHandle;               //注意这是一个全局变量  
 void vAFunction( void )  
 {  
     // 创建任务并保存任务句柄  
     xTaskCreate( vTaskCode, "NAME",STACK_SIZE, NULL, tskIDLE_PRIORITY, &xHandle );  
   
     // ... 剩余代码.  
 }  
 void vTaskCode( void *pvParameters )  
 {  
     for( ;; )  
     {  
         // ... 在这里执行一些其它功能  
         // 挂起自己  
         vTaskSuspend( NULL );  
         //直到ISR恢复它之前，任务会一直挂起  
     }  
 }  
 void vAnExampleISR( void )  
 {  
     portBASE_TYPExYieldRequired;  
     // 恢复被挂起的任务  
     xYieldRequired = xTaskResumeFromISR(xHandle );  
     if( xYieldRequired == pdTRUE )  
     {  
         // 我们应该进行一次上下文切换  
         // 注:  如何做取决于你具体使用，可查看说明文档和例程  
         portYIELD_FROM_ISR();  
     }  
 }  
```