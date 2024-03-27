---
author: Sunny Tsai
title: "[Day 30] 閑的沒事 - HTTP SESSION"
date: 2023-10-13
math: true
tags: [
    "cloud",
    "http session"
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-DDOS：閑的沒事就開始DDOS"]
---
# Http session
Http session 利用了HTTP/1.1中的`Keep-Alive`功能。

# Keep-Alive
早期的Http版本中，每次的Http request&response完成後，連線就會被關閉。也就是說，當網頁有多個元素的時候（圖片、CDN、腳本等），每個元素的請求都要重新建一個TCP連線，因為建立TCP連線需要時間（從TCP&UDP連線可以測得出來，同樣的code UDP協議就是快很多），這個效果是非常不好的。

為了解決這個問題，HTTP/1.1引入了`Persistent Connection`（持久連線），也可以稱作`Keep-Alive`，功能目的是允許在已經建立的TCP連線上執行多次的Http request&response，不用在重新連接。


# Http Flood
Http Get Flood和Http Post Flood兩個都是常見的Http Flood攻擊，跟Http session一樣作用於應用層，同樣更難檢測跟防禦。

但Http Get/Post Flood都是大量發送Request到目標主機，嘗試消耗目標服務器的頻寬與資源；Http Session Flood主要是濫用Http session管理機制嘗試消耗目標服務器的資源，比如說創建大量未完成或畸形的對話連接。

