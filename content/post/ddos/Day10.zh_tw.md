---
author: Sunny Tsai
title: "[Day 10] 閑的沒事就寫benchmark"
date: 2023-09-23
math: true
tags: [
    "cloud",
    "benchmark",
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-DDOS：閑的沒事就開始DDOS"]
---
# 什麼是benchmark
基準測試(benchmark)是一種程式碼的測試方法，在特定時間或特定操作下或功能在一定條件下的測試速度，通常以次數與時間做基本。


# Benchmark
```go
package main

import (
	"runtime"
	"testing"

	"github.com/windasunny/ddos/packet"
)

// Benchmark for the first udp code (using gopacket)
func BenchmarkGopacket(b *testing.B) {
	b.ReportAllocs()

	for i := 0; i < b.N; i++ {
		packet.Udp()
	}

	var memStats runtime.MemStats
	runtime.ReadMemStats(&memStats)
}

// Benchmark for the second udp code (using unix)
func BenchmarkUnixUDP(b *testing.B) {
	b.ReportAllocs()

	for i := 0; i < b.N; i++ {
		packet.SysUdp()
	}

	var memStats runtime.MemStats
	runtime.ReadMemStats(&memStats)
}
```

# 輸出
```shell
# functest                 Number of iterations    Runtime/Iteration    Memory/Iteration    Memory allocations/Iteration
BenchmarkGopacket-8         7158                   167018 ns/op           28496 B/op        279 allocs/op
BenchmarkUnixUDP-8         15555                   150377 ns/op              96 B/op          4 allocs/op
PASS
```

# 結論
unix-udp顯然比gopacket-udp內存消耗更低、執行時間更快。接下來封包都會用unix建立封包。
