---
author: Sunny Tsai
title: "[Day 12] 閑的沒事就寫封包 - UDP Flood之建立封包"
date: 2023-09-25
math: true
tags: [
    "cloud",
    "udp",
    "udp flood"
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-DDOS：閑的沒事就開始DDOS"]
---
# UDP Flood
udp(User Datagram Protocol)是一種非連線型的傳輸協定，意思是說，udp協定只要把封包丟出去就好了，不需要像tcp一樣還要確認連線。攻擊者產生任意PORT的udp封包傳送到伺服器，目標主機收到udp封包後，先判斷有沒有應用程式正在監聽這個PORT，如果沒有就回傳一個ICMP封包回覆此目的地無法到達。意思是說，只要此PORT沒有被應用程式監聽，目標主機收到一個udp封包就會回傳一個ICMP封包。
![](https://imgur.com/6LoWesC.png)

## 如何應對UDP Flood攻擊
1. 由防火牆區別並丟掉無意義的udp封包
    * 限制單一來源的udp封包數量，如果超過某個閥值，直接丟到udp封包
    * 只允許特定或信任的端口接收udp封包。
2. 作業系統限制對ICMP封包回應率
3. IDS/IPS異常監測
4. 加裝DDOS防禦設備等

sysUdp.go
```
// @Title   sysUdp.go
// @Description   Build a udp packet with sys/unix
// @Author   Sunny Tsai
// @Update   2023-09-25
package packet

import (
	"log"
	"math/rand"
	"net"
	"time"

	"golang.org/x/sys/unix"
)

// @title   Udp
// @description   Build a SYN packet with sys/unix
// @auth     Sunny Tsai     Time(2023/09/25)
// @param    targetIP        string        send to target ip address
// @param    targetPort      int           send to target port
// @return   error           error
func Udp(targetIP string, targetPort int) error {

	random := rand.New(rand.NewSource(time.Now().UnixNano()))

	srcIP := "0.0.0.0"
	srcPort := random.Intn(65535)

	// build socket
	fd, err := unix.Socket(unix.AF_INET, unix.SOCK_DGRAM, 0)
	if err != nil {
		log.Fatal(err)
	}
	defer unix.Close(fd)

	localAddr := &unix.SockaddrInet4{Port: srcPort}
	copy(localAddr.Addr[:], net.ParseIP(srcIP).To4())

	err = unix.Bind(fd, localAddr)
	if err != nil {
		log.Fatal(err)
	}

	// set message
	message := []byte("Hello, UDP!") # Payload通常是隨機的或沒有意義的，因為目的不是要傳遞有用的信息，而是要消耗目標服務器的資源。

	sendTo(targetIP, targetPort, fd, message)

	return nil
}

// @title   sendTo
// @description   Send udp packet to target
// @auth     Sunny Tsai     Time(2023/09/24)
// @param    targetIP        string        send to target ip address
// @param    targetPort      int           send to target port
// @param    fd              int           socket fd
// @param    message         []byte        message
func sendTo(targetIP string, targetPort int, fd int, message []byte) {
	remoteAddr := &unix.SockaddrInet4{Port: targetPort}
	copy(remoteAddr.Addr[:], net.ParseIP(targetIP).To4())

	_ = unix.Sendto(fd, message, 0, remoteAddr)
}
```
