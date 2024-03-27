---
author: Sunny Tsai
title: "[Day 25] 閑的沒事 - QOTD反射攻擊"
date: 2023-10-08
math: true
tags: [
    "cloud",
    "qotd",
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-DDOS：閑的沒事就開始DDOS"]
---
# QOTD
[Quote of the Day（每日引言）](https://gist.github.com/alphaolomi/51f27912abe699dd5db95cfbc21d1a2d)協議是為了在網際網絡上提供一種簡單的測試和演示網路通訊的能力而創建的。它的主要目的不是為了傳送實際的有用數據，而是為了展示伺服器之間的基本通信能力和連接性。在現今已經相對不常見，並且在大多數實際應用中已經被淘汰或不再使用。但是還是出現在了[Cloudflare 2021 年第二季 DDoS 攻擊趨勢](https://blog.cloudflare.com/zh-tw/ddos-attack-trends-for-2021-q2-zh-tw/)報告中。

# 測試
在inet設定qotd，qotd佔用port 17
```
> telnet localhost 17
```

# code
跟chargen一樣，使感染的機器把response送到目標機
把port改成17就可以了
