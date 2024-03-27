---
author: Sunny Tsai
title: "[Day 14] 閑的沒事就寫封包 - ICMP Flood之建立封包"
date: 2023-09-27
math: true
tags: [
    "cloud",
    "icmp",
    "icmp flood"
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-DDOS：閑的沒事就開始DDOS"]
---
# ICMP Flood
攻擊者大量發送icmp(Internet Control Message Protocol)回應請求封包使目標機器不刊負荷，導致正常流量無法進入目標機器。
![](https://imgur.com/ICnfXm6.png)

sysIcmp.go
```
// @Title   sysIcmp.go
// @Description   Build an icmp packet with sys/unix
// @Author   Sunny Tsai
// @Update   2023-09-27
package packet

import (
	"log"
	"net"

	"golang.org/x/sys/unix"
)

// @title   Icmp
// @description   Build an icmp packet with sys/unix
// @auth     Sunny Tsai     Time(2023/09/27)
// @param    targetIP        string        send to target ip address
// @return   error           error
func Icmp(targetIP string) error {
	srcIP := "0.0.0.0"

	// build socket
	fd, err := unix.Socket(unix.AF_INET, unix.SOCK_RAW, unix.IPPROTO_ICMP)
	if err != nil {
		log.Fatal(err)
	}
	defer unix.Close(fd)

	localAddr := &unix.SockaddrInet4{}
	copy(localAddr.Addr[:], net.ParseIP(srcIP).To4())

	err = unix.Bind(fd, localAddr)
	if err != nil {
		log.Fatal(err)
	}

	// set message
	message := []byte("")

	remoteAddr := &unix.SockaddrInet4{}
	copy(remoteAddr.Addr[:], net.ParseIP(targetIP).To4())

	sendIcmpTo(fd, message, remoteAddr)

	return nil
}

// @title   sendIcmpTo
// @description   Send ICMP packet to target
// @auth     Sunny Tsai     Time(2023/09/27)
// @param    fd              int           socket fd
// @param    message         []byte        message
// @param    remoteAddr      *unix.SockaddrInet4    remote address
func sendIcmpTo(fd int, message []byte, remoteAddr *unix.SockaddrInet4) {

	icmpPacket := append([]byte{8, 0, 0, 0, 0, 0, 0, 0}, message...) // ICMP类型8表示Echo Request

	// 计算并设置校验和
	checksum := calculateChecksum(icmpPacket)
	icmpPacket[2] = byte(checksum >> 8)
	icmpPacket[3] = byte(checksum)

	_ = unix.Sendto(fd, icmpPacket, 0, remoteAddr)
}

// @title   calculateChecksum
// @description   Calculate the checksum for ICMP packet
// @auth     Sunny Tsai     Time(2023/09/27)
// @param    data            []byte        ICMP packet data
// @return   uint16          checksum value
func calculateChecksum(data []byte) uint16 {
	sum := uint32(0)

	// Sum all 16-bit words
	for i := 0; i < len(data)-1; i += 2 {
		sum += uint32(data[i+1])<<8 + uint32(data[i])
	}

	// Add carry if any
	if len(data)%2 != 0 {
		sum += uint32(data[len(data)-1]) << 8
	}

	// Fold sum to 16 bits
	for (sum >> 16) > 0 {
		sum = (sum & 0xFFFF) + (sum >> 16)
	}

	// Take one's complement
	checksum := ^uint16(sum)
	return checksum
}
```
