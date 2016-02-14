## LibOS as a regression test framework for Linux networking

<span>
<br>
<br>
<br>
<br>

### Hajime Tazaki<br>

2016/02/12

netdev 1.2
</span>

---

# outline

- libOS introduction
- testing framework introduction
- case studies
- QA

>>>


# what is LibOS ?
- Library version of Linux kernel

- presented at netdev0.1, proposed to LKML (2015)

http://www.slideshare.net/hajimetazaki/library-operating-system-for-linux-netdev01

Note:
## the LibOS in a nutshell

- NUSE + DCE
- recently effort devoted to LKL (bit status info.)

- LibOS is
 - a library of Linux kernel
 - to decompose only required part of huge software stack
 - attach to an application
 - hijack calls
 - make a unikernel
 - flexible/agile virtual environment (dlmopen isolation)

>>>

# media

- LWN
 - https://lwn.net/Articles/637658/
- Phoronix
 - http://www.phoronix.com/scan.php?page=news_item&px=Linux-Library-LibOS
- Linux Magazine
 - http://www.linux-magazine.com/Issues/2015/176/Kernel-News
- Hacker News
 - https://news.ycombinator.com/item?id=9259292


>>>

# how to use it ?

- Network Stack in Userspace (NUSE)
 - **LD_PRELOADed** application
 - Network stack personality
- Direct Code Execution (DCE, ns-3 network simulator)
 - Network simulation integration (running Linux network stack on ns-3)

>>>


# what is **NOT** LibOS?

- _not only_ a userspace operating system
- _not only_ a debuging tool
<p>
- but LibOS _is_
 - a library which can link with **any programs**
 - a library to form any purpose of program

Note:
- (img: multi-instance on a single process)
- (img: kernel bypass with full *kernel* stack)

>>>

# anykernel

- introduced by a NetBSD hacker (rump kernel)
- Definition:
>We define an anykernel to be an organization of kernel code which allows the kernel's **unmodified** drivers to be **run in various configurations** such as application libraries and microkernel style servers, and also as part of a monolithic kernel.  -- Kantee 2012.

- can form various kernel for various platforms
- userspace (POSIXy), bare-metal, qemu/kvm, Xen
 - **Unikernel ?**<!-- .element: class="fragment" data-fragment-index="1" -->

Note:

<img src="figs/unikernels.jpg" width=70%>


<small>
http://slides.com/technolo-g/intro-to-unikernels-and-erlang-on-xen-ling-demo/
</small>

>>>



## single purpose operating system

<div class="left" style="width: 40%">
<img src="figs/CloudOSDiagram.png">

<small>
- http://www.linux.com/news/enterprise/cloud-computing/751156-are-cloud-operating-systems-the-next-big-thing-
</small>
</div>

<div class="right" style="width: 50%">
<li> Strip downed software stack
<li> single purpose
<li> resource efficient with speed
<li> boot within TCP 3-way handshake [1]

<br/>
<br/>
<br/>
<br/>
<br/>
<br/>

<small>
[1]: Madhavapeddy et al., Jitsu: Just-In-Time Summoning of Unikernels, USENIX NSDI 2015
</small>

</div>

>>>

## demos with linux kernel library

<div class="left" style="width: 50%">
<img src="figs/franken-linux-ping6.gif" width="840">
Unikernel on Linux (ping6 command embedded kernel library)
</div>

<div class="right" style="width: 50%">
<img src="figs/franken-qemu-arm-hello.gif" width="840">
Unikernel on qemu-arm (hello world)
</div>


>>>

# what's different ?

- User Mode Linux
 - generate executable of Linux kernel in userspace
 - no shared library
- Containers
 - no foreign OS (shared kernel with host)
- nfsim
 - broader coverage of kernel code

>>>

# recent news

- Linux kernel library (LKL) is coming
 - by Octavian Purdila (Intel)
 - since 2007, reborn in late 2015 

***LibOS project is going to migrate to LKL project***<!-- .element: class="fragment" data-fragment-index="1" -->

- port NUSE code to LKL already <!-- .element: class="fragment" data-fragment-index="1" -->
- DCE (ns-3 integration) not yet <!-- .element: class="fragment" data-fragment-index="1" -->
- unikernel in progress <!-- .element: class="fragment" data-fragment-index="1" -->

---

# testing network stack

>>>

# motivation

- testing networking code is hard
 - complex cabling
 - inefficiency with massive VM instances
- You may do
 - in your own large testbed
 - with your test programs

<img src="figs/complicate-cabling.png" width=20%>


>>>

# are we enough ?

