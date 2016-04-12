## 再利用可能なモノリシックカーネル

<span>
<br>
<br>
<br>
<br>

### Hajime Tazaki 
IIJ Innovation Institute
<br>

2016/04/12, IIJlab セミナー
</span>

---

# 背景

- 仮想化技術の発達
- リソース (計算機・ストレージ・ネットワークなど)の抽象化
- 物理的制約の緩和、複雑性の減少など


**仮想化する事で、できなかった事をできるようにする**

<img src="figs/virt-1.jpg">
<img src="figs/virt-2.jpg">

<small>
- http://ableit.ca/business-support-services/server-virtualization/
- http://exelos.com/solutions/virtualization/

>>>

### 様々な仮想化とその用途 (ネットワーク)

- 手法
 - 疑似 (仮想マシン)
 - 名前空間分離
 - カーネル迂回
<!-- - unikernel -->
- 用途
 - 多重化、隔離環境
 - 機能追加への自由度
 - 性能改善
 - 軽量化、分化・専用用途
 - デバッグ・テスト、詳細・再現調査
 - 大規模ネットワーク実験


Note:
<!-- - エミュレーション (namespace/translation) -->

<!--
>>>

## 現在できる事

- OS の再現性を高めて抽象化
 - VM
 - コンテナ
- 専用ユーザ空間ネットワークスタック
 - 何かに特化する (ハードウェア、プロトコル、etc)
 - その分汎用なものを凌駕する性能 (pps, bps)
->>

>>>

## モチベーション (Why)

- 新しい用途でも、実績のあるソフトウェアが利用したい
 - ネットワークスタック (618K LoC, Linux)
 - ファイルシステム (752K LoC, Linux)
 - アプリケーション (a lot)
- VM は重い (再現性向上 => 機動性低下)
 - 抽象化の場所 (プロセッサ、OS、システムコール、etc)
- 仮想化技術から前進させて、変形させる

<img src="figs/msr-vm-usage.png" width=40%>

<small>
Poll: "When you download and run software, how often do you use a virtual machine (to reduce security risks)?"

- Jon Howell, Galen Hunt, David Molnar, and Donald E. Porter, Living Dangerously: A Survey of Software Download Practices, no. MSR-TR-2010-51, May 2010

Note:
US: us citizens (non-US is mostly Indian, and noisy due to all a answers)
OS: OS researcher in MSR
MS: non-OS employee in MSR

>>>

<img src="figs/msr-vm-usage.png" width=70%>

Poll: "When you download and run software, how often do you use a virtual machine (to reduce security risks)?"

<small>
- Jon Howell, Galen Hunt, David Molnar, and Donald E. Porter, Living Dangerously: A Survey of Software Download Practices, no. MSR-TR-2010-51, May 2010

Note:
US: us citizens (non-US is mostly Indian, and noisy due to all a answers)
OS: OS researcher in MSR
MS: non-OS employee in MSR

>>>

<img src="figs/balance.png" width=100%>

## 性能と利便性の両立 ？

- 性能を求めるには既存の資産(コード)は障壁になる
- 既存の資産を生かすと、性能を犠牲にしなければならない

Note:

ゼロからつくれば、イノベーションぽいが、エコシステムも含めて全部置きか
えをするのは現実的ではない (10~20年スパン)

>>>

<!-- .slide: data-background="figs/cs-thomas.png" -->

> Any problem in computer science can be solved with another level of __indirection__.

>(Wheeler and/or Lampson)



<br /><small>
img src: https://www.flickr.com/photos/thomasclaveirole/305073153
</small>

>>>

### モノリシックカーネルの再利用とは (What ?)

- Anykernel: NetBSD rump kernel に由来


>We define an anykernel to be an organization of kernel code which allows the kernel's **unmodified** drivers to be **run in various configurations** such as application libraries and microkernel style servers, and also as part of a monolithic kernel.  -- Kantee 2012.

- 実績あるモノリシックカーネルのソースコードをそのまま利用 (unmodified)
- 追加具材を糊付けし変形させ、ライブラリ化

Note:

様々仮想化技術はあり、用途もあるが、「まだ別の仮想化技術が便利で必要で
すよ」という話と、「その仮想化技術にまだ課題がありますよ」という話を、
Linux カーネルを題材としてお話。

>>>

<div class="left" style="width: 40%">
<br>
<br>
<img src="figs/anykernel-pre.png" width=100%>
</div>


