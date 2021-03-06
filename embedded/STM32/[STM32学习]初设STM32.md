---
title: "[Stm32 Learning]  The First Stm32"
short: ""
date: 08.Apr.2016
tags:
    - c
    - stm32
    - embedded
    - notes
---

# 初涉STM32
## 一、学习入门（从51到stm32）
寄存器映射：外设资源与地址一一对应（sfr 和 sbit）；

STARTUP.A51:启动代码，单片机上电复位后，首先执行启动文件。主要实现了以下功能：清除内部数据存储器、清除外部数
        据存储器、清除外部页储存器、初始化 small 模式下的可重入栈和指针、初始化 large 模式下可重入栈和指针、
        初始化 compact 模式下的可重入栈和指针、初始化 8051 硬件栈指针、传递初始化全局变量的控制命令或者在没
        有初始化全局变量时给 main 函数传递命令
startup_stm32f10x_hd.s：启动文件  功能与51相当
外设基地址:每个外设的起始地址
volatile：volatile int i; i是随时可能变化的

GPIO的操作：
1. 时钟控制：寄存器 RCC,STM32 的外设因为速率的不同，分别挂载到三条总系上：AHB、APB2、APB1，APB
为高速总线，APB2 次之，APB1 再次之。所有的 IO 口都挂载到 APB2 总线上，属于高速外设。ST 设计了三条总线：AHB、APB2 和 APB1，其中 AHB 和 APB2 是高速总线，APB1 是低速总线。不同的外设根据速度不同分别挂载到这三条总线上。其中 APB1 地址又叫做外设基地址，叫做 PERIPH_BASE。
2. 方向控制：端口配置寄存器分为高低两个，每 4bit 控制一个 IO 口，所以端口配置低寄存器：CRL 控制这 IO 口的低 8 位，端口配置高寄存器：CRH控制这 IO 口的高 8bit。
在 4 位一组的控制位中，CNFy[1:0] 用来控制端口的输入输出，MODEy[1:0]用来控制输出模式的速率，即输出时，IO 电平翻转的速度

GPIO工作配置
1. GPIO_Mode_AIN 模拟输入
2. GPIO_Mode_IN_FLOATING 浮空输入
3. GPIO_Mode_IPD 下拉输入
4. GPIO_Mode_IPU 上拉输入
5. GPIO_Mode_Out_OD 开漏输出
6. GPIO_Mode_Out_PP 推挽输出
7. GPIO_Mode_AF_OD 复用开漏输出
8. GPIO_Mode_AF_PP 复用推挽输出

关于GPIO 的8种工作模式的介绍见：http://blog.csdn.net/kevinhg/article/details/17490273
库文件中定义一个 GPIO 寄存器结构体，结构体里面的成员是 GPIO 的寄存器，成
员的顺序按照寄存器的偏移地址从低到高排列，成员类型跟寄存器类型一样。
```c
typedef struct {
    __IO uint32_t CRL;
    __IO uint32_t CRH;
    __IO uint32_t IDR;
    __IO uint32_t ODR;
    __IO uint32_t BSRR;
    __IO uint32_t BRR;
    __IO uint32_t LCKR;
} GPIO_TypeDef;
```
_IO原型是 volatile

                                                                                                                                   2016.5.22



