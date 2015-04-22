---
layout: post
title : How to use gdb to debug ARM kernel in Qemu
---
In host Linux shell, run below command to start gdb first.

    [zsun@ubuntu:~]↥ % gdb-multiarch ~/workspace/code/linux/vmlinux
    GNU gdb (Ubuntu 7.8-1ubuntu4) 7.8.0.20141001-cvs
    Copyright (C) 2014 Free Software Foundation, Inc.
    License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
    This is free software: you are free to change and redistribute it.
    There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
    and "show warranty" for details.
    This GDB was configured as "x86_64-linux-gnu".
    Type "show configuration" for configuration details.
    For bug reporting instructions, please see:
    <http://www.gnu.org/software/gdb/bugs/>.
    Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.
    For help, type "help".
    Type "apropos word" to search for commands related to "word"...
    Reading symbols from /home/zsun/workspace/code/linux/vmlinux...done.
    (gdb) set architecture arm
    The target architecture is assumed to be arm
    (gdb) target remote localhost:1234
    Remote debugging using localhost:1234


Start Qemu. Please note that "-s" is used for debugging.

    qemu-system-arm -M vexpress-a9 -kernel zImage -append "console=tty1 root=/dev/nfs nfsroot=10.0.2.2:/busybox/_install rw ip=dhcp init=/sbin/init" -s

Then you can debug your kernel.