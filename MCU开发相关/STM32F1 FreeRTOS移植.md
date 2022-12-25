
## 参考文档
https://www.freertos.org/zh-cn-cmn-s/FreeRTOS-porting-guide.html

# 实操
到FreeRTOS官网下载源代码。我这里用的是LTS的版本，这个里面没有demo, 会缺少`FreeRTOSConfig.h` 这个文件。需要自己写一个或者到其他地方复制。

打开FreeRTOS-Kernel 这个文件夹，里面就是所需要的所有的源码了。
```
FreeRTOS-Kernel
	│  CMakeLists.txt
	│  croutine.c
	│  event_groups.c
	│  GitHub-FreeRTOS-Kernel-Home.url
	│  History.txt
	│  LICENSE.md
	│  list.c
	│  manifest.yml
	│  queue.c
	│  Quick_Start_Guide.url
	│  README.md
	│  sbom.spdx
	│  stream_buffer.c
	│  tasks.c
	│  timers.c
	├─include
	└─portable
```
在工程下新建FreeRTOS目录，然后将 FreeRTOS-Kernel 根目录下面的 C文件全部复制过去， include文件夹直接复制过去， portable文件夹中，根据自己的架构和编译器选择`port.c` 和 `portmarco.h` 文件，然后将这个文件夹下面的 MemMang 文件夹也复制过去。

我的工程添加以上文件后目录如下(**heap_x.c 选择一个适合自己的就可以了**)
```
└─Source
    │  croutine.c
    │  event_groups.c
    │  LICENSE.md
    │  list.c
    │  queue.c
    │  README.md
    │  stream_buffer.c
    │  tasks.c
    │  timers.c
    │
    ├─include
    │      atomic.h
    │      croutine.h
    │      deprecated_definitions.h
    │      event_groups.h
    │      FreeRTOS.h
    │      list.h
    │      message_buffer.h
    │      mpu_prototypes.h
    │      mpu_wrappers.h
    │      portable.h
    │      projdefs.h
    │      queue.h
    │      semphr.h
    │      StackMacros.h
    │      stack_macros.h
    │      stdint.readme
    │      stream_buffer.h
    │      task.h
    │      timers.h
    │
    └─portable
        ├─ARMClang
        │      Use-the-GCC-ports.txt
        ├─GCC
        │  └─ARM_CM3
        │          port.c
        │          portmacro.h
        │
        └─MemMang
                heap_1.c
                heap_2.c
                heap_3.c
                heap_4.c
                heap_5.c
                ReadMe.url
```

 接着，到官方的Demo工程里找一个 `FreeRTOSConfig.h`文件，复制到你的工程中。我是用的是官方STM32F407中使用的，跟我的使用环境比较类似。
