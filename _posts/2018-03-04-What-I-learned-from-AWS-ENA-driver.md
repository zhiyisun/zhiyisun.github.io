---


---

<p>Haven’t update my blog for quiet a long time!</p>
<p>I’m not just lazy. I’m super lazy. :-)<br>
<img src="https://cdn.shopify.com/s/files/1/0267/4223/products/Superlazy-clean_compact.jpg?v=1510686239" alt="https://cdn.shopify.com/s/files/1/0267/4223/products/Superlazy-clean_compact.jpg?v=1510686239"><br>
from: <a href="https://www.teeturtle.com/products/super-lazy">https://www.teeturtle.com/products/super-lazy</a></p>
<p>Linus Torvalds said “Talk is cheap. Show me the code.” I read AWS Elastic Network Adapter (ENA) driver when I wait my son’s after-school math club this weekend. I learned some details of AWS ENA that I haven’t found from other articles.</p>
<p>ENA is AWS’s elastic network adapter. It is a part of Nitro system which <a href="https://www.youtube.com/watch?v=02EbskIXCOc">AWS announced last year 2017</a>. ENA provides network function for both bare-metal host and VMs of AWS. It “hardwarize” network virtualization for AWS cloud. The ENA driver could be downloaded from <a href="https://github.com/amzn/amzn-drivers">Github</a>.  According to the code from Github, AWS open source their code for Linux kernel, Freebsd and DPDK(user space). The code has been upstream. But it doesn’t mean ENA only support these platforms. Actually it should also supports Windows and iPXE. (Check the host_info_os_type from the code.)</p>
<p>In the init of this PCIe device, module init function ena_init() creates a single thread workqueue as method to defer packet processing. The driver module is named as “ena”. There are four types of PCIe device id( defined in ena_pci_tbl). These IDs are interesting, all are related to “EC2”! :-)</p>
<pre class=" language-c"><code class="prism  language-c"><span class="token macro property">#<span class="token directive keyword">define</span> PCI_DEV_ID_ENA_PF      0x0ec2 </span>
<span class="token macro property">#<span class="token directive keyword">define</span> PCI_DEV_ID_ENA_LLQ_PF  0x1ec2 </span>
<span class="token macro property">#<span class="token directive keyword">define</span> PCI_DEV_ID_ENA_VF      0xec20 </span>
<span class="token macro property">#<span class="token directive keyword">define</span> PCI_DEV_ID_ENA_LLQ_VF  0xec21</span>
</code></pre>
<p>These IDs show the SR-IOV Physical Function (PF) and Virtual Function (VF) support of ENA devices. It is one of the key technologies which AWS network virtualization relies on. It makes VMs to bypass kernel/user space networking process software, operate NIC hardware directly and increase the network performance dramatically. But SR-IOV also has limitations. It is difficult to migrate VMs if they use SR-IOV. ENA’s pci_driver struct <strong>ena_pci_driver</strong> also defines <strong>ena_sriov_configure</strong> as the SR-IOV configure function.</p>
<p>Another feature which can be noticed from the IDs is “LLQ”. It means <strong>Low Latency Queue</strong>. Some of the ENA devices support this operation mode, which “saves several more microseconds”. <strong>Kernel/Documentation/networking/ena.txt</strong> describes as below:</p>
<blockquote>
<p>The ENA driver supports two Queue Operation modes for Tx SQs:</p>
<ul>
<li>Regular mode</li>
</ul>
<ul>
<li>In this mode the Tx SQs reside in the host’s memory. The ENA device fetches the ENA Tx descriptors and packet data from host memory.</li>
</ul>
<ul>
<li>Low Latency Queue (LLQ) mode or “push-mode”.</li>
</ul>
<ul>
<li>In this mode the driver pushes the transmit descriptors and the first 128 bytes of the packet directly to the ENA device memory space. The rest of the packet payload is fetched by the device. For this operation mode, the driver uses a dedicated PCI device memory BAR, which is mapped with write-combine capability.</li>
</ul>
</blockquote>
<p>As the function comment described, <strong>“ena_probe() initializes an adapter identified by a pci_dev structure. The OS initialization, configuring of the adapter private structure, and a hardware reset occur.”</strong> ENA device exposes standard PCI config registers and device specific memory mapped(MMIO) registers to host CPU for hardware configuration during the initialization. For each VM or host OS, a pair of queues: Admin Queue (AQ) and Admin Completion Queue(ACQ) are created for further hardware configuration after the device is initialized.</p>
<p>The following admin queue commands are supported:</p>
<blockquote>
<ul>
<li>Create I/O submission queue</li>
<li>Create I/O completion queue</li>
<li>Destroy I/O submission queue</li>
<li>Destroy I/O completion queue</li>
<li>Get feature</li>
<li>Set feature</li>
<li>Configure AENQ</li>
<li>Get statistics</li>
</ul>
</blockquote>
<p>Besides this, ENA device has another mechanism named Asynchronous Event Notification Queue(AENQ) to report devices status. AENQ has three handlers:</p>
<blockquote>
<ul>
<li>link change: report link up/down</li>
<li>Notification: update parameters from hardware;  like <strong>admin_completion_tx_timeout</strong>, <strong>mmio_read_timeout</strong>, <strong>missed_tx_completion_count_threshold_to_reset</strong>, <strong>missing_tx_completion_timeout</strong>, <strong>netdev_wd_timeout</strong>, etc</li>
<li>keep alive: get keep alive jiffies and rx drops</li>
</ul>
</blockquote>
<p>ENA setup timer service to check <strong>missing_keep_alive</strong>, <strong>admin_com_state</strong>, <strong>missing_completions</strong>, <strong>empty_rx_ring</strong>, etc. If something goes wrong, ENA driver will reset the device. Reset reasons are defined in <strong>ena_regs_reset_reason_types</strong>.</p>
<p>For data path, as shown in above picture, there could be one pare of TX/RX submission queues associated to one vCPU. Each submission queue has a completion queue(CQ) associated with it. The max number of IO queue is 128. But the actual number of IO queue is min of <strong>ena_calc_io_queue_num</strong>, <strong>io_sq_num</strong>, <strong>io_cq_num</strong>, <strong>num_of_onlinecpu</strong>, MSI-X number - 1 (one IRQ for management). This submission queue and completion queue architecture has various benefits. As described in <strong>Kernel/Documentation/networking/ena.txt</strong>:</p>
<blockquote>
<ul>
<li>Reduced CPU/thread/process contention on a given Ethernet interface.</li>
<li>Cache miss rate on completion is reduced, particularly for data cache lines that hold the sk_buff structures.</li>
<li>Increased process-level parallelism when handling received packets.</li>
<li>Increased data cache hit rate, by steering kernel processing of packets to the CPU, where the application thread consuming the packet is running.</li>
<li>In hardware interrupt re-direction.</li>
</ul>
</blockquote>
<p>ENA driver uses NAPI interface as the packet processing mechanism. <strong>ena_io_poll()</strong> is the poll function of ENA’s NAPI. In <strong>ena_io_poll()</strong>, it “cleans” TX IQR and RX IRQ. <strong>ena_clean_tx_irq()</strong> fetches the completion description from complete queue(CQ) of TX. <strong>ena_clean_rx_irq()</strong> fetches the incoming packet description from RX queue.</p>
<p>Overall, the architecture of ENA driver is similar to other popular NIC in the industry. ENA is developed by Annapurna Labs which is acquired by Amazon in 2015. AWS have said that eventually most (or all) instances will use the Nitro hypervisor. Other big cloud vendors, like Microsoft Azure, etc also have their own hardware implementation of network virtualization. Semiconductor companies, like Mallanox, Broadcom released multiple Smart-NIC solutions.</p>
<p><strong>Hardware always matters in cloud.</strong></p>

