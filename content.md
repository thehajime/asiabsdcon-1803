## Performance related flustrations

<span>
<br>
<br>
<br>
<br>

#### Hajime Tazaki 
IIJ Innovation Institute
<br>

2016/08

</span>

---

### (This talk is inspired by a blog "Programming Myth")

<img src="figs/harmful-goto-blog-en.png" width=45%>
<img src="figs/harmful-goto-jp.png" width=45%>

http://videlalvaro.github.io/2015/02/programming-myths.html
http://postd.cc/programming-myths/

>>>


## What I'm doing

- Operating system *re-construction*
 - for a selfishness (personality,specialization)
 - but respect the past dozen effort <br>
   (reusing Linux kernel)

- *Precludes any ossification in software (i.e., Internet)*

- e.g.
 - Userspace network stack
 - Lightweight guest operating system
 - Reproducible network experiment (simulation)


<div class="right" style="width: 15%">
<img src="figs/nuse-overview.png" width=100%>
<img src="figs/CloudOSDiagram.png" width=100%>
<video data-autoplay src="figs/ns-3-dce-mptcp-linux3.5.7-8subf.m4v" width=100%></video>
</div>

Note:

To apply today's agenda (keywords) to my research topic, I rethink
- what i'm doing
- how others were doing
- how we should go

>>>

## High Performance == Speed ?

>>>


<img src="figs/speed-matter-6wind-2.png" width=70%>


<small>
http://www.slideshare.net/6WIND/6wind-speed-matters-the-challenge-2014-contest-winners
</small>

>>>


## Speed as a strong metric

<img src="figs/ix-osdi14-perf.png" width=70%>

- e.g. outperforms 180x 
- e.g. reduce the overhead by 30%

<span>
Belay et al., IX: A Protected Dataplane Operaing System for High Throughput and Low Latency, USENIX OSDI '14
</span>

>>>

## A number describes a system

- an important way to evaluate (good or bad)
- a number never tell lies

> Generic advice on abstract writing for systems papers:
>
> (snip)
> 1. Define the problem you're solving
> 2. Give the key idea for how you solved it
> 3. Describe how you demonstrate the success of your solution
> 4. Give key results, preferably **numerically**
> 5. Describe how this impacts the world/industry/whatever (big picture)



<small>
*Abstracts for Systems Papers* (blog: rdv live from Tokyo)
<br>
 http://rdvlivefromtokyo.blogspot.jp/2011/04/abstracts-for-systems-papers.html
</small>

>>>

## But is number only the way ?
- numbers age quite fast
- improving numbers **often** sacrifice features/functions

<img src="figs/balance.png" width=60%>

Note:
- Today's 8Gbps is not fast result on 10 years later

>>>

<img src="figs/dict-performance.png" width=90%>

>>>

## Any accomplishment !

>>>

## Stuffs about extensive performance improvements

>>>

## Example: kernel bypass technologies
- bypassing for *selfishness* (personality)
- Intel DPDK, netmap, PacketShader, ...
- basic IPv4 stack (ARP, IPv4 reassembles, ..)
- very limited usage for the production use
 - packet drop (DDoS prevention)
 - traffic generator
- no general purpose use<!-- .element: class="fragment" data-fragment-index="1" -->
 - (*toy*)<!-- .element: class="fragment" data-fragment-index="1" -->

<div class="right" style="width: 50%">
<img src="figs/bypass-image.jpg" width=100%>
<small>
https://blog.cloudflare.com/kernel-bypass/ <br>
https://www.flickr.com/photos/londonmatt/11421393074/
</small>
</div>


>>>

