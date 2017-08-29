## /dev/stdpkt with Function Chaining

<span>
<br>
<br>
<br>
<br>

Hajime Tazaki

(w/ Motomu Utsumi: University of Tokyo)

IIJlab camp 2017 Summer

</span>

Note:

## Outline

- NFV, what ?
 - a chain of functions
 - functions (NF)
 - inter-connect (chain)
- benefit
 - reduce costs
 - agility (no h/w installaion)
- challenges
 - performance on virtualization
 - SFC (function chaining)
 - migration
 - 

- NF + interconnect
 - virtual machine/container + soft-sw (bridge/ovs)
   => heavy NF (vm)
 - custom application (mos[nsdi], netvm[??]) + DPDK/vale
   => super fast but less functionality
 - clickos [nsdi] + xen netfront
   => super fast but less functionality

- Q: Is NFV familiar with Unix ?
- What is it ?
- How are others doing ?
- How its' useful ?



- What is NFV ?
- What is UNIX ?

- What if NFV is implemented with UNIX shell ?

---

## Network Function Virtualization

<img src="figs/what-is-nfv.jpg" width=80%>

<small>
https://www.globaltelecomsbusiness.com/article/b11vyg2p88wsqw/nfv-bringing-radical-change-in-way-networks-will-be-planned-built-operated-and-maintained

>>>

## NFV


- Virtualize everything
 - adapt server virtualization <br>into network functions
 - **hardware appliances <br> => software appliances**

- Benefit
 - lower cost
 - precise control <br>(per-flow, per-user)

- Challenges
 - chain of functionality
 - service migration

<div class="right" style="width: 50%">
<br>
<br>
<br>
<img src="figs/what-is-nfv.jpg" width=100%>
</div>

<small>
https://www.globaltelecomsbusiness.com/article/b11vyg2p88wsqw/nfv-bringing-radical-change-in-way-networks-will-be-planned-built-operated-and-maintained


Note:

- ETSI ?
- Virtual appliance ?
- Core idea
 - Chain of Function in Virtualized facility


>>>

- before

<img src="figs/before-nfv.png" width=80%>

- after <!-- .element: class="fragment" data-fragment-index="1" -->

<img src="figs/after-nfv.png" width=80%> <!-- .element: class="fragment" data-fragment-index="1" -->


>>>

## Requirements

- Network Function (NF)
 - Agility (quick instantiation as *inetd*)
 - Feature richness


- Inter-connect (of NF)
 - Flexibility of functions
 - Rich chain expression
 

Note:

- (Isolation of function(s))


>>>

## Function chain ??

- Service Function Chain (SFC)
 - An output of 1st func. can be an input of 2nd func.
<center>
<img src="figs/nfv-ericsson-mirage.png" width=50%>
- function chain == UNIX pipe (connect in/out) <!-- .element: class="fragment" data-fragment-index="1" -->

** SFC can be expressed by UNIX pipes ? ** <!-- .element: class="fragment" data-fragment-index="2" -->

<small>
ref: http://unikernel.org/blog/2016/unikernel-nfv-platform


>>>

## Unix pipe

- Douglas McIlroy (Bell lab., ~1964?*)
- *Make each program do one thing well.*

<small>
> a pipeline is a sequence of processes chained together by their standard streams
</small>


```
% cat data.txt | awk '{print $1}' | grep -i "important"
```



<small>
'* Bell Lab, THE UNIX ORAL HISTORY PROJECT
</small>

>>>

## (function) chaining with pipes

- (function) chaining with pipes

```
% cmd1 | cmd2 | cmd3
```

- function chaining

```
% vm1 | vm2 | vm3 ??
```

- Inter-VM communication by pipes
- ?? bidirectional communication
- ?? feature-rich functions ?
- ?? quick startup of vms

---

## EtherPIPE

- A character device for (FPGA) NIC
 - /dev/etherpipe/{0,r0}
