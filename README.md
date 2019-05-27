Ubuntu 18.04 Bionic-Builder

Bionic-Builder是创建Hikey970的Ubuntu镜像的All-In-One脚本。此脚本是一个交互式的脚本，借助Debootstrap，我们可以构建一个完整的Ubuntu 18.04的镜像。此脚本会自动设置必要的系统配置，因此无需手动编辑任何文件。此脚本可以执行以下操作。

rootft下载地址:https://pan.baidu.com/s/1G8BaVutcc9taXdSfFYBBtg 提取码: ag5x

	A)  通过apt下载基本的Ubuntu 18.04所需的包
	
	B)  下载并安装交叉编译内核所需的ARM64工具链：gcc-arm-8.3-2019.03-x86_64-aarch64-linux-gnu （https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-a/downloads）

	C)  下载内核源代码。此内核源码是专门为Debain或Ubuntu配置的 （https://github.com/Bigcountry907/linux/tree/hikey970-v4.9-Debain-Working）

	D)  编译内核并安装内核设备树和模块。所有的一切都被安装在Ubuntu Bionic镜像中，无需任何手动配置

	E)  考虑到使用WIFI连接Hikey 970开发板的情况并不多，此版本移除了WIFI配置

	F)  集成了一个叫做First-Boot初始化脚本。初始化脚本将完成Ubuntu 18安装：

        1. 自动更新和升级系统
        2. 安装tasksel并运行tasksel install standard来将标准软件包添加到系统中
        3. 如果你愿意，该脚本将自动安装XFCE4 / Xubuntu Desktop环境
        4. 如果你选择不安装桌面，那么Ubuntu将作为Ubuntu Server运行 (推荐)

    G)  修复了HDMI无法正常显示的BUG, console可以通过HDMI正常显示

	H)　如果其他的功能需要被加入，欢迎提出issue或者pull request
				
				
Bionic-Builder的使用说明

注意：此脚本需要在Ubuntu或Debain系统上运行。UBUNTU 16.04在构建rootfs时会遇到与MAN-DB有关的问题而卡住，所以选项1不适用于16.04。但是我们仍可以在Ubuntu 16.04上编译和安装内核。并且，下载apt的包需要耗费一定的时间，你可以直接从百度云盘下载构建好的rootfs，并解压缩到此脚本的目录下。这样，你只需要从选项2开始，就能构建你自己的内核了。
我建议使用Ubuntu 18.04创建Bionic rootfs。我将添加一个预先下载的rootfs来解决ubuntu 16.04问题。

在此脚本里，内核是交叉编译的，因此你无法在Hikey970上运行此脚本（在Hikey970上编译内核大约需要一辈子，人生苦短，我用工作站）。你必须使用服务器或本地的Ubuntu / Debain系统。


1) 安装所需的软件包.

		A) sudo apt-get install -y ccache python-pip build-essential kernel-package fakeroot libncurses5-dev 
		   sudo apt-get install -y libssl-dev gcc  git-core gnupg 
		   sudo apt-get install -y binfmt-support qemu qemu-user-static debootstrap simg2img
		
        B) 摇身一变成为超级用户：
        　　sudo -s

		C) 执行脚本
            ./BB.sh

					Welcome to the Bionic-Builder Main Menu

					(1) CREATE MINIMAL BASE ROOT FILESYSTEM

 					(2) BUILD KERNEL LINUX v4.9.78

					(3) COPY KERNEL & DEVICE TREE / INSTALL KERNEL MODULES in ROOTFS

 					(4) GENERATE FLASHABLE AND COMPRESSED IMAGES

 					(5) UPDATE BOTH GRUB.CFG FILES ( BOARD AND BOOT IMAGE )



BIONIC-BUILDER菜单选项

（1）创建MINIMAL BASE ROOT FILESYSTEM。如果你只想编译自己的内核，那么你可以跳过选项1，直接从选项2开始执行。


    A) 第一个选项使用debootstrap来下载基本根文件系统。rootfs将位于 主目录/Ubuntu-SRC/build/rootfs。

        当你选择选项1时，将创建基本rootfs，你将在创建期间创建提示输入USERNAME和PASSWORD以登录Hikey 970。如果你使用我预先创建好的rootfs，那么用户名和密码都将是: zrji

    B) 原作者添加了3个apt的包镜像，个人推荐第一个镜像。你也可以指定自己的镜像，但是arm64的镜像是如此的稀有，而且大部分都在中国大陆境内（上海交通大学好像有一个镜像），从我这里访问是如此的慢。
    
    C) 默认语言将设置为英语。 此时，你已经拥有了一个完整的rootfs，你永远不需要再次运行选项1，除非你想建立一个新的rootfs或者消磨时间。

    D) 你可以通过执行 chroot 主目录/Ubuntu-SRC/build/rootfs 来切换你的运行系统的root到 主目录/Ubuntu-SRC/build/rootfs。在此之后，你可以执行添加用户和安装包之类的操作。请记住，~~西瓜越大，西瓜皮就越大~~在此阶段添加的包越多，system.img就越大。

    E) 纵使你此时无比的兴奋，你还不能创建img然后flash到你的Hikey970。你还需要编译内核，设备树以及讲modules安装到kernel中。但是不必担心，选项２和选项３将帮助你完成这些步骤。


(2)  编译 LINUX v4.9.78

    A）此选项将自动下载ARM64交叉编译工具链和内核源代码。此两者只会被下载一次。

        内核 REPO：　https://github.com/Bigcountry907/linux.git -b hikey970-v4.9-Debain-Working


    B）内核构建菜单

        选择选项B将自动执行所有内核构建操作。

        选择选项C将运行make menuconfig并允许你更改配置。

        运行选项B或C后选择选项O将运行make oldconfig

    C）内核编译完成后，你可以在　主目录/Install/kernel-install/找到内核：Image-hikey970-v4.9.gz　和设备树：kirin970-hikey970.dtb



(3)  复制内核和设备树并安装内核modules到rootfs中

    A) 在ROOTFS中复制内核和设备树/安装内核模块

        此选项将复制Image-hikey970-v4.9.gz和kirin970-hikey970.dtb到rootfs/boot中。

        内核模块也将自动安装到相应的目录中：rootfs/lib/modules/$ {uname -r}



(４)  生成fastboot flash所需要的镜像文件

    A）运行选项＃1选项＃2和选项＃3后，你就可以利用选项4来创建了用于fastboot flash的镜像。

    Ｂ）你可以使用选项2修改和重新编译内核。然后使用选项3安装新内核。在之后，你可以使用选项4来创建新的img。无需再次运行选项1。
