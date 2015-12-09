## Rethinking Software Architecture for Network Stack Personality
アプリケーション固有ネットワークスタックのためのライブラリOS

<span>
<br>
<br>
<br>
<br>

田崎 創 (ネットワーク研究部門) <br>
2015/10/1
</span>

---


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

### Slow evolution of network protocols

<img src="figs/micchie-ccr-proto-evo.png" width="80%"/>

<span>
Honda et al., Rekindling Network Protocol Innovation with User-Level Stacks, ACM SIGCOMM CCR, Vol.44, Num. 2, April 2014
</span>

Note:
Windowscale, Timestamp: since Windows 2000, Vista: default ON in 2006,
SACK/TS defaulted Linux 1999, WS: 2004

>>>

<!-- .slide: data-background="figs/fat-kernel.png" data-background-size="75%" -->

>>>

### Big-Fat kernel

<img src="figs/fat-kernel.png" width="650">
<small>
img src: http://www.makelinux.net/kernel_map/
</small>

- Taking **long time** to deploy a new network protocol
- Aggressive modification is almost **impossible**
- **Fat architecture** for a small/micro service

>>>

## Problems

- Slow evolution of network stack (**lack of personality**)
 - new protocols is hard to get deployed
 - performance improvement takes also long
 - e.g., Linux MPTCP, socket API sucks

- Kernel space is a *holy* space
 - sometimes untouchable 
 - Can only big parties introduce a new protocol ? (e.g., QUIC)

>>>

## Network stacks

- Writing a network stack in 1-week DIY
- Making a network stack work on real-world is **NOT** 1-week DIY
 - (which is no longer DIY)

<br>

#### Avoiding reinvention of wheels

- **reimplementation** the whole features
 - waste of time
 - breaks inter-operability
- network stack is just a bunch of C code (in conventional OSes)

# 

**``why not reuse it (rather than reimplement it) ?``**  <!-- .element: class="fragment" data-fragment-index="2" -->

>>>

## Related Work

- mTCP [NSDI '14], SandStorm [SIGCOMM '14], MirageOS [ASPLOS '13], Seastar ('15)
 - reimplementing network stack **from scratch isn't practical**
- OSv [USENIX ATC '14], libuinet ('14) 
 - **porting is a headache** when you will track the latest code
- Rump kernel [USENIX ATC '09], DrawBridge [ASPLOS '11]
 - more generic idea on NetBSD/Windows

>>>

<!-- .slide: data-background="figs/cs-thomas.png" -->

> Any problem in computer science can be solved with another level of __indirection__.

>(Wheeler and/or Lampson)



<br /><small>
img src: https://www.flickr.com/photos/thomasclaveirole/305073153
</small>



---


## Userspace network stack / Library operating system

>>>

## Network Stack in Userspace (NUSE)

<div class="left" style="width: 50%">
<img src="figs/nuse-overview.png" width="400">
</div>

<div class="right" style="width: 50%">
<ul>
<li> Network stack as a **library** </li>
<li> Userspace app can choose </li>
<!-- <li> For generic framework (not only for network simulator) </li> -->
<li> Suitable with **kernel bypass** technologies (e.g., netmap, Intel DPDK) </li>
<p>
<li> Benefit </li>
<ul>
<li> network stack personality (like FUSE does) </li>
<li>full feature set of matured network stack </li>
<li>Multi-threading is easy </li>
<li>Rich development facilities </li>
</ul>
</div>

<!-- - Drawback -->
 

>>>

## NUSE Architecture

<div class="left" style="width: 50%">
<ol>
<li> Host backend <br>
 - provide (Linux/POSIX-y) OS interface <br>
 - scheduler (pthread-based) <br>
 - clock (clock_gettime(2)) <br>
 - network (raw sock, netmap, DPDK) <br>
<li> Kernel layer <br>
 - **reuse** kernel code *as-is*
<li> POSIX layer <br>
 - reimplementation of POSIX API <br>
 - system call hijacking/proxy <br>
</ol>
</div>

<div class="right" style="width: 50%">
<img src="figs/nuse-arch.png" width="600">
</div>


>>>

## How it works

```
% LD_PRELOAD=libnuse-linux.so ping www.google.com
```

call stack will be
```
ping(8)
 socket(2)
  nuse_socket()
   sock_sendmsg(lib-kernel)
    ip_local_out(lib-kernel)
     dev_queue_xmit(lib-kernel)
      AF_PACKET(7)
       (via host NIC)
```

>>>

## Network experiment

<video data-autoplay src="figs/ns-3-dce-mptcp-linux3.5.7-8subf.m4v"></video>

<br>
- a python process with 10 different Linux (kernel) instances
- MPTCP v0.86 with ns-3-dce, 8 sub flows

Note:
 - https://www.youtube.com/watch?v=fN_nv7RdFm8

- MPTCP over LTE (IPv4) and Wi-Fi (IPv6) 
 - https://www.youtube.com/watch?v=rvF-yreZElQ


>>>

## Estimation of DNSSEC overhead

<img src="figs/dnssec-topo.png" width="300">
<img src="figs/dnssec-resptime-plot.png" width="440">

- Feasibility study of DNSSEC in your lab
- Bind9 DNS/DNSSEC software + Linux kernel
- **Replay** DNS queries and DNS topology from trace
 - contains 1000 queries (max.) => 581 zones
- see how response time will be changed

<span font-size="8pt">
Paper: Sekiya et al. DNSSEC simulator for realistic estimation of deployment impacts. IEICE ComEX(3):10, 2014.
</span>

>>>

## performance information (Host)
Transmission over 10Gbps Ether link 

<img src="figs/nuse-benchmark-topo-host.png" width="400">
<img src="figs/nuse-benchmark-host.png" width="900">

hmm..

>>>

## performance (L3 forwarding)
IP forwarding over 10Gbps Ether link 

<img src="figs/nuse-benchmark-topo-router.png" width="400">
<img src="figs/nuse-benchmark-router.png" width="900">

``we're not good at all :-)`` <!-- .element: class="fragment" data-fragment-index="1" -->

>>>

## Challenges

- Investigate the overhead of *__indirections__*
- Transparent interface to *__unmodified applications__*
- Performance improvements based on previous studies (mTCP, Sandstorm)
 - connection locality
 - adaptive packet batching
 - system call amortization


Note:

- To be sustainable over the decade
 - most of past projects are disappeared due to many reasons



---


## Summary

- Walk-through review of Linux userspace network stack
- 2 applications
 - Network stack personality 
 - More flexible network experiment

- Future work
 - Analysis of side-effect on **indirections**
 - More POSIX coverage

>>>



## References

- [1] Tazaki et al. Direct Code Execution: Revisiting Library OS Architecture for Reproducible Network Experiments, ACM CoNEXT 2013
- [2] Camara et al. DCE: Test the Real Code of Your Protocols and Applications over Simulated Networks, IEEE Communications Magazine, March 2014
- [3] Sekiya et al. DNSSEC simulator for realistic estimation of deployment impacts. IEICE ComEX(3):10, 2014.
- [4] Tazaki et al. Library Operating System with Mainline Linux Network Stack. Linux netdev0.1 conference, 2015.

---

# Backups

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


