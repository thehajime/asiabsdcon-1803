# Linux library operating system (LibOS)

Hajime Tazaki (University of Tokyo)

3rd July, 2015
at UCL, (Louvain La Nouve, Belgium)

---

## Who am I ?

- Hajime Tazaki
 - Ph.D (2011, supervised by Jun Murai)
- A project lecturer at University of Tokyo
 - NECOMA project (EU FP7/Japan co-project)
- Interests
 - freeform networks (mobile/ad-hoc network architectures)
 - network experimental method
- **(in fact) just a code monkey**  <!-- .element: class="fragment" data-fragment-index="1" -->

>>>

## agenda

- What is Linux Library OS ?
- When it is useful ?
 - Direct Code Execution
 - Network Stack in Userspace
- What's going to be in near future ?
- Summary


---

## The Linux Library operating system

- A **library version** of Linux kernel
- The idea is not brand-new technology (since 90's from MIT)
- ability to link **a single subsystem** to an application
 - only network stack at the present moment
- proposed to Linux upstream, now improving the paches
- LWN featured (http://lwn.net/Articles/639333/)


>>>

## Network stacks ?

- Why kernel space ?
 - (Unit price of) packets were **expensive** at the beginning
- Why not **userspace** ?
 - well grown in decades, costs degrades
 - obtain **network stack personality**
 - **controllable** by userspace utilities

refs: https://fosdem.org/2015/interviews/2015-antti-kantee/

>>>

## What's the matter ?

- we (at least myself) don't want to **reimplement** the whole features
 - breaks inter-operability
 - waste of time
 - lack of flexibility/controllability
- network stack is just a bunch of C code <!-- .element: class="fragment" data-fragment-index="1" -->

# 

### **why not reuse it (rather than reimplement it) ?**  <!-- .element: class="fragment" data-fragment-index="2" -->

>>>

<!-- .slide: data-background="figs/cs-thomas.png" -->

"Any problem in computer science can be solved with another level of ***indirection***." 
(Wheeler and/or Lampson)






img src: https://www.flickr.com/photos/thomasclaveirole/305073153

>>>

## Architecture

<img src="figs/libos-arch.png" width="400">

* Host backend layer
* Kernel layer
* POSIX glue layer

>>>

## Execution Model

- a single entry point via API (i.e., lib_init())
- dlmopen(3) and LD_PRELOAD


>>>

## Applications

- Direct Code Execution (DCE)
 - An integration with network simulator (ns-3)
 - A testing platform for (kernel) network stack
- Network Stack in Userspace (NUSE)
 - network stack personality (ad-hoc network stack)

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

## History

- Initial discussion, 2006 (ns-3 goals, by Mathieu Lacage, INRIA)
- DCE implementation
 - Started around 2007 (Mathieu Lacage)
 - GSoC 2008 (Quagga/Netlink bridge)
 - almost 8 years old
- A forked project, NUSE with Linux LibOS, started, 2014
 - network stack part is shared btw/ NUSE and DCE

---

## Direct Code Execution in a nutshell

>>>

## What is Direct Code Execution ?

- Lightweight virtualization of kernel and application processes, interconnected by simulated networks

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

## Demo

- MPTCP v0.86 with ns-3-dce, 8 sub flows 
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

  
### DCE helps you (tm) ! <!-- .element: class="fragment" data-fragment-index="1" -->

Note:
This will only display in the notes window.

>>>


## High-level Overview
<img src="figs/stack-apps-variants.png" width="1000">

- POSIX application on ns-3 [Lacage 10]
- Linux/FreeBSD kernel on ns-3 [Lacage 10]
- ns-3 Application with Linux/FreeBSD kernel [Tazaki 13]

>>>

## Internals

<img src="figs/dce-arch.png" width="700">

- Virtualization Core Layer
- Kernel layer
- POSIX glue layer

>>>
## How it works ?

- Recompile your code
 - Userspace as Position Independent Executable (PIE)
 - Kernel space code as shared library (libsim-linux.so)

- Run with ns-3
 - Load the executables (binary, library) in an isolated environment among nodes
 - synchronize simulation clocks with apps/kernels clock

>>>
## What to start with ?

- Quick start with *bake* tool
 - https://www.nsnam.org/docs/dce/release/1.6/manual/html/getting-started.html

- 3 modes
 - dce-ns3-|version| (basic mode)
 - dce-linux-|version| (advanced mode)
 - dce-freebsd-|version| (*experimental*)

```
mkdir dce
cd dce
bake.py configure -e dce-linux-1.6
bake.py download
bake.py build
```


>>>
## How to use it ?
- DceManagerHelper / DceApplicationHelper
- (option) LinuxStackHelper
- (option) other submodule helper (QuaggaHelper, Mip6dHelper, etc)
-  (option) Custom command execution

>>>

## Code snippet

~~~c
  // configure DCE with Linux network stack
  DceManagerHelper dce;
  dce.SetNetworkStack ("ns3::LinuxSocketFdFactory", 
                      "Library", StringValue ("liblinux.so"));
  dce.Install (nodes);

  // run an executable at 1.0 second on node 0
  DceApplicationHelper process;
  ApplicationContainer apps;
  process.SetBinary ("your-great-server");
  apps = process.Install (nodes.Get (0));
  apps.Start (Seconds (1.0));
~~~

Note:
```ruby
require 'redcarpet'
markdown = Redcarpet.new("Hello World!")
puts markdown.to_html
```

```javascript
function fancyAlert(arg) {
  if(arg) {
    $.facebox({div:'#foo'})
  }
}
```


>>>

## DCE as a development platform of Linux network stack

- Testing network stack with nightly/each commit
- Measuring testing code coverage
- Debug with gdb or valgrind

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

## Detected kernel bugs (at netdev)

- [v3] ipv6: Fix protocol resubmission (v4.1-rc7)
 - patchwork: http://patchwork.ozlabs.org/patch/482645/
 - detected by: http://ns-3-dce.cloud.wide.ad.jp/jenkins/job/umip-net-next/716/
- [net-next] ipv6: Check RTF_LOCAL on rt->rt6i_flags instead of rt->dst.flags
 - patchwork: http://patchwork.ozlabs.org/patch/467447/
 - detected by: http://ns-3-dce.cloud.wide.ad.jp/jenkins/job/daily-net-next-sim/878/
- [net-next] xfrm6: Fix a offset value for network header in _decode_session6 (v3.19-rc7?)
 - patchwork: http://patchwork.ozlabs.org/patch/436351/



>>>

# DCE Showcases

>>>

## Features/Functions

1. **less development effort**
 - quagga, umip, ccnx, bind9, etc
 - Linux kernel (SCTP, DCCP, IPv4/v6), FreeBSD 10.0 kernel (partialy)
 - Out-of-tree Linux kernel Multi-path TCP
2. **reproducible environment** for Linux kernel experiment
3. **development platform**
 - debugging facility (gdb, valgrind)
 - code coverage (gcov, etc)

>>>

## Less development effort (DCCP)
<img src="figs/dccp-dce-cradle-topo.png" width="460">
<img src="figs/dccp-dce-cradle-plot.png" width="460">

- **10K LoC** of Linux DCCP can be used on ns-3
- with a small amount of code (~10 LoC)!
- and the results are promising

<span>

- https://www.nsnam.org/docs/dce/manual/html/dce-cradle-usecase-4.html
- Tazaki et al. DCE Cradle: Simulate Network Protocols with Real Stacks for Better Realism, WNS3 2013
</span>

>>>

## Less development effort (DNS/DNSSEC)

<img src="figs/dnssec-topo.png" width="300">
<img src="figs/dnssec-resptime-plot.png" width="440">

- bind9 querylog => named.conf/unbound.conf (by *createzones*)
 - contains 1000 queries (max.) => 581 zones
- run bind9/unbound (named, dig, etc) **with/without** DNSSEC	
- see how response time will be changed
- available at a private repository (http://dnssec.sekiya-lab.info)

<span font-size="8pt">
Paper: Sekiya et al. DNSSEC simulator for realistic estimation of deployment impacts. IEICE ComEX(3):10, 2014.
</span>

>>>

## Less development effort (ccnx)

<img src="figs/ccnx.png" width="680">

- an alternative for ndn-sim

>>>

## Replication of a past experiment (MPTCP)

<img src="figs/mptcp-comparison-topo.png" width="350">
<img src="figs/mptcp-comparison-conext13.png" width="550">

1. picked one figure from a paper (NSDI 2012)
2. replicate an experiment with available information on the paper
3. configure an ns-3 scenario **with** the same software (Linux/iperf)


- no significant goodput improvement with buffer size when DCE
- Max goodput range: 2.2 - 2.9Mbps (DCE)  2.0 - 3.2Mbps (NSDI)

<span>

- Tazaki et al. Direct Code Execution: Revisiting Library OS Architecture for Reproducible Network Experiments, CoNEXT 2013
- http://yans.pl.sophia.inria.fr/trac/DCE/wiki/MptcpNsdi12
</span>


---

## Network Stack in Userspace (NUSE)

>>>

## What's NUSE ?
- Userspace network stack running on Linux (POSIX) platform
- Network stack personality
- Full-featured network stack for kernel bypass technology (e.g., netmap, Intel DPDK)
- Flexibility does matter !

>>>

<img src="figs/nuse-arch.png">

>>>

## Outlook

- Memory
 - host malloc(3)
- Time
 - clock_gettime(2)
- Network
 - nuse-vif
 - bound to kernel-bypass tech (raw(7), DPDK, netmap, tap)
- IPC
 - Multi-process support by rump_server
 - system call proxy

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

## Demo 

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

---

## The end of talk

- Walk-through review of Linux LibOS
- 2 applications
 - DCE
 - NUSE


>>>

## End notes

- Be patient :)
 - It's still a young project
 - sparse documents/tools/output

- But it's definitelly useful
 - no need to develop a model
 - improve a realism
 - always reproducible
 - you will get certain benefits !

>>>

## References

- [1] Lacage. Experimentation Tools for Networking Research. PhD thesis, 2010
- [2] Tazaki et al. Direct Code Execution: Revisiting Library OS Architecture for Reproducible Network Experiments, CoNEXT 2013
- [3] Tazaki et al. DCE Cradle: Simulate Network Protocols with Real Stacks for Better Realism, WNS3 2013
- [4] Camara et al. DCE: Test the Real Code of Your Protocols and Applications over Simulated Networks, IEEE Communications Magazine, March 2014
- [5] Sekiya et al. DNSSEC simulator for realistic estimation of deployment impacts. IEICE ComEX(3):10, 2014.

>>>

### Contact

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

