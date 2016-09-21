# 16.3 移植和调试

* 移植的内容
* 驱动程序
* 硬件抽象层
* 上层应用

## 16.3.1 移植的内容

在Android平台中，需要移植的GPS内容位于JNI层下面，即驱动层和硬件抽象层。GPS硬件的驱动程序通常是串口驱动程序，比较容易实现，在Linux系统的用户空间中通常用ttf设备来表示。

GPS的硬件抽象层实现比较复杂，也是Android系统中使用GPS和其他系统的主要区别所在。Android给出了简单的参考实现gps_qemu.c，基本结构可以重新使用，但是需要再次基础上做大量的修改工作和升级工作之后才能使用。

## 16.3.2 驱动程序

GPS一般分为软和硬两种。其中硬GPS直接输出卫星数据，需要应用处理器进行解析计算，再转换成标准的NMEA（National Marine Electronics Association，国际海洋电子协会）数据。而软GPS通常需要主控芯片控制其运行状态，输出的大多是裸卫星数据。需要主控方进行计算，才能得到最终的NMEA数据。两者的共同点是最终输出都是NMEA数据。

GPS的硬件接口相对简单，除开供电，reset等控制，一般仅通过串口和应用处理器连接。因此驱动程序通常是标准的串口驱动。部分GPS只需上电，即可开始找到卫星，并返回携带信息的NMEA数据。找到足够的卫星之后，导航即可开始。也有一部分GPS上电后，允许写入命令，配置信息（包括星历等信息加速定位速度），乃至固件。对于这类GPS，需要根据其要求，在硬件抽象层中实现对应的操作。

将GPS连接上应用处理器的某个串口之后，通常表现为设备节点/dev/ttySx，x表示其对应编号。可以打开该节点进行写操作，对GPS写入数据或命令。而读操作获取到的，一般就是标准的NMEA数据。

## 16.3.3 硬件抽象层
GPS硬件抽象层的主要入口如下所示。

```const GpsInterface* gps_get_hardware_interface();//返回实际硬件的实现接口
const GpsInterface* gps_get_qemu_interface();		//返回模拟器的实现接口
const GpsInterface* gps_get_interface();			//获取GpsInterface接口```


实现硬件抽象层，首先是主入口的实现。Android提供了gps.cpp的标准实现，用于通过宏开关模拟器实现和硬件实现。因此对普通的硬件实现来说，实现gps_get_hardware_interface，返回定义的GpsInterface接口即可。

对于GPS，首先需要初始化，然后对于GPS数据，建立一套“获取——解析——上报”的机制。

首先在GpsInterface.init的实现中，完成对GPS的初始化。Init会在GPS被打开的时候调用，该步骤需要完成GPS内核模块的加载工作，部分GPS还需要进行firmware下载工作。之后打开GPS端口，确认GPS硬件已经正常工作。

GPS初始化完成后，一般都会新开一个polling线程，对GPS端口进行轮询，获取输出的NMEA数据。GpsInterface.start以及stop主要控制该线程启动和停止。

解析的工作在获取数据后进行，这里的目的是将NMEA数据解析成Android卡框架可以识别的结构信息，存放到GpsLocation以及GpsSvInfo等，以便上报。对于NMEA数据的解析，已经有非常多的参考实现。Android在gps_qemu.c中也给出了大部分的参考实现。可以照搬这部分的内容。
上报过程，在解析完后进行，直接调用init时注册的callback函数，并填入获取到的数据结构即可。

## 16.3.4 上层应用

* LocationManager
* initialize()函数
* 在文件GpsLocationProvider.cpp
* 文件android_location_GpsLocationProvider.cpp
* 文件gps.cpp
* 在文件gps_qemu.c
* 构造函数GpsLocationProvider()
* 函数enable()
* 函数enableLocationTracking()
