## Linux rumpkernel
### a librarified monolithic kernel

<span>
<br>
<br>
<br>
<br>

**Hajime Tazaki**

IIJ Research Laboratory

March, 2018, AsiaBSDCon 2018
</span>

---

## Intro

- i'm going to talk about *Linux is great* (sorry)
- but Linux or xxxBSD doesn't matter

- re-composable, re-usable, flexible operating system kernel <br>should make everyone happy

Note:

- Linux in BSD conference
- Network stack evolution
- with NetBSD derived technology (rump)

>>>

## Who I am ?

- Researcher at IIJ Research Laboratory
- working for the Internet


>>>

## The original Internet

  <img src="figs/baran-distributed.png" width="70%"/>

- packet switching network
 - a basis of end-to-end principle
 - a basis of the hugest platform

<small>
P. Baran, On Distributed Communications Networks, IEEE Transactions on Communications Systems, 1964
</small>


Note:

His motivation is to create a sustainable network architecture against nuclear attack, which previous circuit-switching network based on centralized or de-centralized network cannot achieve.  By a newly introduced idea, packet, a chunk of small split message, each node in a distributed network should just forward a packet based on partial header information described in a packet, thus each intermediate node in a network can be replaced by others.

This is super powerful idea, which gives us a flexibility to the message format (a.k.a. protocols), and everybody who wish to can join the network with a small amount of costs.

Thus the Internet has grown to the hugest platform in the world and be a winner.
And important characteristics is people can easily extend network protocols as the need of users.

>>>

## Today's internet

- not yesterday's Internet

<img src="figs/complex-mind.jpg" width="30%"/>
<img src="figs/goverment-control.jpg" width="30%"/>
<img src="figs/security-fast.png" width="15%"/>

various stake holders / controlled system / security fast

<small>
- refs:
 - https://justimagine.aurecongroup.com/solving-complex-problems-forget-what-you-currently-know/
 - https://kentforliberty.liberty.me/letting-government-control-you/


Note:
**Should be self-organized system**

- My dream about free-form internet
- can we ? no..

- pictures
 - (middlebox jam)
 - (governmental control)
 - (security-first: default deny forwarding)

>>>

## Today's internet (cont'd)

<img src="figs/no-more-e2e.png" width="80%"/>

- a packet is hard to deliver to the others without any modifications

<small>
- ref:
https://www.slideshare.net/obonaventure/innovation-is-back-in-the-transport-and-network-layers

>>>

### End of evolution/innovation ??

