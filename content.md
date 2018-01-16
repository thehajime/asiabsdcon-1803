## Library Operating System project
### research review (2017) and plans (2018)

<span>
<br>
<br>
<br>
<br>

**Hajime Tazaki**

16 Jan. 2018
</span>

>>>

## Library OS project

A system research to relax current ossified Internet (protocol, software)

- Why ?
 - hard-to-upgrade kernel (network stack)
 - lack of reusablity
- How ?
 - **reconstruct** (feature-rich) monolithic kernel
 - **as a library**
 - not from scratch (incremental research)
- Core idea
 - Intelligence is in software part
 - Hardware related: should be small

<div class="right" style="width: 30%">
<img src="figs/lkl-arch-new.png" width=100%>
</div>

>>>

## 2017 review

- topics (planned)
 1. performance improvement of userspace network (BBR)
 1. unikernel on hypervisor (as a cloud service)
 1. network stack personality on mobile device (Android)
- topic (others)
 1. unikernel on bare-metal (on embedded devices)
 1. service chaining with Unix pipe (/dev/stdpkt)
 1. network stack personality on container service (docker)
 1. network simulator extensions (ns3, w/ matt)
 1. **operate** a service with PoC using LKL (mptcp proxy)

>>>

## 2017 publications

- papers
 1. Motomu Utsumi, Hajime Tazaki, Hiroshi Esaki, /dev/stdpkt: A Service Chaining Architecture with Pipelined Operating System Instances in a Unix Shell, AINTEC 2017

- talks
 1. "Playing BBR with a userspace network stack", 2017/4/7, Linux netdev 2.1.
 1. Motomu Utsumi, Hajime Tazaki, Hiroshi Esaki, /dev/stdpkt: A Service Chaining Architecture with Pipelined Operating System Instances in a Unix Shell, ACM APSys 2017 (poster)
 1. Cristina Opriceana, Hajime Tazaki, Network stack personality in Android phone Speakers, Linux netdev 2.2
 1. (planned) AsiaBSDCon 2018 keynote, March, 2018

>>>

## 2017 misc. activities

- research collaboration
 - extending Monroe platform (docker-based) with LKL (w/ Hoang Tran-Viet, UCL)
 - multipath TCP/QUIC experimental platform (w/ Quentin De Coninck, UCL)
 - NEDO (w/ Alaxala, etc)
 - EU-JP (Horizon2020/MIC) proposal (w/ NEC Europe, etc)

- services
 1. CFI2017 TPC
 1. WNS3 2017 TPC
 1. WNS3 2018 TPC

>>>

## 2017 trips

1. tazaki: 2017/4/6-4/10 Linux netdev 2.1/Montreal, Canada
1. tazaki: 2017/9/1-9/5 APSys 2017, Mumbai India
1. tazaki: 2017/11/7-11/10 Linux netdev 2.2/Seoul, Korea
1. tazaki: 2017/11/19-11/23 AINTEC 2017, Bangkok, Thailand

>>>

## 2017 budget

- not planned
- 2 android phones

>>>

## 2017 self evaluation

- Performance improvements: fair
- Operation experience with live network: good
- Promote IIJ's presence (publications, talks, services): fair

---

## 2018 challenges

- Expand use-cases of Library OS (LKL)
 - Gain real venues of use-case (lab network, product?)
- Upstream LKL to (mainline) Linux kernel (cont'd)

>>>

## 2018 plans

- topics (planned)
 1. ~~performance improvement of userspace network (BBR)~~
 1. unikernel on hypervisor (as a cloud service)
 1. ~~network stack personality on mobile device (Android)~~
- topic (others)
 1. unikernel on bare-metal (on embedded devices)
 1. ~~service chaining with Unix pipe (/dev/stdpkt)~~
 1. network stack personality on container service (docker)
 1. **operate** a service with PoC using LKL (mptcp proxy)
 1. network simulator extensions (ns3, w/ matt)
 1. network stack personality on **foreign OS (BSD/macOS)**


>>>

### 2018 budget, travels

- Travels
 - HotCloud'18
 - APSys '18
 - Linux netdev 3.1, 3.2
 - (EuroBSDCon ?)


- Budget
 - 1 or 2 VPSs
<br>
<br>
<br>
<br>
