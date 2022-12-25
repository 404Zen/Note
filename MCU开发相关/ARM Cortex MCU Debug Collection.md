# 程序不运行，debug 停在 BKPT 0xAB
_STM32F103, _
具体参考： https://www.cnblogs.com/ChYQ/p/5726020.html

**一般是在重定向fputc后出现该问题，因为stdio中的printf函数需要使用半主机模式，如果想不使用半主机模式，可以使用microlib 或者 重新实现C库中stdio.h中的一些使用了半主机模式的函数。**

- 关于半主机模式的作用
	https://www.bilibili.com/read/cv19478564


在使用串口的时候，代码可以正常编译，没有报任何错误，烧录进MCU内，就是看不到程序正常运行的现象，而把串口部分注释掉就没问题。进入调试模式，发现代码停在 "BKPT　　0xAB" 这里，并不是死循环，按下全速运行键“F5”，代码会立马在该段被终止，不会继续往下跑，这里说明了main函数都没有进入。Google到了ARM的技术支持有提到过这个问题， “ARM: Application Builds Without Error, But Does Not Run”，这个链接描述的现象即是我现在碰到的现象。此处指出，调试时，出现代码停在 “BKPT　　0XAB” 的现象，说明Semihosting 被使能了。

比较推荐的做法是添加一个retarget.c的文件， 文件内容如下
```
#include "stdio.h"

#if defined(__CC_ARM)
#define PUTCHAR_PROTOTYPE int fputc(int ch, FILE *f)
#pragma import(__use_no_semihosting_swi)
#pragma import(__use_no_semihosting)
#elif defined(__ARMCC_VERSION) && (__ARMCC_VERSION >= 6010050)
#define PUTCHAR_PROTOTYPE int fputc(int ch, FILE *f)
__asm(".global __use_no_semihosting");
__asm(".global __use_no_semihosting_swi");
#elif defined(__GNUC__)
#define PUTCHAR_PROTOTYPE int __io_putchar(int ch)
#endif

void _sys_exit(int x)
{
    x = x;
}

struct FILE
{
    int handle;
    /* Whatever you require here. If the only file you are using is */
    /* standard output using printf() for debugging, no file handling */
    /* is required. */
};
/* FILE is typedef’ d in stdio.h. */
FILE __stdout;

/**
 * @brief  Retargets the C library printf function to the USART.
 * @param  None
 * @retval None
 */
// int fputc(int ch, FILE *f)
PUTCHAR_PROTOTYPE
{
    USART_ClearFlag(USART1, USART_FLAG_TC);
    /* Place your implementation of fputc here */
    /* e.g. write a character to the USART */
    USART_SendData(USART1, (uint8_t)ch);

    /* Loop until the end of transmission */
    while (USART_GetFlagStatus(USART1, USART_FLAG_TC) == RESET)
    {
    }

    return ch;
}
```



----

# 串口相关问题

## 复位后串口丢失首字节
_STM32F103_

在初始化完成之后，要清除TC标志位, 为了省事，直接在初始化完成后清楚所有的标志位。
```
/* Clear all flag */
USART_ClearFlag(USART1, 0x03FF);
```

## 重复/一直进入接收空闲中断
_STM32F103_

接收空闲中断需要通过读取 SR 和 DR 寄存器进行清楚
```
uint16_t clr;
clr = USART1->SR;               /* 清除空闲中断, 先读USART_SR，然后读USART_DR */
clr = USART1->DR;  
```

----
