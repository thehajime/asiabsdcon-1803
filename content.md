## (no title)
### LibOS/LKL updates

<span>
<br>
<br>
<br>
<br>

**Hajime Tazaki**

Feb. 2018
</span>

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
(Nov. 2017)
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

## MPTCP on Android

- Linux mptcp isn't upstreamed
- **have to** upgrade firmware with custom image
 - hard to upgrade android kernel
 - kernel version dependency to Android release (5.0 - 8.0)

- Use userspace mptcp (of LKL)
 - ease of deployment
 - a little bitter bits..


>>>

## How it works

- For smooth replacement (i.e., hijack) for Android UI app syscalls (java-based)
 - bionic is more familiar than glibc
 - only socket-related calls are redirected
 - handling a mixture of host and lkl descriptors


<img src="figs/lkl-android-hijack.png" width=40%>

>>>

## Demo

---

## Call for participants

- MPTCP-enabled <br> proxy deployment
 - your browser will benefit <br> multipath connectivity
- No kernel upgrade


<div class="right" style="width: 50%">
<img src="figs/mptcp-proxy-polipo.png" width=100%>
</div>

>>>

## Instruction

- configure a proxy server
 - in your system config, or your browser config
- proxy
 - 202.214.86.168:22
 - auth: lkl/mptcp
- feedback
 - `#lkl-exp` (slack)
 - ML (iijlab@iij.ad.jp)
