---
author: Sunny Tsai
title: "[Day 8] 閑的沒事就寫封包 - gopacket初建封包（未完）"
date: 2023-09-21
math: true
tags: [
    "cloud",
    "gopacket",
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-DDOS：閑的沒事就開始DDOS"]
---
以下是一個簡單的建立單純的TCP SYN封包示例

## Gopacket建立封包
按照文件，code應該要這樣寫
```go
package main

import (
	"log"
	"net"
	"time"

	"github.com/google/gopacket"
	"github.com/google/gopacket/layers"
	"github.com/google/gopacket/pcap"
)

func main() {

	var (
		device      string = "bridge100"  // 換了一張網卡
		snapshotLen int32  = 1024
		promiscuous bool   = false
		err         error
		timeout     time.Duration = 10 * time.Second
		handle      *pcap.Handle
		buffer      gopacket.SerializeBuffer
	)

	var SerializationOptions = gopacket.SerializeOptions{
		FixLengths:       true,
		ComputeChecksums: true,
	}

	// Open device
	handle, err = pcap.OpenLive(device, snapshotLen, promiscuous, timeout)
	if err != nil {
		log.Fatal(err)
	}
	defer handle.Close()

	// SrcMAC跟DstMAC兩個可以不用按照真實案例
	ethernetLayer := &layers.Ethernet{
		SrcMAC:       net.HardwareAddr{0xFF, 0x00, 0xFF, 0x00, 0xFF, 0x00},
		DstMAC:       net.HardwareAddr{0xFF, 0x00, 0xFF, 0x00, 0xFF, 0x00},
		EthernetType: layers.EthernetTypeIPv4,
	}
	ipLayer := &layers.IPv4{
		SrcIP:    net.IP{10, 211, 55, 2},    //source ip
		DstIP:    net.IP{10, 211, 55, 10},
		Version:  4,
		TTL:      64,
		Protocol: layers.IPProtocolTCP,
	}
	tcpLayer := &layers.TCP{
		SrcPort: layers.TCPPort(12345),
		DstPort: layers.TCPPort(12345),
		SYN:     true,
		ACK:     true,
	}
	tcpLayer.SetNetworkLayerForChecksum(ipLayer)

	// And create the packet with the layers
	buffer = gopacket.NewSerializeBuffer()
	gopacket.SerializeLayers(buffer, SerializationOptions,
		ethernetLayer,
		ipLayer,
		tcpLayer,
	)

    // Send serialize packet
	outgoingPacket := buffer.Bytes()
	err = handle.WritePacketData(outgoingPacket)
	if err != nil {
		log.Fatal(err)
	}
}
```

同樣在靶機上攔截封包：
```shell
# 靶機上收到Flag為S(SYN)封包
02:48:47.273581 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto TCP (6), length 64)
    10.37.129.2.12347 > ubuntu-linux-20-04-desktop.12345: Flags [S], cksum 0xd356 (correct), seq 7459, win 65535, options [mss 1460,wscale 6,sackOK,nop,nop,TS val 16909060 ecr 84281096,eol], length 0
```
沒錯只有這樣，正常的SYN封包是會讓靶機回傳ACK/SYN封包做三次握手的第二步驗證的，但沒有，說明這是異常封包。


### 驚喜點
可以在Network Layer層修改任意SrcIP

```go
ipLayer := &layers.IPv4{
		SrcIP:    net.IP{10, 211, 55, 3},    // <- 這邊改成任意IP，就算是網段下沒有配置的IP也可以
		DstIP:    net.IP{10, 211, 55, 10},
		Version:  4,
		TTL:      64,
		Protocol: layers.IPProtocolTCP,
	}
```
然後可以發送。

## 使用net.Dial發送封包
```go
func main() {
	targetIP := "10.211.55.10"
	targetPort := 12345

	tcpConn, err := net.Dial("tcp", fmt.Sprintf("%s:%d", targetIP, targetPort))
	if err != nil {
		log.Fatalf("Failed to connect：%v", err)
	}
	defer tcpConn.Close()
}
```

## 攔截封包
看起來是很正常的TCP三次握手
```shell
02:55:36.431738 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto TCP (6), length 52)
    10.211.55.2.52261 > ubuntu-linux-20-04-desktop.12345: Flags [.], cksum 0x4fe0 (correct), ack 1, win 2058, options [nop,nop,TS val 2224605755 ecr 3013823431], length 0
02:55:36.432070 IP (tos 0x2,ECT(0), ttl 64, id 0, offset 0, flags [DF], proto TCP (6), length 72)
    10.211.55.2.52261 > ubuntu-linux-20-04-desktop.12345: Flags [P.], cksum 0xeed4 (correct), seq 1:21, ack 1, win 2058, options [nop,nop,TS val 2224605755 ecr 3013823431], length 20
02:55:36.432123 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto TCP (6), length 52)
    10.211.55.2.52261 > ubuntu-linux-20-04-desktop.12345: Flags [F.], cksum 0x4fcb (correct), seq 21, ack 1, win 2058, options [nop,nop,TS val 2224605755 ecr 3013823431], length 0
02:55:36.432199 IP (tos 0x0, ttl 64, id 9536, offset 0, flags [DF], proto TCP (6), length 52)
    ubuntu-linux-20-04-desktop.12345 > 10.211.55.2.52261: Flags [.], cksum 0x83d8 (incorrect -> 0x55d8), ack 21, win 509, options [nop,nop,TS val 3013823432 ecr 2224605755], length 0
02:55:36.446463 IP (tos 0x2,ECT(0), ttl 64, id 9537, offset 0, flags [DF], proto TCP (6), length 608)
    ubuntu-linux-20-04-desktop.12345 > 10.211.55.2.52261: Flags [P.], cksum 0x8604 (incorrect -> 0x8842), seq 1:557, ack 22, win 509, options [nop,nop,TS val 3013823446 ecr 2224605755], length 556
02:55:36.446805 IP (tos 0x0, ttl 64, id 9538, offset 0, flags [DF], proto TCP (6), length 52)
    ubuntu-linux-20-04-desktop.12345 > 10.211.55.2.52261: Flags [F.], cksum 0x83d8 (incorrect -> 0x539b), seq 557, ack 22, win 509, options [nop,nop,TS val 3013823447 ecr 2224605755], length 0
02:55:36.446863 IP (tos 0x2,ECT(0), ttl 64, id 0, offset 0, flags [DF], proto TCP (6), length 40)
    10.211.55.2.52261 > ubuntu-linux-20-04-desktop.12345: Flags [R], cksum 0x3974 (correct), seq 2853653572, win 0, length 0
02:55:36.446962 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto TCP (6), length 40)
    10.211.55.2.52261 > ubuntu-linux-20-04-desktop.12345: Flags [R], cksum 0x3974 (correct), seq 2853653572, win 0, length 0
```

明天的目標是：使用gopacket建立一個合法合理的封包。

### 碎碎念
我也不確定行不行？


### 2023-09-22補

gopacket TCP packet先pass，原碼還沒看完。
