# 5.2 IIC操作实例
## 5.2.1.硬件设计
 使用主模式IIC向AT24C16的EEPROM中写入0~255，再从AT24C16中读出写入的内容。对读写内容进行比较，检测AT24C16与S3C6310的IIC借口之间的操作是否正常。
![](/assets/图片 5.2.1.1.png)IIC电路图
## 5.2.2.软件设计
 软件设计部分主要给出主程序、测试IIC程序、IIC写AT24C16程序、 IIC读AT24C16程序。
![](/assets/图片 5.2.2.1.png)测试IIC程序流程
 1. 设置I2C控制寄存器

 1）收发传输：IICCON=0b 1 0 1 0 1111 = 0xAF

 含义：应答使能、时钟分频为 IICCLK = f PCLK /16 、

 中断使能、清除中断标志、预分频值取15。

 2）接收结束传输：IICCON=0b 0 0 1 0 1111 = 0x2F

 含义：禁止应答（非应答）、时钟分频为

 IICCLK = f PCLK /16 、中断使能、清除中断标志、预分频值取15。


2. I2C控制状态寄存器

1）主模式发送、启动传输

 IICSTAT=0b 11 1 1 0 0 0 0 = 0xF0

 含义：主设备发送、启动传输、输出使能、低4位为状态

2）主模式发送、结束传输

 IICSTAT=0b 11 0 1 0 0 0 0 = 0xD0

 含义：主设备发送、结束传输、输出使能、低4位为状态

3）主模式接收、启动传输

 IICSTAT=0b 10 1 1 0 0 0 0 = 0xB0

 含义：主设备接收、启动传输、输出使能、低4位为状态

4）主模式接收、结束传输

 IICSTAT=0b 10 0 1 0 0 0 0 = 0x90

 含义：主设备接收、结束传输、输出使能、低4位为状态


 3. 地址寄存器设置

 1）S3C6410地址寄存器：

 作为从设备地址为0x10（作为主设备无意义）

 2）AT24C16 EEPROM芯片地址：

 作为从设备地址为 0xA0

 注*: AT24C16存储容量为512字节，该器件的编址为1010A2A1P0，P0为页地址，P0=0表示寻址低256字节单元，P0=1表示寻址高256字节单元。


 4. 寻址字节值

 所寻址的“从设备地址+操作控制命令”(R/W):

 1）主设备发送： 0xA0

 2）主设备接收： 0xA1


5. 程序

`#include <string.h>`

`#include "6410addr.h"`

`#include "6410lib.h"`

`#include "def.h "`



U32 _iicStatus;
void Test_Iic(void)

{

 unsigned int i,j;

 static U8 data[256];



 Uart_Printf("[ IIC Test using AT24C16 ]\n");



 rGPEUP |= 0xc000; //Pull-up resistors disable

 rGPECON |= 0xa0000000; //GPE15: IICSDA

 //GPE14: IICSCL

 rIICCON = (1<<7) | (0<<6) | (1<<5) | (0xf); //rIICCON = 0xAF

 rIICADD = 0x10; // S3C6410作为I2C总线从设备的地址

 rIICSTAT = 0x10; //I2C总线数据输出允许(Rx/Tx)


 Uart_Printf("Write test data into AT24C16(0-255)\n");



 for(i=0;i<256;i++)

 _Wr24C16(0xa0,(U8)i,i);



 for(i=0;i<256;i++)

 data[i] = 0;



 Uart_Printf("\nRead test data from AT24C16\n");



 for(i=0;i<256;i++)



 _Rd24C16(0xa1,(U8)i,&(data[i]));


for(i=0;i<16;i++)

 {

 for(j=0;j<16;j++)

 Uart_Printf("%2x ",data[i*16+j]);

 Uart_Printf("\n");

 }

}

void _Wr24C16(U8 slvAddr,U8 addr,U8 data)

{ // slvAddr: 从设备地址，此处为0xa0

 // addr: 待写入数据到芯片的地址

 // data: 待写入的数据功能说明

 rIICDS = slvAddr; //发送从设备地址

 rIICSTAT = 0xf0; //启动发送



 while(rIICCON & 0x10==0); //查询Tx中断状态

 rIICDS = addr; //发送存储器地址

 rIICCON = 0xaf; //清除中断状态



 while(rIICCON & 0x10==0); //查询中断状态

 data = rIICDS; //接收数据

 rIICCON = 0xaf; //清除中断状态

 while(rIICCON & 0x10==0); //查询中断状态

 while(rIICSTAT&1); //等待I2C EEPROM应答ACK



 rIICSTAT = 0xd0; //Stop(Write)

 Delay(1); //等待停止结束生效

}
void _Rd24C16(U8 slvAddr,U8 addr,U8 *data )

{ // slvAddr: 从设备地址，此处为0xa1

 // addr: 待读入数据的芯片地址

 // data: 待读入数据功能说明

 rIICDS = slvAddr; //发送从设备地址

 rIICSTAT = 0xf0; //启动发送



 while(rIICCON & 0x10==0); //查询Tx中断状态

 rIICDS = addr; //发送存储器地址

 rIICSTAT = 0xb0; //启动接收

 rIICCON = 0xaf; //清除中断状态



 while(rIICCON & 0x10==0); //查询Rx中断状态

 rIICDS = data; //接收数据

 rIICCON = 0xaf; //清除中断状态

}














