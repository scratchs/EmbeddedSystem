# 17.3 实现传感器


Android为模拟器提供了一个Sensor硬件抽象层的示例实现，它本身具有实际的功能，可以作为实际系统的传感器硬件抽象层的示例。

在Donut及以前的版本中，Sensor硬件抽象层的代码路径是“development/emulator/sensor/”，在Éclair及其以后的版本中，Sensor硬件抽象层的代码路径是“sdk/emulator/sensors”。无论是什么版本，在上述目录下都会包含一个Android.mk文件和一个源文件sensors_qemu.c，经过编译会后会形成一个单独的模块，即动态库sensors.goldfish.so（中间的goldfish表示产品名）。它将被放置在标准文件系统的“system/lib/hw/”目录中，在运行时将作为一个硬件被加载。