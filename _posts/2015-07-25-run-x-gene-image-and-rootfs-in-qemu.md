Run X-Gene image and rootfs in Qemu
===================
It is inefficient to control the X-Gene board remotely. But fortunately, X-Gene provides Linux source code and root filesystem. I spent some time to make it work with Qemu. 

    qemu-system-aarch64 -machine virt -cpu cortex-a57 -smp 8 -M virt -nographic -m 512 -kernel linux/arch/arm64/boot/Image -- append "earlyprintk console=ttyAMA0 root=/dev/nfs nfsroot=10.0.2.2:xgene/rootfs rw ip=dhcp init=/sbin/init" -netdev user,id=eth0 -device virtio-net-device,netdev=eth0 -redir tcp:2222::22

I use “virt” as the machine. There is no network interface provided in this machine. So I have to specify the network device manually .

Another trick is to modify the /etc/inittab of guest rootfs. Because we are using ttyAMA0 as the terminal to access guest os. 

To access guest OS from host through ssh, redirect tcp port 2222 of host to 22 of guest (in blue above). Then I can login in guest os:

    ssh -p 2222 root@localhost

Then I can boot the X-Gene APM Linux image successfully.

Have fun!