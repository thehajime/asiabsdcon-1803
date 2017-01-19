# 2016 Research Review

<span>
<br>
<br>
<br>
<br>

### Hajime Tazaki 
IIJ Innovation Institute
<br>

2017/1, IIJlab camp Winter '17
</span>

---

## Outlook (of what I'm doing)

- Operating system *re-construction*
 - for a selfishness (personality,specialization)
 - but respect the past dozen effort <br>
   (reusing Linux kernel)

- *Precludes any ossification in software (i.e., Internet)*

- e.g.
 - Userspace network stack
 - Lightweight guest operating system (e.g., unikernel)
 - Reproducible network experiment (simulation)


<div class="right" style="width: 15%">
<img src="figs/nuse-overview.png" width=100%>
<img src="figs/CloudOSDiagram.png" width=100%>
<video data-autoplay src="figs/ns-3-dce-mptcp-linux3.5.7-8subf.m4v" width=100%></video>
</div>

>>>

## 2016 Plan

1. Performance investigation on userspace network stack
1. Show the PoC of unikernel using LKL
1. Upstream LKL code to Linux mainline

<br>
<br>
<br>

- Improve the presence of IIJlab

>>>

### 1. Performance investigation on userspace network stack

- Goals
 - Identify bottlenecks
 - Understand strength/weakness of userspace network stack

- Activities
 - Benchmarks with synthesized traffic (netperf)
 - Measurement of timing accuracy with packet pacing + TCP-BBR (Linux 4.9)

>>>

## Performance improvements (so far)

- Offload features (TSO, checksum by google)
- thread optimization (by Google)
- zero-copy (WIP, by Google)
- use of green-thread (myself)

>>>

## Benchmarks

- ThinkStation P310 x2
 - CPU: Intel Core i7-6700 CPU @ 3.40GHz (8 cores)
 - Memory: 32GB
 - NIC: X540-T2
- Linux 4.4.6-301 (x86_64) on Fedora 23
 - Linux bridge (X540 + tap/raw socket)
- netperf (git ~v2.7.0)
 - netserver (native)
 - netperf (varied)

<img src="figs/bench-topo.png" width=50%>

>>>

### Results (TCP_RR, TCP_STREAM, UDP_STREAM)

<img src="2016-07-12/tx/tcp-rr.png" width=30% onclick="window.open(this.src)">
<img src="2016-07-12/tx/tcp-stream.png" width=30%>
<img src="2016-07-12/tx/udp-stream.png" width=30%>

- LKL won by (native) Linux in some conditions

>>>

### Timing accuracy

```
        netperf(client)                    netserver
        +------+        +---------+       +--------+
        |      |        |         |       |        |
        |sender+--------+middlebox+------+|receiver|
        |      |        |         |       |        |
        +------+        +---------+       +--------+
        cc: BBR         1% pkt loss
        qdisc: fq       100ms delay
```

- BBR uses packet pacing with `fq` queue discipline to avoid burst
- w/ high-resolution timer (nano second resolution)

>>>

### Timing accuracy

<img src="lkl-bbr-result/170111/hrtimer-delay.png" width=50%>

- timer accuracy := (ts of a timer fired) - (ts of a timer scheduled)

- timer event
 - delayed schedule of a packet (i.e., pacing)
 - trying to decrease the queue length of middleboxes

>>>

## Other (non-expected) output

- Proposed a change to TCP-BBR (in low-resolution timer case)

https://groups.google.com/d/msg/bbr-dev/sNwlUuIzzOk/ztFEA8jCDAAJ

>>>

### 1. Performance investigation on userspace network stack

- **What I did**
 - throughput/delay improvements
 - investigation for concerns about userspace facilities
- **Overall (self) rating**
 - so so

>>>

### 2. Show the PoC of unikernel using LKL

- Unikernels are[1]
 - specialized
 - single address space
 - constructed by using<br> library operating systems
- Single purpose
 - massive guests cloud

- Use Linux kernel code (as a library) <br> to build unikernel
 - *generic* library should be used  <br> with different application

<div class="right" style="width: 40%">
<img src="figs/CloudOSDiagram.png" width=100%>
</div>

<small>
[1] Madhavapeddy et al., "Unikernels: Rise of the virtual library operating system.", ACM Queue 11.11 (2013): 30.

</small>

>>>
### 2. Show the PoC of unikernel using LKL (cont'd)

- Hello world-ish applications
- Underlying platform
 - qemu (x86_64, arm)
 - POSIX-ish OS
 - (not yet) Xen, bare-metal
- Upper applications
 - nginx, ghc, netperf

<div class="right" style="width: 40%">
<img src="figs/frankenlibc-stack.png" width=100%>
</div>

>>>
### 2. Show the PoC of unikernel using LKL (cont'd)

- **What I did**
 - Implement rump-hypercall interface
 - provide LKL-specific std library
- **Overall (self) rating**
 - good

>>>
### 3. Upstream LKL code to Linux mainline

- no recent communications (on LKML/netdev)

- **What I did**
 - organized a conference (netdev1.2)
 - talk locally
- **Overall rating**
 - not good
 - keep doing further

>>>

## Publications

- Refereed papers
 - Yuta Tokusashi, Yohei Kuga, Ryo Nakamura, Hajime Tazaki, Hiroki Matsutani, mitiKV: An Inline Mitigator for DDoS Flooding Attacks, Internet Conference 2016
 - 金津 穂, 田崎 創, 宇夫 陽次朗, 山田 浩史, クラウド環境を指向するライブラリ OS の起動にかかる時間の比較およびその分類, Internet Conference 2016