```
/*
 * FreeRTOS V202112.00
 * Copyright (C) 2020 Amazon.com, Inc. or its affiliates.  All Rights Reserved.
 *
 * Permission is hereby granted, free of charge, to any person obtaining a copy of
 * this software and associated documentation files (the "Software"), to deal in
 * the Software without restriction, including without limitation the rights to
 * use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of
 * the Software, and to permit persons to whom the Software is furnished to do so,
 * subject to the following conditions:
 *
 * The above copyright notice and this permission notice shall be included in all
 * copies or substantial portions of the Software.
 *
 * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
 * IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS
 * FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR
 * COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER
 * IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN
 * CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
 *
 * http://www.FreeRTOS.org
 * http://aws.amazon.com/freertos
 *
 * 1 tab == 4 spaces!
 */


#ifndef FREERTOS_CONFIG_H
#define FREERTOS_CONFIG_H

/*-----------------------------------------------------------
 * Application specific definitions.
 *
 * These definitions should be adjusted for your particular hardware and
 * application requirements.
 *
 * THESE PARAMETERS ARE DESCRIBED WITHIN THE 'CONFIGURATION' SECTION OF THE
 * FreeRTOS API DOCUMENTATION AVAILABLE ON THE FreeRTOS.org WEB SITE.
 *
 * See http://www.freertos.org/a00110.html
 *----------------------------------------------------------*/

/* Ensure stdint is only used by the compiler, and not the assembler. */
#if defined(__clang__)  
	#include <stdint.h>
	extern uint32_t SystemCoreClock;
#endif

#define configUSE_PREEMPTION			1                   /* 1-抢占式， 0-时间片轮转 */
#define configUSE_IDLE_HOOK				0                   /* 空闲钩子函数 */
#define configUSE_TICK_HOOK				0                   /* 嘀嗒钩子函数 */
#define configCPU_CLOCK_HZ				( SystemCoreClock ) 
#define configTICK_RATE_HZ				( ( TickType_t ) 1000 )
#define configMAX_PRIORITIES			( 16 )              /* 任务最大优先级 */
#define configMINIMAL_STACK_SIZE		( ( unsigned short ) 130 )      /* 任务最小堆栈大小 */
#define configTOTAL_HEAP_SIZE			( ( size_t ) ( 8 * 1024 ) )     /* FreeRTOS堆栈大小 */
#define configMAX_TASK_NAME_LEN			( 32 )
#define configUSE_TRACE_FACILITY		1
#define configUSE_16_BIT_TICKS			0
#define configIDLE_SHOULD_YIELD			1                   /* 抢占式调度下，1-允许任务调度， 0-时间片时间耗尽才让出CPU使用权 */
#define configUSE_MUTEXES				1                   /* 使用互斥量 */
#define configQUEUE_REGISTRY_SIZE		8                   /* 信号量的最大数目 */
#define configCHECK_FOR_STACK_OVERFLOW	0
#define configUSE_RECURSIVE_MUTEXES		1                   /* 递归互斥量 */
#define configUSE_MALLOC_FAILED_HOOK	0
#define configUSE_APPLICATION_TASK_TAG	0
#define configUSE_COUNTING_SEMAPHORES	1
#define configGENERATE_RUN_TIME_STATS	0

/* Co-routine definitions. */
#define configUSE_CO_ROUTINES 		0
#define configMAX_CO_ROUTINE_PRIORITIES ( 2 )

/* Software timer definitions. */
#define configUSE_TIMERS				1
#define configTIMER_TASK_PRIORITY		( 2 )
#define configTIMER_QUEUE_LENGTH		10
#define configTIMER_TASK_STACK_DEPTH	( configMINIMAL_STACK_SIZE * 2 )

/* Set the following definitions to 1 to include the API function, or zero
to exclude the API function. */
#define INCLUDE_vTaskPrioritySet		1
#define INCLUDE_uxTaskPriorityGet		1
#define INCLUDE_vTaskDelete				1
#define INCLUDE_vTaskCleanUpResources	1
#define INCLUDE_vTaskSuspend			1
#define INCLUDE_vTaskDelayUntil			1
#define INCLUDE_vTaskDelay				1

/* Cortex-M specific definitions. */
#ifdef __NVIC_PRIO_BITS
	/* __BVIC_PRIO_BITS will be specified when CMSIS is being used. */
	#define configPRIO_BITS       		__NVIC_PRIO_BITS
#else
	#define configPRIO_BITS       		4        /* 15 priority levels */
#endif

/* The lowest interrupt priority that can be used in a call to a "set priority"
function. */
#define configLIBRARY_LOWEST_INTERRUPT_PRIORITY			0xf

/* The highest interrupt priority that can be used by any interrupt service
routine that makes calls to interrupt safe FreeRTOS API functions.  DO NOT CALL
INTERRUPT SAFE FREERTOS API FUNCTIONS FROM ANY INTERRUPT THAT HAS A HIGHER
PRIORITY THAN THIS! (higher priorities are lower numeric values. */
#define configLIBRARY_MAX_SYSCALL_INTERRUPT_PRIORITY	5

/* Interrupt priorities used by the kernel port layer itself.  These are generic
to all Cortex-M ports, and do not rely on any particular library functions. */
#define configKERNEL_INTERRUPT_PRIORITY 		( configLIBRARY_LOWEST_INTERRUPT_PRIORITY << (8 - configPRIO_BITS) )
/* !!!! configMAX_SYSCALL_INTERRUPT_PRIORITY must not be set to zero !!!!
See http://www.FreeRTOS.org/RTOS-Cortex-M3-M4.html. */
#define configMAX_SYSCALL_INTERRUPT_PRIORITY 	( configLIBRARY_MAX_SYSCALL_INTERRUPT_PRIORITY << (8 - configPRIO_BITS) )
	
/* Normal assert() semantics without relying on the provision of an assert.h
header file. */
#define configASSERT( x ) if( ( x ) == 0 ) { taskDISABLE_INTERRUPTS(); for( ;; ); }	
	
/* Definitions that map the FreeRTOS port interrupt handlers to their CMSIS
standard names. */
#define vPortSVCHandler SVC_Handler
#define xPortPendSVHandler PendSV_Handler
#define xPortSysTickHandler SysTick_Handler

#endif /* FREERTOS_CONFIG_H */

```


