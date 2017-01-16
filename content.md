## Linux Kernel Library: Reusing Monolithic Kernel

<span>
<br>
<br>
<br>
<br>

### Hajime Tazaki 
IIJ Innovation Institute
<br>

2016/12

ネットワークソフトウェアの設計と実装原理研究会
</span>

---

## Intro.

- 2000-2007 PFU
 - COPS s/v
 - v4 routing (zebra/quagga)
- 2007-2011 SFC
 - MANEMO
- 2011-2013 NICT
 - ns-3 + Linux
- 2013-2016 UTokyo
 - LibOS/LKL
- 2016-     IIJ-II
 - LibOS/LKL

---

## LKL in a nutshell


- Linux kernel library
 - a library of Linux
- **Octavian Purdila (Intel)'s** <br> work (since 2007?)
- Proposed on LKML (Nov. 2015)
 - https://lwn.net/Articles/662953/
 - 2809 LoC (as of Apr. 2016)

<div class="right" style="width: 35%">
<img src="figs/lkl-arch.png" width=100%>

<br>
<small>
Purdila et al., LKL: The Linux kernel library, RoEduNet 2010.
</small>
</div>

>>>

## LKL (cont'd)

- hardware-independent <br> architecture (arch/lkl)
- provide an interface underlying <br>environment
 - *outsource* dependencies
 - clock, memory allocation, <br>scheduler
 - running on Windows, Linux, <br>FreeBSD
- simplify I/O operation of devices
 - virtio host implementation
 - could use the driver (of virtio)<br> in Linux 

<div class="right" style="width: 35%">
<img src="figs/lkl-arch.png" width=100%>

<br>
<small>
Purdila et al., LKL: The Linux kernel library, RoEduNet 2010.
</small>
</div>

>>>

## Benefits

- **operating system personality**
 - userspace library has less deployment cost
 - (less ossification of new features)
- **well-matured code base in various ways**
 - (e.g.) Linux kernel running in userspace
 - (e.g.) Application embeded OS (unikernel)
 - small kernel, a bunch of library
 - but in a different shape


>>>

<!-- .slide: data-background="figs/cs-thomas.png" -->

> Any problem in computer science can be solved with another level of __indirection__.

>(Wheeler and/or Lampson)



<br /><small>
img src: https://www.flickr.com/photos/thomasclaveirole/305073153
</small>

>>>

### What is reusing monolithic kernel ?

- Anykernel: originally in NetBSD rump kernel 


>We define an anykernel to be an organization of kernel code which allows the kernel's **unmodified** drivers to be **run in various configurations** such as application libraries and microkernel style servers, and also as part of a monolithic kernel.  -- Kantee 2012.

- Using (*unmodified*) high-quality code base of monolithic kernel
- on different environment in different shape
- by **gluing** additional stuffs 


Note:

様々仮想化技術はあり、用途もあるが、「まだ別の仮想化技術が便利で必要で
すよ」という話と、「その仮想化技術にまだ課題がありますよ」という話を、
Linux カーネルを題材としてお話。

>>>

<div class="left" style="width: 40%">
<br>
<br>
<img src="figs/anykernel-pre.png" width=100%>
</div>


<div class="right" style="width: 40%">
<img src="figs/anykernel-post-en.png" width=100%><!-- .element: class="fragment" data-fragment-index="1" -->
</div>

>>>

## (a bit of) History

- rump: 2007 (NetBSD)
- LKL: 2007 (Linux)
- DCE/LibOS: 2008 (Linux/FreeBSD)
- LibOS/LKL revival: 2015
 - LibOS merged to LKL

>>>

<img src="figs/lwn-libos-150408.png" width=28%>
<img src="figs/hnews-1503.png" width=28%><br>
<img src="figs/phoronix-1503.png" width=28%>
<img src="figs/mynavi-libos.png" width=28%>