- internet is mature enough (that we don't have to modify)
- we can create another universe
<br>
<br>
- are we satisfied ?
 - people want to innovate but the system is not ready

>>>

## Questions

- why do you want to extend your system ?
 - want to put new idea (I have a great protocol)
 - want to refresh design (socket API sucks)
 - want to optimize implementations (too slow for me)
 - want to secure codes (security fast)

---

## What's the matter ?

>>>

## Problems

- more ossification/no innovation  <!-- .element: class="fragment grow" data-fragment-index="1" -->
- no more end-to-end  <!-- .element: class="fragment shrink" data-fragment-index="1" -->
- no more experimental platform  <!-- .element: class="fragment shrink" data-fragment-index="1" -->
- more low-quality codes  <!-- .element: class="fragment shrink" data-fragment-index="1" -->
- more waste of time  <!-- .element: class="fragment shrink" data-fragment-index="1" -->


<div class="right" style="width: 20%">
<img src="figs/problem.jpg" width="100%"/>

<small>
https://pixabay.com/en/question-problem-think-thinking-622164/
</div>

>>>

## Protocol ossification
- two obstacles for new protocol deployment
 1. middlebox
 1. host OS

>>>

## Ossification: middlebox

<img src="figs/pkt-process-router-olivier.png" width="80%"/>

TCP segments processed by a router
<small>
- ref:
 - https://www.slideshare.net/obonaventure/innovation-is-back-in-the-transport-and-network-layers

>>>

### Ossification: middlebox (cont'd)

<img src="figs/pkt-process-nat-olivier.png" width="80%"/>

TCP segments processed by a NAT router
<small>
- ref:
 - https://www.slideshare.net/obonaventure/innovation-is-back-in-the-transport-and-network-layers

>>>

### Ossification: middlebox (cont'd)

<img src="figs/pkt-process-mbox-olivier.png" width="80%"/>

possible TCP segments processed by typical middlebox today
<small>
- ref:
 - https://www.slideshare.net/obonaventure/innovation-is-back-in-the-transport-and-network-layers


>>>

## Ossification: host OS


<img src="figs/tcp-opt-kensuke.png" width="60%"/>

The deployment of protocol extensions takes long

- Standardized
 - WS,TS: 1992 (RFC1323)
 - SACK: 1996 (RFC2018)
- OS
 - WS, TS: Win 2000/Linux(1999)
 - SACK: defaulted 1999 (Linux), 2004 (Win)

<small>
Fukuda, Kensuke. "An Analysis of Longitudinal TCP Passive Measurements (Short Paper)." Traffic Monitoring and Analysis 40: 29.
</small>

>>>

## Ossification: host OS (cont'd)

- updating base kernel is not an easy task
 - Android still uses older kernel
 - container guests use the host kernel (for network stack)
<br>
<br>

<img src="figs/android-platform-distribution-version.png" width=60%>
<small>
- Android OS distribution with the base Linux kernel version
https://developer.android.com/about/dashboards/index.html
(taken Nov. 2017)


Note:
takeaway: people tend to use older kernel even if vendors release newer software

>>>

## Design pattern

- Multipath TCP (mptcp)
 - an extension to<br> (traditional) TCP
 - multipath communication
 - RFC6824 (experimental)
 - application compatibility <br> (unlike SCTP)

- Good design ?
 - middlebox friendly => **OK**
 - unmodified application => **OK**

<div class="left" style="width: 50%">
<img src="figs/mptcp-tessares-fig.png" width=100%>
</div>

<small>
http://blog.multipath-tcp.org/blog/html/2015/12/25/commercial_usage_of_multipath_tcp.html

Note:
- successfully deployed by apple (since ios7)
 - modified kernel => **??**

>>>

## Ossification: Google's answer

- QUIC (Quick UDP Internet Connection)

- a transport protocol over UDP
- 7% of Internet traffic *1
- why UDP ?
 - **middlebox friendly**
 - with encrypted payload middlebox can't intercept
- why UDP (cont'd) ?
 - can be implemented in userspace
 - **no need to upgrade host OS**

<small>
*1 The QUIC Transport Protocol: Design and Internet-Scale Deployment, ACM SIGCOMM 2017

>>>

## Ossification: can others deploy such a way ?

- no <!-- .element: class="fragment" data-fragment-index="1" -->
- only a giant can <!-- .element: class="fragment" data-fragment-index="1" -->

<div class="left" style="width: 50%">
<img src="figs/giant-dora.jpg" width=100%> <!-- .element: class="fragment" data-fragment-index="1" -->
</div>


>>>

## If you face obstacles

- you would implement from scratch
 - as a name of *specialization*

- lack of maturity of an OS history
- more low-quality codes
- more waste of time (reinventing a wheel)

Note:
- no productive implementation (e.g., kernel bypass)

>>>

## summary of problems
- today's internet is not the original internet
- **no more end-to-end**
<br>
<br>
- to put a break-through
 - be a part of giant (which I won't)
 - or thinks differently ?

---

## Alternatives

- Userspace stack
 - lwip (2002~)
 - Arrakis [OSDI '14]
 - IX [OSDI '14]
 - MegaPipe [OSDI '12]
 - mTCP [NSDI '14]
 - SandStorm [SIGCOMM '14]
 - uTCP [CCR '14]
 - FastSocket [ASPLOS '16]
 - SolarFlare (2007~?)
 - libuinet (2013~)
 - SeaStar (2014~)
 - Snabb Switch (2012~)


- lightweight VM
 - MirageOS [ASPLOS '13]
 - OSv [USENIX '14]
 - ClickOS [NSDI '14]


*Most of them lack feature-richness, or one-shot porting w/o latest feature updates* <!-- .element: class="fragment" data-fragment-index="1" -->

Note:
 - rumpkernel [ATC '09]
 - StackMap [ATC '16]

>>>

## Alternatives (cont'd)

- MegaPipe [OSDI '12]
 - *outperforms baseline Linux .. **582%** (for short connections).*
 - New API for applications (no existing applications benefit)<!-- .element: class="fragment" data-fragment-index="1" -->
- mTCP [NSDI '14]
 - *improves the performance ... **by a factor of 25** compared to the latest Linux TCP stack*
 - implement with very limited TCP extensions<!-- .element: class="fragment" data-fragment-index="1" -->
- SandStorm [SIGCOMM '14]
 - *our approach with the FreeBSD and Linux stacks ..., **demonstrating 2-10x** improvements*
 - specialized (no existing applications benefit)<!-- .element: class="fragment" data-fragment-index="1" -->
- Arrakis [OSDI '14]
 - *improvements of **2-5x in latency and 9x in throughput** .. to a well-tuned Linux implementation.*
 - utilize simplified TCP/IP stack (lwip) (loose feature-rich extensions)<!-- .element: class="fragment" data-fragment-index="1" -->

Note:
- IX [OSDI '14]
 - *improves the throughput ... **by up to 3.6x** and reduces tail latency **by more than 2x***
 - utilize simplified TCP/IP stack (lwip) (loose feature-rich extensions)<!-- .element: class="fragment" data-fragment-index="1" -->
- netmap [ATC '12]
 - *a single core ... can send or receive 14.88 Mpps (**the peak packet rate** on 10 Gbit/s links).*


>>>

## Does speed matter ?

- nope, it's one of metric of a system
- improving numbers **often** sacrifices features/functions

<img src="figs/balance.png" width=35%>

> As the old joke goes, writing a TCP/IP stack from scratch over the weekend is easy, but making it work on the real-world Internet is more difficult [1].

<small>
- [1] Antti Kantee, Rump Kernels No OS? No Problem!, USENIX login; October, 2014

>>>

## Our goal
- Respect the implementation (and experience) of past decades
- Accelerate the innovation of network stack

***discover new values through the past studies***


---

## The project

>>>

## Anykernel
- Anykernel: originally in NetBSD rump kernel 
<small>
>We define an anykernel to be an organization of kernel code which allows the kernel's **unmodified** drivers to be **run in various configurations** such as application libraries and microkernel style servers, and also as part of a monolithic kernel.  -- Kantee 2012.
</small>

- using (*unmodified*) high-quality code base of monolithic kernel
 - on different environment in different shape
 - by **gluing** additional stuffs


Note:

様々仮想化技術はあり、用途もあるが、「まだ別の仮想化技術が便利で必要で
すよ」という話と、「その仮想化技術にまだ課題がありますよ」という話を、
Linux カーネルを題材としてお話。

>>>

<div class="left" style="width: 40%">
<img src="figs/anykernel-pre.png" width=100%>
</div>

<div class="right" style="width: 40%">
<img src="figs/anykernel-post-en.png" width=100%> <!-- .element: class="fragment" data-fragment-index="1" -->
</div>



- transforming a monolithic kernel code into an Anykernel  <!-- .element: class="fragment" data-fragment-index="2" -->

Note:

- History: Rump kernel
 - Detail:

>>>

## Linux Kernel Library (LKL)

- a library (liblkl.{so,a})
- out-of-tree architecture <br> (h/w-independent)
- run Linux code on various ways
 - with a reusable library
- h/w dependent layer
 - on Linux/Windows <br>/FreeBSD uspace, <br> unikernel, on UEFI, Android
 - network simulator (ns-3)
- code
 - 2.4KLoC (h/w independent)
 - 6.6KLoC (h/w dep)

<div class="right" style="width: 40%">
<img src="figs/lkl-arch-new.png" width=100%>
</div>

>>>

## LKL: internals

- core design
 - outsource machine dependent code
 - keep application and<br> kernel code untouched
- components
 1. host backend (host_ops)
 1. CPU independent arch. (arch/lkl)
 1. application interface

<div class="right" style="width: 40%">
<img src="figs/lkl-arch-new.png" width=100%>
</div>

>>>

<img src="figs/lkl-arch-new.png" width=30%>
<img src="figs/baremetal-rumpstack.png" width=60%>

>>>

### 1. host backend

<div class="left" style="width: 45%">
<img src="figs/lkl-arch-host.png" width=100%>
</div>

- environment *dependent* <br>part
 - unify an interface across <br> different platforms
 - (rump-hypercall like)
- device interface with **Virtio**
 - block device <=> disk image
 - networking <=> TAP, <br> raw socket, DPDK, VDE

>>>
### 2. CPU independent architecture

<div class="left" style="width: 45%">
<img src="figs/lkl-arch-kernel.png" width=100%>
</div>



architecture (arch/lkl)
<br><br>
- transparent architecture bind <br> (as CPU arch)
 - require no modification to  <br>the other
- implementation
 - thread information (struct <br> thread_info)
 - irq, timer, syscall handler
 - access to underlying layer <br> by host_ops

>>>

### 3. Application interface

<div class="left" style="width: 45%">
<img src="figs/lkl-arch-api.png" width=100%>
</div>

<br><br><br>
1. use exposed API (LKL syscall)
2. use host libc (LD_PRELOAD)
3. extend (alternative) libc

>>>

### API 1: use exposed API (LKL syscall)

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
### API 2: hijack host standard library

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
## API 3: extend (alternative) libc

- **only** call LKL syscall with our own libc
- also introduce as a virtual CPU architecture
- a program can link this instead of host libc
 - can't access to (underlying) host resource <br>directly via this lkl syscall
- as a patch for musl libc

<div class="right" style="width: 30%">
<img src="figs/lkl-syscall-musl.png" width=100%>
</div>

---

## Usages

>>>

### NUSE (Network Stack in UserspacE)

- What ?
 - Install/Use alternate network stack (i.e., TCP/IP)
 - but it's a full-fledged code (Linux)
 - host network stack isn't involved

- Why ?
 - because kernel is hard to touch
 - Android (long delivery time)
 - container (e.g., docker: shared by others)

<div class="left" style="width: 30%">
<img src="figs/lkl-android-hijack.png" width=100%>
</div>


>>>

## Demo (Android+mptcp)


>>>

## unikernels

- What ?
 - OS instance w/ a single process
 - on bare-metal
 - on hypervisor
 - on userspace program
- **cross-compile** with alt-libc
 - rumprun (Antti Kantee)
 - frankenlibc (Justin Cormack)

- Why ?
 - small footprint
 - quick instantiation

<div class="right" style="width: 40%">
<img src="figs/CloudOSDiagram.png">

<small>
- http://www.linux.com/news/enterprise/cloud-computing/751156-are-cloud-operating-systems-the-next-big-thing-
</small>
</div>

Note:
originally rumprun and frankenlibc are designed to cooperate with NetBSD rump, but can use with LKL (Linux) ofc.

>>>

### rumprun/frankenlibc unikernel

<img src="figs/frankenlibc-stack.png" width=45%>

>>>

## Demo (frankenlibc)

<img src="figs/nginx-frankenlibc-freebsd.png" width="40%">

Note:
- this slides (reveal.js/nginx/lkl+frankenlibc/FreeBSD (on linux))

>>>

## Service Function Chain (SFC)

- What ?
 - SFC by Unix pipe and LKL
 - NF in a shell command
 - `ping.sh | nat.sh | pfilter.sh`
- Why ?
 - a chain w/ VMs is heavyweight
 - Unix pipe is useful enough (e.g., packet filter by `grep`)

<img src="figs/stdpkt-virtio.png" width=50%>

>>>

## Demo

>>>

## How ping looks like ?

- generate raw data to stdout
- next program can receive <br> from stdin

<div class="right" style="width: 50%">
<img src="figs/ping-stdpkt.gif" width=100%>
</div>

>>>

### grep command as firewall


<div class="left" style="width: 40%">
<br>
<br>
<br>
<br>
<img src="slide-figs/stdpkt-grep-topo.png" width=100%>
</div>


<div class="right" style="width: 60%">
<img src="figs/stdpkt-grep-acl.gif" width=100%>
</div>

>>>

## NAT, packet filter


<div class="left" style="width: 45%">
<ul>
<li>microbenchmark
<li> netperf, iptables (NAT/ACL)
<li> measure boot latency
</ul>

<img src="figs/stdpkt-nat-acl.png" width=100%>
</div>


<div class="right" style="width: 50%">
<img src="figs/stdpkt-nat-acl-tcp.png" width=100%>
TCP googput

<img src="figs/stdpkt-boot-latency.png" width=100%>
Boot latency
</div>


*quick boot, reasonable fwd/filter performance, w/o optimizations*  <!-- .element: class="fragment" data-fragment-index="1" -->

>>>


## Network simulation

- What ?
 - network simulation (ns-3) <br> with Linux network stack
- Why ?
 - less abstraction
 - more **realistic**
 - fully **reproducible**

<div class="right" style="width: 45%">
 <!-- img src="figs/dce-arch.png" /-->
 <video data-autoplay src="figs/ns-3-dce-mptcp-linux3.5.7-8subf.m4v"></video>
</div>

Note:
For those who are not familiar with network simulation, NS is a framework which users can deeply investigate a particular network protocol with an abstracted implementation of the protocol.  abstraction is useful in order to focus on the protocol, but is sometime harmful if various external sources affect to the specific protocol behavior.

ns-2, 3, omnet, Qualnet, matlab, 

>>>

## Your experiment

<img src="figs/1link-topo.png" width=50%>

- easy to create in your laptop with VM (UML/Docker/Xen/KVM) 
- ***only IF*** the test is enough to describe

>>>

## Your experiment (cont'd)

<div class="left" style="width: 45%">
<img src="figs/rocketfuel.png" width=100%>
</div>

- huge resources to conduct a test
- not likely to **reproduce**
- tons of **configuration scripts**
- running on different machines/OSes
 - controling is troublesome
 - distributed debugger... 

Note:

## (picture of complex topology)

>>>

## Debugging/Testing

<img src="figs/umip-gdb.png" width="45%">
<img src="figs/valgrind.png" width="50%">

>>>

### Testing with Continuous Integration

<img src="figs/jenkins.png" width="40%">

- Detected bugs (Linux net-next ree)
 - [net-next,v2] ipv6: Do not iterate over all interfaces when finding source address on specific interface. (v4.2-rc0, **during VRF**)
 - [v3] ipv6: Fix protocol resubmission (v4.1-rc7, **expanded from v4 stack**)
 - [net-next] ipv6: Check RTF_LOCAL on rt->rt6i_flags instead of rt->dst.flags (**v4.1-rc1, during v6 improvement**)
 - [net-next] xfrm6: Fix a offset value for network header in _decode_session6 (v3.19-rc7?, **regression only in mip6**)

Note:
 - detected by: http://ns-3-dce.cloud.wide.ad.jp/jenkins/job/daily-net-next-sim/958/testReport/
 - detected by: http://ns-3-dce.cloud.wide.ad.jp/jenkins/job/umip-net-next/716/
 - detected by: http://ns-3-dce.cloud.wide.ad.jp/jenkins/job/daily-net-next-sim/878/
 - patchwork: http://patchwork.ozlabs.org/patch/493675/
 - patchwork: http://patchwork.ozlabs.org/patch/482645/
 - patchwork: http://patchwork.ozlabs.org/patch/467447/
 - patchwork: http://patchwork.ozlabs.org/patch/436351/

---

## Lessons learned

<div class="left" style="width: 30%">
<img src="figs/frankenlibc-stack.png" width=100%>
</div>

- Proof: Anykernel isn't only for NetBSD
 - Can be applied to other kernels
- New (standard?) layers of software stack


>>>

## Summary

- i've talked about Linux,
- but Linux or xxxBSD doesn't matter

- because operating system kernel can be replaced / reused


Note:
- Anykernel, a magic technology for resuability
- Do not re-implement from scratch, but reuse a great code with the library

>>>


<br>
<br>
<br>
<br>
<br>

### Linux rumpkernel

<br>
<br>

Hajime Tazaki (tazaki at iij.ad.jp)
@thehajime

>>>

## References

- Code
 - https://github.com/lkl/linux (LKL)
 - https://github.com/libos-nuse/frankenlibc
 - https://github.com/libos-nuse/rumprun
- Articles
 - https://github.com/thehajime/blog/issues/3 (blog)
 - http://www.iij-ii.co.jp/en/lab/researchers/tazaki/ (my info)

---


<section id="appendix" class="stack">

<h2> Backups </h2>

</section>

<section id="appendix" class="stack">
## (a bit of) History

- rump: 2007 (NetBSD)
- LKL: 2007 (Linux)
- DCE/LibOS: 2008 (Linux/FreeBSD)
- LibOS/LKL revival: 2015
 - LibOS merged to LKL
</section>


<section id="appendix" class="stack">

<img src="figs/lwn-libos-150408.png" width=28% />
<img src="figs/hnews-1503.png" width=28% /><br>
<img src="figs/phoronix-1503.png" width=28% />
<img src="figs/mynavi-libos.png" width=28% />

<small>
http://news.mynavi.jp/news/2015/03/25/285/ <br>
https://news.ycombinator.com/item?id=9259292 <br>
http://www.phoronix.com/scan.php?page=news_item&px=Linux-Library-LibOS <br>
http://lwn.net/Articles/639333/ <br>
</small>

</section>


<section id="appendix" class="stack">


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

</section>


<section id="appendix" class="stack">

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

reimplementation of timer API is required for simulation's feature, time warp

</section>


