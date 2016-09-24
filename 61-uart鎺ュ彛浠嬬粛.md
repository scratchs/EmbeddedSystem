# 6.1 UART接口介绍

## 6.1.1.UART结构及操作 
### 1. UART的特性 

 每个UART 包含一个波特率发生器，发送器，接收器和控制单元。该波特率发生器由pclk ， ext_uclk0或ext_uclk1 进行时钟控制 。发射器和接收器包含64 字节的FIFO 存储器和数据移位寄存器。发送数据之前，首先将数据写入FIFO 存储器 ，然后复制到发送移位寄存器。通过发送数据的引脚(txdn)将数据发送，同时通过数据接收的引脚(rxdn)将接收到的数据从接收移位寄存器复制到FIFO 存储器。

### 2．UART的操作 

数据发送

数据接收

自动流量控制

接收FIFO的操作

发送FIFO的操作

RS-232C接口

中断/DMA请求的产生

UART错误状态FIFO

红外线模式

## 6.1.2.寄存器
UART行控制寄存器

UART控制寄存器

UART 的FIFO控制寄存器

UART Modem控制寄存器

UART 接收(Rx)/(Tx)发送状态寄存器

UART 错误状态寄存器

UART 的 FIFO 状态寄存器

UART 发送缓冲寄存器(保存寄存器和FIFO寄存器)

UART 接收缓冲寄存器(保存寄存器和FIFO寄存器)

UART波特率分频寄存器

波特率错误容限

UART中断处理寄存器

UART中断源处理寄存器

UART中断屏蔽寄存器

UART Modem 状态寄存器

