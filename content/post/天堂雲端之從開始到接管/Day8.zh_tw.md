---
author: Sunny Tsai
title: "[Day 8] 天堂雲端 - 審核雲端環境（藍隊）"
date: 2023-09-21
math: true
tags: [
    "cloud",
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-天堂雲端之從開始到接管"]
---
# 藍隊視角 - 一般情況下雲端環境漏洞

### 配置錯誤（Misconfigurations）
雲端服務提供預設配置，但不能說就完全沒有問題，錯誤的配置、開放的端口和未使用的服務都可能造成漏洞。

### 弱憑證（Weak Credentials）
跟弱密碼一樣，容易被破解的憑證容易導致入侵。

### 缺少加密設定（Lack of Encryption）
數據在存儲或傳輸的時候缺少加密。

### 不安全的APIs（Insecure APIs）
不安全的API介面容易被攻擊者利用，導致數據洩露或是未經授權的訪問。

### 未授權訪問（Unauthorized access）
攻擊者可以透過猜測、竊取，以未經授權的方式訪問雲端資源。

### 內部威脅（Insider Threats）
內部員工或承包商的惡意泄露或疏忽行為可能會導致敏感數據洩露。

### 未修補的漏洞（Unpatched Vulnerabilities）
雲端供應商來不及修補的漏洞，成了供應商跟攻擊者間的競賽。

### 不安全的第三方套件（Insecure Third-party Integrations）
惡意或是不安全的第三方設定容易導致數據洩露或是未經授權的訪問。

### 數據洩露（Data Leakage）
因為程式碼設定不當或是上述環境漏洞導致的數據洩露。

### 日誌與監控不足（Insufficient logging and monitoring）
雲端服務通常都有日誌，沒有正確管理日誌與監控會導致有紀錄攻擊行為但被忽略。
