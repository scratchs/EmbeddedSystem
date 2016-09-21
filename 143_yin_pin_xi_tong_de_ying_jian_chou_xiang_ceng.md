# 14.3 音频系统的硬件抽象层

* Audio硬件抽象层的接口
* 实现Audio硬件抽象层
* Audio策略管理的内容
* 上层的情况和注意事项

14.3.1 Audio硬件抽象层的接口

Audio的硬件抽象层是AudioFlinger和Audio硬件之间的层次，在各个系统的移植过程中可以有不同的实现方式。

Audio硬件抽象层的接口路径为:
 hardware/libhardware_legacy/include/hardware目录中的Audio HardwareInterface.h文件。

在AudioHardwareInterface.h中定义了类：AudioStreamOut、AudioStreamIn和AudioHardwareInterface，分别用于输出、输入和管理。

14.3.2 实现Audio硬件抽象层

在Android 2.3.4中Audio HAL Library的相关文件主要分布在如下的目录。

hardware/libhardware_legacy/include/hardware_legacy：部分头文件。

hardware/alsa_sound：Audio HAL Library的部分头文件和实现文件（.cpp文件）。

在Android 4.x中incude/hardware_legacy/audio目录的头文件仍然存在，只是alsa_sound目录的相应实现文件已经被hardware/libhardware_legacy/audio目录中的文件取代。

## 14.3.3 Audio策略管理的内容

Audio的策略管理是Android 2.1新引入的内容，它是一个辅助Audio系统的功能模块。

Audio的策略管理层的接口由hardware/lihardware_legacy/include/hardware目录中的AudioPolicyInterface.h文件定义。

AudioPolicyInterface类由一些抽象接口来实现，这个类主要包含了基本配置、路径设置、音量设置几个方面的功能。AudioPolicyInterface的大部分接口是按照设置——获取（set和get）成对的关系。

之所以引入AudioPolicyInterface，主要目的是分离Audio系统的核心部分和辅助性功能部分。例如：由于Audio系统和电话系统有关系，因此setPhoneState()接口是相关电话的设置部分；Audio系统可以有多个的输出和输入，setForceUse()接口的功能强行设置输入输出的通道。

实现AudioPolicyInterface需要结合自身系统硬件的特点来实现，不仅涉及Audio系统本身，还涉及蓝牙系统（有关A2DP）、电话部分，以及系统特点的外围电路等内容。由于策略管理的接口是set和get接口成对出现的，因此在实现AudioPolicyInterface的时候，通常使用预设值管理的方式，在这种情况下，AudioPolicyInterface本身只是一些设置量的管理者，本身和实际的功能无关。

## 14.3.4上层的情况和注意事项

Android的Audio子系统对上层提供的接口也分为如下两个层次。

* 本地API：作为libmedia的Audio部分向本地部分提供的接口
* Java API：作为android.media包中的各个类向Java提供的接口

本地API和Java API都提供了控制类和数据类的接口。实际上，由于Java层次的API如果进行数据流的操作，效率是比较低的。因此，虽然Java层也有数据流的操作接口，但是通常不在Java层次进行数据流操作。

 Android系统调试包含了数据流和控制两个方面的内容，可以编写简单的程序直接在Java中调用Audio类。

简单的调试方式是，使用Android中的音频播放器和录音机进行调试，它们可以分别处理Audio的输出流和输入流。

在Audio的调试方面，一个关键的方法就是分离编解码和输入输出环节。Audio的硬件抽象层就是构建在Audio输入输出驱动程序之上的部分。其中，Audio的驱动程序一般比较容易进行单独的测试，对于某些Audio的驱动程序，可以通过读、写设备节点直接获取Audio的数据流。
