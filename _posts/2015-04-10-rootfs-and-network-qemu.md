---
layout: post
RootFS and Network of Qemu
---
After a few days struggling with rootfs, I tend to use the simplest way -- busybox. I got some fails when compiling the buildroot source code. For debugging network code, I think busybox is enough for my requirement.

 1. Download BusyBox
		BusyBox can be downloaded from its git server.

	    $ git clone git://busybox.net/busybox.git
	    $ git checkout remotes/origin/1_NN_stable

 2. Tune BusyBox Setting

	    $ make menuconfig

> 	BusyBox Settings --> Build Options -> Build BusyBox as a static
> binary (no shared libs)  	BusyBox Settings --> Build Options -> CrossCompiler prefix

 3. Other settings, like networking ......

Build BusyBox
$ make install

Update filesystem

$ cd _install
$ mkdir proc sys dev etc etc/init.d

Create _install/etc/init.d/rcS

#!/bin/sh
mount -t proc none /proc
mount -t sysfs none /sys
/sbin/mdev -s

Then set it as an executable file:
$ chmod +x _install/etc/init.d/rcS

$ cd _install
$ find . | cpio -o --format=newc > ../rootfs.img
$ cd ..
$ gzip -c rootfs.img > rootfs.img.gz

Boot Linux with initram image:

$ qemu-system-arm -M vexpress-a9 -kernel workspace/code/linux/arch/arm/boot/zImage -append "console=tty1 root=/dev/ram rdinit=/sbin/init" -initrd ~/workspace/rootfs/rootfs.img

Boot Linux with NFS filesystem and fixed guest IP address:

Install DHCP server

$ sudo service nfs-kernel-server start

Add below line in /etc/exports
/home/zsun/workspace/rootfs/busybox/_install 127.0.0.1(rw,sync,no_subtree_check,no_root_squash,insecure)

Restart NFS server
$ sudo service nfs-kernel-server restart

Then boot Linux:
$ qemu-system-arm -M vexpress-a9 -kernel workspace/code/linux/arch/arm/boot/zImage -append "console=tty1 root=/dev/nfs nfsroot=10.0.2.2:/home/zsun/workspace/rootfs/busybox/_install rw ip=dhcp init=/sbin/init"