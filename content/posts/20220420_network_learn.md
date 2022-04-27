---
title: "网络排查案例"
date: 2022-04-20T22:03:56+08:00
draft: false
toc: false
tags:
- network
- 笔记
---

## 预习
* `traceroute` 查看路由跳转

## Nginx 日志报 connection reset by peer
* Wireshark 抓包分析 
  * 过滤IP：`ip.addr eq my_ip`, `ip.src eq my_ip`或者`ip.dst eq my_ip`
  * 找到RST报文：`tcp.flags.reset eq 1`
  * 过滤序列号：`tcp.ack eq my_num`
  * 过滤时间：`frame.time >= "dec 01,2015 15:49:48"`
  * 过滤表达式前添加 `!` 或者 `not` 表示取反
  * 多个条件可以使用 `and` 或者 `or` 形成复合过滤器

## 定位防火墙问题
* 客户端和服务端在同一时间开始抓包
* 使用IP过滤
* 查看Wireshark的Expert Information，关注可疑的（如Warning）报文并追踪TCP流
* 对比两侧文件，如何找到另一端对应的TCP流：TCP序列号（裸序列号）
  * 设置去掉 `Wireshark` -> `Preferences` -> `Protocols` -> `Relative sequence numbers`
* Wireshark 支持自定义列，在主窗口中显示。进入详情之后，右键需要在主窗口中显示的数据，选择 `Apply as column`
* 同样的服务端，在三次握手中（SYN+ACK 报文）的 TTL 是 59，在导致连接中断的 RST 包里却变成了 64！显然，这个 RST 包并不是跟我们握手的那个服务端发出的（防火墙）
  * ![wireshark-ttl](/images/network_learn_wireshark_ttl.png)

## 长肥管道
* 带宽很大、RTT 很长的网络，被冠以一个特定的名词，叫做长肥网络，英文是 Long Fat Network
* TCP 传输的核心公式：`速度 = 窗口 / 往返时间`
* 传输速度的上限就是 window/RTT = 64KB/134ms = 478KB/s

## TCP Window Full
* 一般说到 TCP Window，如果没有特别指明，就是指接收窗口。
* TCP 的下个序列号（Next Sequence Number）等于序列号和段长度之和，即 NextSeq = Seq + Len。
* 在途数据计算 `Bytes_in_flight = latest_nextSeq - latest_ack_from_receiver`
* 在 Statistics 下拉菜单下的 I/O Graph 工具，可以直观地展示传输速度图。
* 同是 Statistics 菜单下的 TCP Stream Graphs 的 Window Scaling 工具，可以直观的展示 TCP Window Full 历史曲线图。
* 名称
  * Acknowledgement Number：确认号
  * Next Sequence Number：下个序列号
  * Caculated Window Size：计算后的接收窗口
  * Bytes in flight：在途字节数
* velocity = acked_data/RTT

## TCP探测拥塞
* 概念
  * 慢启动：每收到一个 ACK，拥塞窗口（CW）增加一个 MSS。
  * 拥塞避免：策略是“和性增长乘性降低”，每一个 RTT，CW 增加一个 MSS。
  * 快速重传：接收到 3 次或者以上的重复确认后，直接重传这个丢失的报文。
  * 快速恢复：结合快速重传，在遇到拥塞点后，跳过慢启动阶段，进入线性增长。
* Wireshark 序列号（会根据自己发送的数据量进行增加）
  * Acknowledgment: Ack序号
  * nextSeq: 下一个序号
* 拥塞控制算法 （命令 sysctl）
  1. 修改拥塞控制算法 `sudo sysctl net.ipv4.tcp_congestion_control=bbr`
  2. net.ipv4.tcp_congestion_control
     1. cubic
     2. bbr (如果内核大于 4.9，Linux 默认带有 BBR)
  3. net.core.default_qdisc
     1. fq

## 重传
* TCP 的确认报文如果丢失了，发送端还会不会重传呢？为什么？
  1. 如果发送端先后发送了报文n，n+1，接收端也先后确认了这两个报文，如果ACK[n] 传输时丢失，ACK[n+1] 正常传输，由于后者的确认范围涵盖了前者，那么即使 ACK[n] 丢失也不会有什么影响，则不会重传 ACK[n]。
  2. 如果在情况2中，第 n+1 个报文是当前数据流的最后一个报文，且 ACK[n+1] 传输时丢失了，发送端经过1个RTO时间后会触发超时重传，接收端在接收到报文后会重传 ACK[n+1]。

* 超时重传(Retransmission Timeout,RTO)
  * 在报文发送出去后就开始计时，在时限内对方回复 ACK 的话，计时器就清零
  * 报文在发送途中丢失，没有到达接收方，那接收方也不会回复确认包。
  * 报文到达接收方，接收方也回复了确认，但确认包在途中丢失。
  * 超时重传也还是可能会丢包，此时发送方一般会以 RTO 为基数的 2 倍、4 倍、8 倍等时间倍数去尝试多次
  * `PUSH 和 重传的关系？`
  * RTO
    * RFC6298规定：在一条 TCP 连接刚刚开始，还没有收到任何回复的时候，这时的超时 RTO 为 1 秒
    * 在连接成功建立后，Linux 会根据 RTT 的实际情况，动态计算出 RTO。常见，RTO 200ms
    * 上限值：120s; 下限值: 200ms

![timeout-retransmission](/images/timeout_retransmission.png)

* Spurious重传
  * A 发送了报文 X，B 回复了确认，A 再次发送 X。
  * A 收到了 B 发过来的报文 X，A 也回复了确认，但 B 再次发送 X。

![spurious-retransmission](/images/spurious_retransmission.png)

* 快速重传
  * 如果对端回复连续 3 个 DupAck 即重复确认，我就把序列号等于这个 ACK 号的包重传。

* SACK
  * `TCP Option - SACK`
    * 记录 左右两个丢包后已经被确认的界限
    * SACK 要能工作，还需要 `TCP Option - SACK permitted` 这个 TCP 扩展属性的支持
    * 受限于 TCP Option 长度，SACK 部分最多只能容纳 4 个块

![sack-1](/images/sack_1.png)
![sack-2](/images/sack_2.png)
![sack-3](/images/sack_3.png)

* Ack可以只确认一部分包（确认号是在中间位置）
![ack-percent](/images/ack_percent.png)

## DDos攻击
* NTP 反射放大攻击
* SSDP 反射型攻击