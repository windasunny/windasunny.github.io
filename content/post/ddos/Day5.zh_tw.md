---
author: Sunny Tsai
title: "[Day 5] 閑的沒事就建立socket - sys"
date: 2023-09-18
math: true
tags: [
    "cloud",
    "ddos",
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-DDOS：閑的沒事就開始DDOS"]
---
在syscall還未被棄用的時後是用syscall建立socket指定來源PORT。現在使用[sys](https://pkg.go.dev/golang.org/x/sys)
（主題是做DDOS，其實有沒有指定PORT不是重點，但這個在後面做raw socket是重點）

# Overview
This repository holds supplemental Go packages for low-level interactions with the operating system.

## Directories
|  packet   | description  |
|  ----  | ----  |
| cpu  | Package cpu implements processor feature detection for various CPU architectures. |
| execabs  | Package execabs is a drop-in replacement for os/exec that requires PATH lookups to find absolute paths. |
| unix | Package unix contains an interface to the low-level operating system primitives. |
| windows | Package windows contains an interface to the low-level operating system primitives. |

做socket用到sys.unix庫

## Socket
```go
package main

import (
	"fmt"
	"log"
	"net"

	"golang.org/x/sys/unix"
)

func main() {
	srcIP := "0.0.0.0"  // if not know, set 0.0.0.0
	srcPort := 23457
	targetIP := "10.211.55.10"
	targetPort := 12345

	// build socket
	fd, err := unix.Socket(unix.AF_INET, unix.SOCK_STREAM, unix.IPPROTO_TCP)
	if err != nil {
		log.Fatal(err)
	}
	defer unix.Close(fd)

	// set source IP and source PORT
	localAddr := &unix.SockaddrInet4{Port: srcPort}
	copy(localAddr.Addr[:], net.ParseIP(srcIP).To4())

	err = unix.Bind(fd, localAddr)
	if err != nil {
		log.Fatal(err)
	}

	// connnect
	remoteAddr := &unix.SockaddrInet4{Port: targetPort}
	copy(remoteAddr.Addr[:], net.ParseIP(targetIP).To4())

	err = unix.Connect(fd, remoteAddr)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println("SYN packet sent successfully!")
}

```
