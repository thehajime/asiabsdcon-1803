## Network stack personality in Android phone

<span>
<br>
<br>
<br>
<br>

Cristina Opriceana, **Hajime Tazaki**  (IIJ Research Lab.)

Linux netdev 2.2, Seoul, Korea

08 Nov. 2017
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

## Librarified Linux taLks (LLL)

- Userspace network stack (NUSE) in general (netdev0.1)
- kernel CI with libos and ns-3 (netdev1.1)
- Network performance improvement of LKL (netdev1.2, by Jerry Chu)
- How bad/good with LKL and hrtimer (BBR) (netdev2.1)
- Updating Android network stack (netdev2.2)<!-- .element: class="fragment" data-fragment-index="1" -->

>>>

## Android
### a platform of billions devices

- billions installed Linux kernel

- Questions
 - When our upstreamed code <br> available ?
 - What if I come up with <br> a great protocol ?

<div class="right" style="width: 50%">
<img src="figs/android-platform-distribution-version.png" width=100%>

<small>
https://developer.android.com/about/dashboards/index.html
</div>


Note:
- slow delivery of software <br>updates

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


**Long delivery to all billions devices**

>>>

### Approaches to alleviate the issue

- Virtualization (KVM on Android)
 - Overhead isn't negligible to embedded devices
- Project Treble (since Android O)
 - More modular platform implementation
- Fushia
 - Rewrite OS from scratch
- QUIC (transport over UDP)
 - Rewrite transport protocols on UDP


<img src="figs/treble_blog_after.png" width=60%>



<small>

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

<img src="figs/lego-toy.jpg" width=30%>
<img src="figs/lego-toy2.jpg" width=30%>


- Motivation<!-- .element: class="fragment" data-fragment-index="1" -->
 - Trying to present that a Toy is practically useful<!-- .element: class="fragment" data-fragment-index="1" -->


Note:

- Don't care whether toy or not<!-- .element: class="fragment" data-fragment-index="1" -->
- Do care whether it's useful or not<!-- .element: class="fragment" data-fragment-index="1" -->

