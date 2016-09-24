# 4.2 GPIO操作实例
## 4.2.1.硬件设计
 利用4个发光二极管LED与GPIO口相连，实现LED的点亮和熄灭。GPIO选用GPF端口的GPF4~GPF7分别与发光二极管LED1~LED4相连。发光二极管的正极与S3C6410的3.3V高电压相连，负极与S3C6410的GPF4~GPF7引脚相连(通过限流电阻)。

 GPF4~GPF7通过GPFCON寄存器[15:8]位设置为输出，再通过GPFDAT寄存器的[7:4]位写入1或0使GPF4~GPF7引脚输出高电平或低电平。当GPF4~GPF7输出高电平时，LED熄灭，当GPF4~GPF7输出低电平时，LED点亮。
![](/assets/图片 4.2.2.png)
LED控制电路

## 4.2.2.软件设计
 重点是对GPIO口参数的设置上，首先对配置寄存器进行设置，即将选择的GPF4~GPF7设置为输出状态；其次对GPFPUD寄存器进行设置，即禁止GPF4~GPF7上拉使能；最后当基本设置设置完毕后，依次点亮发光二极管后再依次熄灭发光二极管
![](/assets/图片 4.2.22.png) 程序流程图

寄存器设置

为了实现控制LED的目的，需要通过配置GPFCON寄存器将GPF4、GPF5、GPF6、GPF7设置为输出属性。如：配置GPFCON[9:8]两位为“01”，可实现将GPF4设置为输出属性。

通过设置GPFDAT寄存器实现点亮与熄灭LED。如：配置GPFDAT[4]为“0”，可实现点亮LED4。配置GPFDAT[4]为“1”，可实现关闭LED4。

对于本例来说，GPFUP可以不用设置。
程序的编写

1、相关寄存器定义

`#define rGPFCON (*(volatile unsigned *)0x56000050)`

//端口F的控制寄存器

`#define rGPFDAT (*(volatile unsigned *)0x56000054)`

 //端口F的数据寄存器

`#define rGPFUP (*(volatile unsigned *)0x56000058)`

 //端口F的上拉控制寄存器 

2、端口初始化

void port_init(void)

{

//=== PORT F GROUP

//端口: GPF7 GPF6 GPF5 GPF4 GPF3 GPF2 GPF1 GPF0

//信号: LED_1 LED_2 LED_3 LED_4 PS2_INT CPLD_INT1 KEY_INT BUT_INT1

//设置属性: Output Output Output Output EINT3 EINT2 EINT1 EINT0

//二进制值: 01 01, 01 01, 10 10, 10 10

 rGPFCON = 0x55aa;

 rGPFUP = 0xff; // GPF所有端口都不加上拉电阻

}

3、开启LED

void led_on(void)

{

 int i,nOut;

 nOut=0xF0;

 rGPFDAT=nOut & 0x70; //点亮LED1

 for(i=0;i<100000;i++);

 rGPFDAT=nOut & 0x30; //点亮LED1 LED2

 for(i=0;i<100000;i++);

 rGPFDAT=nOut & 0x10; //点亮LED1 LED2 LED3

 for(i=0;i<100000;i++);

 rGPFDAT=nOut & 0x00; //点亮LED1 LED2 LED3 LED4

 for(i=0;i<100000;i++);

}

4、关闭LED

void led_off(void)

{

 int i,nOut;

 nOut=0;

 rGPFDAT = 0;

 for(i=0;i<100000;i++);

 rGPFDAT = nOut | 0x80; //关闭LED1

 for(i=0;i<100000;i++);

 rGPFDAT |= nOut | 0x40; //关闭LED2

 for(i=0;i<100000;i++);

 rGPFDAT |= nOut | 0x20; //关闭LED3

 for(i=0;i<100000;i++);

 rGPFDAT |= nOut | 0x10; //关闭LED4

 for(i=0;i<100000;i++);

}

5、所有LED交替亮灭

void led_on_off(void)

{

 int i;

 rGPFDAT=0; //所有LED全亮

 for(i=0;i<100000;i++);

 rGPFDAT=0xF0; //所有LED全灭

 for(i=0;i<100000;i++);

}