## Example: Clean-slate approach
- design/implement from scratch
- often good to start to concentrate on *new ideas*
- Example
 - mTCP [NSDI '14], excellent TCP throughput (**25x Linux**)
 - no offload, recent extensions (TFO, Crypt, CC algorithm, ...)
- toy<!-- .element: class="fragment" data-fragment-index="1" -->

<img src="figs/clean-slate.jpg" width=40%>
<img src="figs/mtcp-results.jpg" width=35%>
<div class="left" style="width: 50%">
<small>
http://www.braveandhappy.com/2015/10/a-clean-slate.html
</small>
</div>

Note:

if you started something based on the current situation, your idea will be restricted on the current limitations

so starting from scratch, as known as clean-slate approach, will give you a freedom to imagine a great idea.

>>>

## How Others Say

- MegaPipe [OSDI '12]
 - *outperforms baseline Linux .. **582%** (for short connections).*
 - New API for applications (no existing applications benefit)<!-- .element: class="fragment" data-fragment-index="1" -->
- mTCP [NSDI '14]
 - *improves the performance ... **by a factor of 25** compared to the latest Linux TCP stack*
 - implement with very limited TCP extensions<!-- .element: class="fragment" data-fragment-index="1" -->
- SandStorm [SIGCOMM '14]
 - *our approach with the FreeBSD and Linux stacks ..., **demonstrating 2-10x** improvements*
 - specialized (no existing applications benefit)<!-- .element: class="fragment" data-fragment-index="1" -->
- Arrakis [OSDI '14]
 - *improvements of **2-5x in latency and 9x in throughput** .. to a well-tuned Linux implementation.*
 - utilize simplified TCP/IP stack (lwip) (loose feature-rich extensions)<!-- .element: class="fragment" data-fragment-index="1" -->
- IX [OSDI '14]
 - *improves the throughput ... **by up to 3.6x** and reduces tail latency **by more than 2x***
 - utilize simplified TCP/IP stack (lwip) (loose feature-rich extensions)<!-- .element: class="fragment" data-fragment-index="1" -->

Note:
- netmap [ATC '12]
 - *a single core ... can send or receive 14.88 Mpps (**the peak packet rate** on 10 Gbit/s links).*

>>>

Sigh

>>>

## A good system research (paper) should be
 (by Tazaki 2016.)


- Address real problem(s), **without (functionality) degradation**
- **With exhaustive evaluations**
 - temptation (numbers are attractive)
 - be honest (as usual)
 - respect prior work (as usual)

<br>
### No Silver Bullet (Frederick Brooks Jr., 1986)<!-- .element: class="fragment" data-fragment-index="1" -->

Note:

revealing the strong point is not an evaluation



>>>

## Summary

- Let the performance be a general-class metric
- *discover new thing through past studies*
 - respect prior work
 - and think different(ly)

>>>

### References

- Papers
 - [mTCP'14] Jeong et al., mTCP: a Highly Scalable User-level TCP Stack for Multicore Systems, NSDI'14
 - [IX'14] Belay et al., IX: A Protected Dataplane Operating System for High Throughput and Low Latency, OSDI'14
 - [Arrakis'14] Peter et al., Arrakis: The Operating System is the Control Plane, OSDI'14
 - [MegaPipe'12] Han et al., MegaPipe: A New Programming Interface for Scalable Network I/O, OSDI'14
 - [SandStorm'14] Marinos et al., Network stack specialization for performance, SIGCOMM '14
 - [PacketShader'10] Han et al., PacketShader: a GPU-accelerated Software Router, ACM SIGCOMM '10

>>>

### References (cont'd)

- Blog
 - http://videlalvaro.github.io/2015/02/programming-myths.html
 - http://postd.cc/programming-myths/
 - http://rdvlivefromtokyo.blogspot.jp/2011/04/abstracts-for-systems-papers.html (Abstracts for Systems Papers)
- Quotes
 - Frederick P. Brooks, Jr., "No Silver Bullet - Essence and Accidents of Software Engineering", IFIP 10th World Computing

Note:
 - Premature optimization
 - premature optimization is the root of all evil (knuth)
 - Dijkstra's Goto

>>>

## Random thoughts

- Performance (Optimization) vs Functionality/Flexibility
 - trade off
 - Performance gain => Functional degradation
 - Functional gain => Performance degradation

- Generalization Tax
- Specialization Tax
