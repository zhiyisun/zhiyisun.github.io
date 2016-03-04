---
layout: post
title: Build busybox for ARM Qemu
---
 - Download busybox

```
git clone git://busybox.net/busybox.git
```
 - Configure busybox

```
make menuconfig
```
```
Change below settings:
  Busybox Setting -> Build Options -> Build BusyBox as a static binary
  Busybox Setting -> Build Options -> Cross Compiler Prefix 
```

 - Build busybox

```
make; make install
```
 - Create necessary directory in rootfs

```
cd _install
mkdir -p proc sys dev etc etc/init.d
cat << EOF > etc/init.d/rcS
  #!/bin/sh
  mount -t proc none /proc
  mount -t sysfs none /sys
  /sbin/mdev -s
  EOF
chmod +x etc/init.d/rcS
find . | cpio -o --format=newc > ../../busybox.img
```

 - Run Qemu

```
qemu-system-arm -M vexpress-a9 -kernel zImage -initrd busybox.img -serial stdio -append "root=/dev/ram rdinit=/sbin/init console=ttyAMA0" -curses
```

    
