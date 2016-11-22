---
layout: post
title: Port Taobao Tair Service on ARM Server and Some Thoughts on Performance
---

Tair is a distributed key/value storage system developed by Taobao.com. It is an open source project since 2010. And it can be downloaded from [its website](http://tair.taobao.org/). It is not as popular as Redis/Memcached. But it is used by Taobao and some Chinese companies. 

It provides four services: fdb, kdb, ldb and mdb. (In Tair's homepage, it also mentioned rdb. But we don't see it in source code which is opened to public.) fdb, kdb, ldb are persistent storage services. They are based on [firebird](http://firebirdsql.org/), [Kyoto Cabinet](http://fallabs.com/kyotocabinet/) and [Google's levelDB](https://github.com/google/leveldb). mdb is mdb is cache based solution. It is based on [memcached](https://memcached.org/). 

Basically, Tair system includes three components. Config server, Data Server and Client. 

![enter image description here](http://cdn1.infoqstatic.com/statics_s2_20161122-0331/resource/articles/taobao-tair/zh/resources/image1.JPG)


(Source: [http://www.infoq.com/cn/articles/taobao-tair](http://www.infoq.com/cn/articles/taobao-tair))

**Porting:**

For client, tair defines a set of RESTful interface. So user can write their own client code. It could be platform/language independent.

For Config and Data server, it is written mainly in C++. To make it run on ARM platform, it needs some porting effort. The most issue which blocks you to compile Tair on ARM platform is its inline x86 assembly code. Fortunately, those assembly are related to atomic operations. These function/code can be easily replaced by GNU builtin functions. Just like I contributed to [stress-ng](http://kernel.ubuntu.com/~cking/stress-ng/). 

**Performance:**

Among server side components, most of workload are running on data server. So performance optimization work should focus on data server side. 

More than 50% CPU are consumed by receiving requests from network and transmitting responses back to network. I saw same phenomenon on Redis and Memcached. It includes both kernel TCP/IP stack and user space code. For kernel space, NIC interrupt affinity and some TCP/IP parameters can be tuned to get better performance. 

In user space, it depends on the architecture of Tair. For open source project, Tair uses tb-common-utils as the low level lib for rx/tx.  

Here is some notes I made after reading Tair code.

    When everything is initialised, transport::listen is listening on specific server interface IP and port. When new connection request is coming, it creates a new TCPComponent for this client in TCPAcceptor::handleReadEvent. And it will also add this socket to _socketEvent.  All further service requests/response will use this TCPComponent. In transport:eventLoop, it calls EPollSocketEvent::epoll_wait to wait all events registered to this epoll. Then calls iocomponent’s handleReadEvent/handleWriteEvent based on event type. iocomponent could be TCPAcceptor or TCPComponent. TCPAcceptor is used to received connection setup request from client. data_server will setup a new connection for this client’s service. This new connection is TCPComponent. TCPComponent’s handleReadEvent/handleWriteEvent functions will call TCPConnection’s readData/writeData functions. TCPConnection’s readData will call Connection::handlePacket and finally call tair_server::handlePacket or handleBatchPacket to push packets to related queues or handle the packet directly. For each queue(before these queues, all works are only in one thread, tair’s threads is for each queue), tair_server::handlePacketQueue is the function to process all packets based on packet type “pcode”. Then for example, GET/PUT operation are handled by request_processor::process. tair_manager’s put/get operation are called after that. 

It has two thread modes. One is multiple thread mode. It uses one thread for rx. Then push these requests to a queue. Other threads are launched to retrieve requests from these queues and do get/put operation from related DB. Another mode is single thread mode. It receives requests from TCP socket and do get/put operation on DB.  

To me, it doesn't make sense to use multiple threads architecture. Because, when using multiple threads mode, the bottleneck is the one thread which uses to RX/TX. And it also wastes CPU to switch between multiple threads. To get better performance, it would be better to use single thread mode and launch multiple instances on one server node. Just like we did in Redis.

One thing that I notice is that in tbnet, it calls syscall to set TCP Quick ACK in each packet receive process!!! Based on my test, it consumes about 20% CPU for this syscall. TCP delayed acknowledgement is a kernel technology which combined the ACK, window update and the response data into one segment, to reduce protocol overhead. If there is no response from remote side, the ACK will be sent out after 500ms delay. To set TCP Quick ACK, Kernel TCP stack will send ACK after it get TCP packet immediately. For Tair application, each request from client side will have a response from data server side. So it makes TCP Quick ACK not so useful in this case. And TCP Quick ACK setting will be disabled automatically by kernel based on TCP stack status. So if you want to use it, you have to set it each time. That downgrade the performance a lot. For tair case, I would suggest to not set TCP Quick ACK in packet receive handler.

Another thought is Tair is using mur_mur_hash2 as hash function. For example it is used for mdb to store its entry in a hash table. It would be better to replace with a hash function optimized for ARM platform. 

