---
author: Sunny Tsai
title: "[Day 2] 閑著沒事就DDOS - 什麼是DDOS"
date: 2023-09-15
math: true
tags: [
    "cloud",
    "ddos",
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-DDOS：閑的沒事就開始DDOS"]
---
# 什麼是DDOS
分散式阻斷服務（Distributed Denial of Service），主要是通過製造大流量來攻擊特定服務器或基礎設施，以阻斷正常流量，中斷用戶與服務器或應用程序的連接，造成流量阻塞。

什麼是**DOS**？
這個很多人都會搞混，DOS（Denial of Service），同樣是大規模封包攻擊特定服務器或基礎設施，但由於是單一來源，攻擊規模較小，而且一旦識別出IP後，即可阻止攻擊。

因此，DDOS可以當作分布式的DOS。DDOS發送的封包會更複雜更多樣化，以避免用戶封鎖特定IP或特定協議來阻擋攻擊。DDOS攻擊可以針對OSI模型的不同層發起不同類型的攻擊。
![](https://imgur.com/2AdZYgO.png)
[來源](https://www.imperva.com/learn/application-security/osi-model/)

### 第一層 Physical Layer
這一層的攻擊主要是物理攻擊，例如說切斷網絡連接、摧毀網絡設備、割斷電纜等。跟主題無關。

### 第二層 Data Link Layer
攻擊會針對網路交換器、橋接器、或其他設備進行攻擊。可能通過MAC Spoofing、ARP Spoofing或利用數據鏈路協議漏洞來導致網絡連接中斷或混亂。
行為：MAC Spoofing、ARP Spoofing

### 第三層 Network Layer
Network layer攻擊包括ICMP Flood、Fragment Packet Flood和Route Attack。
行為：ICMP Flood、Fragment Packet Flood、Route Attack

### 第四層 Transport Layer
這一層攻擊主要針對TCP、UDP協定，包括SYN Flood、UDP Flood、RST Attack等，試圖耗盡目標系統的連接資源或混淆傳輸協議以干擾正常通信。
行為：SYN Flood、UDP Flood、RST Attack、Reflection Attack

### 第五層 Session Layer
這些攻擊針對Session Layer會話和通信，如HTTP Flood、SSL/TLS Flood。攻擊者可能試圖使目標網站無法處理用戶的請求或使安全通信協議無法建立。
行為：HTTP Flood、SSL/TLS Flood

### 第六層 Presentation Layer
利用Presentation Layer協議漏洞來造成損害，但通常會依據依賴的程式碼，所以並不常見。例如說：加密漏洞、數據解碼漏洞等。

### 第七層 Application Layer
Application Layer攻擊是最常見的DDoS攻擊類型，包括HTTP Flood、DNS Flood、SMTP Flood等。攻擊者通常針對特定應用程式及服務，試圖使其無法正常運作。
行為：HTTP Flood、DNS Flood、SMTP Flood、Amplification Attack
