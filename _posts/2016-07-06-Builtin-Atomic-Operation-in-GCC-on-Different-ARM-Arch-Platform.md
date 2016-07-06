---
layout: post
title: Builtin Atomic Operation in GCC on Different ARM Arch Platform
---

ARMv6 provided a pair of synchronisation primitives, LDREX and STREX in the ARM instruction set. These instructions are used to implement atomic operations. But please note that ARMv6 doesn't support one byte exclusive operations. It begins to support byte level exclusive operations from ARMv7 and ARMv6_K/ARMv6_ZK. 

To implement an atomic operation, you can write it in assembly code by yourself. Like this.

```

locked    EQU 1
unlocked  EQU 0

; lock_mutex
; Declare for use from C as extern void lock_mutex(void * mutex);
    EXPORT lock_mutex
lock_mutex PROC
    LDR     r1, =locked
1   LDREX   r2, [r0]
    CMP     r2, r1        ; Test if mutex is locked or unlocked
	BEQ     %f2           ; If locked - wait for it to be released, from 2
    STREXNE r2, r1, [r0]  ; Not locked, attempt to lock it
    CMPNE   r2, #1        ; Check if Store-Exclusive failed
    BEQ     %b1           ; Failed - retry from 1
    ; Lock acquired
    DMB                   ; Required before accessing protected resource
    BX      lr

2   ; Take appropriate action while waiting for mutex to become unlocked
    WAIT_FOR_UPDATE
    B       %b1           ; Retry from 1
    ENDP


; unlock_mutex
; Declare for use from C as extern void unlock_mutex(void * mutex);
    EXPORT unlock_mutex
unlock_mutex PROC
    LDR     r1, =unlocked
    DMB                   ; Required before releasing protected resource
    STR     r1, [r0]      ; Unlock mutex
    SIGNAL_UPDATE	
    BX      lr
    ENDP
    
```

Or you can leverage on GNU GCC's builtin functions __atomic_xxx (GCC version >= 4.7.0) if you are happen to use GNU GCC. But there is a tricky problem if you decide to use GCC for this. 

As we know, usually, 32-bit GNU GCC supports a lot of arch from armv2 to armv8-a. So we must be careful of which arch you are setting to compiler. For example, if you are using armv6 as the arch to compile the code of signle byte atomic incr operation, GCC will link that function to legacy __sync_xxx functions. **These functions don't use LDREX/STREX at all!!!**

Let's check below sample. It will atomic increase 1 to the data stored at address 0x80000000.

```

__atomic_add_fetch((char *)0x80000000, (unsigned int)1, __ATOMIC_SEQ_CST);

```

On my Raspberry Pi 3B modle (ARMv8-a 32-bit OS), the default arch of GCC is armv6. 

```

[user]% gcc -Q --help=target | grep arch  
  -march=  armv6
  
```

So above code will be compiled as below assembly code by toolchain on the target natively.

```

[user]% gcc sample.c
[user]% objdump -D sample.out 

000103e8 <main>:
   103e8:       e92d4800        push    {fp, lr}
   103ec:       e28db004        add     fp, sp, #4
   103f0:       e3a03102        mov     r3, #-2147483648        ; 0x80000000
   103f4:       e1a00003        mov     r0, r3
   103f8:       e3a01001        mov     r1, #1
   103fc:       eb000265        bl      10d98 <__sync_add_and_fetch_1>
   10400:       e3a03000        mov     r3, #0
   10404:       e1a00003        mov     r0, r3
   10408:       e8bd8800        pop     {fp, pc}

000107a4 <__sync_fetch_and_add_1>:
  107a4:       e92d47f0        push    {r4, r5, r6, r7, r8, r9, sl, lr}
  107a8:       e200a003        and     sl, r0, #3
  107ac:       e3a050ff        mov     r5, #255        ; 0xff
  107b0:       e1a0a18a        lsl     sl, sl, #3
  107b4:       e59f8040        ldr     r8, [pc, #64]   ; 107fc <__sync_fetch_and_add_1+0x58>
  107b8:       e1a05a15        lsl     r5, r5, sl
  107bc:       e1a07001        mov     r7, r1
  107c0:       e3c09003        bic     r9, r0, #3
  107c4:       e1e06005        mvn     r6, r5
  107c8:       e5990000        ldr     r0, [r9]
  107cc:       e1a02009        mov     r2, r9
  107d0:       e0004005        and     r4, r0, r5
  107d4:       e0061000        and     r1, r6, r0
  107d8:       e1a04a34        lsr     r4, r4, sl
  107dc:       e0843007        add     r3, r4, r7
  107e0:       e0053a13        and     r3, r5, r3, lsl sl
  107e4:       e1831001        orr     r1, r3, r1
  107e8:       e12fff38        blx     r8
  107ec:       e3500000        cmp     r0, #0
  107f0:       1afffff4        bne     107c8 <__sync_fetch_and_add_1+0x24>
  107f4:       e6af0074        sxtb    r0, r4
  107f8:       e8bd87f0        pop     {r4, r5, r6, r7, r8, r9, sl, pc}
  107fc:       ffff0fc0                        ; <UNDEFINED> instruction: 0xffff0fc0
  
```

But if you specify GCC arch as armv7 or armv8-a, you will get what you expected.

```

[user]% gcc -march=armv8-a sample.c
[user]% objdump -D sample.out 

000103e8 <main>:
   103e8:       e52db004        push    {fp}            ; (str fp, [sp, #-4]!)
   103ec:       e28db000        add     fp, sp, #0
   103f0:       e3a03102        mov     r3, #-2147483648        ; 0x80000000
   103f4:       e1d32e9f        ldaexb  r2, [r3]
   103f8:       e2822001        add     r2, r2, #1
   103fc:       e1c31e92        stlexb  r1, r2, [r3]
   10400:       e3510000        cmp     r1, #0
   10404:       1afffffa        bne     103f4 <main+0xc>
   10408:       e3a03000        mov     r3, #0
   1040c:       e1a00003        mov     r0, r3
   10410:       e24bd000        sub     sp, fp, #0
   10414:       e49db004        pop     {fp}            ; (ldr fp, [sp], #4) 
   10418:       e12fff1e        bx      lr
   
```

That's very tricky. Because you are compiling your code on a ARMv8 Cortex A53 node with native toolchain. But it is actually building a ARMv6 code for you. So please be careful when you build code on ARM platform natively. Set -march=xxx all the time to make sure GCC know which platform it is working on.

