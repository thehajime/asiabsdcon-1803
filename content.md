## Linux rumpkernel
### yet another virtualization with a librarified kernel

<span>
<br>
<br>
<br>
<br>

**Hajime Tazaki**

IIJ Research Laboratory

March, 2018, Abscission 2018
</span>

---

## Intro

- Linux in BSD conference
- Network stack evolution
- with NetBSD derived technology (rump)

>>>

## Who I am ?
- 

>>>

## The original Internet

  <img src="figs/baran-distributed.png" width="70%"/>

- Packet switching network
 - a basis of end-to-end principle
 - a basis of hugest platform

<small>
P. Baran, On Distributed Communications Networks, IEEE Transactions on Communications Systems, 1964
</small>


Note:

- baran's diagram
- e2e

>>>

## today's internet

- not yesterday's Internet

<img src="figs/complex-mind.jpg" width="30%"/>
<img src="figs/goverment-control.jpg" width="30%"/>
<img src="figs/security-fast.png" width="15%"/>

various stake holders / governmental control / security fast

<small>
- refs:
 - https://justimagine.aurecongroup.com/solving-complex-problems-forget-what-you-currently-know/
 - https://kentforliberty.liberty.me/letting-government-control-you/


Note:

- My dream about free-form internet
- can we ? no..

- pictures
 - (middlebox jam)
 - (governmental control)
 - (security-first: default deny forwarding)

>>>

## today's internet (cont'd)

<img src="figs/no-more-e2e.png" width="80%"/>

- a packet is hard to deliver to the others without any modifications

<small>
- ref:
https://www.slideshare.net/obonaventure/innovation-is-back-in-the-transport-and-network-layers

>>>

### the end of evolution/innovation ??