<div class="right" style="width: 40%">
<img src="figs/anykernel-post.png" width=100%><!-- .element: class="fragment" data-fragment-index="1" -->
</div>

>>>

### モノリシックカーネルの再利用とは (cont'd)

- 機能のアウトソース (環境依存コード)
 - 時計
 - メモリ
 - スケジュール
 - デバイス (NIC/ブロック) I/O

- 様々な実装でこの仕組みを利用
 - SeaStar/OSv, mTCP, LKL/LibOS, rumpkernel

- Anykernel として作るには、カーネル内に*透過的*にサブシステムを導入する必要があり
 - 環境**非依存**コード
 - CPU architecture として
 - 独立したサブシステムとして


>>>

## 

## Who else ?

- Full scratch (ゼロから実装)
 - Mirage [ASPLOS 2013]
 - IncludeOS [CloudCom 2016]
 - SeaStar
 - mTCP [NSDI 2014]
- porting (既存 OS の移植)
 - OSv [USENIX 2014]
 - libuinet
 - SandStorm [SIGCOMM 2014]
- Anykernel
 - NetBSD rump [USENIX 2009]
 - UML
 - DrawBridge (?)

Note:

- Drawbridge も？ コードがないからわからない
- 


- virtualization
 - qemu/vmm
- isolation
 - namespace/jail
- 一部再利用 (Wine, Win subssytem for Linux on Win10)

>>>

## 特定用途カーネルの作り方

<large>

|                    | 性能 | 既存資産  | 最新版追従|
| ------------------ |:---------:|: ----------:|:-------:|
| full scratch |  **++**   |   **--**     | **+** | 
| porting |  **+**       |   **+/-**     | **--** |
| anykernel |  **-**         |   **++**     | **++** |

<!-- | (virtualization) |  **-**         |   **++**     | **++** | -->



Note:

---

## Linux Kernel Library (LKL)


>>>

## Linux Kernel Library



- Linux カーネルのライブラリ
- ハードウェア非依存のアーキテクチャ
- 下位ホストのインターフェースを規定
 - 依存部をアウトソース
 - Windows, Linux, FreeBSDで動作
- デバイスI/Oを簡素化
 - virtio ホスト実装
 - Linux カーネル内のドライバ利用

<div class="right" style="width: 45%">
<img src="figs/lkl-arch.png" width=100%>

<br>
<small>
Purdila et al., LKL: The Linux kernel library, RoEduNet 2010.
</small>
</div>


Note:
 - with plenty of network features (of Linux)

>>>

## 歴史

- rump: 2007 (NetBSD)
- LKL: 2007 (Linux)
- DCE/LibOS: 2008 (Linux/FreeBSD)
- LibOS/LKL revival: 2015
 - LibOS merged to LKL

>>>

## LKL v.s. LibOS

<div class="left" style="width: 50%">
<img src="figs/lkl-arch.png" width=100%>
LKL
</div>

<div class="right" style="width: 50%">
<img src="figs/libos-arch.png" width=75%>
<br>
LibOS
</div>

>>>

## LKL v.s. LibOS (cont'd)

- LoC:
 - arch/lkl (LKL) < arch/lib (LibOS)
 - スタブコードの量に依存
- 共通点
 - カーネルコンテキストの表現 (POSIX thread)
 - ホストインターフェース (時計、メモリ、スケジュール)
 - CPU 非依存の arch として実装
 - 他カーネルコードへの修正なし
- 相違点
 - LibOS: 高位 API (timer, irq, kthread)を pthread で再実装
 - LKL: pthread で IRQ, kthread, timer などを API 再実装はせずに表現


>>>

## 内部構造
<div class="left" style="width: 48%">
<img src="figs/lkl-arch.png" width=100%>
</div>


<br>
1. ホストバックエンド (host_ops)
1. CPU 非依存アーキテクチャ (arch/lkl)
1. アプリケーションインターフェース


Note:
 - bridge (real) userspace with library kernel

(FIXME: add diagram)


>>>

## 1. ホストバックエンド

<div class="left" style="width: 35%">
<img src="figs/lkl-arch-host.png" width=100%>
</div>

- 環境依存部のインターフェース
 - 異プラットフォーム間で共通化
 - (rump ハイパーコール like)
- Virtio によるデバイスインターフェース
 - ブロックデバイス <=> ディスクイメージ
 - ネットワーク <=> TAP, DPDK, VDE

>>>
## 2. CPU 非依存アーキテクチャ

