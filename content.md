## LKL updates Oct.2016-Feb.2017

<span>
<br>
<br>
<br>
<br>

### Hajime Tazaki
IIJ
<br>

2017/3
</span>


Note:

## Outline

- What is LKL ?
- How are LKL or userspace networking thought ?
- How far is LKL from the (original) Linux ?
- Steps toward the Linux with tcp_bbr.c/sch_fq.c


---


## Motivation

<normal>
- MegaPipe [OSDI '12]
 - *outperforms baseline Linux .. **582%** (for short connections).*
 - New API for applications (no existing applications benefit)<!-- .element: class="fragment" data-fragment-index="1" -->
- mTCP [NSDI '14]
 - *improve... **by a factor of 25** compared to the latest Linux TCP*
 - implement with very limited TCP extensions<!-- .element: class="fragment" data-fragment-index="1" -->
- SandStorm [SIGCOMM '14]
 - *our approach ..., **demonstrating 2-10x** improvements*
 - specialized (no existing applications benefit)<!-- .element: class="fragment" data-fragment-index="1" -->
- Arrakis [OSDI '14]
 - *improvements of **2-5x in latency and 9x in throughput** .. to Linux*
 - utilize simplified TCP/IP stack (lwip) (loose feature-rich extensions)<!-- .element: class="fragment" data-fragment-index="1" -->
- IX [OSDI '14]
 - *improves throughput ... **by 3.6x** and reduces latency **by 2x***
 - utilize simplified TCP/IP stack (lwip) (loose feature-rich extensions)<!-- .element: class="fragment" data-fragment-index="1" -->

Note:
- netmap [ATC '12]
 - *a single core ... can send or receive 14.88 Mpps (**the peak packet rate** on 10 Gbit/s links).*

>>>

## Motivation (cont'd)

- Reasonable speed with generalized userspace network stack
 - **feature-richness**
 - **x1 speed of the original**


>>>

## LKL outlooks

- h/w independent

- on Linux/Windows/FreeBSD uspace, <br> unikernel, on UEFI, 

<div class="right" style="width: 40%">
<img src="figs/fig2-lkl-arch.png" width=100%>
</div>

>>>

## How others think about userspace ?

> DPDK is not Linux (@ netdev 1.2) <!-- .element: class="fragment" data-fragment-index="1" -->

- The model of DPDK isn't compatible with Linux <!-- .element: class="fragment" data-fragment-index="2" -->
 - break security model (protection never works)
- XDP is Linux  <!-- .element: class="fragment" data-fragment-index="2" -->

>>>

## userspace network stack ?

- Concerns about timer accuracy
 - how LKL behaves with BBR ?
 - Having network stack in userspace is a *ridiculous* idea
- Concerns about multiple network stack
 - *userspace networking lives in different universe*

>>>

## TCP BBR

- **B**ottleneck **B**andwidth and **R**ound-trip propagation time
- Control Tx rate
 - congestion not based on the packet loss
 - estimate MinRTT **and** MaxBW (on each ACK)

<img src="figs/bbr-acm-queue-optimal-points.png" width=25%>
<img src="figs/bbr-acm-queue-vs-cubic.png" width=50%>

http://queue.acm.org/detail.cfm?id=3022184

>>>

## TCP BBR (cont'd)

BBR requires
- packet pacing
- precise RTT measurement

```
function onAck(packet) 
  rtt = now - packet.sendtime 
  update_min_filter(RTpropFilter, rtt) 
  delivered += packet.size 
  delivered_time = now 
  deliveryRate = (delivered - packet.delivered) /
          (delivered_time - packet.delivered_time) 
  if (deliveryRate > BtlBwFilter.currentMax ||
           ! packet.app_limited) 
     update_max_filter(BtlBwFilter, deliveryRate) 
  if (app_limited_until > 0) 
     app_limited_until = app_limited_until - packet.size
```

http://queue.acm.org/detail.cfm?id=3022184

>>>

## TCP BBR (cont'd)

<div class="right" style="width: 45%">
<img src="figs/bbr-acm-queue-b4-wan-thput.png" width=100%>
</div>

- On Google's B4 WAN <br> (across North America, EU, Asia)
- Migrated from cubic to bbr at 2016
- x2 - x25 improvements

>>>

## 1st Benchmark (Oct. 2016)

- netperf (TCP_STREAM, -K bbr/cubic)
- 2-node 10Gbps b2b link
- No loss, no bottleneck, close link

```
                 netperf(client)         netserver
                 +------+               +--------+
                 |      |               |        |
                 |sender+--------------+|receiver|
                 |      |==============>|        |
                 |      |               |        |
                 +------+               +--------+
                 Linux-4.9-rc4          Linux-4.6             
                 (host,LKL)
                 bbr/cubic             cubic,fq_codel        
                                       (default) 

```

|cc| tput (Linux)  |tput (LKL)|
|-|-|-|
|bbr | 9414.40 Mbps |    **456.43** Mbps| 
|cubic| 9411.46 Mbps  | 9385.28 Mbps |


>>>

## What ??

- only BBR + LKL shows bad


- ack timestamp used by RTT measurement needed a precise time event (clock)
- providing high resolution timestamp improve the BBR performance

>>>

## Change HZ (tick interval)

|cc| tput (Linux)  |tput (LKL)| tput (LKL) |
|-|-|-|-|
|bbr | 9414.40 Mbps | **456.43 Mbps** | **6965.05 Mbps** |
|cubic| 9411.46 Mbps  | 9385.28 Mbps | 9393.35 Mbps|

>>>

## Timestamp on (ack) receipt

From

```
unsigned long long __weak sched_clock(void)
{
	return (unsigned long long)(jiffies - INITIAL_JIFFIES)
					* (NSEC_PER_SEC / HZ);
}
```

To

```
unsigned long long sched_clock(void)
{
	return lkl_ops->time(); // i.e., clock_gettime()
}
```


|cc| tput (Linux)  |tput (LKL)| tput (LKL sched_clock) |
|-|-|-|-|
|bbr | 9414.40 Mbps |    456.43 Mbps| **9409.98** Mbps |

>>>

#### What happens if `sched_clock()` isn't available ?

- low throughput due to longer RTT measurement
- A patch to torelate lower in jiffies resolution 


```
diff --git a/net/ipv4/tcp_input.c b/net/ipv4/tcp_input.c
index 56fe736..b0f1426 100644
--- a/net/ipv4/tcp_input.c
+++ b/net/ipv4/tcp_input.c
@@ -3196,6 +3196,8 @@ static int tcp_clean_rtx_queue(struct sock *sk, int prior_fackets,
                ca_rtt_us = skb_mstamp_us_delta(now, &sack->last_sackt);
        }
        sack->rate->rtt_us = ca_rtt_us; /* RTT of last (S)ACKed packet, or -1 */
+       if (sack->rate->rtt_us == 0)
+               sack->rate->rtt_us = jiffies_to_usecs(1);
        rtt_update = tcp_ack_update_rtt(sk, flag, seq_rtt_us, sack_rtt_us,
                                        ca_rtt_us);
 
diff --git a/net/ipv4/tcp_rate.c b/net/ipv4/tcp_rate.c
index 9be1581..981c48e 100644
--- a/net/ipv4/tcp_rate.c
+++ b/net/ipv4/tcp_rate.c
@@ -148,7 +148,9 @@ void tcp_rate_gen(struct sock *sk, u32 delivered, u32 lost,
         * measuring the delivery rate during loss recovery is crucial
         * for connections suffer heavy or prolonged losses.
         */
-       if (unlikely(rs->interval_us < tcp_min_rtt(tp))) {
+       if (rs->interval_us == 0) {
+               rs->interval_us = jiffies_to_usecs(1);
+       } else if (unlikely(rs->interval_us < tcp_min_rtt(tp))) {
                if (!rs->is_retrans)
                        pr_debug("tcp rate: %ld %d %u %u %u\n",
                                 rs->interval_us, rs->delivered,
```

|cc| tput (Linux)  |tput (LKL)| tput (LKL patched) |
|-|-|-|-|
|bbr | 9414.40 Mbps |    456.43 Mbps| 9413.51 Mbps |


- https://groups.google.com/forum/#!topic/bbr-dev/sNwlUuIzzOk


>>>

## 2nd Benchmark

- delayed, lossy network but 10Gbps
 - netem (middlebox)


```
        netperf(client)                    netserver
        +------+        +---------+       +--------+
        |      |        |         |       |        |
        |sender+--------+middlebox+------+|receiver|
        |      |======= |======== |======>|        |
        |      |        |         |       |        |
        +------+        +---------+       +--------+
        cc: BBR         1% pkt loss
     **fq-enabled**     100ms delay
     **tcp_wmem=100M**

```

|cc| tput (Linux)  |tput (LKL)|
|-|-|-|
|bbr |  Mbps |    145.32 Mbps| 
|cubic|  Mbps  | 118.71 Mbps |

>>>

## Memory configurations w/ TCP

- socket, TCP has configurable parameter
 - sysctl -w net.ipv4.tcp_wmem="4096 16384 100000000"
 - delay and loss requires increased buffer

>>>

## Memory w/ TCP (cont'd)

- default memory size: 64MiB
 - the size affects the sndbuf size

```
static bool tcp_should_expand_sndbuf(const struct sock *sk)
{
(snip)
	/* If we are under global TCP memory pressure, do not expand.  */
	if (tcp_under_memory_pressure(sk))
		return false;
(snip)
}
```

>>>

## Timer relates

- CONFIG_HIGH_RES_TIMERS enabled
 - fq scheduler uses
 - properly transmit packets with probed BW
- fq configuration
 - instead of `tc qdisc add fq`


>>>

## How slow high-resolution timer ?

<img src="figs/hrtimer-delay-cdf.png" width=70%>

**errata:** usec => nsec
>>>

### How timer works ?

1. schedule an event
1. add (hr)timer list queue (hrtimer_start())
1. (check expired timers in timer irq) (hrtimer_interrupt())
1. invoke callbacks (__run_hrtimer())

>>>

### how timer interrupt works ?

**native thread ver.**
1. timer_settime ()
 - instantiate a pthread
1. notify via signal
1. trigger a timer interrupt (update jiffies, etc)

**green thread ver.**
1. instantiate a green thread
 - malloc/mmap, add to sched queue
1. schedule an event
 - clock_nanosleep() until next event)
 - or do something (goto above)
1. trigger a timer interrupt (update jiffies, etc)

>>>

## Scheduler

- LKL's scheduler
 - outsourced (w/ thread impl.)
- 2 implementations
 - native thread
 - green thread
- minimum delay of timer interrupt
 - 20 usec (native thread)
 - 60 usec (green thread)

>>>

## Scheduler improvement

1. avoid system call (clock_nanosleep) when block
 - busy poll (watch clock instead) if sleep is < 10usec
 - 60 usec => 20 usec

```
sleep(u64 nsec) {
	/* fast path */
	while (1) {
	  	if (nsec < 10*1000) {
                   clock_gettime(CLOCK_MONOTONIC, &now);
		   if (now - start > nsec)
		      return;
		}
	}

	/* slow path */
	return syscall(SYS_clock_nanosleep)
}
```

1. reuse green thread stack (avoid mmap per a timer irq)
 - 20 usec => 3 usec

>>>

## Timer delay improved ?

<img src="figs/hrtimer-delay-cdf.png" width=45%>
<img src="figs/hrtimer-delay-cdf-170303.png" width=45%>

>>>

## TSO/Checksum offload

- virtio based
- guest-side: use Linux driver

>>>

## 

<div class="left" style="width: 49%">
<img src="figs/tcp-stream-tx-170306.png" width=100%>
TCP_STREAM (cubic, no delay)
</div>

<div class="right" style="width: 49%">
<img src="figs/tcp-stream-10000M-fq-170306.png" width=100%>
TCP_STREAM(bbr, delay link)
</div>

>>>

## IPv6 ready

<img src="figs/tcp6-stream-1G-none-170306.png" width=80%>

>>>

## Summary

- We dont have to an OS from scratch


>>>

# Backup

>>>

## Steps

1. add sched_clock() 
 - allow measurement of sub-millisecond RTT
 - fill 10G link with netperf

1. patch tcp_rate.c/tcp_bbr.c to torelate lower jiffies resolusion 
 -  https://groups.google.com/forum/#!topic/bbr-dev/sNwlUuIzzOk
1. memory(ies)
 - tcp_mem (mem=xxx)
 - tcp_wmem (sysctl)
1. enable hrtimer (CONFIG_HIGH_RES_TIMERS)
1. qdisc
1. timer lib rewrite
1. avoid clock_nanosleep (clock_gettime watch instead) for short block (<60 usec, lowest hrtimer 20 usec)
1. avoid mmap (reuse fiber thread stack) (lowest: 3usec)
