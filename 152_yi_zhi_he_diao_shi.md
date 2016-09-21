# 15.2 移植和调试

* 协议和驱动程序
* 本地实现
* JNI层
* Java FrameWork层


## 15.2.1 协议和驱动程序

在Linux中，WiFi的部分包括协议和驱动程序两个部分的内容，对用户空间提供的是网络驱动程序。

WiFi协议部分的主要接口是include/net/目录中的wireless.h。

WiFi协议部分的实现源文件在net/wireless/目录中。
在内核配置菜单中，WiFi协议的内容在“NetWorking support”>“Wireless”中选定，其中Wireless extensions和Wireless extensions sysfs files选项表示了使用ioctl或者sysfs对无线网络进行附加的控制。

WiFI的驱动程序在drivers/net/wireless/目录中。在内核配置菜单中，WiFi驱动程序配置选项为：“Device Drivers”>“Network device support”>“Wireless LAN”。WiFi驱动部分的文件比较多，包含的多个子目录是不同的WiFi设备的驱动程序。其中可能包含的内容表示了不同的无线局域网实现的驱动程序，配置通常还需要设置固件（firmware）的路径。WiFi驱动程序的形式通常是一个网络设备（net_device）。


## 15.2.2 本地实现

本地实现部分主要包括wpa_supplicant及wpa_supplicant适配层。WPA是WiFi Protected Access的缩写，中文含义为“WiFi网络安全存取”。WPA是一种基于标准的可互操作的WLAN安全性增强解决方案，可大大增强现有及未来WiFi系统的数据保护和访问控制水平。

在移植过程中，需要特别注意wpa_supplicant，这是一个标准的开源项目，已经被移植到很多平台上，我们比较关心的是wpa_supplicant在接收到上层的命令之后是怎么将命令发给DRIVER的。DRIVER在接收到命令后的解析动作，以及之后调用驱动功能函数的流程及驱动对寄存器控制的细节。由于需要注意代码保密，所以之后不会提及具体使用了哪块WiFi芯片，也不会提及此WiFi DRIVER是在什么平台什么产品。


wpa_supplicant的标准结构框图如下所示。我们重点关注框图的下半部分，即wpa_supplicant是如何与Driver进行联系的。整个过程以AP发出SCAN命令为主线，其实用NDIS也一样，整个流程相差不远。





## 15.2.3 JNI层
Android中的WiFi系统的JNI部分实现的源代码如下。
frameworks/base/core/jni/android_net_wifi_Wifi.cpp


JNI层的接口注册到Java层的源代码如下。
frameworks/base/wifi/java/android/net/wifi/WifiNative.java


WifiNative将为WifiService、WifiStateTracker、WifiMonitor等几个框架内部组件提供底层操作支持。


此处实现的本地函数都是通过调用wpa_supplicant适配层的接口来实现的（包含适配层的头文件wifi.h）。wpa_supplicant适配层是通用的wpa_supplicant的封装。在Android中作为WiFi部分的硬件抽象层来使用。wpa_supplicant适配层主要用于封装与wpa_supplicant守护进程的通信，以提供给Android框架使用。它实现了加载、控制和消息监控等功能。wpa_supplicant适配层的头文件如下所示。
hardware/libhardware_legacy/include/hardware_legacy/wifi.h


## 15.2.4 Java FrameWork层

WiFi系统的Java部分代码实现的目录如下所示。
frameworks/base/wifi/java/android/net/wifi/		//WiFi服务层的内容
frameworks/base/services/java/com/android/server	//WiFi部分的接口

WiFi系统Java层的核心是根据IWifiManager接口所创建的Binder服务器端和客户端，服务器端是WifiService，客户端是WifiManager。

编译IWifiManager.aidl生成文件IWifiManager.java，并生成IWifiManager.Stub（服务器端抽象类）和IWifiManager.Stub.Proxy（客户端代理生成类）。WifiService通过继承IWifiManager.Stub实现，而客户端通过getService()函数获取IWifiManager.Stub.Proxy（即Service的代理类），将其作为参数传递给WifiManager，供其与WifiService通信时使用。



* WifiManager是WiFi部分与外界的接口，用户通过它来访问WiFi的核心功能，WifiWatchdogService这一系统组件也是用WifiManager来执行一些具体操作。
* WifiSerivce是服务器端的实现，作为WiFi的核心，处理实际的驱动加载、扫描，链接/断开等命令，以及底层上报的事件。对于主动的命令控制，WiFi是一个简单的封装，针对来自客户端的控制命令，调用相应的WifiNative底层实现。
* （3） WifiWatchdogService是ConnectivityService所启动的服务，但它并不是通过Binder来实现的服务。它的作用是监控同一个网络内的接入点（Access Point），如果当前接入点的DNS无法Ping通，就自动切换到下一个接入点。WifiWatchdogService通过WifiManger和WifiStateTracker辅助完成具体的控制动作。在WifiWatchdogServcie初始化时，通过registerForWifiBroadcasts再注册获取网络变化的BroadcastReciver，也就是捕获WifiStateTracker所发出的通知消息，并开启一个WifiWatchdogThread线程来处理获取的消息。通过更改Setting.Secure.WIFI_WARCHDOG_ON的配置，可以开启和关闭WifiWatchdogService。

