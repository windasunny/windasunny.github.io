---
author: Sunny Tsai
title: "[Day 1] 天堂雲端從開始到接管-鐵人賽開賽"
date: 2023-09-14
math: true
tags: [
    "cloud",
    "security",
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-天堂雲端之從開始到接管"]
---

## 鐵人賽開賽前言

不例外的第一天先來介紹一下接下來地獄30天會寫什麼。題目是雲端攻擊，但是還沒建立雲端環境（對我沒有現成的）哪來的target來攻擊？所以接下來的鐵人30天讓我們一起建構ㄧ套雲端架構然後再把它打掉。

## 大綱

先介紹雲服務種類架構，再來是一般架構/雲端架構、上雲優缺點。既然要上雲了，那雲供應商提供的安全性跟使用者需要自己注意的安全性是如何分攤？把前情提要都介紹完了後，那我們開始建立靶機跟POC

## 為什麼想要寫這個主題

雲端攻擊算是最近我有在研究的方向，雲端架構脫胎於一般落地架構，所以前面幾個章節會先大概講一下雲的基礎知識跟使用開源套件建立的架構，再來雲端架構，然後是雲端攻擊。因為我並不是專業的雲端架構師，所以對於雲端架構方面可能不是很專業的那種，但本次鐵人賽目的是建起來再打掉，所以並不會在雲端架構上面有多琢磨。
前面對於一些已經有雲端服務知識的人可能感覺沒什麼好看的，可以等到後面POC再回來。
