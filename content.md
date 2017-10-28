## Linux Kernel Library on Android

<span>
<br>
<br>
<br>
<br>

Hajime Tazaki

15 Sep. 2017
</span>

Note:

## Outline

- LKL
- MPTCP as an example of extension
- Benchmarks
 - setup
- Single path TCP (netperf)
- Multi-path TCP (netperf)

---


## What is LKL ?

- Run Linux code on various ways
- h/w independent arch (layer)
- h/w dependent layer
 - on Linux/Windows <br>/FreeBSD uspace, <br> unikernel, on UEFI, 
 - (ns-3)<!-- .element: class="fragment" data-fragment-index="1" -->
 - ** Android<!-- .element: class="fragment" data-fragment-index="2" -->**

<div class="right" style="width: 40%">
<img src="figs/fig2-lkl-arch.png" width=100%>
</div>


>>>

## Extensions

- Android (arm/arm64) support
- MPTCP port to LKL

- HOWTO
 - LD_PRELOAD=liblkl.so netperf XXX (console app)
 - setprop wrap.app LD_PRELOAD=liblkl.so (Java app)

>>>

<img src="figs/amiusingmptcp-nexus5.jpg" width=35%>
<img src="figs/amiusingmptcp-nexus5-2nd.jpg" width=35%>

>>>

## Why LKL on Android ?

- Hard to upgrade a kernel
 - 5 yrs old android kernel (3.4.0+ on android-6)
 - Long time to ship
 - Out-of-tree code is hard too


>>>

## NUSE: network stack personality

- Linux MPTCP as an example
- Goal
 - To use Linux mptcp w/o replacing kernel
 - With tolerable amount of overhead

- Questions
 - Is NUSE working fine (Will users wanna use it) ?
 - How different from native Linux kernel ?



---

## netperf measurement

- Machines
 - Nexus5 anrdoid 6.01 (rooted)
 - Ubuntu 16.04 (amd64) on KVM
- Network
 - Nexus5: LTE, wifi
 - Ubuntu: virtio/Etherlink (100 Mbps)
- Software
 - netperf 2.7.x
 - 10 seconds TCP_STREAM
 - LKL arm/android patched
 - Ubuntu: mptcp-4.4.70 (v0.92),

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

- Comparable goodput
- CPU utilization: LKL < native 

<p>
<p>
<p>



>>>

## Single path (Wi-Fi only, 9/19)

<div class="left" style="width: 50%">
<img src="figs/tcp-stream-sp-tx-170919.png" width=100%>

Tx (TCP_STREAM)
</div>

<div class="right" style="width: 50%">
<img src="figs/tcp-stream-sp-tx-170919.png" width=100%>

Rx (TCP_MAERTS)
</div>

- LKL consumes more CPU than 9/14 and native

>>>

## Multipath (LTE+WiFI, 9/14)

<div class="left" style="width: 50%">
<img src="figs/tcp-stream-mp-tx-170913.png" width=100%>

Tx (TCP_STREAM)
</div>

<div class="right" style="width: 50%">
<img src="figs/tcp-stream-mp-rx-170913.png" width=100%>

Rx (TCP_MAERTS)
</div>



<br>
<br>
<br>
<br>
<br>
<br>

- LKL: aggreated (wifi+lte)
- native: Single path (wifi)
- CPU utilization: LKL > native

>>>

## Multipath (LTE+WiFI, 9/19)

<div class="left" style="width: 50%">
<img src="figs/tcp-stream-mp-tx-170919.png" width=100%>

Tx (TCP_STREAM)
</div>

<div class="right" style="width: 50%">
<img src="figs/tcp-stream-mp-rx-170919.png" width=100%>

Rx (TCP_MAERTS)
</div>

>>>

## Multipath (LTE+WiFI, 9/19-2 new mpctl.apk)

<div class="left" style="width: 50%">
<img src="figs/tcp-stream-mp-tx-170919-2.png" width=100%>

Tx (TCP_STREAM)
</div>

<div class="right" style="width: 50%">
<img src="figs/tcp-stream-mp-rx-170919-2.png" width=100%>

Rx (TCP_MAERTS)
</div>

- CPU util. of LKL always shows 25% (maybe bug)
- LTE link often (every 2~5 mins?) changed IP address
 - some results can't use multiple flows
 - especially LKL doesn't raise netlink events after execution

>>>

### Multipath (LTE+WiFI, 9/19-3, no addr changes)

<div class="left" style="width: 50%">
<img src="figs/tcp-stream-mp-tx-170919-3.png" width=100%>

Tx (TCP_STREAM)
</div>

<div class="right" style="width: 50%">
<img src="figs/tcp-stream-mp-rx-170919-3.png" width=100%>

Rx (TCP_MAERTS)
</div>

- LTE link and IP addr are stable (no changes)
 - due to network condition ?
- Tx with native kernel offers less goodput
 - even it's using multipath

>>>

## Multipath (LTE+WiFI, 9/22)

<div class="left" style="width: 50%">
<img src="figs/tcp-stream-mp-tx-170922.png" width=100%>

Tx (TCP_STREAM)
</div>

<div class="right" style="width: 50%">
<img src="figs/tcp-stream-mp-rx-170922.png" width=100%>

Rx (TCP_MAERTS)
</div>


---

- Limitation
 - DHCP only boot time (handover will fail)
 - IPv4 only on cellular interface (rmnet0)

- Required tweaks
 - grant NET_RAW permission (AndroidManifest.xml, Xposed)
 - disable selinux (setenforce 0)
 - need filter out RST packet from host <br>
 (iptables -A OUTPUT -p tcp --tcp-flags RST RST  -j DROP)


>>>

## Further investigations

- LKL/MPTCP v.s. Native MPTCP
 - to see the overhead by *network personality*
- test with http://amiusingmptcp.de

Note:
- and iOS11 ?