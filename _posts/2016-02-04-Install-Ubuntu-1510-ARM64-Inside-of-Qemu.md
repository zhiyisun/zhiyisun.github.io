---
layout: post
title: Install Ubuntu 15.10 ARM64 inside of Qemu
date:  2016-02-04 19:33:42 +0800
categories: geek 
---
In previous post, I described how to run ARM64 Linux inside Qemu. But that is stock Linux. In this post, I will describe how to install Ubuntu 15.10 for ARM64 platform inside Qemu. 

There are two types of Ubuntu can be installed in Qemu. The first is to use Ubuntu Cloud image. But it needs setup ssh key to connect to it. And it is an installed image. I can play it without any customization or installation. I don't like it. I don't want a image which can be deployed on Amazon cloud. The second one is what I will describe here. It is to install Ubuntu from scratch! Just like we did on real system.

 1. Download [kernel](http://ports.ubuntu.com/ubuntu-ports/dists/wily/main/installer-arm64/current/images/netboot/ubuntu-installer/arm64/linux) and [initrd](http://ports.ubuntu.com/ubuntu-ports/dists/wily/main/installer-arm64/current/images/netboot/ubuntu-installer/arm64/initrd.gz) from [Ubuntu web site](http://ports.ubuntu.com/). Please note that these two files are only used for boot and installation. After we installed Ubuntu on Hard Disk, we don't need them anymore.
 2. Create a qcow2 image as Hard Disk. I set the size as 10G. It should be enough for light use. 

    `qemu-img create -f qcow2 ubuntu.qcow2 10G`

 3. Boot qemu by using above kernel、initrd. The qcow2 is as the HD. It takes for a while. I recommend to read some articles like this [one](http://wiki.qemu.org/Documentation/Networking).

    `qemu-system-aarch64 -machine virt -cpu cortex-a57 -nographic -smp 1 -m 512 -kernel linux -initrd initrd.gz --append "earlyprintk console=ttyAMA0 root=/dev/sda1" -netdev user,id=eth0 -device virtio-net-device,netdev=eth0 -redir tcp:2222::22 -drive if=none,file=ubuntu.qcow2,id=hd0 -device virtio-blk-device,drive=hd0`

 4. During above installation, flash boot will be failed. But it doesn't matter.
 5. After that, we get qcow2 image which Ubuntu installed on that. Then we need to retrieve Ubuntu's vmlinuz and initrd.img from it.

    `sudo modprobe nbd max_part=16
    sudo qemu-nbd -c /dev/nbd0 ubuntu.qcow2
    mkdir -p /mnt
    sudo mount /dev/nbd0p1 /mnt
    mkdir -p boot
    sudo cp -rf /mnt/* boot/
    sudo umount /mnt
    sudo qemu-nbd -d /dev/nbd0
    sudo killall qemu-nbd``
 6. Now we have all we need to boot up ARM64 based Ubuntu inside Qemu! Let run it.
    `qemu-system-aarch64 -machine virt -cpu cortex-a57 -nographic -smp 8 -m 512 -kernel vmlinuz -initrd initrd.img --append "earlyprintk console=ttyAMA0 root=/dev/vda2 init=/sbin/init" -netdev user,id=eth0 -device virtio-net-device,netdev=eth0 -redir tcp:2222::22 -drive if=none,file=ubuntu.qcow2,id=hd0 -device virtio-blk-device,drive=hd0`
