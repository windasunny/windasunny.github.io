---
author: Sunny Tsai
title: "[Day 1] DDOS開賽前言"
date: 2023-09-14
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
## 鐵人賽開賽前言

其實這是我今年7月就準備寫的一個side project，結果拖到了現在一個code都沒有寫好。
今年7月我寫了一個Go的併發爬蟲專案，併發爬蟲並丟到資料庫，跑了一個超快超迅速，沒有特定釋出memory的情況下還不吃memory，然後我就突然愛上Go了。我原本有用python寫關於底層封包設定，後來想想，既然要寫DDOS，代表說機器丟封包可能需要丟得越快越好，然後我就決定改用Go寫。~~然後我就拖到現在都沒寫，還去參加了救生員訓練，delay了3個禮拜~~


## 大綱

整個鐵人賽程的大綱就是，我為什麼要寫這個？寫這個有什麼作用？~~我為什麼要在這裡？~~


### 碎碎念
基於我還沒有動筆的原因，其實我也不太確定能不能寫好寫滿30天。而且中間架構我該不會會改來改去之類的。但主題是SideProject30，我寫side project的時候可能看到別人好的架構或是好的封裝或是用到好用的套件我就會把整個都改一遍，有空的話還會寫benchmark證明一下我為什麼要改，因為經驗不足的原因，看到好用的我都想用用看。希望大家包容一下~