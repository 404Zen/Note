
# 为什么还要使用标准库？
主要是国内一众厂商目前提供的SDK和ST标准库的写法更加相似。加上最近ST更新了STM32F1的标准库，目前正好有某些方面的需求，重新学习一下。

# ST标准库的下载
点开[这个链接](https://www.st.com/zh/embedded-software/stm32-standard-peripheral-libraries.html?querycriteria=productId=LN1939)，然后再图片中选择对应的系列的MCU，填写邮箱等信息之后，会将下载链接以邮件的形式发出。目前的最新版本是 V3.6.0
![[Pasted image 20221224123554.png]]

# 编译工程
下载过来之后，直接打开目录`en.stsw-stm32054_v3-6-0\STM32F10x_StdPeriph_Lib_V3.6.0\Project\STM32F10x_StdPeriph_Template\MDK-ARM`  下的工程文件，应该是可以正常编译通过的。一般情况下，拿这个工程根据你的板子定义进行修改也都够用了。


# 新建工程的方式
新建工程的方式有很多种，可以直接使用Keil新建工程，然后根据选择需要的包就好，但是这样不仅要求一个国际标准的互联网连接，建出来的工程可能其他人无法很好地使用。

我们通过将CMSIS源文件与StdPeriph_Lib库文件添加到自己工程的方式进行新建工程。这样新建出来的工程的缺点是当CMSIS或者StdPeriph_Lib升级之后，需要手动替换这些文件。
建好的工程目录如下
```
+---CMSIS
|   \---5.8.0
|       +---CMSIS
|                                   
+---Project
|   |   Project.sct
|   |   Project.uvoptx
|   |   Project.uvprojx
|   |       
|   \---Objects
|
+---Startup
|       startup_stm32f10x_hd.s
|       
+---STM32F10x_StdPeriph_Driver
|           
\---User
        main.c
        stm32f10x.h
        stm32f10x_conf.h
        stm32f10x_it.c
        stm32f10x_it.h
        system_stm32f10x.c
        system_stm32f10x.h
```

其中 CMSIS 文件夹从Keil的安装目录中复制；Project目录为Keil的工程目录；Startup文件夹只需要放对于的 startup_xxx.s 启动文件即可；STM32F10x_StdPeriph_Driver为标准库。

_从Keil中复制CMSIS的目的是使工程支持AC6编译。_

