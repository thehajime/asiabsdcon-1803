# Freeform Internet by (TBD)

Hajime Tazaki (University of Tokyo)


---

  <img src="figs/baran-distributed.png" width="120%"/>

<span>
P. Baran, On Distributed Communications Networks, IEEE Transactions on Communications Systems, 1964
</span>

---

## Motivation

- my ultimate goal
 - make the Internet more democratic
 - **Freeform Internet**
<!--
even though the reasons for those problems (deployment issue, architectural issue) are not technical issue, 
but the others like political worries, stuck community, 
technologies should be ready to answer the real problem which we faced.
-->

- by 'democratic' I meant
 - *any entities (users, software, networks) should do anything as they wish IF there is an interesting idea.*

Note:
--- this is a lesson learnt from 3.11

Outline
- (global) goal
- overview of 3 projects
 - problems
 - ideas
- DCE detail
 - nutshell
 - presence
 - problems/solutions
 - use cases
- NUSE detail
 - nutshell
 - presence
 - problems/solutions
 - use cases (evaluations)
- future directions
 - Linux LibOS project (more generic ideas)
 - software engineering challenge (NUSE)
- summary

## Outline (LibOS version)

- What's libos
- Motivations
 - lack of network stack personality
 - lack of controlled environment for network experiment
 - lack of reliable testing environment for protocol development
- History
 - microkernel
   UNIX => fat UNIX => slim kernel + rich servers => (performance issue) => fat UNIX
   ~1980                     mid 1980'                 1990' - 2000' mid        2015

   heart of u-kernel
   - multiple OSes (purpose)
   - multiple admin domains 
 - why now ?
  - mtcp (NSDI'14), Sandstorm (SIGCOMM 14')
  - 0) monolithic kernel
  - 1) micro-kernel with multiple (small) OSes (exokernel/libos)
  - 2) hypervisor with multiple monolithic kernel
  - 3) hypervisor with multiple (small) kernel
 - userspace stack (utah, alpine, nsc, etc)
 - JavaVM
 - unikernels (mirage, osv/seastar, clickos, rumpkernel,)
- Solutions
 - DCE (CoNEXT 13')
 - NUSE (linux netdev0.1)
   - LWN/Phoronix featured
- Architecture
- Demo
- Use cases
- What's next ?


>>>

## three projects

1. Floating ground architecture (IEEE/ACM ANCS 12')  <!-- .element: class="fragment shrink" data-fragment-index="1" -->
 - tackled with current issue of non-flexible mobile network
 - solve with *virtual overlay* (called Floating Ground) in the middle
2. **Direct code execution** (ACM CoNEXT 13')
 - tackled with 3 issues of network protocol experiments
  - 1) lack of timing realism, 2) lack of functional realism, 3) lack of debuggability
 - solved with real kernel implementation with **indirections**
3. **Network stack in userspace/Linux LibOS** (ongling)
 - Problem: no network stack personality on a system
 - solved with another indirection by dynamically translating an application code
 - end-to-end principle for network stack (toward E2E principle for OSes)

---

## An experimental framework for network protocol
Toward full reproducible network experiments

(Joint work among INRIA, UW, ns-3 project, and Keio/UTokyo/NICT)

>>>
<!-- .slide: class="two-floating-elements" -->
## Direct Code Execution in a nutshell
### (DCE)

> Lightweight virtualization of kernel and application processes, interconnected by simulated networks

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

<video data-autoplay src="figs/ns-3-dce-mptcp-linux3.5.7-8subf.m4v"></video>

- MPTCP v0.86 with ns-3-dce, 8 sub flows

Note:
 - https://www.youtube.com/watch?v=fN_nv7RdFm8

- MPTCP over LTE (IPv4) and Wi-Fi (IPv6) 
 - https://www.youtube.com/watch?v=rvF-yreZElQ



>>>
## Why DCE ?

