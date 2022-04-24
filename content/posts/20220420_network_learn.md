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