<small>
http://news.mynavi.jp/news/2015/03/25/285/ <br>
https://news.ycombinator.com/item?id=9259292 <br>
http://www.phoronix.com/scan.php?page=news_item&px=Linux-Library-LibOS <br>
http://lwn.net/Articles/639333/ <br>
</small>

>>>

## LKL v.s. LibOS

<div class="left" style="width: 50%">
<img src="figs/lkl-arch.png" width=100%>
LKL
</div>

<div class="right" style="width: 50%">
<img src="figs/libos-arch.png" width=75%>
<br>
LibOS
</div>

>>>

## LKL v.s. LibOS (cont'd)

- LoC:
 - arch/lkl (LKL) < arch/lib (LibOS)
 - diff: the amount of stub code
- commons
 - no modification to the original Linux code
 - description of kernel context (by POSIX thread)
 - outsourced resources (clock, memory, scheduler)
 - CPU independent architecture
- diffs
 - LibOS: implemented with higher API (timer, irq, kthread) by pthread
 - LKL: implement IRQ, kthread, timer with pthread in lower layer

Note:

reimplementation of timer API is required for simulation's feature, time warp,


---

## Implementation

>>>

## Internals
<div class="left" style="width: 40%">
<img src="figs/lkl-arch.png" width=100%>
</div>


<br>
1. Host backend (host_ops)
1. CPU independent arch. (arch/lkl)
1. Application interface


Note:
 - bridge (real) userspace with library kernel

(FIXME: add diagram)


>>>

## 1. host backend

<div class="left" style="width: 40%">
<img src="figs/lkl-arch-host.png" width=100%>
</div>

- environment *dependent* <br>part
 - unify an interface across <br> different platforms
 - (rump-hypercall like)
- device interface with **Virtio**
 - block device <=> disk image
 - networking <=> TAP, <br> raw socket, DPDK, VDE

>>>
## 2. CPU independent architecture

<div class="left" style="width: 40%">
<img src="figs/lkl-arch-kernel.png" width=100%>
</div>



architecture (arch/lkl)
- transparent architecture bind <br> (as CPU arch)
 - require no modification to  <br>the other
- 2800 LoC
 - thread information (struct <br> thread_info)
 - irq, timer, syscall handler
 - access to underlying layer <br> by host_ops

>>>

## 3. Application interface

<div class="left" style="width: 40%">
<img src="figs/lkl-arch-api.png" width=100%>
</div>

<br><br>
1. use exposed API (LKL syscall)
2. use host libc (LD_PRELOAD)
3. extend (alternative) libc

>>>

## API 1: use exposed API (LKL syscall)

- call entry points of LKL kernel
 - *lkl_sys_open()*, *lkl_sys_socket()*
- *almost* same as ordinal syscalls
 - return value, *errno* notification are<br> different
- can use LKL syscall *and* host syscall <br> simultaneously
 - read ext4 file by lkl_sys_read() => <br>write into host (Windows) by write()

<div class="right" style="width: 30%">
<img src="figs/lkl-syscall-1.png" width=100%>
</div>

>>>
## API 2: hijack host standard library

- dynamically replace symbols <br> of host syscalls (of libc)
 - LD_PRELOAD
 - socket() => lkl_sys_socket()
- can use host binary (executable) as-is
- limitation of replaceable symbols
- needs syscall translation on <br>non-linux host

<div class="right" style="width: 30%">
<img src="figs/lkl-syscall-hijack.png" width=100%>
</div>

>>>
## API 3: extend (alternative) libc

- **only** call LKL syscall with our own libc
- also introduce as a virtual <br>CPU architecture
- a program can link this instead of<br> host libc
 - can't access to (underlying) host <br>resource directly via this lkl syscall
- as a patch for musl libc

<div class="right" style="width: 30%">
<img src="figs/lkl-syscall-musl.png" width=100%>
</div>

>>>

## Usecase (applications)

- Use Case 1: instant kernel bypass
- Use Case 2: programs reusing kernel code in userspace
- Use Case 3: unikernel

