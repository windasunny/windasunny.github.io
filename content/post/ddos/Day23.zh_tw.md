---
author: Sunny Tsai
title: "[Day 23] 閑的沒事 - CLDAP放大攻擊"
date: 2023-10-06
math: true
tags: [
    "cloud",
    "cldap",
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-DDOS：閑的沒事就開始DDOS"]
---
# CLDAP
Connectionless Lightweight Directory Access Protocol(CLDAP)無連結輕量級目錄協議，與LDAP類似，基於UDP協議。為了效能與便利性，使用UDP 389 port進行傳輸。

# code
```
package main

import (
	"log"
	"net"
)

func main() {

	targetIP := "10.211.55.10"
	targetPort := 389

	conn, err := net.DialUDP("udp", nil, &net.UDPAddr{
		IP:   net.ParseIP(targetIP),
		Port: targetPort,
	})
	if err != nil {
		log.Fatalf("Failed to create UDP connection: %v", err)
	}
	defer conn.Close()

	cldapPacket := []byte{
		...
	}

	_, err = conn.Write(cldapPacket)
	if err != nil {
		log.Fatalf("Failed to send CLDAP packet: %v", err)
	}
}
```
先奉上用Dial寫的udp packet。cldapPacket那邊應該要用結構體寫，原本是硬寫，好像不太對。明明改改。

CLDAP廣泛用於Windows伺服器的AD，哎但我沒有可以測的..