- internet is mature enough (that we don't have to modify)
- we can create another universe

(figure: end of the world)

- are we satisfied ? no probably
 - people want to but the system is not ready

>>>

## Questions

- Why do you want to update your network stack ?
 - want to put new idea
 - want to refresh design (e.g., socket API sucks)
 - want to optimize implementations
 - want to secure codes

---

## What's the matter ?

>>>

## Problems

- more ossification/no innovation  <!-- .element: class="fragment grow" data-fragment-index="1" -->
- more low-quality codes  <!-- .element: class="fragment shrink" data-fragment-index="1" -->
- more waste of time  <!-- .element: class="fragment shrink" data-fragment-index="1" -->
- no productive implementation  <!-- .element: class="fragment shrink" data-fragment-index="1" -->
- no more end-to-end  <!-- .element: class="fragment shrink" data-fragment-index="1" -->
- no more experimental platform  <!-- .element: class="fragment shrink" data-fragment-index="1" -->


<div class="right" style="width: 20%">
<img src="figs/problem.jpg" width="100%"/>

<small>
https://pixabay.com/en/question-problem-think-thinking-622164/
</div>

>>>

## More ossification
- two obstacles for new protocol deployment
 - middlebox
 - host OS

>>>

## Ossification: middlebox

<img src="figs/pkt-process-router-olivier.png" width="80%"/>

TCP segments processed by a router
<small>
- ref:
 - https://www.slideshare.net/obonaventure/innovation-is-back-in-the-transport-and-network-layers

>>>

## Ossification: middlebox (cont'd)

<img src="figs/pkt-process-nat-olivier.png" width="80%"/>

TCP segments processed by a NAT router
<small>
- ref:
 - https://www.slideshare.net/obonaventure/innovation-is-back-in-the-transport-and-network-layers

>>>

## Ossification: middlebox (cont'd)

<img src="figs/pkt-process-mbox-olivier.png" width="80%"/>

Possible TCP segments processed by typical middlebox today
<small>
- ref:
 - https://www.slideshare.net/obonaventure/innovation-is-back-in-the-transport-and-network-layers


>>>

## Ossification: host OS

<img src="figs/micchie-ccr-proto-evo.png" width="60%"/>

- Windowscale, Timestamp
 - since Windows 2000, Vista: default ON in 2006
- SACK/TS defaulted Linux 1999, Windows: 2004

<small>
Honda et al., Rekindling Network Protocol Innovation with User-Level Stacks, A
CM SIGCOMM CCR, Vol.44, Num. 2, April 2014
</small>

>>>

## Ossification: host OS (cont'd)

- Updating base kernel is not an easy task
 - Android still uses older kernel
 - Container guests use the host kernel (for network stack)
<br>
<br>

<img src="figs/android-platform-distribution-version.png" width=60%>

Android OS distribution with the base Linux kernel version

<small>
https://developer.android.com/about/dashboards/index.html
(taken Nov. 2017)


>>>

## An example: Multipath TCP

- An extension to (traditional) TCP
 - multipath communication
 - RFC6824 (experimental)
 - application compatibility <br> (unlike SCTP)

- Good design ?
 - middlebox friendly => **OK**
 - unmodified application => **OK**
 - modified kernel => **??**

<div class="left" style="width: 50%">
<img src="figs/mptcp-tessares-fig.png" width=100%>
</div>

<small>
http://blog.multipath-tcp.org/blog/html/2015/12/25/commercial_usage_of_multipath_tcp.html

Note:
- successfully deployed by apple (since ios7)

>>>

## Ossification: a Google's answer

- QUIC (Quick UDP Internet Connection)

- a transport protocol over UDP
- why UDP ?
 - with encrypted payload middlebox can't intercept
 - **middlebox friendly**
- why UDP (cont'd) ?
 - can be implemented in userspace
 - **no need to upgrade host OS**

>>>

## Ossification: can others deploy such a way ?

- No <!-- .element: class="fragment" data-fragment-index="1" -->
- Only a giant can <!-- .element: class="fragment" data-fragment-index="2" -->

<div class="left" style="width: 50%">
<img src="figs/giant-dora.jpg" width=100%> <!-- .element: class="fragment" data-fragment-index="2" -->
</div>


>>>

## if you face obstacles

- you would implement from scratch
 - as a name of *specialization*

- lack of maturity of an OS history
- more low-quality codes
- more waste of time (reinventing a wheel)
- no productive implementation (e.g., kernel bypass)

>>>

## summary of problems
- today's internet is not the original internet
- **no more end-to-end**
- to be flexible
 - be a part of giant (which I won't)
 - or thinks differently

---

## Alternatives

- lwip
- seastar
...

>>>

## numbers (alternatives)
(from labcamp's slides 1608)

>>>

sigh 

>>>

## Does speed matter ?

(from labcamp's slides 1608)
- not !
- don't forget about feature-richness

*writing an OS is one-week DIY, but writing a production quality of an OS is not a week DIY*
(quote from antti's usenix login;)

>>>

## Our goal
- Respect the implementation (and experience) of past decades
- Accelerate the innovation of network stack

***discover new values through the past studies***


---

## Our approach:

>>>

## Anykernel
 - History: Rump kernel
 - Detail:
 - transforming a monolithic kernel code into an Anykernel

>>>

## LKL: nutshell
- overview
- LoC
- feature sets

>>>

## LKL: internals
- Internals
 - design overview
 - host ops / rump hypcall
 - h/w independent layer

>>>

## 3 APIs
(from lab seminar slides)

>>>

## Execution

x86_64-rumprun-{linux,netbsd}-gcc

>>>

## LKL: side history
- LibOS, UML, ...

---

## Usages

>>>

## NUSE (uspace net)
- hijack library

>>>

## android/docker

>>>

## NUSE: performance num

>>>

## unikernel (frankenlibc/rumprun)
 - bare-metal

- Demo
 - this slides (reveal.js/nginx/lkl+frankenlibc/FreeBSD (on linux))

>>>

## NFV

>>>

## network simulation

>>>

## Debugging/Testing
- gdb/valgrind

>>>

## Summary

- Anykernel