>>>

## Use Case 1: instant kernel bypass

- syscall redirection by LD_PRELOAD
- can use both LKL and host syscalls

<br> <br>

new feature without touching host kernel

```
LD_PRELOAD=liblkl-super-tcp++.so firefox
```

>>>

## Use Case 2: programs reusing kernel code in userspace
- use kernel code without **porting**
 - mount a filesystem w/o root privilege
- can use both LKL and host syscalls
<p>
- e.g., access to disk image of ext4 format on Windows
 1. open disk image (*CreateFile()*)
 1. Mount (*lkl_sys_mount()*)
 1. read a file in the disk image (*lkl_sys_read()*)
 1. write a file to windows side (*WriteFile()*)


>>>

## Use Case 3: Unikernel

- single-application contained LKL
 - python + LKL, nginx + LKL
- only LKL syscalls available
 - musl libc extension
- rump hypcall (frankenlibc)
 - running on non-OS environment
 - (on Xen Mini-OS via rumprun)
- Work in progress

<div class="right" style="width: 35%">
<img src="figs/CloudOSDiagram.png">

<small>
- http://www.linux.com/news/enterprise/cloud-computing/751156-are-cloud-operating-systems-the-next-big-thing-
</small>
</div>

<!--
<div class="right" style="width: 40%">
<img src="figs/frankenlibc-stack.png" width=70%>
</div>
-->

>>>

## demos with linux kernel library

<div class="left" style="width: 50%">
<img src="figs/franken-linux-ping6.gif" width="840">
Unikernel on Linux (ping6 command embedded kernel library)
</div>

<div class="right" style="width: 50%">
<img src="figs/franken-qemu-arm-hello.gif" width="840">
Unikernel on qemu-arm (hello world)
</div>

---

## Kernel bypass/userspace networking

>>>

## Network Stack