- A device abstraction of NIC
 - together with Unix shell
 - Unix commands as VNF

<img src="figs/etherpipe.png" width=45%>
<small>
- Kuga et al., EtherPIPE: an Ethernet character device for network scripting, ACM HotSDN 2013

>>>

- packet dump by `cat` command
<small>
```
% cat /dev/ethpipe/0
0000000000000000 A0369F1850e5 001C7E6ABAD1 0800 45 00 00 4E 00 00 40 00 40 11 FB 32 0A 00 00 6E 0A 00 00 02 04 04 00 89 00 3A 38 03 10 FD 01 10 00 01 00 00 00 00 00 00 20 46 45 45 4E 45 42 46 45 46 44 46 46 46 4A 45 42 43 4E 45 49 46 41 43 41 43 41 43 41 43 41 43 41 00 00 20 00 01
0000000000000000 A0369F1850e5 001C7E6ABAD1 0800 45 00 00 4E 00 00 40 00 40 11 FB 32 0A 00 00 6E 0A 00 00 02 04 04 00 89 00 3A 38 03 10 FD 01 10 00 01 00 00 00 00 00 00 20 46 45 45 4E 45 42 46 45 46 44 46 46 46 4A 45 42 43 4E 45 49 46 41 43 41 43 41 43 41 43 41 43 41 00 00 20 00 02
0000000000000000 A0369F1850e5 001C7E6ABAD1 0800 45 00 00 4E 00 00 40 00 40 11 FB 32 0A 00 00 6E 0A 00 00 02 04 04 00 89 00 3A 38 03 10 FD 01 10 00 01 00 00 00 00 00 00 20 46 45 45 4E 45 42 46 45 46 44 46 46 46 4A 45 42 43 4E 45 49 46 41 43 41 43 41 43 41 43 41 43 41 00 00 20 00 03
0000000000000000 A0369F1850e5 001C7E6ABAD1 0800 45 00 00 4E 00 00 40 00 40 11 FB 32 0A 00 00 6E 0A 00 00 02 04 04 00 89 00 3A 38 03 10 FD 01 10 00 01 00 00 00 00 00 00 20 46 45 45 4E 45 42 46 45 46 44 46 46 46 4A 45 42 43 4E 45 49 46 41 43 41 43 41 43 41 43 41 43 41 00 00 20 00 04
```
</small>
- mac addres filtering
```
$ awk '$1=="001122334455"{print $0}' \ 
       < /dev/ethpipe/0 > /dev/ethpipe/1
```
- VLAN untagging
```
$ sed -e 's/8100 00 01 //' \
   < /dev/ethpipe/0 > /dev/ethpipe/1
```
- port mirroring
```
$ cat /dev/ethpipe/0 \
   | tee /dev/ethpipe/1 > /dev/ethpipe/2
```

>>>

## EtherPIPE (cont'd)

- Useful for simple packet <br>processing
 - packet capture
 - layer2 forwarding (w/ fdb)
- Not practical to do a complex job

<div class="right" style="width: 45%">
<img src="figs/etherpipe.png" width=100%>
</div>


>>>

## Lightweight guest OS

