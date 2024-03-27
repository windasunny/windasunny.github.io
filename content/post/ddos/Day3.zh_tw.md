---
author: Sunny Tsai
title: "[Day 3] 閑的沒事就找套件 - Go原生套件'net'"
date: 2023-09-16
math: true
tags: [
    "cloud",
    "ddos",
    "go",
    "net"
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-DDOS：閑的沒事就開始DDOS"]
---
# 用什麼套件建構封包
先來測一下有go有什麼packet interface好用的套件，像其他語言例如c、python可以用socket建構封包。python還有另一個套件庫scapy，專門處理各種交互式數據封包操作庫。

回來go。首先，先建立一個SYN封包。
使用Go原生庫中的[net](https://pkg.go.dev/net)
> **Overview**
> Package net provides a portable interface for network I/O, including TCP/IP, UDP, domain name resolution, and Unix domain sockets.

```go
import (
	"fmt"
	"net"
)

func main() {
    // 設定標的IP與PORT
	targetIP := "10.211.55.10"     // 這是我建的一台測試VM
	targetPort := 12345            // 我在VM上起了flask web服務，web佔用12345 port

	// 建立TCP連接
	conn, err := net.Dial("tcp", fmt.Sprintf("%s:%d", targetIP, targetPort))
	if err != nil {
		fmt.Println("Error connecting:", err)
		return
	}
	defer conn.Close()
}
```
因為SYN是TCP/IP建立連線使用的握手信號，所以建立net.Dial得時候使用的是tcp
看一下[net.Dial源碼註釋](https://github.com/golang/go/blob/master/src/net/dial.go#L351)
```go
// Dial connects to the address on the named network.
//
// Known networks are "tcp", "tcp4" (IPv4-only), "tcp6" (IPv6-only),
// "udp", "udp4" (IPv4-only), "udp6" (IPv6-only), "ip", "ip4"
// (IPv4-only), "ip6" (IPv6-only), "unix", "unixgram" and
// "unixpacket".
//
// For TCP and UDP networks, the address has the form "host:port".
// The host must be a literal IP address, or a host name that can be
// resolved to IP addresses.
// The port must be a literal port number or a service name.
// If the host is a literal IPv6 address it must be enclosed in square
// brackets, as in "[2001:db8::1]:80" or "[fe80::1%zone]:80".
// The zone specifies the scope of the literal IPv6 address as defined
// in RFC 4007.
// The functions JoinHostPort and SplitHostPort manipulate a pair of
// host and port in this form.
// When using TCP, and the host resolves to multiple IP addresses,
// Dial will try each IP address in order until one succeeds.
//
// Examples:
//
//	Dial("tcp", "golang.org:http")
//	Dial("tcp", "192.0.2.1:http")
//	Dial("tcp", "198.51.100.1:80")
//	Dial("udp", "[2001:db8::1]:domain")
//	Dial("udp", "[fe80::1%lo0]:53")
//	Dial("tcp", ":80")
//
// For IP networks, the network must be "ip", "ip4" or "ip6" followed
// by a colon and a literal protocol number or a protocol name, and
// the address has the form "host". The host must be a literal IP
// address or a literal IPv6 address with zone.
// It depends on each operating system how the operating system
// behaves with a non-well known protocol number such as "0" or "255".
//
// Examples:
//
//	Dial("ip4:1", "192.0.2.1")
//	Dial("ip6:ipv6-icmp", "2001:db8::1")
//	Dial("ip6:58", "fe80::1%lo0")
//
// For TCP, UDP and IP networks, if the host is empty or a literal
// unspecified IP address, as in ":80", "0.0.0.0:80" or "[::]:80" for
// TCP and UDP, "", "0.0.0.0" or "::" for IP, the local system is
// assumed.
//
// For Unix networks, the address must be a file system path.
```
可以看出Dial支援多種傳輸類型；也可以使用host代替ip
targetIP也可以使用"example.com"等hostname

## TCP SYN 封包
接下來建立TCP SYN封包
![](https://imgur.com/hUm3gHN.png)
[來源](https://www.geeksforgeeks.org/how-many-bits-are-reserved-for-the-tcp-flags/)
1. Source Port(16 bits/2 bytes): 來源端口號
2. Destination Port(16 bits/2 bytes): 目的端口號
3. Sequence Number(32 bits/4 bytes): 3次握手建立連結後，建立一個隨機初始序列號/起始數據位，隨後按傳輸的字節數遞增。
4. Acknowledgment Number(32 bits/4 bytes): 預測下一個TCP的序列號。
5. Header length field(4 bits): 標示TCP Header大小。
6. Reserved bits(6 bits): 設置為0，保留供以後使用。
7. Flag bits(6 bits): 包括SYN、ACK、RST、FIN、URG和PSH等標籤。
8. Window Size(16 bits/2 bytes): 指定接收端的窗口大小，用於流量控制。
9. Checksum(16 bits/2 bytes): 校驗TCP Header與Data的完整性。
10. Urgent Pointer(16 bits/2 bytes): 只有設定URG標示才會使用。
11. Options/optional data(0-40 Bytes):包含可選訊息。可不寫，所以TCP Header總大小在20~60

## 建立TCP Header
```
seqNum := uint32(123456)

flags := uint16(0x02) // TCP SYN標籤

// TCP Header
tcpHeader := make([]byte, 20)
tcpHeader[0] = byte(srcPort >> 8)
tcpHeader[1] = byte(srcPort)
tcpHeader[2] = byte(dstPort >> 8)
tcpHeader[3] = byte(dstPort)
tcpHeader[4] = byte(seqNum >> 24)
tcpHeader[5] = byte(seqNum >> 16)
tcpHeader[6] = byte(seqNum >> 8)
tcpHeader[7] = byte(seqNum)
tcpHeader[13] = byte(flags >> 8)
tcpHeader[12] = byte(flags)
```

好像缺了什麼？明天繼續
