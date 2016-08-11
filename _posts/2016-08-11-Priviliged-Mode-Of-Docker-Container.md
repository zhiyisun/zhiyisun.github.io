---
layout: post
title: Privileged mode of Docker Container
---

Below test is inspired by a conversion about docker with my friend. 

Docker is quiet hot in recent years. We discussed whether it is possible to access devices inside of docker. So I did a test to find out the answer. 

First of all, I run a Ubuntu container on my Ubuntu host. 

```
docker run -i -t ubuntu /bin/bash
```

In this container, there is even no lsmod, insmod, rmmod binaries. That can be easily resolved by copy these binaries from the host or mount /sbin of host into the container directly(-v option of docker).  

I also write a simple hello world kernel module. My goal is to install this module inside of container. Since docker container use same kernel as host. If I can install this dummy kernel module inside of container, I can hack host OS too. 

Docker is designed to be "portable" and lightweight. So as default, it doesn't include kernel module in it. And thanks for the kernel namespace and cgroup features, it isolates one container from others and host. So in above setup, I got "permission deny" error as expected.

I Googled a solution that suggests to run container in privileged mode.  In that way, I am able to install my kernel module inside of container. And actually, it is installed on host os's kernel. 

```
docker run -i -t --privileged ubuntu /bin/bash
```

Docker team claimed this helps people to run docker within docker. But it is more powerful than this. We are able to do whatever we want inside of container. It also has more capability control options to control which  device/resource we can access. 

This is a very powerful and danger feature. Because Docker daemon runs as root. That means even if you only expose such kind of container to others, you actually expose the whole system to others with root privilege. With this option, container is not you think it is. Docker user should be careful about this. 