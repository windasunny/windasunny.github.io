---
author: Sunny Tsai
title: "[Day 27] 閑的沒事 - 碎片攻擊（Fragmentation Attack）之理論"
date: 2023-10-10
math: true
tags: [
    "cloud",
    "fragmentation attack",
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-DDOS：閑的沒事就開始DDOS"]
---
# Fragmentation Attack
Fragmentation Attack（碎片攻擊）指的是利用IP協議的碎片化（fragmentation）機制來達成攻擊目的。IP碎片是指將一個較大的IP數據包拆分成多個較小的數據包，以適應網絡中最大傳輸單位（MTU）的限制。
以太網的MTU是1500，如果IP層有要傳輸的封包長度超過1500，那IP層就要對封包進行分片(fragmentation)，使每個封包的長度都小於1500。
![](https://imgur.com/Jusk9GW.png)
使用netstat -i 查看MTU大小

碎片化在正常的網絡通信中是一個合法且有用的過程，但攻擊者可以利用這一機制進行惡意操作。

# 主要特徵與特點
### 碎片化利用
攻擊者可以將payload分割成小碎片，然後將這些碎片發送到目標主機，如此可以避掉防火牆或是IPS等防禦機制的檢查。

### 碎片重組
目標主機收到碎片後會重組成原始數據包，這個時候有些老舊系統就有點不行了，容易造成崩潰或拒絕服務。


# 具體例子

### Ping of Death
Ping of Death是一種利用Internet Control Message Protocol（ICMP）協議的碎片攻擊，攻擊者發送一個長度超過65535的封包，目標主機在重組分片的時候超出系統的緩衝區容量，造成緩衝區溢出。這種溢出可以導致系統崩潰或者讓攻擊者執行任意代碼。

#### 手法
使用ping發出一條超出緩衝區容量的封包
```shell
> ping -c 1 -s 65537 10.211.55.3
```
輸出
ping of death是一個很老的攻擊手段，現在的作業系統已經不讓我們做壞事了
```shell
ping: packet size 65537 is too large. Maximum is 65507
```

65507是65535-20(IP layer)-8(payload部分預留)=65507


### Teardrop attack
在Teardrop攻擊中，攻擊者發送一系列畸形的IP碎片，這些碎片在重組時會互相重疊。目標系統可能無法正確處理這些重疊的碎片，從而導致系統崩潰或重啟。
