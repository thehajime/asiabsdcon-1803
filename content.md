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

>>>

## Intro

>>>

## Who I am ?
- 

>>>

## The original Internet
- baran's diagram
- e2e

>>>

## today's internet
 - yesterday's internet
 - My dream about free-form internet
 - can we ? no..

>>>

## the end of evolution/innovation ??
- internet is mature enough
- that we don't have to modify

- really ? no probablly

>>>

## Question
- Why do you want to update your network stack ?
 - want to put new idea
 - want to refresh design (e.g., socket API sucks)
 - want to optimize implementations
 - want to secure codes

---

## Problems/Frustrations/Reality (2)
 - more ossification/no innovation
 - more low-quality codes
 - more waste of time
 - no productive implementation
 - no more end-to-end


>>>

## more ossification
- two obstacles for new protocol deployment
 - middlebox
 - host OS

>>>

## ossification: middlebox issue
- olivier's slide
https://www.slideshare.net/obonaventure/innovation-is-back-in-the-transport-and-network-layers

>>>

## ossification: host OS upgrade
- Android's case (netdev22 slide)
- Docker container

- an example of mptcp
 - (current) middlebox friendly extension => **OK**
 - unmodified application => **OK**
 - modified kernel => **??**

- successfully deployed by apple (since ios7)

>>>

## ossification: a Google's answer

- QUIC (Quick UDP Internet Connection)

- a transport protocol over UDP
- why UDP ?
 - with encrypted payload middlebox can't intercept
 - **middlebox friendly**
- why UDP (cont'd) ?
 - can be implemented in userspace
 - no need to upgrade host OS

>>>

## ossification: can others deploy such a way ?

- No
- Only a giant can

https://dora-world.com/uassets/c0c/eddeebb9668c08fd2a8ffec379c64/dora511A_004_v1.mov.jpg

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
