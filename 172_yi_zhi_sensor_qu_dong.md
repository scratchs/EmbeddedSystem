# 17.2 移植Sensor驱动



Android传感器系统自传感器硬件抽象层接口以下的部分是非标准的，因此传感器系统移植包括传感器的驱动程序和硬件抽象层。

Sensor的硬件抽象层被Sensor的JNI（SensorManager）调用，Sensor的JNI被Java的程序调用。因此传感器系统实现的核心是硬件抽象层，Sensor的HAL必须满足硬件抽象层的接口。

传感器的硬件抽象层使用了Android中标准的硬件模块的接口，这是一种纯C语言的接口，基本依靠填充函数指针来实现。

Android中Sensor的驱动程序是非标准的，只是为了满足硬件抽象层的需要。

## 17.2.1 移植驱动驱动

从Linux操作系统的角度，Sensor的驱动程序没有公认的标准定义。因此在Android中构建的Sensor驱动程序也没有标准，属于非标准的Linux驱动程序。我们编写的Sensor驱动程序的目的是从硬件中获取传感器的信息，并通过接口将这些信息传递给上层，我们可以通过如下接口来实现Sensor驱动程序。

* 使用Event设备：因为传感器本身就是一种获取信息的工具，所以使用Event设备非常自然。通过使用Event设备，可以实现用于阻塞poll调用，在中断到来的时候将poll解除阻塞，然后通过read调用将数据传递给用户空间，当使用Event设备时，可以使用input驱动框架中定义的数据类型。
* 使用Misc杂项字符设备：和使用Event设备方式类似，可以直接通过file_operations中的read、poll和ioctl接口来实现对应的功能。
* 实现一个字符设备的主设备：和上面的使用Misc杂项字符设备方式相同。
* 使用Sys文件系统：可以实现基本的读、写功能，对应驱动中的show和store接口实现。虽然使用Sys文件系统可以实现阻塞，但是通常不这样做。

## 17.2.2 移植硬件抽象层


Sensor传感器系统HAL层的实现文件目录是“hardware/libhardware/include/hardware/”。我们先看看其中的Android.mk文件，其代码如下所示。

```LOCAL_PATH:= $(call my-dir)
include $(CLEAR_VARS)
LOCAL_SHARED_LIBRARIES := libcutils
LOCAL_INCLUDES += $(LOCAL_PATH)
LOCAL_CFLAGS  += -DQEMU_HARDWARE
QEMU_HARDWARE := true
LOCAL_SHARED_LIBRARIES += libdl
LOCAL_SRC_FILES += hardware.c
LOCAL_MODULE:= libhardware
include $(BUILD_SHARED_LIBRARY)```


在此需要注意对LOCAL_MODULE的赋值，这里的模块名字都是定义好了的，具体可以参考文件“hardware/libhardware/hardware.c”。



在文件sensors.h中实现了Sensor传感器系统硬件层的接口，这是一个标准的Android硬件模块。其中，SENSOR_TYPE_*等常量表示各种传感器的类型。
Sensor模块sensors_module_t的定义如下所示：

```struct sensors_module_t {
    struct hw_module_t common;```

    
     // Enumerate all available sensors. The list is returned in "list".
    // @return number of sensors in the list
    int (*get_sensors_list)(struct sensors_module_t* module,

    struct sensor_t const** list);
};


## 17.2.3 实现上层部分

传感器部分的上层包括了以下内容：

* 传感器的JNI部分和传感器的Java框架。
* 在Java Framework中对传感器部分的调用。
* 在应用程序中对传感器部分的调用。
* 实现传感器的JNI部分和Java框架部分。

Android中Sensor传感器系统的JNI部分的实现文件是“frameworks/base/core/jni/android_hardware_SensorManager.cpp”，它提供了对类android.hardware.Sensor.Manage的本地支持。此文件是Sensor的Java部分和硬件抽象层的接口。这部分内容是Sensor的Java部分和硬件抽象层接口，Sensor的JNI部分直接调用硬件抽象层，需要包含本地的头文件“hardware/sensors.h”。实际上，Java层得到的Sensor数据，是在这里获得并且赋值的。文件com_android_server_SensorService.cpp和android_hardware_SensorManager.cpp联合使用，通过文件“android\hardware\libhardware\hardware.c”与sensor.so实现通信。

* 在Java Framework中调用传感器的部分

Sensor传感器系统的Java部分在“frameworks/base/include/core/java/android/hardware/”目录中定义，包含了以下几个文件。

SensorManager.java：实现传感器系统核心的管理类SensorManager

Sensor.java：单一传感器的描述性文件Sensor，此类是通过    SensorManager实现的。类Sensor的初始化工作是在SensorManager JNI代码中实现的，在SensorManager.java中维护了一个Sensor列表。

SensorEvent.java：实现传感器系统的事件类SensorEvent。

SensorEventListener.java：传感器事件的监听者SensorEventListener接口。

其中SensorManager、Sensor和SensorEvent是3个类，SensorEventListener和SensorListener是2个接口。这几个文件都是Android平台API的接口。

* 在应用程序中调用传感器

在Java应用层中可以调用SensorManager，通常通过SensorEventListener注册回调函数的方式实现对Sensor系统的调试。