# 关于 heap_x.c的区别
https://www.cnblogs.com/FutureHardware/p/14220238.html

## heap_1

-   在调度程序运行前，提前动态分配一大段内存空间，不管任务用与不用，用多少，内存占用是固定的
-   商业，安全领域，禁止动态分配内存
    -   basic version api: `pvPortMalloc()`，没有vPortFree()！
    -   `configTOTAL_HEAP_SIZE` in FreeRTOSConfig.h

> 每创建一个任务，都需要一个`task control block (TCB)`

## heap_2

-   heap_2 是为了向后兼容才保留的，建议新的设计中使用heap_4代替。
-   heap_2 允许释放内存
-   heap_2 与 heap_1一样需要分割 `configTOTAL_HEAP_SIZE` 所静态分配的内存
-   最合适内存分配算法，比如要分配20bytes, 现在有5，25，50等内存区域，heap_2就会将25分配出20bytes。
-   但heap_2 不能整合相邻的空闲内存区域, heap_4可以。
-   heap_2 适合于，重复性申请和释放内存的操作，并且每次内存大小都一样。

## heap_3

-   heap_3 使用标准库 `malloc()` 和 `free()` 函数， `configTOTAL_HEAP_SIZE`将不起作用。 heap 大小决定于linker配置

## heap_4

-   heap_4 与heap_1、heap_2 一样是从内存数组分配出小的内存块
-   内存数组大小决定于`configTOTAL_HEAP_SIZE`，这就造成一种现象：虽然没有任何内存被分配，但程序已经消耗了很多的内存。
-   heap_4 采用 内存适配算法 和 相邻内存整合算法（减少了内存碎片化的风险）

## heap_5

-   heap_5 分配和释放内存的算法 和 heap_4 一样。
-   heap_5 不局限于从静态内存数组中分配内存，它可以从 多个、不连续的内存空间中分配内存。
-   当使用heap_5时，`vPortDefineHeapRegions()`必须在 pvPofrtMalloc、内核对象之前调用。


# 根据实际情况实现Assert宏
鉴于某些情况下可能无法使用debuger调试器，可以改写ASSERT宏使用printf 输出错误的位置。
```
#if 0               /* 进入死循环 */
#define configASSERT( x ) if( ( x ) == 0 ) { taskDISABLE_INTERRUPTS(); for( ;; ); }	
#else               /* printf 打印错误位置 */
#define vAssertCalled(char,int)     {printf("Error: Assert failed :%s, L:%d\r\n",char,int); for(;;);}
#define configASSERT(x)             if((x)==0) vAssertCalled(__FILE__,__LINE__)
#endif
```