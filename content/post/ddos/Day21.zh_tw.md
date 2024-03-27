---
author: Sunny Tsai
title: "[Day 21] 閑的沒事 - LDAP放大攻擊之sladp發送ldap協定"
date: 2023-10-04
math: true
tags: [
    "cloud",
    "ldap",
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-DDOS：閑的沒事就開始DDOS"]
---
# LDAP
Lightweight Directory Access Protocol（LDAP）是一種輕量的目錄服務協定（Directory Access Protocol，DAP）。目錄服務通常提供了有組織的記錄集合，這些記錄通常具有層級結構，也就是說這些資料之間存在著父子或是兄弟的關係。
LDAP的常見用途之一是提供中央位置來進行身份驗證，其中包括用戶帳戶和密碼的儲存，例如在`Docker`、`Jenkins`、`Kubernetes`、`Open VPN`和`Linux Samba`等服務中用於驗證帳戶和密碼。此外，系統管理員還可以使用LDAP單一登入來控制對LDAP資料庫的訪問。
實作了LDAP的軟體程式可以被視為是專門用來讀取資料的資料庫(這些資料可能經常被讀取，但不常被變動)，例如說經常用來儲存姓名、電子郵件、電話、地址等人事資料，或是拿來做使用者權限的驗證。
# 測試
### 安裝slapd
確定389 port被佔用
![](https://imgur.com/eHjfuWg.png)

### 測試
我拿之前測試用的kali主機當ldap的client端
client端發送request
![](https://imgur.com/d7c9BIX.png)

### 攔截封包
ldap主機攔截封包
![](https://imgur.com/33ORJm3.png)


明天寫攔截並發送ldap協定封包。