- Why in kernel space ?
 - the cost of packet was <br>expensive at the era ('70s)
 - now much cheaper

- Getting fat (matured) <br>after decades
 - code path is longer <br> (and slower)
 - hard to add new features
 - faced unknown issues

<div class="right" style="width: 45%">
<img src="figs/fat-kernel.png">
<small>
img src: http://www.makelinux.net/kernel_map/
</small>
</img>
</div>

>>>

## Alternate network stacks

- lwip (2002~)
- Arrakis [OSDI '14]
- IX [OSDI '14]
- MegaPipe [OSDI '12]
- mTCP [NSDI '14]
- SandStorm [SIGCOMM '14]
- uTCP [CCR '14]
- rumpkernel [ATC '09]
- FastSocket [ASPLOS '16]
- SolarFlare (2007~?)
- StackMap [ATC '16]
- libuinet (2013~)
- SeaStar (2014~)
- Snabb Switch (2012~)

Note:

218	lwip
85	Arrakis
78	IX
63	MegaPipe
44	mTCP
32	SandStorm
17	uTCP
13	rumpkernel
2	FastSocket
1	SolarFlare
1	StackMap
1	libuinet
1	SeaStar

>>>

## Motivations

- Socket API sucks
 - StackMap, MegaPipe, uTCP, SandStorm, IX
 - New API: no benefit with existing applications
- Network stack in kernel space sucks
 - FastSocket, mTCP, lwip (SolarFlare?)
- Compatibility is (also) important
 - rumpkernel, libuinet, Arrakis, IX, SolarFlare
- Existing programming model sucks
 - SeaStar


Note:

## Socket API issues
- VFS overhead
- non-batchable (**{send,recv}mmsg**)
## kernel space is a source of ossification?
## generalization costs a lot
- rumpkernel + netmap 
## seastar ?

>>>

## Techniques

- batching (syscall/NIC access)
 - Arrakis, IX, MegaPipe, mTCP, SandStorm, uTCP
- Utilize feature-rich kernel stack
 - rumpkernel, fastsocket, StackMap
- Porting to userspace stack
 - libuinet, SandStorm
- Kernel bypass (userspace network stack)
 - mTCP, SandStorm, uTCP, rumpkernel, libuinet, lwip, SeaStar
- bypass technique itself
 - netmap, PF_RING, raw socket, Intel DPDK
- Connection locality (multi-core scalability)
 - SeaStar, MegaPipe, mTCP, fastsocket, .....

Note:
- Full scratch
 - lwip (Arrakis, IX, SolarFlare), mTCP, uTCP, SeaStar

>>>

## Implementation

- Full scratch
 - lwip (Arrakis, IX, SolarFlare?), mTCP, uTCP, SeaStar
- Porting based
 - libuinet, SandStorm
- New API
 - MegaPipe, StackMap
- Anykernel
 - rumpkernel, (LKL)

>>>

## What's still missing ?

- some solves problems by **specialization**
 - avoiding generality tax
 - performance w/ specialization v.s. more features w/ generalization
 - e.g., less TCP stack features, new API breaks existing applications support.
- **specialized** v.s. **generalized**
 - generalization often involves **indirection**
 - indirection usually introduces complexity (Wheeler/Lampson)
- performant **and** generalized ?

>>>

### Specialization v.s. Generalization

<normal>
- MegaPipe [OSDI '12]
 - *outperforms baseline Linux .. **582%** (for short connections).*
 - New API for applications (no existing applications benefit)<!-- .element: class="fragment" data-fragment-index="1" -->
- mTCP [NSDI '14]
 - *improve... **by a factor of 25** compared to the latest Linux TCP*
 - implement with very limited TCP extensions<!-- .element: class="fragment" data-fragment-index="1" -->
- SandStorm [SIGCOMM '14]
 - *our approach ..., **demonstrating 2-10x** improvements*
 - specialized (no existing applications benefit)<!-- .element: class="fragment" data-fragment-index="1" -->
- Arrakis [OSDI '14]
 - *improvements of **2-5x in latency and 9x in throughput** .. to Linux*
 - utilize simplified TCP/IP stack (lwip) (loose feature-rich extensions)<!-- .element: class="fragment" data-fragment-index="1" -->
- IX [OSDI '14]
 - *improves throughput ... **by 3.6x** and reduces latency **by 2x***
 - utilize simplified TCP/IP stack (lwip) (loose feature-rich extensions)<!-- .element: class="fragment" data-fragment-index="1" -->

Note:
- netmap [ATC '12]
 - *a single core ... can send or receive 14.88 Mpps (**the peak packet rate** on 10 Gbit/s links).*

>>>

Sigh

---

## Performance study

>>>

## Conditions

- ThinkStation P310 x2
 - CPU: Intel Core i7-6700 CPU @ 3.40GHz (8 cores)
 - Memory: 32GB
 - NIC: X540-T2
- Linux 4.4.6-301 (x86_64) on Fedora 23
 - Linux bridge (X540 + tap/raw socket)
 - *no DPDK*... (can't with hijack, etc)
- netperf (git ~v2.7.0)
 - netserver (native)
 - netperf (varied)

<img src="figs/bench-topo.png" width=50%>

>>>

## Conditions (cont'd)

- combinations
 - netperf (sendmmsg) + host stack (**native**)
 - \+ hijack library, native thread (**hijack**)
 - \+ frankenlibc/lkl, green thread (**lkl-musl**)
 - netperf (sendmmsg) + lkl extension + frankenlibc (**lkl-musl (skb pre alloc)**)
- pinned a processor
 - using taskset command
- disable/enable offload features (tso/gso/gro, rx/tx cksum)

>>>

<img src="2016-07-12/tx/tcp-rr.png" width=80%>

### TCP_RR (netperf)

>>>

<img src="2016-07-12/tx/udp-stream.png" width=80%>

### UDP_STREAM (netperf)

>>>

<img src="2016-07-12/tx/udp-stream-pps.png" width=80%>

### UDP_STREAM (pps, netperf)

>>>

<img src="2016-07-12/tx/tcp-stream.png" width=80%>

### TCP_STREAM (netperf)

>>>

<img src="2016-08-03/tx/tcp-stream.png" width=80%>

### TCP_STREAM+TSO (netperf)

>>>

## (ref.) LibOS results (as of Feb. 2015)

<img src="figs/nuse-benchmark-host.png" width=100%>

- 1024 bytes UDP, own-crafted tool

- throughput: <10% of Linux native

>>>

## Observations (of benchmark)

- Native thread vs Green thread
 - better TCP_RR w/ native thread (pthread)
 - better TCP_STREAM/UDP_STREAM w/ green thread
 - ???
- avoiding dynamic allocation contributes a lot
- penalized over MTU-sized payload on host stack (?)

---

## Summary

- Morphing monolithic kernel into an Anykernel
- Various use cases
 - Userspace network stack (kernel bypass)
 - Unikernel
- Performance study in progress


https://github.com/lkl/linux


>>>

## Reference
- Linux Kernel Library
 - Purdila et al., LKL: The Linux kernel library, RoEduNet 2010.
 - https://github.com/lkl/linux
- Rumpkernel (dissertation)
 - Kantee, Flexible Operating System Internals: The Design and Implementation of the Anykernel and Rump Kernels, Ph.D Thesis, 2012
- Linux LibOS in general
 - Tazaki et al. Direct Code Execution: Revisiting Library OS Architecture for Reproducible Network Experiments, CoNEXT 2013
 - http://libos-nuse.github.io/ (LibOS in general)
 - https://lwn.net/Articles/637658/


---

## Backups

>>>


## Recent Updates


>>>

## Updates (diff to lkl)

- (musl) libc integration
- rump hypercall interface
 - via frankenlibc tools (for POSIX environment)
 - via rumprun framework (for baremetall/xen/kvm environment)
- more applications
 - netperf (signal handling, etc)
 - nginx
 - ghc (Haskell runtime)
- performance study

>>>

## libc integration

- standard lib for LKL
 - all syscall direct to LKL
 - application can use LKL *transparently* <br>
   no special modifications or hijack needed
- based on musl libc
 - introduce new (sub) architecture **lkl**

<div class="right" style="width: 30%">
<img src="figs/lkl-syscall-musl.png" width=100%>
</div>

>>>

## rump hypercall interface

- replacement of LKL host_ops
 - or yet-another new host environment (rump)
- has two thread primitives
 - pthread-based (as LKL does)
 - ucontext-based (more efficient on non-MP)

- can reduce
 - the effort of host_ops maintainance
 - complexity of tall abstraction turtle

<div class="right" style="width: 30%">
<img src="figs/lkl-rumphypcall.png" width=100%>
</div>

>>>

## rump hypcall (cont'd)

- integration of
 - libc (musl for LKL, netbsd libc for rumpkernel)
 - rump hypcall (on linux, freebsd, netbsd, qemu-arm, spike)
 - host (platform) support code
- frankenlibc
 - has two *namespaced* libc(s)
 - hyper call implementation can use libc
<br>
- provides
 - a **libc.a**
 - cross-build toolchains (rumprun-cc, etc)

>>>

## Usage

- build

```
% ./configure CC=rumprun-cc ; make
```

- execution (with rexec launcher)

```
% rexec ./nginx disk-nginx.img tap:tap0 -- -c nginx.conf
```

rexec executable [disk image file] [NIC] -- [executable specific options]

>>>

## Codes

- https://github.com/libos-nuse/lkl-linux
- https://github.com/libos-nuse/musl
- https://github.com/libos-nuse/frankenlibc
- https://github.com/libos-nuse/rumprun
- https://github.com/libos-nuse/nginx
- https://github.com/libos-nuse/ghc



