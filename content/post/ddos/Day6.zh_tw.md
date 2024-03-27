---
author: Sunny Tsai
title: "[Day 6] 閑的沒事就攔截封包 - 封包解析"
date: 2023-09-19
math: true
tags: [
    "cloud",
    "syn",
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-DDOS：閑的沒事就開始DDOS"]
---
昨天用sys.unix寫好了SYN封包

## 發送封包
```
% go run
```

## 攔截封包

使用tcpdump攔截封包
```shell
$ sudo tcpdump -i eth0 host 10.211.55.10 and port 12345 -v
```

```shell
# 收到封包
17:38:27.106775 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto TCP (6), length 64)
    # 主機ubuntu-linux-20-04-desktop的port 12345收到TCP Flags S(SYN)E(ECE)W(CWR)封包，三次握手第一步封包
    10.211.55.2.23457 > ubuntu-linux-20-04-desktop.12345: Flags [SEW], cksum 0x0e0f (correct), seq 600188989, win 65535, options [mss 1460,nop,wscale 6,nop,nop,TS val 2770151296 ecr 0,sackOK,eol], length 0
17:38:27.106987 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto TCP (6), length 60)
    # 主機ubuntu-linux-20-04-desktop回傳SYN/ACK封包，Flags S(SYN)E(ECE)，這是TCP三次握手的第二步，注意ACK是上一個封包的SEQ數值的+1值。
    ubuntu-linux-20-04-desktop.12345 > 10.211.55.2.23457: Flags [S.E], cksum 0x83e0 (incorrect -> 0xe89b), seq 2971612303, ack 600188990, win 65160, options [mss 1460,sackOK,TS val 2914428151 ecr 2770151296,nop,wscale 7], length 0
17:38:27.107208 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto TCP (6), length 52)
    # 主機10.211.55.2回傳確認連接封包
    10.211.55.2.23457 > ubuntu-linux-20-04-desktop.12345: Flags [.], cksum 0x0e27 (correct), ack 1, win 2058, options [nop,nop,TS val 2770151296 ecr 2914428151], length 0
17:38:27.107293 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto TCP (6), length 52)
    # 主機10.211.55.2發送TCP Flag F(FIN)，通知主機ubuntu-linux-20-04-desktop要結束連接。
    10.211.55.2.23457 > ubuntu-linux-20-04-desktop.12345: Flags [F.], cksum 0x0e25 (correct), seq 1, ack 1, win 2058, options [nop,nop,TS val 2770151297 ecr 2914428151], length 0
17:38:27.108219 IP (tos 0x0, ttl 64, id 28745, offset 0, flags [DF], proto TCP (6), length 52)
    # ubuntu-linux-20-04-desktop發送ACK + 1，此為對上一個封包的確認。
    ubuntu-linux-20-04-desktop.12345 > 10.211.55.2.23457: Flags [.], cksum 0x83d8 (incorrect -> 0x142f), ack 2, win 510, options [nop,nop,TS val 2914428153 ecr 2770151297], length 0
17:38:27.108563 IP (tos 0x0, ttl 64, id 28746, offset 0, flags [DF], proto TCP (6), length 52)
    # ubuntu-linux-20-04-desktop發送TCP Flag F(FIN)封包，通知主機10.211.55.2要結束連接。
    ubuntu-linux-20-04-desktop.12345 > 10.211.55.2.23457: Flags [F.], cksum 0x83d8 (incorrect -> 0x142e), seq 1, ack 2, win 510, options [nop,nop,TS val 2914428153 ecr 2770151297], length 0
17:38:27.108788 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto TCP (6), length 52)
    # 主機10.211.55.2回傳確認封包
    10.211.55.2.23457 > ubuntu-linux-20-04-desktop.12345: Flags [.], cksum 0x0e20 (correct), ack 2, win 2058, options [nop,nop,TS val 2770151299 ecr 2914428153], length 0

    # 兩邊主機暫停連接。
```
