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