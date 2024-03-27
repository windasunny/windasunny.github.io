---
author: Sunny Tsai
title: "[Day 7] 閑的沒事就找套件 - gopacket"
date: 2023-09-20
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
# Gopacket
[gopacket](https://pkg.go.dev/github.com/google/gopacket)是go的第三方庫，主要用於封包捕獲和解析。可以把它當作libpcap（tcpdump的流量捕獲庫） 和 npcap（Wireshark的流量捕獲庫）的 go 封装。


以下使用gopacket做一些範例

## 捕獲主機上所有封包
```go
package main

import (
	"fmt"
	"log"
	"github.com/google/gopacket"
    "github.com/google/gopacket/layers"
    _ "github.com/google/gopacket/layers"
	"github.com/google/gopacket/pcap"
)

func main() {
	iface := "en0"
	handle, err := pcap.OpenLive(iface, 1600, true, pcap.BlockForever)
	if err != nil {
		log.Fatal(err)
	}
	defer handle.Close()

    // Create packet
	packetSource := gopacket.NewPacketSource(handle, handle.LinkType())

	for packet := range packetSource.Packets() {
		ethLayer := packet.Layer(layers.LayerTypeEthernet)
		if ethLayer != nil {
			ethPacket, _ := ethLayer.(*layers.Ethernet)
			fmt.Println("Ethernet Source MAC:", ethPacket.SrcMAC)
			fmt.Println("Ethernet Destination MAC:", ethPacket.DstMAC)
		}

		ipLayer := packet.Layer(layers.LayerTypeIPv4)
		if ipLayer != nil {
			ipPacket, _ := ipLayer.(*layers.IPv4)
			fmt.Println("IP Source IP:", ipPacket.SrcIP)
			fmt.Println("IP Destination IP:", ipPacket.DstIP)
		}

		tcpLayer := packet.Layer(layers.LayerTypeTCP)
		if tcpLayer != nil {
			tcpPacket, _ := tcpLayer.(*layers.TCP)
			fmt.Println("TCP Source Port:", tcpPacket.SrcPort)
			fmt.Println("TCP Destination Port:", tcpPacket.DstPort)
		}
	}
}

```
結果就會出現主機收到的所有封包
Ethernet Source MAC: {MAC}
Ethernet Destination MAC: {MAC}
IP Source IP: {IP}
IP Destination IP: {IP}
TCP Source Port: {PORT}
TCP Destination Port: {PORT}
...


### 解析packet
```go
package main

import (
	"encoding/hex"
	"fmt"
	"log"
	"time"

	"github.com/google/gopacket"
	"github.com/google/gopacket/layers"
	"github.com/google/gopacket/pcap"
)

var (
	device       string = "en0"
	snapshot_len int32  = 1024
	promiscuous  bool   = false
	err          error
	timeout      time.Duration = 30 * time.Second
	handle       *pcap.Handle
	ethLayer     layers.Ethernet
	ipLayer      layers.IPv4
	tcpLayer     layers.TCP
	udpLayer     layers.UDP
	tlsLayer     layers.TLS
	ipv6Layer    layers.IPv6
)

func main() {
	handle, err = pcap.OpenLive(device, snapshot_len, promiscuous, timeout)
	if err != nil {
		log.Fatal(err)
	}
	defer handle.Close()
	packetSource := gopacket.NewPacketSource(handle, handle.LinkType())
	for packet := range packetSource.Packets() {
		printPacket(packet)
	}
}

func printPacket(packet gopacket.Packet) {
	parser := gopacket.NewDecodingLayerParser(
		layers.LayerTypeEthernet,
		&ethLayer,
		&ipLayer,
		&tcpLayer,
		&udpLayer,
		&tlsLayer,
		&ipv6Layer,
		&gopacket.Payload{},
	)
	foundLayerTypes := []gopacket.LayerType{}
	err := parser.DecodeLayers(packet.Data(), &foundLayerTypes)
	if err != nil {
		fmt.Println("Trouble decoding layers: ", err)
	}
	for _, layerType := range foundLayerTypes {
		switch layerType {
		case layers.LayerTypeEthernet:
			fmt.Println("Ethernet: ", ethLayer.SrcMAC, "->", ethLayer.DstMAC)
		case layers.LayerTypeIPv4:
			fmt.Println("IPv4: ", ipLayer.SrcIP, "->", ipLayer.DstIP)
		case layers.LayerTypeTCP:
			fmt.Println("TCP Port: ", tcpLayer.SrcPort, "->", tcpLayer.DstPort)
			fmt.Println("TCP SYN:", tcpLayer.SYN, " | ACK:", tcpLayer.ACK)
		case layers.LayerTypeUDP:
			fmt.Println("UDP Port: ", udpLayer.SrcPort, "->", udpLayer.DstPort)
		case layers.LayerTypeTLS:
			fmt.Println("TLS Info:", tlsLayer.Handshake)
		case layers.LayerTypeIPv6:
			fmt.Println("IPv6: ", ipv6Layer.SrcIP, "->", ipv6Layer.DstIP)
		}

		if appLayer := packet.ApplicationLayer(); appLayer != nil {
			payload := appLayer.Payload()
			fmt.Println("Payload:", hex.EncodeToString(payload))
		}
	}
}

```
記得要把協定都加進去，我這邊只有加了常見的幾個
輸出結果為：
來源IP/PORT -> 目的IP/PORT
Payload

### 碎碎念
code寫起來比做socket簡單多了。由於gopacket專門用於封包的捕獲及解析，提供的interface更簡單高效。明天就來寫寫看用gopacket寫單純的封包發送。