<div class="left" style="width: 30%">
  <img src="figs/numcommit-net.png">
  - the number of commit per day
</div>

- frequently changing codebase 
 - many commits (30~40 commits/day)
 - out of 982K LoC (```cloc net/```)
 - may have increased num of regression bugs

Note:
git log --since "$1" --format=format:"%cd" --date=short --no-merges --all | sort
 | uniq -c  | \
gnuplot -e  "set terminal png; set output \"/tmp/test.png\"; set xdata time; set
 timefmt \"%Y-%m-%d\"; set xlabel \"date\"; set ylabel \"\#commits/day\"; set xt
ics rotate by 45 right; plot '-' using 2:1 with boxes title \"net/\" "


>>>

# your test

<img src="figs/1link-topo.png" width=50%>

- easy to create in your laptop with VM (UML/Docker/Xen/KVM) 
- ***only IF*** the test is enough to describe

>>>

# your test (cont'd)

<div class="left" style="width: 45%">
<img src="figs/rocketfuel.png" width=100%>
</div>

- huge resources to conduct a test
- not likely to **reproduce**
- tons of **configuration scripts**
- running on different machines/OSes
 - controling is troublesome
 - distributed debugger... 

Note:

## (picture of complex topology)

>>>

## many terminal windows with gdb

<img src="figs/many-gdb-consoles.png" width=80%>


Note:
## (picture of many terminal windows with gdb)

- hard to create with them
- ease to conduct in LibOS


<img src="figs/high-loadavg.jpg" width=60%>

>>>

# other projects

