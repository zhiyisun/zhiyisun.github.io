---
layout: post
Linux kernel development with Qemu
---

To study Linux kernel, I need to setup a development environment to run/debug modified kernel. Two VM solutions are considered. One is [KVM](http://www.linux-kvm.org/page/Main_Page). And another is [QEMU](http://wiki.qemu.org/Main_Page). 

> [KVM](http://www.linux-kvm.org/page/Main_Page) (Kernel Virtual
> Machine) is a Linux kernel module that allows a user space program to
> utilize the hardware virtualization features of various processors.
> Today, it supports recent Intel and AMD processors (x86 and x86_64),
> PPC 440, PPC 970, S/390, ARM (Cortex A15), and MIPS32 processors.
> 
> [QEMU](http://wiki.qemu.org/Main_Page) can make use of KVM when
> running a target architecture that is the same as the host
> architecture. For instance, when running qemu-system-x86 on an x86
> compatible processor, you can take advantage of the KVM acceleration -
> giving you benefit for your host and your guest system.

My requirement is to debug ARM Linux on X86 machine. So I choose [QEMU](http://wiki.qemu.org/Main_Page). 

Prerequisites
--------------
 - Linux source code
	This could be download from git repository of kernel.org
 - Linaro GNU/Linux toolchain for ARM
    > apt-get install gcc-arm-linux-gnueabi
 - cpio, an utility to create the Linux filesystem image
    > apt-get install cpio

 - QEMU, qemu-system-arm
>apt-get install qemu-system-arm
 - etc
	like native gcc for x86 and other tools needed for kernel compilation

Target board selection
----------------------
Kernel source code package includes some default config file for each arch. They are located in **arch/arm/configs**.

[QEMU](http://wiki.qemu.org/Main_Page) also has some machines supported by default. Type "**qemu-system-arm -M help**" to show the supported machines list.

According to Balau's blog, I choose **ARM Versatile Express for Cortex-A9** as my target board.

Linux Kernel:
Below environment variables should be set before compile kernel. 

    export ARCH=arm
    export CROSS_COMPILE=arm-linux-gnueabi-

The config file for ARM Versatile Express for Cortex-A9 board is vexpress_defconfig.

    make vexpress_defconfig

Then build the kernel:

    make -j 4 all

The generated zImage is located in **arch/arm/boot/zImage**.

Hello World
-----------
Write a simple c program "hello world". Compile it with below command.

    arm-linux-gnueabi-gcc -static init.c -o init

Then create ramdisk which includes this hello world.

    echo init|cpio -o --format=newc > initramfs
Run kernel and hello world
-----------
**-M** vexpress-a9 means using Versatile Express board
**-kernel** specify the kernel image
**-initrd** specify the ramdisk
**-serial stdio** QEMU can redirect the serial port of the emulated system on the host terminal
**"console=ttyAMA0"** Linux can display its messages on the first serial port by passing "console=ttyAMA0" as a kernel parameter
**-curses** Normally, QEMU uses SDL to display the VGA output. With this option, QEMU can display the VGA output when in text mode using a curses/ncurses interface. Nothing is displayed in graphical mode

    qemu-system-arm -M vexpress-a9 -kernel workspace/code/linux/arch/arm/boot/zImage -initrd workspace/code/hello_world/initramfs -serial stdio -append "console=ttyAMA0" -curses


If you need GUI, you can execute below command.

    qemu-system-arm -M vexpress-a9 -kernel workspace/code/linux/arch/arm/boot/zImage -initrd workspace/code/hello_world/initramfs -append "console=tty1"

Done!

Reference
-----------

 1. https://balau82.wordpress.com/2012/03/31/compile-linux-kernel-3-2-for-arm-and-emulate-with-qemu/
 2. http://saurorja.org/2011/07/04/creating-a-minimal-kernel-development-setup-using-kvmqemu/
 3. http://blog.vmsplice.net/2011/02/near-instant-kernel-development-cycle.html
 4. https://blog.nelhage.com/2013/12/lightweight-linux-kernel-development-with-kvm/
 5. http://ytliu.info/blog/2014/10/22/debugging-linux-kernel-using-gdb-and-qemu/
