---
layout: post
title: How to Use Performance Monitor Unit(PMU) of 64-bit ARMv8-A in Linux
---

**Performance Monitor** is an optional feature in ARMv8-A architecture. Performance Monitor in ARMv8-A includes a 64-bit cycle counter, a number of 32-bit event counters and control component.

From programmer perspective, it is a handy tool for performance monitoring and tuning. We can get processor status, like cycle, instruction executed, branch taken, cache miss/hit, memory read/write, etc from these PMU event counters. 

Performance counters support has been added in Linux Kernel since 3.6. Kernel has a utility named **perf** to view CPU PMU event statistics. Perf supports raw event id or named event. Due to the difference architecture of CPUs, only a few events are common defined in kernel. All other events related to specific CPU architecture can only be accessed by using raw event id. For detailed usage of perf utility, refer to [perf wiki tutorial page](  https://perf.wiki.kernel.org/index.php/Tutorial).  

Perf can be used when measure the whole software program. But if only a piece of code is interested in debugging, how to monitoring the CPU performance event counters for it? There are some [articles](http://neocontra.blogspot.sg/2013/05/user-mode-performance-counters-for.html) describe how to make it for ARMv7. But few of them mention ARMv8. This article will try to cover ARMv8's PMU. 

There are two ways I know so far.

 - **Access PMU registers by assembly code directly**

The basic way is write assembly code to access PMU registers directly. Please note that ARMv8-A architecture allows access PMU counters from EL0(means in user space of Linux). (This article will not cover all register detail. Please refer to **ARMv8 Architecture Reference Manual** for details. ) 

So the first thing is to create a kernel module to enable user-mode access to PMU counters. Below is the code to set PMU register PMUSERENR_EL0 to enable user-mode access. 

```
/*Enable user-mode access to counters. */
asm volatile("msr pmuserenr_el0, %0" : : "r"((u64)ARMV8_PMUSERENR_EN_EL0|ARMV8_PMUSERENR_ER|ARMV8_PMUSERENR_CR));

/*   Performance Monitors Count Enable Set register bit 30:0 disable, 31 enable. Can also enable other event counters here. */ 
asm volatile("msr pmcntenset_el0, %0" : : "r" (ARMV8_PMCNTENSET_EL0_ENABLE));

/* Enable counters */
u64 val=0;
asm volatile("mrs %0, pmcr_el0" : "=r" (val));
asm volatile("msr pmcr_el0, %0" : : "r" (val|ARMV8_PMCR_E));
```

After this kernel module is loaded, user space application can access PMU event counters.

```
/* Access cycle counter */
asm volatile("mrs %0, pmccntr_el0" : "=r" (r));

/* Setup PMU counter to record specific event */
/* evtCount is the event id */
evtCount &= ARMV8_PMEVTYPER_EVTCOUNT_MASK;
asm volatile("isb");
/* Just use counter 0 here */
asm volatile("msr pmevtyper0_el0, %0" : : "r" (evtCount));
/*   Performance Monitors Count Enable Set register bit 30:1 disable, 31,1 enable */
uint32_t r = 0;
asm volatile("mrs %0, pmcntenset_el0" : "=r" (r));
asm volatile("msr pmcntenset_el0, %0" : : "r" (r|1));

/* Read counter */
asm volatile("mrs %0, pmevcntr0_el0" : "=r" (r));

/*   Disable PMU counter 0. Performance Monitors Count Enable Set register: clear bit 0*/
uint32_t r = 0;
asm volatile("mrs %0, pmcntenset_el0" : "=r" (r));
asm volatile("msr pmcntenset_el0, %0" : : "r" (r&&0xfffffffe));

```

This is a simple way to access PMU. But it also has limitation. It could conflict with other performance tools running in background (Like perf).  

 - **Using perf_event_open system call**

Another way is to use Linux perf infrastructure. Software can use perf_event_open system call to get PMU event counters from kernel. So above ugly kernel module is not needed. [PAPI](http://icl.cs.utk.edu/papi/) is a tool to access hardware performance counters. But unfortunately, it doesn't support ARMv8-A yet. [Austin Seipp](https://www.blogger.com/profile/08003235138924772402) suggests to use GNU C’s \__attribute__((constructor)) and \__attribute__((destructor)) routines. The constructor invokes the system call which returns a file descriptor. We can later read from the file descriptor to get the cycle count from the processor. 

```
static int fddev = -1; 
__attribute__((constructor)) static void
init(void)
{
        static struct perf_event_attr attr;
        attr.type = PERF_TYPE_HARDWARE;
        attr.config = PERF_COUNT_HW_CPU_CYCLES;
        fddev = syscall(__NR_perf_event_open, &attr, 0, -1, -1, 0); 
}

__attribute__((destructor)) static void
fini(void)
{
        close(fddev);
}

static inline long long
cpucycles(void)
{
        long long result = 0;
        if (read(fddev, &result, sizeof(result)) < sizeof(result)) return 0;
        return result;
}
```

In above sample,  **attr.type** could be below types. Since this article is talking about processor's PMU, hardware's Perf types are **PERF_TYPE_HARDWARE**,  **PERF_TYPE_HW_CACHE**,  **PERF_TYPE_RAW**.

```
/*
 * attr.type
 */
enum perf_type_id {
        PERF_TYPE_HARDWARE    = 0,
        PERF_TYPE_SOFTWARE    = 1,
        PERF_TYPE_TRACEPOINT  = 2,
        PERF_TYPE_HW_CACHE    = 3,
        PERF_TYPE_RAW         = 4,
        PERF_TYPE_BREAKPOINT  = 5,

        PERF_TYPE_MAX,        /* non-ABI */
};
```

**attr.config** could be picked from enum **perf_hw_id**, combination of **(perf_hw_cache_id | perf_hw_cache_op_id | perf_hw_cache_op_result_id)**, or raw hardware PMU event id, like 0x011. Please check the details in **include/uapi/linux/perf_event.h** in kernel.



But please note that this method(system call) involves additional latency comparing to access PMU registers directly. Because it needs to switch between user context and kernel context. And perf's infrastructure is complicated.

There are other methods could get PMU events. For example, JTAG tools, like ARM's DS-5 with [DSTREAM](http://ds.arm.com/ds-5/debug/dstream/) could use PM hardware to record cycles per instructions. [OProfile](http://oprofile.sourceforge.net/) provides the ocount tool for collecting raw event counts on a per-application, per-process, per-cpu, or system-wide basis. 

Based on the work from [Austin Seipp](https://www.blogger.com/profile/08003235138924772402) , I added ARMv8 support for PMU. My sample code is hosted on [Github](https://github.com/zhiyisun/enable_arm_pmu/tree/dev) in dev branch.

Reference

 1. http://neocontra.blogspot.sg/2013/05/user-mode-performance-counters-for.html
 2. http://stackoverflow.com/questions/30709432/how-to-get-cpu-performance-counter-for-a-piece-of-code
 3. http://web.eece.maine.edu/~vweaver/projects/perf_events/perf_event_open.html
 4. http://lists.infradead.org/pipermail/linux-arm-kernel/2014-November/299228.html
 5. https://community.arm.com/groups/embedded/blog/2015/03/08/using-the-arm-performance-monitor-unit-pmu-linux-driver