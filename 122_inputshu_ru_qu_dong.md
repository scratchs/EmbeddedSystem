# 12.2 Input输入驱动

Input输入驱入程序是Linux输入设备的驱动程序，可以进一步分成游戏杆（joystick）、鼠标（mouse和mice）和事件设备三种驱动程序。其中事件驱动程序是目前通用的驱动程序，可以支持键盘、鼠标、触摸屏等多种输入设备。

Input驱动程序的主设备号是13，每一种Input设备占用5位，因此每种设备包含的个数是32个。Event设备在用户空间使用如下三种文件系统来操作接口。
* Read：用于读取输入信息。
* Ioctl：用于获得和设置信息。
* Poll：调用可以进行用户空间的阻塞，当内核有按键等中断，通过在中断中唤醒poll的内核实现，这样在用户空间的poll调用也可以返回。
* 
Event设备在文件系统中的设备节点为/dev/input/eventX目录。主设备号为13，次设备号按照递增顺序生成，为64~95，各个具体的设备保存在misc、touchscreen和keyboard等目录中。

Android输入设备驱动程序的头文件是include/linux/input.h，核心文件是drivers/input/input.c，Event部分的代码文件是drivers/input/evdev.c。

##12.2.1文件input.h

在手机系统中使用的键盘（keyboard）和小键盘（kaypad）属于按键设备EV_KEY，轨迹球属于相对设备EV_REL，触摸屏属于绝对设备EV_ABS。

input.h中定义了struct input_dev结构，它表示你Input驱动程序的各种信息，对于Event设备分为同步设备、键盘、相对设备（鼠标）、绝对设备（触摸屏）等。

input_dev中定义并归纳了各种设备的信息，例如按键、相对设备、绝对设备、杂项设备、LED、声音设备、强制反馈设备、开关设备等。

在具体实现Event驱动程序时，如果得到按键的事件，通常需要通过以下的接口向上进行通知。

##12.2.2 文件KeycodeLabels.h


触摸屏和轨迹球上报的是坐标、按下、抬起等信息，信息量比较少。按键处理的过程稍微复杂，从驱动程序到Android的Java层受到的信息，键表示方式经过了两次转化。

键扫描码Scancode是由Linux的Input驱动框架定义的整数类型。键扫描码Scancode经过一次转化后，形成按键的标签KeycodeLabel，是一个字符串的表示形式。按键的标签KeycodeLabel经过转换后，再次形成整数型的按键码keycode。在Android应用程序层，主要使用按键码keycode来区分。

在文件KeycodeLabels.h中，按键码整数值的格式，在此文件中是通过枚举实现的。进而定义了数组KEYCODES[]，功能是存储从字符串到整数的映射关系。左列内容即表示按键标签KeyCodeLabel，右列的内容为按键码KeyCode（与KeyCode的数值对应）。在按键信息第二次转化的时候，已经将字符串类型KeyCodeLabel转换成整数的KeyCode。

##12.2.3 文件KeyCharacterMap.h


文件frameworks/base/include/ui/KeyCharacterMap.h也是本地框架层libui的头文件，在其中定义了按键的字符映射关系。其实KeyCharacterMap只是一个辅助的功能，因为按键码只是一个与UI无关的证书，通常用程序对其进行捕获处理。如果将按键事件转换为用户可见的内容，需要经过这个层次的转换。

##12.2.3 文件KeyCharacterMap.h


KeyCharacterMap需要从本地层传送到Java层，JNI的代码路径如下所示：
frameworks/base/core/jni/android_text_KeyCharacterMap.cpp

KeyCharacterMap Java框架层的代码如下：
* frameworks/base/core/Java/android/view/KeyCharacterMap.Java
* android.view.KeyCharacterMap类是Android平台的API，可以在应用程序中使用这个类。
* android.text.method中有各种Linstener，相互之间可以监听KeyCharacterMap相关的信息。