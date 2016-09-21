# 12.1 用户输入子系统
Android中，用户输入系统的结构比较简单，实现输入功能的硬件设备包括键盘、触摸屏和轨迹球等。在Android的上层应用中，可以通过获得这些设备产生的事件，并对设备的事件做出响应。在Java框架和应用程序层，通常使用运动事件来获得触摸屏和轨迹球等设备的信息，用按键事件获得各种键盘的信息。

## 12.1.1 Android输入系统的结构
Android输入系统的结构比较简单，从下到上包含了驱动程序、本地库处理部分、Java类对输入事件的处理、对Java的接口等。Android用户输入系统的结构如图所示。

从下到上，Android的用户输入系统分成以下几个部分：

驱动程序：保存在/dev/input目录中，通常是Event类型的驱动程序。

EventHub：本地框架层的EventHub是libui中的一部分，它实现了对驱动程序的控制，并从中获得信息。

KeyLayout（按键布局）和KeyCharacterMap（按键字符映射）文件。同时，libui中有相应的代码对其操作。定义按键布局和按键字符映射需要运行时配置文件的支持，它们的后缀名分别为kl和kcm。

Java框架层的处理：在Java框架层具有KeyInputDevice等类用于处理由EventHub传送上来的信息，通常信息由数据结构RawInputEvent和KeyEvent表示。在通常情况下，对于按键事件，则直接使用KeyEvent来传送给应用程序层，对于触摸屏和轨迹球等事件，则由RawInputEvent经过转换后，形成MotionEvent事件传送给应用程序层。

Android应用程序层：通过重新实现onTouchEvent和onTrackballEvent等函数来接收运动事件（MotionEvent），通过重新实现onKeyDown和onKeyUp等函数来接收按键事件（KeyEvent）。这些类包含在android.view包中。


## 12.1.2 移植工作

在移植Android输入系统时需要完成下面的两个工作：

移植输入（input）驱动程序。

在用户空间中动态配置“kl”和“kcm”文件。

因为Android输入系统的硬件抽象层是libui库中的EventHub，此部分是系统的标准部分。所以在实现特定硬件平台的Android系统的时候，通常改变输入系统硬件抽象层。
        
 EventHub使用Linux标准的输入设备作为输入设备，并且大多使用实用的Event设备。基于上述原因，为了实现Android系统的输入，必须使用Linux标准输入驱动程序作为标准的输入。
        
由此可见，输入系统的标准化程度较高，在用户空间实现时一般不需要更改代码。唯一的变化时使用不同的“ki”和“kcm”文件，使用按键的布局和按键字符映射关系。
        