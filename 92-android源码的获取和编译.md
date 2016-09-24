# 9.2 Android源码的获取和编译

Android的源码获取需要安装Git和Repo。

Git是Linus Torvalds 为了帮助管理Linux 内核开发而开发的一个开源的分布式版本控制系统，用以敏捷高效的处理任何或大或小的项目。Git的读音为\/gɪt\/。尽管最初Git的开发是为了辅助Linux内核开发的过程，但是我们发现在很多其他自由软件项目或公司项目管理也使用了Git。

Linux下软件的安装方法有很多种。在Ubuntu下我们可以简单实用如下命令直接安装。

> $ sudo apt-get install git
> 
> $ sudo su
> 
> $ curl [http:\/\/commondatastorage.googleapis.com\/git-repo-downloads\/repo](http://commondatastorage.googleapis.com/git-repo-downloads/repo) &gt; ~\/bin\/repo
> 
> $ cd ~\/bin\/
> 
> $ chmod a+x repo

## 9.2.1 获取Android源码

### 1. repo init创建仓库

初始化android源代码仓库，下载最新的代码，使用如下方法：

> $ mkdir myAndroid
> 
> $ cd myAndroid
> 
> $ repo init -u [https:\/\/android.googlesource.com\/platform\/manifest](https://android.googlesource.com/platform/manifest)

或者想下载某个分支，例如android-4.4.2\_r1，可以使用如下命令：

> $ repo init -u [https:\/\/android.googlesource.com\/platform\/manifest](https://android.googlesource.com/platform/manifest) -b android-4.4.2\_r1

### 2. 同步源代码到本地仓库

> $ repo sync

在同步代码的过程中可能因为网络环境的原因会经常停止，我们可以编写一个脚本文件来让他中断后又自动下载。脚本参考如下：

# !\/bin\/bash

> echo “start sync……”
> 
> repo sync
> 
> while \[ $? == 1 \]; do
> 
> echo “sync interrupted! start again……”
> 
> sleep 5
> 
> repo sync
> 
> done

用这个脚本文件代替repo sync，当网络连接不上的而终止的时候，终端会延迟5毫秒后自动运行repo sync命令。具体操作如下：

> $ sudo su
> 
> $ cd myAndroid
> 
> $ vim AutoLoad.sh\(创建一个名为AutoLoad.sh的脚本文件，然后将上述的内容复制进去，保存退出\)
> 
> $ chmod a+x AutoLoad.sh
> 
> $ .\/AutoLoad.sh

## 9.2.2获取Android Linux Kernel源码

Android内核的下载相对简单，只需要使用git clone命令即可：

> $ git clone git:\/\/android.git.kernel.org\/kernel\/common.git kernel
> 
> $ cd kernel
> 
> $ git branch

显示如下分支:\* android-2.6.27

说明你现在在android-2.6.27这个分支上，也是kernel\/common.git的默认主分支。

显示所有head分支：git branch –a。

* 设置环境变量（仅共参考，需根据具体平台而定）

> $ vim \/root\/.bashrc
> 
> 在末尾增加如下几行：
> 
> CROSS\_COMPILE=arm-arago-linux-gnueabi- 这是指定使用的交缠编译工具链
> 
> PATH="${PATH}:\/opt\/Tools\/linux-devkit\/bin" 将交叉编译链加入到环境变量中
> 
> export PATH CROSS\_COMPILE
> 
> $ source \/root\/.bashrc

* 设定交叉编译参数

打开kernel目录下的Makefile文件，修改ARCH和CROSS\_COMPILE两个参数，指定为arm和指定平台的交叉编译链。参考如下：

> export KBUILD\_BUILDHOST := $\(SUBARCH\)
> 
> ARCH ?= arm
> 
> CROSS\_COMPILE ?= arm-arago-linux-gnueabi-

* 设定编译选项

编译之前需要我们配置适合我们自己平台的编译选项，可以通过图形化界面来配置我们需要编译的选项。

> $ make menuconfig

执行该命令后会出现类似下面的界面，这样可以很直观的配置内核选项，正确配置、合适裁剪，编译生成我们需要的内核。

![](/assets/9.2.2.png)

配置完成后编译信息都保存到了kerne目录下的.config文件中，同样我们也可以修改该文件。

同样内核源码中也为我们准备了一些平台的的配置文件，位于kernel\/arch\/arm\/configs\/目录下，如果有你需要支持的平台则可以直接通过make命令配置。

![](/assets/9.2.2-2.png)

* 编译内核映像

以am335x平台为例，我要编译内核只需要执行以下操作:

> $ cd myAndroid\/kernel
> 
> $ make am335x\_evm\_defconfig
> 
> $ make（使用make uIamge可以生成uImage映像）

* 测试生成的内核映像

> $ emulator -avd myavd -kernel ~\/android\/kernel\/arch\/arm\/boot\/zImage
> 
> 参考：[http:\/\/source.android.com\/source\/downloading.html](http://source.android.com/source/downloading.html)

## 9.2.3 编译Android

进入到源代码根目录直接make即可，同时可以通过TARGET\_PRODUCT指定你要编译的目标，默认情况是编译生成out\/target\/product\/generic\/。编译过程需要很久，与机器的CPU性能和内存有关。例如我要编译生成可运行于am335x平台的镜像，那么可以输入以下编译命令：

> $ make TARGET\_PRODUCT=am335xevm

编译完成之后会生成的am335x平台的文件会在out\/target\/product\/am335x\/目录下。

编译完成之后可以在模拟器上运行编译好的android，emulator在myAndroid\/out\/host\/linux-x86\/bin下，生成的ramdisk.img、system.img和userdata.img在myAndroid\/out\/target\/product\/generic下。其中ramdisk 是比较小的根文件系统，其他镜像分别挂载到根文件系统相应的目录下，\/system和\/data。为了方便，我们可以将其加入到环境变量中，在\/root\/.bashrc中新增环境变量，如下：

> export ANDROID\_PRODUCT\_OUT=~\/android\/out\/target\/product\/generic
> 
> ANDROID\_PRODUCT\_OUT\_BIN=~\/android\/out\/host\/linux-x86\/bin
> 
> export PATH=${PATH}:${ANDROID\_PRODUCT\_OUT\_BIN}:${ANDROID\_PRODUCT\_OUT};

将该变量应用到系统中，运行我们编译好的android，如果最后进入android界面就说明编译成功。

> $ source \/root\/.bashrc
> 
> $ cd ~\/myAndroid\/out\/target\/product\/generic
> 
> $ emulator -system system.img -data userdata.img -ramdisk ramdisk.img

如果需要单独编译模块，这需要在源码根目录下执行$ source build\/envsetup.sh，就会多出如下命令：

> * croot: Changes directory to the top of the tree.
> 
> * m: Makes from the top of the tree.
> 
> * mm: Builds all of the modules in the current directory.
> 
> * mmm: Builds all of the modules in the supplied directories.
> 
> * cgrep: Greps on all local C\/C++ files.
> 
> * jgrep: Greps on all local Java files.
> 
> * resgrep: Greps on all local res\/\*.xml files.
> 
> * godir: Go to the directory containing a file.