<div class="left" style="width: 35%">
<img src="figs/lkl-arch-kernel.png" width=100%>
</div>



アーキテクチャ (arch/lkl)
<br><br>
- CPU アーキテクチャと同等に扱える
 - 他コードへの影響をなくす
- 2400 LoC
 - スレッド構造体 (struct thread_info) 定義
 - irq, timer, syscall handler を実装
 - 下位層リソースへのアクセス(環境依存)は、
   <br>host_opsにて表現

>>>

## 3. アプリケーションインターフェース

<div class="left" style="width: 40%">
<img src="figs/lkl-arch-api.png" width=100%>
</div>

<br><br><br>
- Case 1: use exposed API (LKL syscall)
- Case 2: use host libc (LD_PRELOAD)
- Case 3: extend (alternative) libc

>>>
## Case 1: use exposed API (LKL syscall)

- カーネル内のシステムコール入り口を直接 call
 - *lkl_sys_open()*, *lkl_sys_socket()*
- 通常のシステムコールとほぼ同じ
 - 復帰値、エラー番号の通知が違う
- アプリケーションはこの API と、ホストの持つ<br> API を両方利用可能
 - LKL で ext4 ファイルを lkl_sys_read() => <br>ホスト (Windows) で write()

<div class="right" style="width: 30%">
<img src="figs/lkl-syscall-1.png" width=100%>
</div>

>>>
## Case 2: ホスト標準ライブラリのハイジャック

- ホストの標準ライブラリ (libc) を<br>
  LKL システムコールに実行時に置きかえ
 - LD_PRELOAD
 - socket() => lkl_sys_socket()
- ホストのプログラムバイナリをそのまま利用可能
- 置きかえ可能なシンボルに制限あり
- 非 Linux ホストでのシステムコール変換は必要

<div class="right" style="width: 30%">
<img src="figs/lkl-syscall-hijack.png" width=100%>
</div>

>>>
## Case 3: extend (alternative) libc

- LKL システムコール**のみ**を呼びだす<br>標準ライブラリ
- カーネルに対応して、(仮想的な) <br>CPU アーキテクチャ拡張
- プログラムは、通常の標準ライブラリとして<br>リンクする
 - システムコールを経由して直接ホストの<br>資源アクセスは不可
- musl libc の拡張として実装

<div class="right" style="width: 30%">
<img src="figs/lkl-syscall-musl.png" width=100%>
</div>

>>>

## 使い方 (アプリケーション)

- Use Case 1: ネットワークシミュレータとの結合
- Use Case 2: 簡易カーネルバイパス
- Use Case 3: カーネルコードを再利用したアプリケーション開発
- Use Case 4: Unikernel

>>>

### Use Case 1: ネットワークシミュレータとの結合

<div class="left" style="width: 40%">
<video data-autoplay src="figs/ns-3-dce-mptcp-linux3.5.7-8subf.m4v"></video>

Linux Multipath-TCP の実験可視化
</div>



- ネットワーク状態の調査
- ns-3 ネットワークシミュレータ
- 多種のモデル
 - NIC
 - ノード移動
 - トラフィック
 - タイミング
- 単一プロセスにて複数ノード動作
 - dlmopen(3) にて実現 (シンボル退避)
 - システムコール再実装 (ノード振り分け)
- **再現性 100 %** (実験・バグの再現)

>>>
### Use Case 1: ネットワークシミュレータとの結合

- カーネルのネットワークスタックテストツールとしても利用
 - Regression テスト (を複雑なネットワークで)
 - 再現率 100 % (仮想クロックを利用)
 - コードカバレッジ測定 (シミュレータの疑似乱数利用)
 - Valgrind でデバッグ

<img src="figs/jenkins.png" width="30%">
<img src="figs/gcov.png" width="30%">
<p>
<img src="figs/rocketfuel.png" width="20%">
<img src="figs/valgrind.png" width="30%">

Note:
注： LKL へは未移植

>>>

## Use Case 2: 簡易カーネルバイパス

- LD_PRELOAD により、実行時にシステムコール迂回
- 置きかえ可能なシステムコール (シンボル) に制限あり
- LKL と外界 (ホストOS)の両方のシステムコールを利用可能

<br> <br>

ホストカーネルに影響なく新機能を導入可能

```
LD_PRELOAD=liblkl-super-tcp++.so firefox
```

>>>