- Talks
 - 第38回 Linux Kernel Library: 再利用可能なモノリシックカーネル / 2016年4月
 - Direct Code Execution as a regression test framework for Linux networking, AIST invited talk vol.1 July 22, 2016
 - Linux Kernel Library - Reusing Monolithic Kernel, AIST invited talk vol.2 July 22, 2016

>>>

## Other activities

- Conference organization
 - TPC Chair: Workshop on ns-3 2016
 - TPC: Workshop on ns-3 2017
 - TPC: International Workshop on Vehicular Adhoc Networks for Smart Cities (IWVSC '16)
 - TPC Chair: Internet Conference 2016 (IC2016)
 - Organization Chair: Linux netdev 1.2 conference

>>>

## Open source

- LKL
 - https://github.com/lkl/linux
 - https://github.com/libos-nuse/frankenlibc
 - https://github.com/libos-nuse/rumprun
- ns-3
 - http://code.nsnam.org/ns-3-dce

>>>

## Budget: R/D HW/SW/etc.

none.

<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>

>>>

## Budget: Trip

- Oversea
 - (MUST) WNS3, Seattle/USA (June)
 - ~~(SHOULD) OSIO, Cambridge/UK (April)~~
 - ~~(SHOULD) APsys 16, Hong Kong/CN (August)~~
- Domestic
 - (MUST) Linux netdev1.2, Tokyo (October)
 - (SHOULD) IC2016, Tokyo (October)

- not planned
 - USENIX ATC 16, Denver/USA (June)
 - JAIST meetup, JAIST (December)



---

# 2017 Research Plan

>>>

## Research Plan

1. (cont'd) Performance investigation on userspace
 - measurement, analysis, and improvement
1. (cont'd) LKL Unikernel
 - Run real applications/services
 - Run on commercial cloud service
1. <span style="color: gray"> (cont'd, LKL upstreaming)</span>

>>>

## Budget: R/D HW/SW/etc.

- iPad mini
 - for paper reading

>>>

## Budget: Trip

- Oversea
 - (SHOULD) ATC 17, Santa Clara/USA (July)
 - (SHOULD) netdev 2.1, Montreal/Canada (April)
 - (SHOULD) netdev 2.2, Portugal (October)
 - (MAY) FOSDEM, Brussels/Belgium (February)
- Domestic
 - (MUST) IC 2017, ?? (October/November)



>>>

## Paper story line
### (for userspace network stack)

1. Current OS sucks
 - no user-oriented extensibility
 - no speedy userspace network stack & feature-richness
1. Contributions
 - benefit both **userspace flexibility** and **feature richness**
 - Understand concerns about userspace facilities (timing accuracy)
1. Implementations
 - to improve speed
 - to improve timing accuracy
1. Experiments
 - benchmark (expectation: comparable to native)
 - timing accuracy (high-resolution timer in userspace)

>>>

## Schedule

- (2017.1) IIR deadline
- (2017.2) USENIX ATC paper
- (2017.2) netdev2.1 proposal
- (2017.4) netdev2.1
- (2017.7) USENIX ATC
- (2017.8) netdev2.2 proposal
- (2017.10) netdev2.2
- (2017.10,11?) IC2017


---


# Backups

---

1. Background
1. Motivations
 - Why we're working on this
 - generalization has beaten by specialization
1. Solutions
 - How to address the issue ?
1. Evaluation
 - How this is good idea or not
 1. Throughput
 1. Timer accuracy
1. Progress
 - How far to the goal ?
 - proposed a change to TCP-BBR
 - investigation on high-resolution timer (hrtimer)
1. Ongoings
 - complete hrtimer investigation (accuracy issues)
 - champion data collections (best case, + worst case)


---

## Motivations (why I'm doing this ?)

1. Software is no longer soft
 - i.e., new feature (in operating system core) is hard to deploy 
1. Existing userspace network stacks escape from the feature-richness
 - i.e., Linux sucks

<br>
- ex. Userspace facilities are not considered to do a critical job
 - lack of privilege, overhead (of accessing various resources)

> Providing timing accuracy in userspace TCP stack is a ridiculous idea.
>
> Dave S. Miller (@ netdev 1.2 tokyo)

>>>

# Motivations (cont'd)

- Restructure a network stack to address those issue
- Don't reimplement from scratch
 - Reuse existing code as much as possible

- Anykernel architecture

---

# Solutions

- Linux kernel library
 - A library of Linux (kernel code)
 - Anykernel architecture
- No port, no reimplement, no full-scratch
 - Benefit feature-richness

<div class="right" style="width: 35%">
<img src="figs/lkl-arch.png" width=100%>

<br>
<small>
Purdila et al., LKL: The Linux kernel library, RoEduNet 2010.
</small>
</div>

>>>

## LKL in a nutshell

- .

>>>

## Features

- .

---

## Evaluation


- How this is good idea or not
 1. Throughput
 1. Timing accuracy

>>>

## application goodput (TCP_STREAM)


>>>

## application goodput (TCP_RR)

- ping/pong (req/rep)

>>>

## application goodput (UDP)



>>>

## How a timer (clock related event) works in LKL ?

- clocksource
 - read/provide the current clock value
 - a system call (clock_gettime(posix), GetSystemTime(win32))
- clockevent
 - (host's) timer API
 - e.g., POSIX timer (timer_create(posix), CreateTimerQueue(win32))

- clockevent is relatively expensive than usual kernel
 - hw interrupt (timer)

- clocksource read is not that much
 - a system call v.s. a (single) instruction
 - clock-related syscalls are not that much expensive (vdso)

>>>

<img src="lkl-bbr-result/170111/hrtimer-delay.png" width=65%>

- timer accuracy := (ts of a timer fired) - (ts of a timer scheduled)

- timer event
 - delayed schedule of a packet (i.e., pacing)
 - trying to decrease the queue length of middleboxes


