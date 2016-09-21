# 12.2 Input输入驱动

Input输入驱入程序是Linux输入设备的驱动程序，可以进一步分成游戏杆（joystick）、鼠标（mouse和mice）和事件设备三种驱动程序。其中事件驱动程序是目前通用的驱动程序，可以支持键盘、鼠标、触摸屏等多种输入设备。

Input驱动程序的主设备号是13，每一种Input设备占用5位，因此每种设备包含的个数是32个。Event设备在用户空间使用如下三种文件系统来操作接口。
* Read：用于读取输入信息。
* Ioctl：用于获得和设置信息。
* Poll：调用可以进行用户空间的阻塞，当内核有按键等中断，通过在中断中唤醒poll的内核实现，这样在用户空间的poll调用也可以返回。

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

KeyCharacterMap需要从本地层传送到Java层，JNI的代码路径如下所示：
frameworks/base/core/jni/android_text_KeyCharacterMap.cpp

KeyCharacterMap Java框架层的代码如下：
* frameworks/base/core/Java/android/view/KeyCharacterMap.Java
* android.view.KeyCharacterMap类是Android平台的API，可以在应用程序中使用这个类。
* android.text.method中有各种Linstener，相互之间可以监听KeyCharacterMap相关的信息。
*

##12.2.4  kl格式文件


Android默认提供的按键布局文件主要包括qwerty.kl和AVRCP.kl。其中qwerty.kl是全键盘布局文件，是系统中主要按键使用的布局文件。AVRCP.kl用于多媒体的控制，ACRCP的含义为Audio/Video Remote Control Profile。

在按键布局文件中，第1列为按键的扫描码，是一个整数值；第2列为按键的标签，是一个字符串。即完成了按键信息的第一次转换，将整形的扫描码，转换成字符串类型的按键标签。第3列表示按键的Flag，带有WAKE字符，表示此按键可以唤醒系统。

扫描码来自驱动程序，显然不同的扫描码可以对应一个按键标签。表示物理上的两个按键可以对应同一个功能按键。

##12.2.5   kcm格式文件


kcm格式文件是按键字符映射文件，用于表示按键字符的映射关系，功能是将整数类型按键码（keycode）转化成可以显示的字符。kcm文件将被makekcharmap工具转化成二进制的格式，放在目标系统的/system/usr/keychars/目录中。

除了QWERTY映射类型，还可以映射Q14（单键多字符对应的键盘）和NUMERIC（12键的数字键盘）。

kcm文件将被makekcharmap工具转化成二进制的格式，放在目标系统的/system/usr/keychars/目录中。

##12.2.6 文件EventHub.cpp


文件EventHub.cpp位于libhui库中的frameworks/base/libs/ui目录下，此文件是输入系统的核心控制文件，整个输入系统的主要功能都是在此文件中实现的。例如当按下电源键后，系统把scanCode写入对应的设备节点，文件EventHub.cpp会读这个设备节点，并把scanCode通过kl文件对应成keyCode发送到上层。
 
在具体处理过程时，在函数openPlatformInput()中通过调用scan_dir()函数搜索路径下面所有Input驱动的设备节点，函数scan_dir()会从目录中查找设备，找到后调用open_device()函数以打开查找到的设备。

EventHub的getEvents()函数负责处理中完成，处理过程是在一个无限循环之内，调用阻塞的函数等待事件到来。

poll()函数将会阻塞程序的运行，此时为等待状态，无开销，直到Input设备的相应事件发生，事件发生后poll()将返回，然后通过read()函数读取Input设备发生的事件代码。

注意，EventHub默认情况可以在/dev/input之中扫描各个设备进行处理，通常情况下所有的输入设备均在这个目录中。

实际上，系统中可能有一些input设备可能不需要被Android整个系统使用，也就是说不需要经过EventHub的处理，在这种情况下可以根据EventHub中open_device()函数的处理，设置驱动程序中的一些标志，屏蔽一些设备。open_device()中处理了键盘、轨迹球和触摸屏等几种设备，对其他设备可以略过。另外一个简单的方法就是将不需要EventHub处理的设备的设备节点不放置在/dev/input之中。



