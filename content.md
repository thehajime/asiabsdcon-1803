## Linux Kernel Library updates

<span>
<br>
<br>
<br>
<br>

### Hajime Tazaki 
IIJ Innovation Institute
<br>

2016/06
</span>

---

## Introduction

- Hajime Tazaki
- Japanese
- @thehajime
- http://about.me/thehajime

>>>

## LKL in a nutshell


- Linux kernel library
 - a library of Linux
- **Octavian Purdila's** work (since 2007?)
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

- hardware-independent architecture (arch/lkl)
- provide an interface underlying environment
 - outsource dependencies
 - running on Windows, Linux, FreeBSD
- simplify I/O operation of devices
 - virtio host implementation
 - could use the driver (of virtio) in Linux 

<div class="right" style="width: 35%">
<img src="figs/lkl-arch.png" width=100%>

<br>
<small>
Purdila et al., LKL: The Linux kernel library, RoEduNet 2010.
</small>
</div>

>>>


## (a bit of) History

- rump: 2007 (NetBSD)
- LKL: 2007 (Linux)
- DCE/LibOS: 2008 (Linux/FreeBSD)
- LibOS/LKL revival: 2015
 - LibOS merged to LKL

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
 - description of kernel context (by POSIX thread)
 - interface to underlying host (clock, memory, scheduler)
 - CPU independent architecture
 - no modification to the original Linux code
- diffs
 - LibOS: implemented with higher API (timer, irq, kthread) by pthread
 - LKL: implement IRQ, kthread, timer with pthread in lower layer


>>>

## Internals
<div class="left" style="width: 48%">
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

<div class="left" style="width: 30%">
<img src="figs/lkl-arch-host.png" width=100%>
</div>

- environment *dependent* part
 - unify an interface across different platforms
 - (rump-hypercall like)
- device interface with **Virtio**
 - block device <=> disk image
 - networking <=> TAP, DPDK, VDE

>>>
## 2. CPU independent architecture

<div class="left" style="width: 35%">
<img src="figs/lkl-arch-kernel.png" width=100%>
</div>



architecture (arch/lkl)
<br><br>
- transparent architecture bind (as CPU arch)
 - require no modification to the other
- 2800 LoC
 - thread information (struct thread_info)
 - irq, timer, syscall handler
 - access to underlying layer by host_ops

>>>

## 3. Application interface

<div class="left" style="width: 40%">
<img src="figs/lkl-arch-api.png" width=100%>
</div>

<br><br><br>
- Case 1: use exposed API (LKL syscall)
- Case 2: use host libc (LD_PRELOAD)
- Case 3: extend (alternative) libc

>>>

## Case 1: use exposed API (LKL syscall)

- call entry points of LKL kernel
 - *lkl_sys_open()*, *lkl_sys_socket()*
- *almost* same as ordinal syscalls
 - return value, *errno* notification are different
- can use LKL syscall *and* host syscall <br> simultaneously
 - read ext4 file by lkl_sys_read() => <br>write into host (Windows) by write()

<div class="right" style="width: 30%">
<img src="figs/lkl-syscall-1.png" width=100%>
</div>

>>>
## Case 2: hijack host standard library

- dynamically replace symbols <br> of host syscalls (of libc)
 - LD_PRELOAD
 - socket() => lkl_sys_socket()
- can use host binary (executable) as-is
- limitation of replaceable symbols
- needs syscall translation on non-linux host

<div class="right" style="width: 30%">
<img src="figs/lkl-syscall-hijack.png" width=100%>
</div>

>>>
## Case 3: extend (alternative) libc

- **only** call LKL syscall with our own libc
- also introduce as a virtual CPU architecture
- a program can link this instead of host libc
 - can't access to (underlying) host resource <br>directly via this lkl syscall
- as a patch for musl libc

<div class="right" style="width: 30%">
<img src="figs/lkl-syscall-musl.png" width=100%>
</div>

>>>

## Usecase (applications)

- Use Case 1: instant kernel bypass
- Use Case 2: programs reusing kernel code in userspace
- Use Case 3: Unikernel

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

<div class="right" style="width: 40%">
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


---

# Performance study

>>>

## Conditions

- ThinkStation P310 x2
 - CPU: Intel Core i7-6700 CPU @ 3.40GHz (8 cores)
 - Memory: 32GB
 - NIC: X540-T2
- Linux 4.4.6-301 (x86_64) on Fedora 23
 - Linux bridge (X540 + tap)
 - *no DPDK*... can't with hijack, etc
- netperf (git ~v2.7.0)
 - netserver (native)
 - netperf (varied)

<img src="figs/bench-topo.png" width=50%>

>>>

## Conditions (cont'd)

- 4 combinations
 - netperf (sendmmsg) + host stack (**native**)
 - \+ hijack library (**hijack**)
 - \+ frankenlibc/lkl (**lkl-musl**)
 - netperf (sendmmsg) + lkl extension + frankenlibc (**lkl-musl (skb pre alloc)**)
- pinned a processor
 - using `taskset` command

>>>

<img src="2016-06-07/tcp-rr.png" width=80%>

### TCP_RR (netperf)

>>>

<img src="2016-06-07/udp-stream.png" width=80%>

### UDP_STREAM (netperf)

>>>

<img src="2016-06-07/udp-stream-pps.png" width=80%>

### UDP_STREAM (pps, netperf)

>>>

## (ref.) NUSE results

<img src="figs/nuse-benchmark-host.png" width=100%>

- 1024 bytes UDP, own-crafted tool
- as of Feb. 2015


>>>

## Observations (of benchmark)



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

### Use Case: Integration with ns-3 network simurator

<div class="left" style="width: 40%">
<video data-autoplay src="figs/ns-3-dce-mptcp-linux3.5.7-8subf.m4v"></video>

Visualize Linux Multipath-TCP experiments
</div>



- Investigation of network issues
- ns-3 network simulator
- plenty of models
 - NIC
 - node movement
 - traffic
 - timing
- multiple node running inside a single process
 - by dlmopen(3) (avoid symbol conflicts)
 - syscall re-implementation (node distinction)
- **100 % reproducible** (experiment, bugs)

>>>

### Use Case: Integration with ns-3 network simurator

- test tools for kernel network stack
 - Regression tests (in a complex scenario)
 - 100 % reproducible (virtual, deterministic clock)
 - code coverage measurements (with pseudo random variables of ns-3)
 - memory issue debug with Valgrind

<img src="figs/jenkins.png" width="30%">
<img src="figs/gcov.png" width="30%">
<p>
<img src="figs/rocketfuel.png" width="20%">
<img src="figs/valgrind.png" width="30%">

Note:
note： not ported to LKL yet



