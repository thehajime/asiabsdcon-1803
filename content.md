## Network stack personality in Android phone

<span>
<br>
<br>
<br>
<br>

Cristina Opriceana, **Hajime Tazaki**  (IIJ Research Lab.)

Linux netdev 2.2, Seoul, Korea

07 Nov. 2017
</span>

Note:

## Outline

- Android situation
 - Treble
 - Fushia
 - KVM on Android (https://www.linux-kvm.org/images/a/a9/Kvm-forum-2013-Windows8-on-Android-with-KVM.pdf)
 - Sucks...
 - Another google product
  - QUIC: (SIGCOMM)
- Too long to deliver your code
- The approach
 - network stack personality
- LKL
 - overview
 - current status
- extension for android support
 - raw socket extension
 - hijack library
  - bionic is more familiar than glibc ?
  - only socket-related calls are redirected
  - mixture of host and lkl descriptors (can avoid)
- MPTCP as an example of extension
 - mptcp
 - demo (cat /proc/net/mptcp base detection)
   https://github.com/frootmig/amiusingmptcp/
- Benchmarks
 - setup
 - Single path TCP (netperf)
 - Multi-path TCP (netperf)
 - Simple perf
  - flamegraph
  - CPU cycles per packet (over wifi, lkl vs native)

---

## Android
### a platform of billions devices

- billions installed Linux kernel
- slow delivery of software updates

- Questions
 - When our upstreamed code <br> available ?
 - What if I come up with <br> a great protocol ?

<div class="right" style="width: 50%">
<img src="figs/android-platform-distribution-version.png" width=100%>

<small>
https://developer.android.com/about/dashboards/index.html
</div>


Note:

- Android O is 4.4 based ?
<img src="figs/android-platform-distribution.png" width=100%>

>>>

## Android (cont'd)

- <span style="color: gray">When our upstreamed code available ? </span>
 - wait until base kernel is upgraded
 - backport specific function

- <span style="color: gray">What if I come up with a great protocol ?</span>
 - craft your own kernel and put into your image


<br>
<br>
<br>
<br>
<br>


**Looong delivery to all billions devices**

>>>

### Approaches to alleviate the issue

- Virtualization (KVM on Android)
 - Overhead isn't negligible to embedded devices
- Project Treble (since Android O)
 - More modular platform implementation
- Fushia
 - Rewrite OS from scratch


<img src="figs/treble_blog_after.png" width=40%>

https://source.android.com/devices/architecture/treble

Note:

 - KVM on Android (https://www.linux-kvm.org/images/a/a9/Kvm-forum-2013-Windows8-on-Android-with-KVM.pdf)

>>>

## An alternate approach

- network stack personality
 - use own network stack implemented in userspace
 - no need to replace host kernels
 - but (try to) preserve the application compatibility

- NUSE (network stack in userspace)
 - No delay of network stack update
 - Application can choose a network stack if needed

>>>

## Userspace implementations

- Toys, Misguided People
- Selfish


- Don't care whether toy or not<!-- .element: class="fragment" data-fragment-index="1" -->
- Do care whether it's useful or not<!-- .element: class="fragment" data-fragment-index="1" -->

(TODO: replace image)

Note:

FUSE or not FUSE ? (FAST '17)


---

## Linux Kernel Library intro (again)

- Out-of-tree architecture <br> (h/w-independent)
- Run Linux code on various ways
 - with a reusable library
- h/w dependent layer
 - on Linux/Windows <br>/FreeBSD uspace, <br> unikernel, on UEFI, 
 - (ns-3)
 - **Android<!-- .element: class="fragment" data-fragment-index="2" -->**

<div class="right" style="width: 40%">
<img src="figs/lkl-arch-new.png" width=100%>
</div>

>>>

## LKL: current status

- Sent RFC (Nov. 2015)
 - no update on LKML since then
- have evolved a lot
 - various extensions (offload, SMP, json config, unikernel, etc)


https://github.com/lkl/linux

>>>

## Extensions to LKL

- Android (arm/arm64) support (lkl/linux#372)
- raw socket extension (only handle ETH_P_IP) (not yet)
- hijack library enhance (not yet)

- HOWTO

```
% LD_PRELOAD=liblkl-hijack.so netperf XXX # console app
% setprop wrap.app LD_PRELOAD=liblkl-hijack.so # Java app
```

>>>

## hijack library

- For smooth replacement for Android UI app (java-based)
 - bionic is more familiar than glibc ?
 - only socket-related calls are redirected
 - mixture of host and lkl descriptors (can avoid)


**TODO** (a fig of hijack)

Note:

## Why LKL on Android ?

- Hard to upgrade a kernel
 - 5 yrs old android kernel (3.4.0+ on android-6)
 - Long time to ship
 - Out-of-tree code is hard too


>>>

## New feature introduction

- Example
 - Multipath TCP (http://multipath-tcp.org/)
 - out-of-tree for long time
 - verify site (cat /proc/net/mptcp base detection)
   http://amiusingmptcp.de/

Note:
   https://github.com/frootmig/amiusingmptcp/

>>>

## Demo

Note:

Nexus5 VNC
amiusingmptcp.de


<img src="figs/amiusingmptcp-nexus5.jpg" width=35%>
<img src="figs/amiusingmptcp-nexus5-2nd.jpg" width=35%>

---

## No penalty with userspace network stack ?

- Condition
 - To use Linux mptcp w/o replacing kernel
 - With tolerable amount of overhead

- Questions
 - Is NUSE working fine (Will users wanna use it) ?
 - How different from native Linux kernel ?

>>>

## netperf measurement

- Client
 - Nexus5 anrdoid 6.01 (rooted)
 - LTE, wifi
 - LKL arm/android patched
 - or native kernel
- Server
 - Ubuntu 16.04 (amd64) on KVM
 - virtio/Etherlink (uplink: 100 Mbps)
 - mptcp-4.4.70 (v0.92)
- Software
 - netperf 2.7.x
 - 10 seconds TCP_STREAM,<br>TCP_MAERTS
 - 5 trials, over 64-64K byte packet


<div class="left" style="width: 35%">
<img src="figs/mptcp-exp-topology.png" width=100%>
</div>

Note:
 - Android hammerhead:6.0.1 (M4B30Z)


>>>

## Single path (Wi-Fi only, 9/14)

<div class="left" style="width: 50%">
<img src="figs/tcp-stream-sp-tx-170914.png" width=100%>

Tx (TCP_STREAM)
</div>

<div class="right" style="width: 50%">
<img src="figs/tcp-stream-sp-tx-170914.png" width=100%>

Rx (TCP_MAERTS)
</div>

- Condition
 - phone: LKL v.s. native kernel


- Comparable goodput
- CPU utilization: LKL < native

>>>

## Multipath TCP

<div class="left" style="width: 50%">
<img src="figs/tcp-stream-mp-tx-170919-3.png" width=100%>

Tx (TCP_STREAM)
</div>

<div class="right" style="width: 50%">
<img src="figs/tcp-stream-mp-rx-170919-3.png" width=100%>

Rx (TCP_MAERTS)
</div>

- Condition
 - phone: LKL v.s. mptcp kernel


- Tx with native kernel <br> offers less goodput
 - even it's using multipath
- results are unstable

>>>

## Multipath TCP (Korea/KT)

(TBA)

- Condition
 - phone: LKL v.s. mptcp kernel

- (TODO) To be measured

>>>

## Observation

- IP conflicts may heavier
 - processed twice (host/lkl) per packet
- Results are always unstable
 - difficult measurement under wireless media

**TODO** (a fig of overhead)

---

## Limitations

- Implementations
 - DHCP only boot time (handover will fail)
 - IPv4 only on cellular interface (rmnet0)

- Required tweaks
 - grant NET_RAW permission (packet socket)
 - need filter out RST packet from host <br>

```
iptables -A OUTPUT -p tcp --tcp-flags RST RST  -j DROP
```


>>>

## Further investigations

- profiling
- other platform (iOS11 shipped userspace implementation)

>>>

## Summary

- Use out-of-tree kernel as a library on Android
 - make your code easier to distribute
 - But with privileged installation/operation
- Comparable performance wireless communication

