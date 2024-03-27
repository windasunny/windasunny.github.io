---
author: Sunny Tsai
title: "[Day 9] 閑的沒事就寫封包 - gopacket建立UDP封包"
date: 2023-09-22
math: true
tags: [
    "cloud",
    "gopacket",
    "udp"
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-DDOS：閑的沒事就開始DDOS"]
---
# 建立封包

UDP封包

```go
package main

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
		SrcIP:    net.IP{10, 211, 55, 2},
		DstIP:    net.IP{10, 211, 55, 10},
		Version:  4,
		TTL:      64,
		Protocol: layers.IPProtocolUDP,
	}

	udpLayer := &layers.UDP{
		SrcPort: srcPort,
		DstPort: dstPort,
	}
	udpLayer.SetNetworkLayerForChecksum(ipLayer)

	payload := []byte("Hello World")

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

## 靶機攔截封包
攔截封包
```shell
22:03:52.573287 IP (tos 0x0, ttl 64, id 0, offset 0, flags [none], proto UDP (17), length 39)
    10.211.55.2.26598 > ubuntu-linux-20-04-desktop.12345: UDP, length 11
```

## 與net.Dial比對
使用net.Dial發送UDP封包
```go
func main() {
	targetIP := "10.211.55.10"
	targetPort := 12345
	conn, err := net.Dial("udp", fmt.Sprintf("%s:%d", targetIP, targetPort))
	if err != nil {
		fmt.Println("Error connecting:", err)
		return
	}
	defer conn.Close()
}
```

攔截封包
```shell
22:09:05.603943 IP (tos 0x0, ttl 64, id 40361, offset 0, flags [none], proto UDP (17), length 39)
    10.211.55.2.63766 > ubuntu-linux-20-04-desktop.12345: UDP, length 11
```

UDP成功！
