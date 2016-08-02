## Toward Freeform Internet
#### an introduction to my research (hacking) activities

<span>
<br>
<br>
<br>
<br>

### Hajime Tazaki 
IIJ Innovation Institute
<br>

2016/08

</span>

---


## Distributed System

  <img src="figs/baran-distributed.png" width="120%"/>

<span>
P. Baran, On Distributed Communications Networks, IEEE Transactions on Communications Systems, 1964
</span>

Note:

I would like to start with this figure

This is my favorite picture, which was designed in 1950 by Paul
Baran, and the Internet has been growing based on this idea, I
believe.

The key concept is 'distributed system', which 
1) everybody can be replaced by everybody else, 
2) everybody can do any ideas they wish


>>>

## I'm dreaming about ...

- my ultimate goal
 - Make the Internet more *democratic*
 - **Freeform Internet**
<!--
even though the reasons for those problems (deployment issue, architectural issue) are not technical issue, 
but the others like political worries, stuck community, 
technologies should be ready to answer the real problem which we faced.
-->

- by 'democratic' I meant
 - *any entities (users, software, networks) CAN do anything as they wish IF there is (even small) a concensus*

Note:
--- this is a lesson learnt from 3.11

>>>

  <img src="figs/baran-distributed.png" width="60%"/>
  <img src="figs/baran-distributed-nhop.png" width="30%"/>


>>>

### 1. Floating Ground Architecture

- Redesign of the mobile network architecture
 - but not from scratch <br> (no clean-slate)
 - redesign the shape of last 1-hop links
- Contribution
 - improve the handoff performance
 - less mobility signaling

<div class="right" style="width: 30%">
  <img src="figs/floating-ground.png" width="100%"/>
</div>

<br>
<br>
<br>

<span>
*Tazaki et al. Floating ground architecture: overcoming the one-hop boundary of current mobile internet, ACM/IEEE ANCS 2012*
</span>

>>>

### 2. Direct Code Execution

- An extension to ns-3 network simulator
- Benefits
 - Implementation **realism**
 - in **controlled topologies** <br> (including wireless environments)
 - Model **availability**
 - fully **reproducible**
 - Debugging a whole network <br> within a single process

<div class="right" style="width: 40%">
  <img src="figs/dce-arch.png" />
</div>

<br>
<br>

<span>
*Tazaki et al. Direct Code Execution: Revisiting Library OS Architecture for Reproducible Network Experiments, ACM CoNEXT 2013*
</span>

Note:
- Limitations
 - Not as scalable as pure simulation
 - Tracing more limited
 - Configuration different


>>>

<!-- .slide: class="two-floating-elements" -->
## DCE in a nutshell
#### a framework to use actual implementation on simulations

> Lightweight virtualization of kernel and application processes, interconnected by simulated networks



<video data-autoplay src="figs/ns-3-dce-mptcp-linux3.5.7-8subf.m4v"></video>

- MPTCP v0.86 with ns-3-dce, 8 sub flows

Note:
 - https://www.youtube.com/watch?v=fN_nv7RdFm8

- MPTCP over LTE (IPv4) and Wi-Fi (IPv6) 
 - https://www.youtube.com/watch?v=rvF-yreZElQ


- Benefits
 - Implementation **realism**
 - in **controlled topologies** (including wireless environments)
 - Model **availability**
 - fully **reproducible**
 - Debugging a whole network within a single process
- Limitations
 - Not as scalable as pure simulation
 - Tracing more limited
 - Configuration different

>>>

### 3. Reusing Linux as a Library

- monolithic kernel as a multi-purpose library
- Benefit
 - *operating system personality*
 - userspace library has less deployment cost
 - **bypass**ing kernel
 - tiny operating system (unikernel)

>>>

## A view of Network Stack

- Why in kernel space ?
 - the cost of packet was <br>expensive at the era ('70s)
 - now much cheaper

- Getting fat (matured) <br>after decades
 - code path is longer <br> (and slower)
 - hard to add new features
 - faced unknown issues

<div class="right" style="width: 50%">
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

- **Socket API** sucks
 - StackMap, MegaPipe, uTCP, SandStorm, IX
 - New API: no benefit with existing applications
- Network stack in **kernel space** sucks
 - FastSocket, mTCP, lwip (SolarFlare?)
- **Compatibility** is (also) important
 - rumpkernel, libuinet, Arrakis, IX, SolarFlare
- **Existing programming model** sucks
 - SeaStar

>>>

## Challenges

- (Obviously) Performance
 - memory copy
 - emulation overhead
 - (lack of performance improvements) features


>>>
## Performance Study
### Conditions

- ThinkStation P310 x2
 - CPU: Intel Core i7-6700 CPU @ 3.40GHz (8 cores)
 - Memory: 32GB
 - NIC: X540-T2
- Linux 4.4.6-301 (x86_64) on Fedora 23
 - Linux bridge (X540 + tap/raw socket)
 - *no DPDK*... can't with hijack, etc
- netperf (git ~v2.7.0)
 - netserver (native)
 - netperf (varied)

<img src="figs/bench-topo.png" width=50%>

>>>

## (ref.) LibOS results (as of Feb. 2015)

<img src="figs/nuse-benchmark-host.png" width=100%>

- 1024 bytes UDP, own-crafted tool

- throughput: <10% of Linux native

>>>

### Recent implementation (LKL), TCP_STREAM (netperf)

<img src="2016-07-30/tx/tcp-stream.png" width=80%>

>>>


## Everything is on the web

- https://github.com/thehajime/

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

# Backup

>>>

## Our approach: Kernel bypass

<div class="left" style="width: 40%">
<br>
<br>
<img src="figs/anykernel-pre.png" width=100%>
</div>


<div class="right" style="width: 40%">
<img src="figs/anykernel-post-en.png" width=100%><!-- .element: class="fragment" data-fragment-index="1" -->
</div>

>>>

## three projects

1. **Floating ground architecture** (IEEE/ACM ANCS '12)
 - tackled with current issue of non-flexible mobile network
 - solve with *virtual overlay* (called Floating Ground) in the middle
2. **Direct code execution** (ACM CoNEXT '13)
 - tackled with 3 issues of network protocol experiments
  - 1) lack of timing realism, 2) lack of functional realism, 3) lack of debuggability
 - run real kernel implementation in a userspace process with **indirections**
3. **Network stack in userspace/Linux LibOS** (ongling)
 - Problem: no network stack personality on a system
 - solved with another indirection by dynamically translating an application code
 - end-to-end principle for network stack (toward E2E principle for OSes)
