# 7.2 ADC操作实例

## **7.2.1.硬件设计**

读取一个模拟输入信道的值，并进行AD转换。转换结束后，读取转换结果，实现模拟信号到数字信号的转换。

![](/assets/7.2.1.png)

## 7.2.2.软件设计

### 1.主要参数

* A\/D转换器的通道0获取模拟数据，并将转换后的数字量以波形的形式在LCD上显示。

* 模拟输入信号的电压范围必须是0～2.5V。


### 2. 函数分析

实现将转换后的数据在LCD上以波形的方式显示，主要步骤如下：

* 首先建立通道

* 其次启动通道，并判断通道是否启动成功

* 然后判断转换是否结束

* 若转换结束，则将ADCDAT0中的低十位的数据返回


1．定义与A／D转换相关的寄存器

定义如下：

\#define rADCCON（\*（volatile unsigned\*）0x58000000） \/\/ADC控制寄存器

\#define rADCTSC（\*（volatile unsigned\*）0x58000004） \/\/ADC触摸屏控制寄存器

\#define rADCDLY（\*（volatile unsigned\*）0x58000008） \/\/ADC启动或间隔延时寄存器

\#define rADCDAT0（\*（volatile unsigned\*）0x5800000c） \/\/ADC转换数据寄存器0

\#define rADCDAT1（\*（volati1e unsigned\*）0x58000010） \/\/ADC转换数据寄存器

2．对A\/D转换器进行初始化

程序中的参数ch表示所选择的通道号，程序如下：

void AD\_Init（unsigned char ch）｛

rADCDLY=100; \/\/ADC启动或间隔延时

rADCTSC=0; \/\/选择ADC模式

rADCCON=（1&lt;&lt;14）\|（0x&lt;&lt;6）\|（ch&lt;&lt;3）\| （0&lt;&lt;2）\|（0&lt;&lt;1）\|（0）; \/\/设置ADC控制寄存器

｝

3.获取A\/D的转换值

程序中的参数ch表示所选择的通道号，程序如下：

int Get\_AD（unsigned char ch）｛

int i；

int val= 0;

i f （ch&gt;7） return 0; \/\/通道不能大于7

for（i=0; i&lt; 16; i++）｛ \/\/为转换准确，转换16次

rADCCON \|=0x1; \/\/启动A\/D转换

rADCCON= rADCCON＆0xffc7 \|（ch&lt;&lt;3）;

while （rADCCON＆0x1）; \/\/避免第一个标志出错

while（！（rADCCON＆0x8000））; \/\/避免第二个标志出错

val +=（rADCDAT0＆0x03ff）;

Delay（10）;

}

return（val &gt;&gt; 4）; \/\/为转换准确，除以16取均值

}

4.主函数
实现将转换后的数据在LCD上以波形的方式显示，程序如下：

void Main（void）｛

int i，P＝0;

unsigned short buffer\[Length\]; \/\/ 显示缓冲区

Target\_Init（）；

GUI\_Init（）； \/\/ 图形界面初始化

Set\_Color （GUI\_BLUE）； \/\/画显示背景界面

Fill\_Rect（0,0,319,239）；

Set\_Color（GUI\_RED）；

Draw Line（0,119,319,119）；

Set\_Font（＆GUI\_Font 8x16）； \/\/设定字体类型API

Set\_Color（GUI\_WHITE）；

Set\_BKColor（GUI\_BLUE）； \/\/设定背景颜色API

Fill\_Rect（0,0,319,3）；

Fill\_Rect（0,0,3,239）；

Fill\_Rect（316,0,319,239）；

Fill\_Rect（0,236,319,239）；

Disp\_String（“ADC DEMO”，（320 – 8\*8）／2, 30）；

for（i＝0；i&lt; Length；i++）

buffer［i］＝0；

while（1）｛

p＝0；

for（i＝0；i&lt; Length；i ++）｛

buffer\[p\] =Get \_AD（0）; \/\/从通道获取转换后的数据

Delay（20）；

p++；

｝

p＝0；

for（i＝0；i（Length；i++）｛

Uart \_Printf（“％d\n”，buffer\[p\]）；

P++；

｝

P＝0；

for（i＝0；i（ Length；i++）｛

buffer \[p\]＝AD2Y（buffer\[p\]）；

P++；

｝

P = 0；

for（i＝0; i＜Length；i++）｛

Uart \_Printf（"量化后：％d\n"，buffer\[p\]）；

P++；

｝

ShowWavebuffer（buffer）; \/\/在LCD上显示A\/D转换后的波形

Delay（1000）；

｝

｝