## Use Case 3: カーネルコードを再利用したアプリケーション開発
- カーネルのコードを**移植なし**にユーザ空間でライブラリとして利用
- LKL と外界 (ホストOS)の両方のシステムコールを利用可能
<p>
- カーネル実装を利用したファイルシステムアクセス
- 例： ext4 フォーマットのディスクイメージにアクセス
 1. ディスクイメージを開く (*CreateFile()*)
 1.  lkl_sys_mount()
 1.  lkl_sys_read()
 1. ホストの別ファイルへ書きこみ (*WriteFile()*)


>>>

## Use Case 4: Unikernel

- 単一アプリケーションにリンクさせる LKL
 - python + LKL, nginx + LKL
- LKL システムコールのみ利用可能
 - musl libc 移植
- rump ハイパーコール利用 (frankenlibc)
 - OS のないホスト上でも動作 (rumprun)
 - (on Xen Mini-OS)
- Work in progress

<div class="right" style="width: 40%">
<img src="figs/CloudOSDiagram.png">

<small>
- http://www.linux.com/news/enterprise/cloud-computing/751156-are-cloud-operating-systems-the-next-big-thing-
</small>
</div>

<!--
<div class="right" style="width: 40%">
<img src="figs/frankenlibc-stack.png" width=70%>
</div>
-->

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

<img src="figs/frankenlibc-nginx.png" width="30%">

nginx (linked) LKL running on Linux

>>>

## ベンチマーク

- 10Gbps Ethernet リンク (LibOS, 素の Linux)
- 8コア CPU
- ネットワークバックエンド (DPDK, netmap, rawソケットなど)
- RTT と Throughput 測定 (UDP 1024 byte パケット)

<img src="figs/nuse-benchmark-topo-host.png" width=40%>
<img src="figs/nuse-benchmark-host.png" width=70%>

>>>

## ベンチマーク (cont'd)

<img src="figs/nuse-benchmark-host.png" width=70%>

- 単純にカーネル迂回しただけでは速くはならない
- 原因
 - キャッシュミス
 - 多量の(ホスト)システムコールの数

>>>

## ボトルネック分析

- ネットワークスタック部
 - CPU コアローカルではないプロセス内通信
 - 1パケット毎の処理多数 (バルク処理)
 - システムコールのオーバヘッド (ホストバックエンド実装)
- LKL 本体
 - 迂回 (Indirection)の影響

>>>

<large>

|                    | 性能 | 既存資産  | 最新版追従|
| ------------------ |:---------:|: ----------:|:-------:|
| full scratch |  **++**   |   **--**     | **+** | 
| porting |  **+**       |   **+/-**     | **--** |
| anykernel |  **<span style="color:red"><large>-</span>**         |   **++**     | **++** |

>>>

<large>

|                    | 性能 | 既存資産  | 最新版追従|
| ------------------ |:---------:|: ----------:|:-------:|
| full scratch |  **++**   |   **--**     | **+** | 
| porting |  **+**       |   **+/-**     | **--** |
| anykernel |  **<span style="color:red"><large>+</span>**         |   **++**     | **++** |


---

## まとめ

- カーネルのコードは再利用するものです!
 - 20 年近くの資産を捨てるのは無駄
 - (面倒な)移植なしに
 - ライブラリとして (ユーザ空間プログラムに限らず)
 - 必要な部分のみ利用可能
<br>
- オープンソースプロジェクトとして開発
 - https://github.com/lkl/linux

Note:

**肩ひじ張らずに**使える	


(モノリシック)カーネルのソースコードを変形
- ライブラリ化
- ユーザ空間完結 (カーネル迂回)
- 機能の移植・修正なしに他プログラムにて再利用
- 不要機能(コード)を削ぎおとしたカーネル (Unikernel)

- 課題
 - 性能と利便性・機能の両立
 - 迂回されたコードパスによるオーバヘッド


>>>

## 参考資料
- Linux Kernel Library
 - Purdila et al., LKL: The Linux kernel library, RoEduNet 2010.
 - https://github.com/lkl/linux
- Rumpkernel (dissertation)
 - Kantee, Flexible Operating System Internals: The Design and Implementation of the Anykernel and Rump Kernels, Ph.D Thesis, 2012
- Linux LibOS 一般
 - Tazaki et al. Direct Code Execution: Revisiting Library OS Architecture for Reproducible Network Experiments, CoNEXT 2013
 - http://libos-nuse.github.io/ (LibOS in general)
 - https://lwn.net/Articles/637658/


