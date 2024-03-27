---
author: Sunny Tsai
title: "[Day 16] 閑著沒事 - 反射攻擊理論"
date: 2023-09-29
math: true
tags: [
    "cloud",
    "reflection",
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-DDOS：閑的沒事就開始DDOS"]
---
# 什麼是反射攻擊

反射攻擊指的就是利用大量代理伺服器或是開放的資源來放大流量，通常使用允許小的request生成大量的response，這些response會被重定向的目標伺服器


## 步驟

1. 攻擊者準備已被控制的合法伺服器，作為反射服務器
2. 攻擊者向這些反射服務器發送request，來源地址偽裝成目標地址。通常使用UDP封包，因為UDP不需連接即可發送。
3. 攻擊者選定一些適合做反射攻擊的協議，可以以小博大。例如DNS、NTP、SSDP、CLDAP、Memcached、CharGen 或 QOTD。


### DNS反射攻擊
攻擊者向反射服務器發送解析DNS request，反射服務器向目標主機發送大量的DNS response回覆給目標主機
例如:
```
> dig ANY http://{TARGET HOST} @x.x.x.x
```


### NTP反射攻擊
> NTP : 通過網路協議使主機時間同步化
攻擊者利用偽造的IP地址向NTP（Network Time Protocol/網路時間協議）反射伺服器發送Request
```
> ntpdc -n -c monlist x.x.x.x | wc -l
```

### SSDP反射攻擊
攻擊者使用偽造的IP地址向SSDP（Simple Service Discovery Protocol/簡單服務發現協議）伺服器發送Request
SSDP用於Universal Plug and Play（UPnP）設備上，用來發現設備。特別應用在物聯網及智能家電上。
（我沒有需要UPnP的設備，端口打不開，先跳過）
如果需要測試的話可以先用nmap測試是否可以收到response
```
nmap x.x.x.x -p 1900 -sU --script=upnp-info
```


### SNMP反射攻擊
攻擊者利用偽造的IP地址向SNMP（Simple Network Management Protocol/簡單網絡管理協議）伺服器發送請求

### Chargen反射攻擊
Chargen是一個用於測試的協議，攻擊者可以使用偽造的IP地址向Chargen伺服器發送Request，伺服器將以無限數據流形式回應。
查看port 19是否有chargen服務傳輸字元
```
telnet -r localhost 19
```


### LDAP放大攻擊
攻擊者使用偽造的IP地址向LDAP（輕量級目錄訪問協議）伺服器發送Request

### CLDAP
攻擊者使用偽造的IP地址向CLDAP（無連接輕量級目錄訪問協議）伺服器發送Request

### Memcached反射攻擊
攻擊者利用Memcached伺服器中存儲的大量數據，將攻擊目標的IP地址偽造為請求的來源，然後發送一個小的請求，Memcached伺服器會回應大量數據到目標IP地址。

### QOTD
攻擊者使用偽造的IP地址向QOTD（每日引言）伺服器發送Request
聽起來很怪但是出現在2021 cloudflare ddos報告中..