FUSE or not FUSE ? (FAST '17)


---

## Linux Kernel Library intro (again)

- Out-of-tree architecture <br> (h/w-independent)
- Run Linux code on various ways
 - with a reusable library
- h/w dependent layer
 - on Linux/Windows <br>/FreeBSD uspace, <br> unikernel, on UEFI, 
 - network simulator (ns-3)
 - **Android<!-- .element: class="fragment" data-fragment-index="2" -->**

<div class="right" style="width: 40%">
<img src="figs/lkl-arch-new.png" width=100%>
</div>

>>>

## LKL: current status

- Sent RFC (Nov. 2015)
 - no update on LKML since then
- have evolved a lot
 - fast syscall path
 - offload (csum, TSO/LRO)
 - CONFIG_SMP (WIP)
 - json config
 - qemu baremetal (unikernel)
 - on UEFI


https://github.com/lkl/linux

>>>

## Extensions to LKL

- Android (arm/arm64) support (lkl/linux#372)
- raw socket extension (only handle ETH_P_IP) (not upstreamed yet)
- hijack library enhance (not upstreamed yet)

>>>


## HOWTO

```
% LD_PRELOAD=liblkl-hijack.so netperf XXX # console app
% setprop wrap.app LD_PRELOAD=liblkl-hijack.so # Java app
```


```
{
"gateway": "10.206.211.1",
"interfaces": [
 {
 "ifgateway": "202.214.86.129",
 "ip": "202.214.86.168",
 "mac": "02:87:f8:27:22:02",
 "masklen": "26",
 "param": "/dev/tap23",
 "type": "macvtap"
 }
],
"debug": "0",
"singlecpu": "1",
"delay_main": "500000",
"sysctl": "net.ipv4.tcp_wmem=4096 87380 2147483647;net.mptcp.mptcp_debug=1"
}
```

>>>

## hijack library

- For smooth replacement (i.e., hijack) for Android UI app syscalls (java-based)
 - bionic is more familiar than glibc
 - only socket-related calls are redirected
 - handling a mixture of host and lkl descriptors


<img src="figs/lkl-android-hijack.png" width=40%>


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

Note:
   https://github.com/frootmig/amiusingmptcp/

>>>

## Multipath TCP

- An extension to TCP subsystem
- application compatibility <br> (unlike SCTP)
- Use multiple paths
 - better throughput <br> (aggregation)
 - smooth recovery from failure <br> (handover)

<div class="left" style="width: 50%">
<img src="figs/mptcp-tessares-fig.png" width=100%>
</div>

<small>
http://blog.multipath-tcp.org/blog/html/2015/12/25/commercial_usage_of_multipath_tcp.html

>>>

## Demo

- verify site (cat /proc/net/mptcp base detection)
  http://amiusingmptcp.de/

Note:

Nexus5 VNC
amiusingmptcp.de


<img src="figs/amiusingmptcp-nexus5.jpg" width=35%>
<img src="figs/amiusingmptcp-nexus5-2nd.jpg" width=35%>

---

## No penalty with userspace network stack ?

- Condition
 - To use Linux mptcp w/o replacing kernel

- Questions
 - Is NUSE working fine (Will users wanna use it) ?
 - How different from native Linux kernel ?
 - With tolerable amount of overhead ?

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

## Single path (Wi-Fi only)

<div class="left" style="width: 50%">
<img src="figs/tcp-stream-sp-tx-170914.png" width=100%>

Tx (TCP_STREAM)
</div>

<div class="right" style="width: 50%">
<img src="figs/tcp-stream-sp-tx-170914.png" width=100%>

Rx (TCP_MAERTS)
</div>

- Condition
 - phone: LKL v.s. (stock) kernel


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


- Goodput (Tx) LKL > native
 - even it's using multipath
- CPU: unstable (LKL)
 - LKL > native

>>>

## Multipath TCP (Korea/KT)

<div class="left" style="width: 50%">
<img src="figs/tcp-stream-mp-tx-171108.png" width=100%>

Tx (TCP_STREAM)
</div>

<div class="right" style="width: 50%">
<img src="figs/tcp-stream-mp-rx-171108.png" width=100%>

Rx (TCP_MAERTS)
</div>


- Condition
 - phone: LKL v.s. (stock) kernel
 - native uses single-path/<br>LKL uses multi-path
 - at Ibis hotel


- Goodput: No much gain with LKL
 - even it's using multipath
- CPU: unstable (LKL)
 - LKL > native

>>>

## Observations

- IP conflicts may heavier
 - processed **twice** (host/lkl) <br> per packet
- Results are often unstable
 - difficult measurement under <br> wireless media

<div class="right" style="width: 50%">
<img src="figs/packet-drop-iptables.png" width=100%>
</div>

---

## Limitations

- Implementations
 - DHCP only boot time (handover will fail)
 - IPv4 only on cellular interface (rmnet0)
- Fundamental limitations of hijack library
 - asynchronous signal unsafe
 - MT unsafe

- Required tweaks
 - grant NET_RAW permission (packet socket)
 - need filter out RST packet from host <br>

```
iptables -A OUTPUT -p tcp --tcp-flags RST RST  -j DROP
```


>>>

## Further investigations

- other platform
 - iOS11 now shipped userspace implementation
- profiling

>>>

## Summary

- Use out-of-tree kernel as a library on Android
 - *make your code easier to distribute*
 - with privileged installation/operation
- Comparable goodput over WiFi/LTE
- Unstable CPU utilization with LKL
- You can prepare your library file for your own purpose

---

## Backups


>>>

## Alternate network stacks

- lwip (2002~)
- mTCP [NSDI '14]
- SandStorm [SIGCOMM '14]
- rumpkernel [ATC '09]
- SolarFlare (2007~?)
- libuinet (2013~)
- SeaStar (2014~)



**None of them are feature-rich, or one-shot porting <!-- .element: class="fragment" data-fragment-index="1" -->**
