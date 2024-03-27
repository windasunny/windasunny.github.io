---
author: Sunny Tsai
title: "[Day 28] 閑的沒事 - Fragmentation TCP"
date: 2023-10-11
math: true
tags: [
    "cloud",
    "fragmentation attack",
    "tcp"
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-DDOS：閑的沒事就開始DDOS"]
---
# Code
使用Dail發送tcp封包
Payload暫定長度1360
```golang
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

	conn, err := net.DialTimeout("tcp", fmt.Sprintf("%s:%d", targetIP, targetPort), 10*time.Second)
	if err != nil {
		log.Fatal(err)
	}
	defer conn.Close()

	// build payload, length 1360
	payload := `Hello World\nHello World\nHello World\nHello World\nHello World\nHello World\n \
				Hello World\nHello World\nHello World\nHello World\nHello World\nHello World\n \
				Hello World\nHello World\nHello World\nHello World\nHello World\nHello World\n \
				Hello World\nHello World\nHello World\nHello World\nHello World\nHello World\n \
				Hello World\nHello World\nHello World\nHello World\nHello World\nHello World\n \
				Hello World\nHello World\nHello World\nHello World\nHello World\nHello World\n \
				Hello World\nHello World\nHello World\nHello World\nHello World\nHello World\n \
				Hello World\nHello World\nHello World\nHello World\nHello World\nHello World\n \
				Hello World\nHello World\nHello World\nHello World\nHello World\nHello World\n \
				Hello World\nHello World\nHello World\nHello World\nHello World\nHello World\n \
				Hello World\nHello World\nHello World\nHello World\nHello World\nHello World\n \
				Hello World\nHello World\nHello World\nHello World\nHello World\nHello World\n \
				Hello World\nHello World\nHello World\nHello World\nHello World\nHello World\n \
				Hello World\nHello World\nHello World\nHello World\nHello World\nHello World\n \
				Hello World\nHello World\nHello World\nHello World\nHello World\nHello World\n \
				Hello World\nHello World\nHello World\nHello World\nHello World\nHello World\n \
				`
	fmt.Println(len(payload))

	// _, err = conn.Write([]byte(payload))
	// if err != nil {
	// 	log.Fatal(err)
	// }
	chunkSize := 1000  # 設定每個分片分割長度為1000
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
payload長度1360，分割為每個分片不超過長度1000，可以正常發送碎片化封包
payload長度1360，分割為每個分片不超過長度100，出現write: broken pipe錯誤，顯示目標主機主動關閉連接。
大概發送10次以上就有可能出現write: broken pipe錯誤。