- Test suites/projects
 - LTP (Linux test project, https://linux-test-project.github.io/)
 - kselftest (https://kselftest.wiki.kernel.org/)
 - autotest (http://autotest.github.io/)
 - ktest (in tree, http://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/tree/tools/testing/ktest?id=HEAD)
 - kernelci (https://kernelci.org/)
 - NetDEF CI (quagga)
<p>
- those are great but **networking is always hard**
 - controlling remote hosts is (sometimes) painful
 - combination of userspace programs are unlimited
 - timing is not deterministic, across distributed networks

>>>

# why LibOS ?

- **single process model** with multiple nodes
 - ease of debug/test/development
- **deterministic behavior** (by ns-3 network simulator)
- rich network configuration by **ns-3 network simulator**
- ease of testing by automation (on public CI server)

>>>

## public CI server (circleci.com)

<img src="figs/circleci.png" width=70%/>

- test per commit (push)
- test *before* commit
- easily detect regressions

>>>

# architecture

<div class="left" style="width: 50%">
<ol>
<li> Virtualization Core Layer<br>
 - deterministic clock of simulator<br>
 - stack/heap management<br>
 - isolation via **dlmopen(3)** <br>
 - **single process model** <br>
<li> Kernel layer<br>
 - reimplementation of API <br>
 - glue/stub codes for kernel code <br>
 - use as-is
<li> POSIX glue layer<br>
 - reimplementation of POSIX API<br>
 - hijack host system calls<br>
</ol>
</div>

<div class="right" style="width: 50%">
<img src="figs/dce-arch.png" />
</div>

>>>

# How ?

- a single scenario script (C++, sorry) to describe all
 - application, network stack (kernel as a lib), traffic, link, topology, randomness, timing, etc

<br>

1. Recompile your code
 - Userspace as Position Independent Executable (PIE)
 - Kernel space code as shared library (libsim-linux.so)
2. Run with ns-3
 - Load the executables (binary, library) in an isolated environment among nodes
 - synchronize simulation clocks with apps/kernels clock


>>>

# features

- app supports
 - routing protocols (Quagga)
 - configuration utilities (iproute2)
 - traffic generator (iperf/ping/ping6)
 - others (bind9, unbound, dig)
<p>
- protocol supports
 - IPv4/ARP/IPv6/ND
 - TCP/UDP/DCCP/SCTP/(mptcp)
 - L2TP/GRE/IP6IP6/FOU


>>>

# what's _not_ useful

- performance study of the computation
 - deterministic clock assumes **unlimited** computation/storage resources
 - e.g., you can define 100Tbps link without any packet loss

Note:
- performance study under resource **constraint** is reasonable
 - e.g., observe congestion control algorithm on 1Gbps link (max)


# performance

- speed in a simple case, complex cases
- time warp
<p>
- scalability
 - parallel/distributed simulation
 - maximum: xxxK (cite:)

>>>

# test suite list
- verify results
 - socket (raw{6},tcp{6},udp{6},dccp{6},sctp{6})
 - encapsulation (lt2p,ip6ip6,ip6gre,fou)
 - quagga (rip,ripng,ospfv{2,3},bgp4,radvd)
 - mptcp
 - netlink
 - mip6 (cmip6,nemo)
<p>
- simple execution
 - iperf
 - thttpd
 - mptcp+iperf handoff
 - tcp cc algo. comparison
 - ccnd

>>>

## bugs detected by DCE (so far)

- having nightly tested with the latest net-next (since Apr. 2013~=4yrs)
<p>
- [net-next,v2] ipv6: Do not iterate over all interfaces when finding source address on specific interface. (v4.2-rc0, **during VRF**)
 - detected by: http://ns-3-dce.cloud.wide.ad.jp/jenkins/job/daily-net-next-sim/958/testReport/
- [v3] ipv6: Fix protocol resubmission (v4.1-rc7, **expanded from v4 stack**)
 - detected by: http://ns-3-dce.cloud.wide.ad.jp/jenkins/job/umip-net-next/716/
- [net-next] ipv6: Check RTF_LOCAL on rt->rt6i_flags instead of rt->dst.flags (**v4.1-rc1, during v6 improvement**)
 - detected by: http://ns-3-dce.cloud.wide.ad.jp/jenkins/job/daily-net-next-sim/878/
- [net-next] xfrm6: Fix a offset value for network header in _decode_session6 (v3.19-rc7?, **regression only in mip6**)

Note:
 - patchwork: http://patchwork.ozlabs.org/patch/493675/
 - patchwork: http://patchwork.ozlabs.org/patch/482645/
 - patchwork: http://patchwork.ozlabs.org/patch/467447/
 - patchwork: http://patchwork.ozlabs.org/patch/436351/

---


# Use Case

>>>

## network simulator in a nutshell

- (mainly research purpose)
- flexible parameter configurations
- usually in a single process
 - can be extended distributed/parallel processes for speedup
- usually with abstracted protocol implementation
 - but **no abstraction** this time (thanks to LibOS)
- always produce same results (**deterministic**)
 - can inject pseudo-randomness
 - not realistic sometimes
 - but useful for the test (always reproducible)

>>>

# workflow

0. (installation of DCE)
```
make testbin -C tools/testing/libos
```
1. <strike> develop a model (of interests) </strike>
 - (you already have: the Linux network stack)
2. write a simulation scenario
 - write a network topology
 - parameters configuration (randomization seed, link, traffic, applications)
3. test it
 - one-shot (locally)
 - nightly, per-commit, per-push, etc


>>>

## simulation scenario

~~~c
int main(int argc, char **argv)
{
  // create nodes
  NodeContainer nodes;
  nodes.Create (100);

  // configure DCE with Linux network stack
  DceManagerHelper dce;
  dce.SetNetworkStack ("ns3::LinuxSocketFdFactory", 
                      "Library", StringValue ("libsim-linux-4.4.0.so"));
  dce.Install (nodes);

  // run an executable at 1.0 second on node 0
  DceApplicationHelper process;
  ApplicationContainer apps;
  process.SetBinary ("your-great-server");
  apps = process.Install (nodes.Get (0));
  apps.Start (Seconds (1.0));

  Simulator.Stop (Seconds(1000.0))
  Simulator.Run ()  
}
~~~

>>>

## API (of DCE helpers)

- userspace app
 - ```ns3::DceApplicationHelper class```
- kernel configuration
 - sysctl with ```LinuxStackHelper::SysctlSet()``` method
- printk/log
 - generated into ```files-X``` directory (where X stands for the node number)
 - syslog/stdout/stderr tracked per process (files-X/var/log/{PID}/)
- an instant command (```ip```)
 - ```LinuxStackHelper::RunIp()```
- **manual**
 - https://www.nsnam.org/docs/dce/manual/html/index.html

>>>

# test it !

- use ```waf``` for a build the script
```
cd tools/testing/libos/buildtop/source/ns-3-dce/
./waf
```
- run the script with ```test.py``` to generate XUnit test results
```
./test.py -s exapmle -r
```
- run the script with valgrind
```
./test.py -s exapmle -g
```
- a wrapper in Makefile
```
make test ARCH=lib ADD_PARAM=" -s example"
```

(*the directories may be changed during upstream (etc)*, **sorry 'bout that**)


>>>

## case study: encapsulation test


ns-3-dce/test/addons/dce-linux-ip6-test.cc

<img src="figs/encap-test-topology.png" width=50%/>

- unit tests for encapsulation protocols
 - ip6gre, ip6-in-ip6, l2tp, fou
 - with ```iproute2```, ```ping6```, libsim-linux.so (libos)
<p>
- full script
 - https://github.com/direct-code-execution/ns-3-dce/blob/master/test/addons/dce-linux-ip6-test.cc

>>>

## encap protocols tests

1) tunnel configurations

```
 LinuxStackHelper::RunIp (nodes.Get (0), Seconds (0.5),
                          "-6 tunnel add tun1 remote 2001:db8:0:1::2 "
                          "local 2001:db8:0:1::1 dev sim0");
 LinuxStackHelper::RunIp (nodes.Get (1), Seconds (0.5),
                          "-6 tunnel add tun1 remote 2001:db8:0:1::1 "
                          "local 2001:db8:0:1::2 dev sim0");
```

2) set up ping6 command to generate probe packet

```
 dce.SetBinary ("ping6");
 dce.AddArgument ("2001:db8:0:5::1");
 apps = dce.Install (nodes.Get (1));
 apps.Start (Seconds (10.0));
```

3) verify if the encap/decap work fine or not
```
 if (found && icmp6hdr.GetType () == Icmpv6Header::ICMPV6_ECHO_REPLY) {
    m_pingStatus = true;
 }

```


Note:

# Preparation

- edit the script in a special directory
- putting a file into ```ns-3-dce/test/addons/``` will be (internally) treated as a test script
```
vi tools/testing/libos/buildtop/source/ns-3-dce/test/addons/dce-linux-ip6-test.cc
```
- run it
```
make test ARCH=lib ADD_PARAM=" -s linux-ip6-test -r"
```

>>>

# That's it. Test Test Test !

>>>


## XUnit test result generation

- ```make test ARCH=lib ADD_PARAM=" -s linux-ip6-test -r"``` gives you a test result retained

```
% head testpy-output/2016-02-08-09-49-32-CUT/dce-linux-ip6.xml

<Test>
  <Name>dce-linux-ip6</Name>
  <Result>PASS</Result>
  <Time real="3.050" user="2.030" system="0.770"/>
  <Test>
    <Name>Check that process &#39;plain&#39; completes correctly.</Name>
    <Result>PASS</Result>
    <Time real="0.800" user="0.370" system="0.310"/>
  </Test>
  <Test>
    <Name>Check that process &#39;ip6gre&#39; completes correctly.</Name>
    <Result>PASS</Result>
    <Time real="0.600" user="0.460" system="0.100"/>
  </Test>
  <Test>
    <Name>Check that process &#39;ip6ip6&#39; completes correctly.</Name>
    <Result>PASS</Result>
    <Time real="0.520" user="0.410" system="0.090"/>
  </Test>
```

>>>

<!-- .slide: data-background="figs/jenkins-test-failure-example.png" -->

>>>

# git bisect

you can now _bisect_ a bug with a single program !

- prepare a bisect.sh

```
#!/bin/sh

git merge origin/nuse --no-commit
make clean ARCH=lib
make library ARCH=lib OPT=no

make test ARCH=lib ADD_PARAM=" -s dce-umip"

RET=$?
git reset --hard

exit $RET
```

- run it !

```
git bisect run ./bisect.sh 
```

Note:

something needs to be merged since libos is out-of-tree code


>>>

## gcov (coverage measurement)

-  coverage measurement across multiple nodes

~~~
make library ARCH=lib COV=yes
make test ARCH=lib
~~~
(the **COV=yes** option does the job for you)


<img src="figs/jenkins-gcov-output.png" />

>>>

# gdb (debugger)

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

# valgrind

<img src="figs/valgrind.png" width="640">

- Memory error detection
 - among distributed nodes
 - in a single process
- Use **Valgrind**

<span>
http://yans.pl.sophia.inria.fr/trac/DCE/wiki/Valgrind
</span>

Note:

# how useful

- ```git bisect```
- code coverage (gcov)
- ```gdb```
- ```valgrind```
- contiunous integration (CI)


---

# Summary

- walk through review of testing framework with LibOS + DCE

- uniqueness of experiemnt with the library (LibOS)
 - multiple (host) instances in a single process
 - flexible network configurations
 - deterministic scheduler (i.e., bugs are always reproducible)


>>>

# future directions

- merging to LKL (Linux Kernel Library)
 - part of LibOS has done
- continuous testing to net-next branch
 - I'm watching at you (don't get me wrong.. :))

>>>

# resources

- Web
 - https://www.nsnam.org/overview/projects/direct-code-execution/ (DCE specific)
 - http://libos-nuse.github.io/ (LibOS in general)
- Github
 - https://github.com/libos-nuse/net-next-nuse
- LKL (Linux Kernel Library)
 - https://github.com/lkl/linux

