## Linux library operating system (LibOS) in 5 minutes

 
### Hajime Tazaki

Note:
17th August, 2015
at Apple HQ (Cupertino, CA)

>>>

## The Linux LibOS

- A **library version** of Linux kernel
- ability to link **a single subsystem** to an application
 - only network stack at the present moment

- proposed to Linux upstream, now improving the paches
- LWN featured (http://lwn.net/Articles/639333/)

Note:
- The idea is not brand-new technology (since 90's from MIT)

- A userspace network stack
 - userspace is well grown in decades, costs degrades
 - obtain **network stack personality**
 - **controllable** by userspace utilities
 - rich **debug** tools
 - a **deterministic** clock (to ensure test results)
 - flexible network configuration for broader tests

>>>

## LibOS Architecture

<img src="figs/libos-arch.png" width="450">

* Host backend layer
* Kernel layer
* POSIX glue layer

>>>

<!-- .slide: data-background="figs/cs-thomas.png" -->

"Any problem in computer science can be solved with another level of ***indirection***." 
(Wheeler and/or Lampson)






img src: https://www.flickr.com/photos/thomasclaveirole/305073153

>>>

## What you can do ?

1. Direct Code Execution (**DCE**)
 - An integration with network simulator (ns-3)
 - A testing platform for (kernel) network stack

2. Network Stack in Userspace (**NUSE**)
 - network stack personality (ad-hoc network stack)

>>>

## 1. LibOS as a development platform

- debug with gdb or valgrind
- measuring code coverage
- nightly/commit-lly tests

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

## 2. Network Stack in Userspace (NUSE)

- Userspace network stack running on Linux (POSIX) platform
- Network stack personality
- Full-featured network stack for kernel bypass technology (e.g., netmap, Intel DPDK)
- Flexibility does matter !

>>>

## (extended) Architecture

<img src="figs/nuse-arch.png">

>>>

## What NUSE looks like ?

```
% LD_PRELOAD=libnuse-linux.so ping www.google.com
```

That's it: welcome to new world :)

>>>

## Summary

- The LibOS network stack benefits
 - development platform with userspace facilities
 - fine-grained testing environment (DCE)
 - network stack personality (NUSE)

- **Toward deeper investigation of network stack**
- **Toward a new shape of network stack**



---

## Backups

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