1. You want to investigate a protocol, but the **model isn't available**,
2. You don't want to **maintain two versions of implementation** btw/ ns-3 and (UNIX/POSIX) socket applications,
3. You want to evaluate a protocol implemented in Linux kernel, but
 - you need many nodes with high-volumed traffic (which **doesn't scale** coz computation resource of VM)
 - or you want to conduct a (fully) **reproducible experiment**


# 

  
### DCE helps you (tm) !

Note:
This will only display in the notes window.

>>>

## Related Work
### Realtime emulation

- Container Based Emulation
 - Mininet-HiFi [CoNEXT 12'] (LXC), Netkit [2014] (UML)
 - provides lightweight virtualization
 - Timing realism still limited by hardware resources
 - No debugging support

>>>

## Related Work
### Virtual time emulation

- Time Dilation [NSDI’06]
 - Clock adjustment between different systems Constant time dilation factor
- Slice Time [NSDI’12]
 - Uses synchronizer to adjust speeds between VMs and underlying emulated network
- Time Travel VM (TTVM) [ATC’05]
 - Support debugging with bw/fw navigation

<div class="right" style="width: 20%">
  <img src="figs/vtime.png" />
</div>

**Still hard to control across distributed nodes.**

>>>

## High-level Overview

|                    | Simulators | Emulators  | DCE (ours)   |
| ------------------ |:---------:|: ----------:|:-------:|
| functional realism |  **--**   |   **++**     | **+** | 
| timing realism |  **++**       |   **-/+**     | **++** |
| debuggability |  **+**         |   **-**     | **+** |


<span>


**fill the gaps of network simulations and emulations**
</span>

>>>


## Internals

<div class="left" style="width: 50%">
<ol>
<li> Virtualization Core Layer<br>
 - Use virtual (deterministic) clock of simulator<br>
 - stack/heap management<br>
 - isolation via dlmopen(3) <br>
<li> Kernel layer<br>
 - glue codes for kernel code
<li> POSIX glue layer<br>
 - reimplementation of POSIX API<br>
 - hijack host system calls<br>
</ol>
</div>

<div class="right" style="width: 50%">
  <img src="figs/dce-arch.png" />
<!--
  <br /><small>
    https://flic.kr/p/8N1hWh
  </small>
-->
</div>


>>>

## How it works ?

- Recompile your code
 - Userspace as Position Independent Executable (PIE)
 - Kernel space code as shared library (libsim-linux.so)

- Run with ns-3
 - Load the executables (binary, library) in an isolated environment among nodes
 - synchronize simulation clocks with apps/kernels clock

>>>

# DCE Use cases

- Replicate an existing experiment in the literature (**reproducibility**)

- Debug a complicate protocol of distributed protocol (**debuggability**)

>>>

## Replication of a past experiment (MPTCP)

<img src="figs/mptcp-comparison-topo.png" width="350">
<img src="figs/nsdi12-mptcp-fig9.png" width="450">

1. picked one figure from a paper (NSDI 2012)
2. replicate an experiment with available information on the paper
3. configure an ns-3 scenario **with** the same software (Linux/iperf)

<span>
Reference: Raiciu et al. How hard can it be? Designing and implementing a deployable multipath tcp, USENIX NSDI’12
</span>

>>>

## Replication of a past experiment (cont'd)

<img src="figs/mptcp-comparison-conext13.png" width="850">

- slight diffs in goodput improvement v.s. buffer size
- Max goodput range: 2.2 - 2.9Mbps (DCE),  2.0 - 3.2Mbps (original)


<span>
Tazaki et al. Direct Code Execution: Revisiting Library OS Architecture for Reproducible Network Experiments, CoNEXT 2013
</span>

>>>


## DCE as a development platform of Linux network stack

- Testing network stack with nightly/each commit
- Rich userspace development facilities
 - Measuring testing code coverage
 - Debug with gdb/valgrind

>>>

## Development platform (gdb)

<img src="figs/umip-gdb-topo.png" width="440">
<img src="figs/umip-gdb.png" width="440">

- Inspect codes during experiments
 - among distributed nodes
 - in a single process
- perform a simulation to reproduce a bug
- see how badly handling a packets in Linux kernel


<span>
http://yans.pl.sophia.inria.fr/trac/DCE/wiki/GdbDce
</span>

>>>

## Development platform (valgrind)

<img src="figs/valgrind.png" width="640">

- Memory error detection
 - among distributed nodes
 - in a single process
- Use **Valgrind**

<span>
http://yans.pl.sophia.inria.fr/trac/DCE/wiki/Valgrind
</span>

>>>

## Development platform (code coverage)

<img src="figs/gcov.png" width="1000">

<span>
http://yans.pl.sophia.inria.fr/trac/DCE/wiki/MptcpCoverageTest
</span>

>>>

## Development platform (Jenkins CI)

<img src="figs/jenkins.png" width="800">

http://ns-3-dce.cloud.wide.ad.jp/jenkins/job/daily-net-next-sim/

>>>

## Detected kernel bugs (at Linux netdev)

- [net-next,v2] ipv6: Do not iterate over all interfaces when finding source address on specific interface. (v4.2-rc0)
 - patchwork: http://patchwork.ozlabs.org/patch/493675/
 - detected by: http://ns-3-dce.cloud.wide.ad.jp/jenkins/job/daily-net-next-sim/958/testReport/
- [v3] ipv6: Fix protocol resubmission (v4.1-rc7)
 - patchwork: http://patchwork.ozlabs.org/patch/482645/
 - detected by: http://ns-3-dce.cloud.wide.ad.jp/jenkins/job/umip-net-next/716/
- [net-next] ipv6: Check RTF_LOCAL on rt->rt6i_flags instead of rt->dst.flags
 - patchwork: http://patchwork.ozlabs.org/patch/467447/
 - detected by: http://ns-3-dce.cloud.wide.ad.jp/jenkins/job/daily-net-next-sim/878/
- [net-next] xfrm6: Fix a offset value for network header in _decode_session6 (v3.19-rc7?)
 - patchwork: http://patchwork.ozlabs.org/patch/436351/


>>>

## Summary of DCE

- DCE can simulate with real code (kernel/POSIX app)
- An experiment is **fully reproducible**
- Plenty of use cases


<span> <small>
- Tazaki et al. Direct Code Execution: Revisiting Library OS Architecture for Reproducible Network Experiments, CoNEXT 2013
- Camara et al. DCE: Test the Real Code of Your Protocols and Applications over Simulated Networks, IEEE Communications Magazine, March 2014
</span></small>

---

## Userspace Network Stack

>>>

## Network Stack in Userspace in a nutshell
### (What is NUSE ?)

- Userspace network stack running on Linux (POSIX) platform
- For Generic application (not only for network simulator)
- Suitable with kernel bypass technologies (e.g., netmap, Intel DPDK)

- Benefit
 - network stack personality (like FUSE does)
 - full feature set of matured network stack
 - Multi-threading is easy
 - Rich development facilities
 - etc.
<!-- - Drawback -->
 


>>>

## Problems

- Slow evolution of network stack (lack of personality)
 - new protocols is hard to get deployed
 - performance improvement takes also long
 - e.g., Linux MPTCP, socket API sucks

- Kernel space is a *holy* space
 - sometimes untouchable 
 - Can only big parties introduce a new protocol ? (QUIC)

>>>

## Network stacks

- Why kernel space ?
 - (Unit price of) packets were **expensive** at the beginning
- Why not **userspace** ?
 - well grown in decades, costs degrades
 - obtain **network stack personality**
 - **controllable** by userspace utilities

refs: https://fosdem.org/2015/interviews/2015-antti-kantee/

>>>

## but we don't want to reimplement from scratch

- we (at least myself) don't want to **reimplement** the whole features
 - waste of time
 - breaks inter-operability
- network stack is just a bunch of C code <!-- .element: class="fragment" data-fragment-index="1" -->

# 

### **why not reuse it (rather than reimplement it) ?**  <!-- .element: class="fragment" data-fragment-index="2" -->

>>>

## Related Work

- mTCP [NSDI 14'], SandStorm [SIGCOMM 14'], MirageOS [ASPLOS 13']
 - reimplementing network stack from scratch isn't practical
- OSv [USENIX ATC 14'], libuinet, 
 - porting is headache when you will track the latest code
- Rump kernel [USENIX ATC 09']
 - more generic idea on NetBSD 

>>>

<!-- .slide: data-background="figs/cs-thomas.png" -->

> Any problem in computer science can be solved with another level of __indirection__.

>(Wheeler and/or Lampson)



<br /><small>
img src: https://www.flickr.com/photos/thomasclaveirole/305073153
</small>



>>>

## Architecture

<div class="left" style="width: 50%">
<ul>
<li> Host backend <br>
 - provide (Linux/POSIX-y) OS interface <br>
 - clock (clock_gettime(2)) <br>
 - network (raw sock, netmap, DPDK) <br>
 - scheduler (pthread-based) <br>
<li> POSIX layer <br>
 - system call proxy for IPC <br>
</ul>
</div>

<div class="right" style="width: 50%">
<img src="figs/nuse-arch.png" width="600">
</div>


>>>

## Execution

```
% LD_PRELOAD=libnuse-linux.so ping www.google.com
```

call stack will be
```
ping(8)
 socket(2)
  nuse_socket()
   raw(7)
    (via host NIC)
```

>>>

## Demo (if time permits)

>>>

## performance information (Host)

<img src="figs/nuse-benchmark-topo-host.png" width="400">
<img src="figs/nuse-benchmark-host.png" width="800">

we're not so good right now :( <!-- .element: class="fragment" data-fragment-index="1" -->

>>>

## performance information (L3 forwarding)

<img src="figs/nuse-benchmark-topo-router.png" width="600">
<img src="figs/nuse-benchmark-router.png" width="800">

we're not good :( **(patches are welcome !!)** <!-- .element: class="fragment" data-fragment-index="1" -->

>>>

## Challenges

- Performance improvements based on previous studies (mTCP, Sandstorm)
- To be sustainable over the decade
 - most of past projects are dissapeared due to many reasons


---


## Future Directions

- More generalized framework for userspace network stack
 - more applications, more host backend (bare metal, HV)
 - performed well (at least x1 speed of native kernel)
- More generalized framework for any kernel subsystem on userspace
 - operating system on userspace (i.e., Library OS)
- as rump kernel in NetBSD

>>>

## Linux LibOS project

- Abstract two projects (DCE/NUSE) as an identical component
- The idea is not brand-new technology (since 90's from MIT)
 - but recent revival (Drawbridge, OSv, Mirage, Rumpkernel)
- ability to link **a single subsystem** to an application
 - only network stack at the present moment
 - **make an OS more flexibly composable**

- proposed to Linux upstream, now improving the paches
 - featured Linux Weekly News (http://lwn.net/Articles/639333/)
 - Slide: http://www.slideshare.net/hajimetazaki/library-operating-system-for-linux-netdev01

---


## The end of talk

- Walk-through review of Linux LibOS
- 2 applications
 - DCE (experimental framework)
 - NUSE (network stack personality)

- **Demonstrated userspace network stacks is flexible enough**
- Toward network stack more **democratic**
- Toward the goal of **Freeform Internet**

>>>



## References

- [1] Lacage. Experimentation Tools for Networking Research. PhD thesis, 2010
- [2] Tazaki et al. Direct Code Execution: Revisiting Library OS Architecture for Reproducible Network Experiments, CoNEXT 2013
- [3] Tazaki et al. DCE Cradle: Simulate Network Protocols with Real Stacks for Better Realism, WNS3 2013
- [4] Camara et al. DCE: Test the Real Code of Your Protocols and Applications over Simulated Networks, IEEE Communications Magazine, March 2014
- [5] Sekiya et al. DNSSEC simulator for realistic estimation of deployment impacts. IEICE ComEX(3):10, 2014.

>>>

## Information

- NUSE
 - Web (http://libos-nuse.github.io/)
 - Mailing list (https://groups.google.com/forum/#!forum/libos-nuse)
 - Github (kernel) (https://github.com/libos-nuse/net-next-nuse)
 - Github (tools) (https://github.com/libos-nuse/linux-libos-tools)
- DCE
 - Web (https://www.nsnam.org/overview/projects/direct-code-execution/)
 - Mailing list (ns-3-users@googlegroups.com)
 - Github (https://github.com/direct-code-execution)


>>>

# Questions ?

---

## Backups

>>>

## History

- Initial discussion, 2006 (ns-3 goals, by Mathieu Lacage, INRIA)
- DCE implementation
 - Started around 2007 (Mathieu Lacage)
 - GSoC 2008 (Quagga/Netlink bridge)
 - almost 8 years old
- A forked project, NUSE with Linux LibOS, started, 2014
 - network stack part is shared btw/ NUSE and DCE

>>>

## High-level Overview (cont'd)
<img src="figs/stack-apps-variants.png" width="1000">

- POSIX application on ns-3 [Lacage 10]
- Linux/FreeBSD kernel on ns-3 [Lacage 10]
- ns-3 Application with Linux/FreeBSD kernel [Tazaki 13]


>>>

## Alternatives

- Container (LXC, OpenVZ, vimage)
 - share kernel with host operating system (no flexibility)
- Userspace virtualization (user mode linux = UML)
 - need full virtualization
 - rely on the *underlying* Linux features
- other Library OSs
 - full scratch: mtcp, Mirage, lwIP
 - Porting: OSv, Sandstorm, libuinet (FreeBSD), Arrakis (lwIP), OpenOnload (lwIP?)
 - Glue-layer: LKL (Linux-2.6), rumpkernel (NetBSD), ours (Linux LibOS)

>>>

## LibOS Architecture

Abstraction on more generic rump framework

* Host backend layer
 - new abstraction
* Kernel layer
 - replaceable
* POSIX glue layer

<p class="right" style="width: 60%">
 <img src="figs/libos-arch.png" width="400"/>
<!--
  <br /><small>
    https://flic.kr/p/8N1hWh
  </small>
-->
</p>


