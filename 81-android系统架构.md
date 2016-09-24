# 8.1 Android系统架构

Android将系统架构分为了4层，从上到下依次为：应用程序层（Application）、应用程序框架层（Application Framework）、系统运行库层（Libraries）以及Linux内核层（Linux Kernel）。

![](/assets/8.1.png)

* 应用程序层（Application）

应用程序层包含一个核心应用集合，如SMS短消息程序、联系人管理、E-mail客户端、浏览器、地图、同时，开发人员利用Java语言设计和编写的程序（如音乐播放器、游戏等）也都运行在这一层上。

* 应用程序框架层（Application Framework）

应用程序框架层是Android应用开发的基础，开发人员可以使用Google发布的API框架方便地进行程序开发，简化了程序开发的架构设计，但必须遵守框架的开发原则。

* 系统运行库层（Libraries）

当使用Android应用框架时，Android系统会通过一些C\/C++库来支持我们使用的各个组件，使其能更好地位我们服务

* Linux内核层

Android以Linux内核为基础，借助Linux内核提高核心系统服务，例如安全性、内存管理、进程管理、网络协议栈和驱动模型等等方面。Linux内核作为一个抽象层，存在于硬件和软件栈之间；。Android 4.0之前基于Linux 2.6系列内核，4.0及之后的版本使用更新的Linux 3.X内核，并且两个开源项目之间有了互通。Linux 3.3内核中正式包括了一些Android代码，可以让开发人员利用Linux内核运行于Android系统，为Android内核或Linux内核开发驱动程序，减少甚至彻底消除对面向Android内核开发人员发布的不同补丁进行维护的负担。Linux 3.4增添了电源管理等更多功能，以增加与Android的硬件兼容性，使Android在更多设备上得到支持。

Android在Linux内核的基础上进行了增强，增加了一些面向移动计算的特有功能。例如，低内存管理器LMK（Low Memory Killer）、匿名共享内存（Ashmem），以及轻量级的进程间通信Binder进制等。这些内核的增强使Android在继承Linux内核安全机制的同时，进一步提升了内存管理、进程间通信等方面的安全性。







