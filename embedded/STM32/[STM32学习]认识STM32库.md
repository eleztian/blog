---
title: "[Stm32 Learning]  About Stm32's Lib"
short: ""
date: 15.Apr.2016
tags:
    - c
    - stm32
    - embedded
    - notes
---

STM32库文件的之间的关系
--------------

CMSIC标准（软件抽象层）

 ![这里写图片描述](http://img.blog.csdn.net/20160713172446125)
 
CMSIS 标准中最主要的是 CMSIS 核心层，它包括 ：
 内核函数层 ：其中包含用于访问内核寄存器的名称、地址定义，主要由 ARM 公司提供。
	设备外设访问层 ：提供了片上的核外外设的地址和中断定义，主要由芯片生产商提供。可见 CMSIS 层位于硬件层与操作系统或用户层之间，提供了与芯片生产商无关的硬件抽象层，可以为接口外设、实时操作系统提供简单的处理器软件接口，屏蔽了硬件差异，这对软件的移植有极大的好处。STM32 固件库就是按照CMSIS 标准建立的。
	
1.	core_cm3.c 和头文件 core_cm3.h：
	它们的作用是为采用 Cortex-M3 核设 计 SoC 的芯片商设计的芯片外设提供一个进入 CM3 内核的接口。较重要的是在core_cm3.c 文件中包含了 stdin.h，些新类型定义如uint8_t  等等。但是在STM32f10x.h 文件中也定义了u8 u16 u32等等. 所有 CM3 芯片的库都带有这个文件。
2.	system_stm32f10x．c 文件：
	该文件的功能是设置系统时钟和总线时钟。STM32 整个系统就以 8M 为时钟协调整个处理器的工作。我们还要通过 CM3 核的核内寄存器来对 8M 的时钟进行倍频、分频，或者使用芯片内部的时钟。
3.	stm32f10x. h 文件：
	它包含了 STM32 中寄存器地址和结构体类型定义，在使用到 STM32 固件库的地方都要包含这个头文件。
4.	启动文件：
	cl：互联型产品，stm32f105/107 系列
	vl：超值型产品，stm32f100 系列
	xl：超高密度（容量）产品，stm32f101/103 系列
	ld：低密度产品，FLASH 小于 64K
	md：中等密度产品，FLASH=64 or 128
	hd：高密度产品，FLASH 大于 128
总的来说，启动文件的作用是 ：
	1. 初始化堆栈指针 SP;
	2. 初始化程序计数器指针 PC;
	3. 设置堆、栈的大小;
	4. 设置异常向量表的入口地址;
	5. 配置外部 SRAM 作为数据存储器（这个由用户配置，一般的开发板可没有外部SRAM）;
	6. 设置 C库的分支入口__main（最终用来调用 main 函数）;
	7. 在 3.5 版的启动文件还调用了在 system_stm32f10x.c 文件中的 SystemIni() 函数配置系统时钟，在旧版本的工程中要用户进入 main 函数自己调用 SystemIni() 函数。
	 
 ![这里写图片描述](http://img.blog.csdn.net/20160713172608126)

5.	STM32F10x_StdPeriph_Driver 文件夹：inc（include 的缩写）和 src（source的缩写）这两个文件夹，这都属于 CMSIS 的设备外设函数部分，我们把这类外设文件统称为 ：stm32f10x_ppp.c 或stm32f10x_ppp.h文件，ppp 表示外设名称。其中一个特别misc.c 文件，这个文件提供了外设对内核中的NVIC（中断向量控制器）的访问函数，在配置中断时，我们必须把这个文件添加到工程中。
6.	stm32f10x_it. c 和 stm32f10x_conf. h 文件：用库建立一个完整的工程时，还需要添加这个目录下的 stm32f10x_it.c、stm32f10x_it.h 和 stm32f10x_conf.h 这三个文件，其中stm32f10x_it.c是专门用来编写中断服务函数的.中断服务程序的自定义需要我们结合启动文。stm32f10x_conf.h 文件被包含进 stm32f10x.h 文件，是用来配置使用了什么外设的头文件，用这个头文件我们可以很方便地增加或删除上面 Driver 目录下的外设驱动函数库。stm32f10x_conf.h 这个文件还可配置是否使用“断言”编译选项，在开发时使用“断言”可由编译器检查库函数传入的参数是否正确，软件编写成功后，去掉“断言”编译选项可使程序全速运行。可通过定义 USE_FULL_ASSERT 或取消定义来配置是否使用“断言”。
7.	各个文件之间的关系如下图:

 ![文件关系图](http://img.blog.csdn.net/20160713171912601)


							学习《零死角玩转STM32 -- MINI版》
