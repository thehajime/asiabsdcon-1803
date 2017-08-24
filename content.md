## (no title)

<span>
<br>
<br>
<br>
<br>

Hajime Tazaki
</span>

Note:

## Outline

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

- Benefit
 - lower cost
 - decouple of functions

- Each box as a function
 - Virtual Network Function (VNF)

- Further usecases
 - chain of functionality
 - service migration

<div class="right" style="width: 45%">
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

## Example

- Unix pipeline

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

## Internals

<img src="figs/g1677.png" width=100%>

>>>

## Examples (Chain)

```
% ping.sh | nat.sh | dest.sh
```

<img src="figs/nfv-ericsson-mirage.png" width=60%>

>>>

## Examples (Chain)

```
==> ping.sh <==
LKL_HIJACK_NET_IFTYPE0=fifo \
 LKL_HIJACK_NET_IFPARAMS0="/tmp/fifo1|/dev/stdout" \
 LKL_HIJACK_NET_IP0=192.168.0.1 \
 LKL_HIJACK_NET_GATEWAY=192.168.0.254 \
 LKL_HIJACK_NET_NETMASK_LEN0=24 \
 ./bin/lkl-hijack.sh \
 ./ping 8.8.8.8

==> nat.sh <==
LKL_HIJACK_DEBUG=0x100 \
 LKL_HIJACK_SYSCTL="net.ipv4.ip_forward=1" \
 LKL_HIJACK_NET_IFTYPE0=fifo \
 LKL_HIJACK_NET_IFPARAMS0="/dev/stdin|/tmp/fifo1" \
 LKL_HIJACK_NET_IP0=192.168.0.254 LKL_HIJACK_NET_NETMASK_LEN0=24 \
 LKL_HIJACK_NET_IFTYPE1=fifo LKL_HIJACK_NET_IFPARAMS1="/tmp/fifo3|/dev/stdout" \
 LKL_HIJACK_NET_IP1=8.8.8.4 LKL_HIJACK_NET_NETMASK_LEN1=24 \
 ./bin/lkl-hijack.sh \
 iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -d 8.8.8.0/24 

=> dest.sh <==
LKL_HIJACK_DEBUG=0x100 \
 LKL_HIJACK_NET_IFTYPE0=fifo LKL_HIJACK_NET_IFPARAMS0="/dev/stdin|/tmp/fifo3" \
 LKL_HIJACK_NET_IP0=8.8.8.8 LKL_HIJACK_NET_NETMASK_LEN0=24 LKL_HIJACK_NET_GATEWAY=8.8.8.4 \
 ./bin/lkl-hijack.sh /bin/true


```

>>>

## Examples

- Branch
 - use `tee` command with pipe


```
% fifo3 > LB.sh | tee fifo1 | tee fifo2
% server1 < fifo1 > fifo3
% server2 < fifo2 > fifo3
```

>>>

## Branch: load balancing

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

