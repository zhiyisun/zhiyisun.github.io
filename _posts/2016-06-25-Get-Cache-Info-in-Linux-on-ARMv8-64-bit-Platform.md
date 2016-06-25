---
layout: post
title: Get Cache Info in Linux on ARMv8 64-bit Platform
---

On x86 platform, there are many ways to get cache information from a running Linux system. For example, by checking /proc/cpuinfo, or by using tools, like dmidecode, lshw, hwloc, etc. But don't forget "Everything is a file. From my opinion, the fundamental method to get whole picture of cache information is by checking sysfs. 
```
grep . /sys/devices/system/cpu/cpu*/cache/index*/*
```

These files are populated by device initcall **cache_sysfs_init(void)**. 

Now let's talk about ARM platform. 

On ARM 32-bit arch, at least some versions of ARM arch, like ARMv7A, ARMv6, do have registers(cache type register) which store cache information. But Linux doesn't support to populate cache information for those CPUs yet. 

On ARM 64-bit arch, Sudeep Holla provided a patch (commit 246246cbde5e840012f853e27630ebb59f409486) in 2014 which adds support for providing processor cache information to userspace through sysfs interface. It is based on already existing implementations(x86, ia64, s390 and powerpc). 

The basic process is device initcall **cacheinfo_sysfs_init(void)** calls **detect_cache_attributes(unsigned int cpu)** to get cache information from hardware and device tree. In this function, it calls hardware specific function **init_cache_level(unsigned int cpu)** and **populate_cache_leaves(unsigned int cpu)** to get cache information from hardware registers. In ARM 64-bit case, these registers are CCSIDR, CLIDR, CSSELR. From these registers, cache line size, number of sets, cache hierarchy can be obtained. Then it will call **cache_shared_cpu_map_setup (unsigned int cpu)** to get cache information from device tree. Because some of cache hierarchy information is out of CPU core's view. For example, which cores are shared L2 cache. These information can only be get from device tree. 

It is good to get a ARM 64-bit box and practice it on that. We can use qemu-system-aarch64 as an alternative. But you probably will say "There is no cache info in sysfs!". That's true. :-) Because there is no "next-level-cache" in machine virt's default device tree. Function cache_shared_cpu_map_setup will return error. You can see below log from dmesg.

```
Unable to detect cache hierarchy from DT for CPU 0
```

So what should we do now? Fortunately, Qemu provides a way (-dtb) to load your own dtb file. We can modify the device tree to add "next-level-cache". But where can we get the default device tree which we can based on? Fortunately, again, Qemu provides a way (-machine dumpdtb=xxx.dtb) to dump the default device tree. Here is steps.

```
Step 1:
Dump the default dtb file

qemu-system-aarch64 -machine virt dumpdtb=virt_default.dtb <followed by other options>

Step 2:
Decompile dtb file to dts file

dtc -I dtb -O dts virt_default.dtb > virt.dts

Step 3: 
Add "next-level-cache" to each cpu node. For how to add that, just type "grep -r "next-level-cache" ./" in your Kernel source code. You should get a plenty of samples. 

Step 4:
Use dtc to compile the modified dts file to dtb file.

Step 5:
Load this new dtb file in Qemu.
qemu-system-aarch64 -dtb virt_new.dtb <followed by other options>
```

Then you should be able to see cache information from your sysfs.
