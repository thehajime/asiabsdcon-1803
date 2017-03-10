## ライブラリOSをとりまく現況

<span>
<br>
<br>
<br>
<br>

### 田崎 創
技術研究所
<br>

2017/3, IIR最新号のポイントと活用方法
</span>

---

## アウトライン

- 今なぜ新しい OS が必要なのか
- 新しい用途は新しいOSを生む？
- Linux Kernel Library (LKL) の概要
 - 通常の OS との比較
 - 簡単な性能検証

Note:
 - クラウド (軽量さが重要)
 - アプリ専用OS (機能追加のしやすさ)

>>>

## オペレーティングシステム

- 幅広く利用されるソフトウェア
 - PC
 - 携帯電話・タブレット
- 目的
 - 様々な計算資源を管理する
- 絶えず進化を続ける
 - 機能追加 (OS アップデート)
 - 不具合修正 (セキュリティアップデート)


<img src="figs/android-update-entry.jpg" width=20%>
<img src="figs/windows-update_logo.png" width=10%>
<small>
- http://vba-geek.jp/blog-entry-309.html
- http://appllio.com/20131229-4708-android-os-update-anatomy

>>>

## オペレーティングシステム (cont'd)

- 新たな用途
 - クラウドサービス (XaaS)
 - アプリケーション専用 OS
 - 組込み機器
- 手法
 - 移植
 - サブセットのみ実装
 - 0から(再)実装

<div class="right" style="width: 30%">
<img src="figs/FlowChartCloud.png" width=100%>
<img src="figs/embeded-dev.jpg" width=100%>
</div>

>>>

## クラウド環境でのゲストOS

- 特定サービスを提供するための環境
- 単一機能でしか利用しない<br>場合でもまるごと(一般化)

- 結果
 - ゲスト OS の厳密な資源割り当て
 - 多重度の上限が決まる

- スリムな OS が登場
 - OSv (Cloudius Systems)
 - CoreOS (+Docker)

<div class="right" style="width: 45%">
<br>
<br>
<br>
<br>
<img src="figs/osv-stack.png" width=100%>

<small>
http://osv.io/
</div>

>>>

## アプリケーション専用 OS

- OS の一部の機能の置き換え
 - FUSE (ファイルシステムの実装をアプリケーションで追加)
 - QUIC (Google Chrome, 独自のトランスポートプロトコル)
 - Intel DPDK (ホストカーネルを迂回)


<img src="figs/hispeed.jpg" width=30%>
<img src="figs/bypass-image.jpg" width=25%>
<img src="figs/quic-quick-udp-internet-connections.png" width=40%>

<small>
- https://blog.cloudflare.com/kernel-bypass/
- https://bitmovin.com/advanced-transport-options-dash-quic-http2/

>>>

## ライブラリOSの再興

- 1990年代に登場
- マイクロカーネルの流れを組む小さなカーネル
- ハードウェアへのアクセス(ドライバ)、ネットワークなどはライブラリとして実装

- 2010 年頃より、再び注目される
 - MirageOS (Unikernel)
 - DrawBridge (Windows)
 - Rump kernel (NetBSD/Anykernel)


<img src="figs/iir_vol34_trend01.png" width=70%>


>>>

## 質問：新たな用途には新しい OS は必要か ？

<br>
<br>
<br>

### **いいえ!** <!-- .element: class="fragment" data-fragment-index="1" -->

>>>

# なぜならば

- OS は長年成熟し続けてきたソフトウェア <!-- .element: class="fragment" data-fragment-index="1" -->
 1. 再実装はその資産を捨てる (車輪の再発明)
 1. 移植は今後の成熟を追従できない

<img src="figs/square-tire.jpg" width=50%> <!-- .element: class="fragment" data-fragment-index="1" -->
<small>
- http://calledintowork.com/articles/article.asp?articleID=31 <!-- .element: class="fragment" data-fragment-index="1" -->

>>>

## Linux Kernel Library

- オープンソースの実装
- Linux ベースのライブラリOS
 - 商用利用も多い
 - 開発者も多い
 - **最もモダンな OS !! (私見)**

>>>

## LKL の特徴

<br>

- Linuxカーネルのソースは無修正
- 追加部材(抽象層)のみで別用途実現
 - on Linux (カーネル迂回)
 - on Windows (仮想マシンなし)
 - on qemu/kvm/xen (アプリ専用OS)
 - on UEFI/Android (BIOSで<br>ファイル読み書き)

<div class="right" style="width: 45%">
<img src="figs/iir_vol34_trend02.png" width=100%>
</div>

>>>

## 性能との格闘

- *抽象化はペナルティを生む*
<p>
- ペナルティはある程度は許容
- 可能な速度向上は実現

<img src="figs/balance.png" width=55%>

>>>

## パケット送信性能

- netperf (TCP/UDP)
- 送信側: LKL + netperf
- 受信： (通常) Linux


<img src="figs/iir_vol34_trend03.png" width=45%>
<img src="figs/iir_vol34_trend04.png" width=45%>

TCP 送信性能 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;   UDP 送信性能


>>>

## まとめ

- 用途毎の OS の利用例が増えてきている
- 共通化できるものは(OSでも)ライブラリ化
- 既存資産を最大限利用した新規 OS の開発

---

# Backup

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