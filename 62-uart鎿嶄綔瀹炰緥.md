# 6.2 UART操作实例
## 6.2.1 RS232原理
电气特性

RS-232与TTL转换

接口定义

常用引脚说明

## 6.2.2.硬件设计
 采用MAX232芯片进行电平转换，即将S3C6410的逻辑1信号变成-3V~-15V，将S3C6410的逻辑0信号变成+3V~+15V，从而实现串行通信。
![](/assets/图片 6.2.2.1.png)RS232电路连接图

## 6.2.3.软件设计
初始化串口
![](/assets/图片 6.2.3.1.png)

发送流程
![](/assets/图片 6.2.3.2.png)

接收流程
![](/assets/图片 6.2.3.3.png)

1．定义与UART相关的寄存器

 以UART0为例，需要定义的寄存器如下：
•#define rULCON0（*（volatile unsigned*）0x50000000）

 //UART0行控制寄存器

•#define rUCON0（*（volatile unsigned*）0x50000004）

 //UART0控制寄存器

•#define rUFCON0（*（volatile unsigned*）0x50000008）

 //UART0 FIFO控制寄器

•#define rUMCON0（*（volatile unsigned*）0x5000000c）

 //UART0 Modem控制寄存器

•#define rUTRSTAT0（*（volatile unsigned*）0x50000010）

 //UART0 Tx/Rx状态寄存器

•#define rUERSTAT0（*（volatile unsigned*）0x50000014）

 //UART0 Rx错误状态寄存器

•#define rUFSTAT0（*（volatile unsigned*）0x50000018）

 //UART0 FIFO状态寄存器
•#define rUMSTAT0（*（volatile unsigned*）0x5000001c）

 //UART0 Modem状态寄存器

•#define rUBRDIV0（*（volatile unsigned*）0x50000028）

 //UART0波特率系数寄存器

•#ifdef__BIG_ENDIAN

 //大端模式

•#define rURXH0（*（volatile unsigned char*）0x50000023）

 //UART0发送缓冲寄存器

•#define rURXH0（*（volatile unsigned char*）0x50000027）

 //UART0接收缓冲寄存器

•#define WrUTXH0（ch）（*（volatile unsigned char*）0x50000023）＝（unsigned char）（ch）

•#define RdURXH0（）（*（volatile unsigned char*）0x50000027）
•#define UTXH0（0x50000020+3）

 //DMA使用的字节访问地址

•#define URXH0（0x50000024+3）

•#else //小端模式

•#define rUTXH0（*（volatile unsigned char*）0x50000020）

 //UART0 Transmission Hold

•#define rURXH0（*（volatile unsigned char＊）0x50000024）

 //UART0 Receive buffer

•#define WrUTXH0（ch）（*（volatile unsigned char*）0x50000020）＝（unsigned char）（ch）

•#define RdURXH0（）（*（volatile unsigned char*）0x50000024）

•#define UTXH0（0x50000020）

 //DMA使用的字节访问地址

•#define URXH0（0x50000024）

•#endif
2. 初始化操作

 参数pclk为时钟源的时钟频率，band为数据传输的波特率，初始化函数Uart_Init （ ）的实现如下：

void Uart_Init（int pclk，int baud）｛

 int I;

 if （pclk= =0）

 pclk＝PCLK;

 rUFCON0=0x0; //UARTO FIFO控制寄存器，FIFO禁止

 rUFCON1=0x0; //UART1 FIFO控制寄存器，FIFO禁止

 rUFCON2=0x0; //UART2 FIFO控制寄存器，FIFO禁止

 rUMCON0=0x0; //UARTO MODEM控制寄存器，AFC禁止

 rUMCONI=0x0; //UART1 MODEM控制寄存器，AFC禁止

 //UART0
 rULCON0＝0x3 ;

//行控制寄存器: 正常模式，无奇偶校验，1位停止位，8位数据位

 rUCON0＝0x245 ; //控制寄存器

 rUBRDIV0 =（（int）（pclk/16．/baud+0.5）-1） ;

 //波特率因子寄存器

 //UART1

 rULCON1=0x3;

 rUCON1=0x245;

 rUBRDIV1=（（int）（pclk/16．/baud）-1）;

 //UART2

 rULCON2=0x3;

 rUCON2＝0x245;

 rUBRDIV2=（（int）（pclk/16．/baud）-1）;

 for（i＝0; i< 100; i++）;

 ｝

3．发送数据

 其中whichUart为全局变量，指示当前选择的UART通道，使用串口发送一个字节的代码如下：

void Uart_SendByte（int data）｛

 if（whichUart= =0）｛

 if（data= =’＼n’）｛

 while（！（rUTRSTAT0＆0x2））；

 Delay（10）； //延时，与终端速度有关

WrUTXH。（’＼r’）；

 ｝

while（！（rUTRSTAT0＆0x2））； //等待，直到发送状态就绪

 Delay（10）；

 WrUTXH0（data）；

 ｝

 else if（whichUart= =1）｛

 if（data=＝’＼n’）｛

 while（！（rUTRSTAT1＆0x2））；

 Delay（10）； //延时，与终端速度有关

 rUTXH1=’＼r’；

 ｝

 while（！（rUTRSTAT1＆0x2））； //等待，直到发送状态就绪

 Delay（10）；

 rUTXH1＝data；

 ｝

 else if（whichUart= =2）｛

 if （data= =’\n’）｛

 while（！（rUTRSTAT2＆0x2））；

 Delay（10）； //延时，与终端速度有关

 rUTXH2＝’＼r’；

 ｝

 while（！（rUTRSTAT2＆0x2））； //等待，直到发送状态就绪

 Delay（10）；

 rUTXH2＝data；

 ｝

 ｝

4．接收数据

如果没有接收到字符则返回0。使用串口接收一个字符的代码如下：

char Uart_GetKey（void）｛

 if（whichUart＝＝0）｛

 if（rUTRSTAT0＆0x1） //UARTO接收到数据

 return RdURXH0（）；

 else

 return 0；

 ｝

 else if（whichUart= =1）｛

 if（rUTRSTAT1＆0x1） //UART1接收到数据

 return RdURXH1（）；

 else

 return 0；

 ｝

 else if（whichUart= =2）｛

 if（rUTRSTAT2＆0x1） //UART2接收到数据

 return RdURXH2（）；

 else

 return 0；

 ｝else

 return 0；

 ｝

5．主函数：实现的功能为从UART0接收字符，然后将接收到的字符再分别从UART0和UART1送出去，其中Uart_Select（n）用于选择使用的传输通道为UARTn。代码如下：

＃include＜string．h>

＃include”．．\INC\config．h”

void Main（void）｛

char data；

Target_Init（）；

while（1）｛

data=Uart GetKey（）； //接收字符

if（data！＝0x0）｛

Uart_Select（0）；． //从UART0发送出去

Uart_Printf（”key＝％c\n”，data）；

Uart_elect （1）； //从UART1发送出去

Uart_Printf（”key＝％c \n”，data）；

Uart_Select（0）；

｝

｝

｝