- Custom middlebox <br> programming
 - ClickOS (NSDI '14)
 - OSv (ATC '14)
 - NetVM (NSDI '14)
 - mos (NSDI '17)
- fast, agile, scalable
- no rich features

- boot time == <br> console ready

<div class="right" style="width: 65%">
<img src="figs/clickos-boottime.png" width=100%>
</div>

<small>
Martins et al., ClickOS and the Art of Network Function Virtualization, NSDI 2014
<br>(figure from iijlab seminar)

>>>

|                    | agility   | feature richness| performance |
| ------------------ |:---------:|: ----------:|:-------:|
| EtherPIPE |  **++**      | **--** |  **+** |
| ClickOS/NetVM |  **++**    | **--** |  **++** |
| OSv | **+**    | **+** | **++** |
| (conventional) VM | **--**    | **++** | **-** |
| /dev/stdpkt (LKL)<!-- .element: class="fragment" data-fragment-index="1" --> |  **++** <!-- .element: class="fragment" data-fragment-index="1" -->       | **++** <!-- .element: class="fragment" data-fragment-index="1" -->| **?** <!-- .element: class="fragment" data-fragment-index="1" -->|  


---

## /dev/stdpkt

<img src="figs/stdpkt-pipe.png" width=80%>

- An inter-connect for userspace network stack
- two modes (as EtherPIPE)
 - ASCII mode
 - raw mode

>>>

## Internals

<img src="figs/g1677.png" width=80%>

- Inter VM communication by pipe
- Can interact with Unix commands

>>>

## Example: NAT

```
% ping.sh | nat.sh | dest.sh
```

<img src="figs/nfv-ericsson-mirage.png" width=60%>

>>>

## Example: NAT

```
==> ping.sh <==
LKL_CONFIG=1st.conf \
 ./bin/lkl-hijack.sh \
 ./ping 8.8.8.8

==> nat.sh <==
LKL_CONFIG=2nd.conf \
 ./bin/lkl-hijack.sh \
 iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -d 8.8.8.0/24 

=> dest.sh <==
LKL_CONFIG=3rd.conf \
 ./bin/lkl-hijack.sh /bin/true
```

Note:

```
- 1st
LKL_HIJACK_NET_IFTYPE0=fifo \
 LKL_HIJACK_NET_IFPARAMS0="/tmp/fifo1|/dev/stdout" \
 LKL_HIJACK_NET_IP0=192.168.0.1 \
 LKL_HIJACK_NET_GATEWAY=192.168.0.254 \
 LKL_HIJACK_NET_NETMASK_LEN0=24 \

- 2nd
LKL_HIJACK_DEBUG=0x100 \
 LKL_HIJACK_SYSCTL="net.ipv4.ip_forward=1" \
 LKL_HIJACK_NET_IFTYPE0=fifo \
 LKL_HIJACK_NET_IFPARAMS0="/dev/stdin|/tmp/fifo1" \
 LKL_HIJACK_NET_IP0=192.168.0.254 LKL_HIJACK_NET_NETMASK_LEN0=24 \
 LKL_HIJACK_NET_IFTYPE1=fifo LKL_HIJACK_NET_IFPARAMS1="/tmp/fifo3|/dev/stdout" \
 LKL_HIJACK_NET_IP1=8.8.8.4 LKL_HIJACK_NET_NETMASK_LEN1=24 \

- 3rd
LKL_HIJACK_DEBUG=0x100 \
 LKL_HIJACK_NET_IFTYPE0=fifo LKL_HIJACK_NET_IFPARAMS0="/dev/stdin|/tmp/fifo3" \
 LKL_HIJACK_NET_IP0=8.8.8.8 LKL_HIJACK_NET_NETMASK_LEN0=24 LKL_HIJACK_NET_GATEWAY=8.8.8.4 \

```

>>>

## Example: Port mirroring

```
# mirror by tee
% app1.sh | tee pipe1 | app2.sh

# read mirrored packets by tcpdump
% ./pktparser < pipe1 | \
   text2pcap - - | \
   tcpdump -r -
```

>>>

## Example: Load balancing

- use `tee` command with pipe


```
% fifo3 > LB.sh | tee fifo1 | tee fifo2
% server1 < fifo1 > fifo3
% server2 < fifo2 > fifo3
```

>>>

## Example: Load balancing

- forward path

```
ping --> tee fifo(atob) ---> fifo(atoc)
               |             |
               |             |
               |             |
               |             |
              srv1          srv2
```

- return path

```
srv1 ---+
        |
        +-- fifo(btoa) --> ping
        |
srv2 ---+
```

>>>

## Host a chain of NF

<img src="figs/netperf-usecase-struct.png" width=80%>



---

## Goodput

<!-- img src="figs/netperf-direct-connected_tmp.png" width=100% -->
<img src="figs/netperf-direct-connected.png" width=70%>

- Point-to-Point netperf goodput
<small>
 - `lkl-hijack.sh netperf -H 1.2.3.4 | lkl-hijack.sh netserver -D`
</small>
- Looks similar to Linux performance

>>>

## Boot time

<img src="figs/boot-latency.png" width=80%>

- boot time = (startup) <-> (1st packet: DAD or ARP)
- LKL/OSv/Linux(KVM): 0.02-0.09/1.1-48.5/7.8-12.9 (sec)

>>>

## Future Work

- Performance improvement
 - Linux (named) pipe to custom IPC
 - preserve the shell syntax ('|')
- Dynamic chain configuration
- *inetd* like launcher


>>>


## APsys paper review

(reject, reject, weak reject)

- Goal is unclear
- Resulted cmdline is not simple at all
 - => refine complex part of configuration
- Discussion isn't enough
 - Isolation (of VM) v.s. Short boot-time
 - superiority and inferiority shall be described 
 - why not Unix socket for IPC ?

<div class="right" style="width: 25%">
<img src="figs/apsys-review.png" width=100%>
</div>

---

## Backups

>>>

## Branch: load balance (cont'd)

- srv1

```
 % LKL_HIJACK_NET_IFTYPE0=fifo LKL_HIJACK_NET_IFPARAMS0="atob|btoa" \
   LKL_HIJACK_NET_IP0=192.168.0.2 LKL_HIJACK_NET_NETMASK_LEN0=24 \
   LKL_HIJACK_DEBUG=0x100 \
   ./bin/lkl-hijack.sh /bin/true
```

- srv2

```
 % LKL_HIJACK_NET_IFTYPE0=fifo LKL_HIJACK_NET_IFPARAMS0="atoc|btoa" \
   LKL_HIJACK_NET_IP0=192.168.0.3 LKL_HIJACK_NET_NETMASK_LEN0=24 \
   LKL_HIJACK_DEBUG=0x100 \
   ./bin/lkl-hijack.sh /bin/true
```

- ping

```
 % LKL_HIJACK_NET_IFTYPE0=fifo LKL_HIJACK_NET_IFPARAMS0="btoa|stdout" \
   LKL_HIJACK_NET_IP0=192.168.0.1 LKL_HIJACK_NET_NETMASK_LEN0=24 \
   ./bin/lkl-hijack.sh ./fping 192.168.0.2 192.168.0.3 \
   | tee atob | > atoc
```


>>>

## What ?

- OS as a tool/function
- Concatinating OSs in a UNIX shell


>>>

## Benefits

- an OS (LKL) serves as a *filter*
 - filtering, routing, nat, conntrack (any of in Linux)
- super quick bootstrap (< 0.4msec)
- Make each function small
 - (*make each program do one thing well*)
 - but feature-rich network stack (thanks to LKL)

Note:
- Virtualization makes it flexible to replace/rewrite/modify

>>>

## Challenges

- Uni-directional (nature of pipe)
 - combination of anonymous/named pipes
- Repurposing pipe = NIC
 - more interanal buffers (pipe, vNIC, socket, app, etc)
 - new pipe w/ private shell ?

>>>

## UNIX Philosophy (Mike Gancarz)

1. **Small is beautiful.**
1. **Make each program do one thing well.**
1. Build a prototype as soon as possible.
1. **Choose portability over efficiency.**
1. Store data in flat text files.
1. Use software leverage to your advantage.
1. Use shell scripts to increase leverage and portability.
1. Avoid captive user interfaces.
1. **Make every program a filter.**

>>>

## NFV (cont'd)

- per user/flow traffic handling
 - NAT, ACL, load balancing
 - Virus Scan, IDS/firewall
 - (typically) implemented as VMs (virtual appliances)

- Benefit after passing the middleboxes
