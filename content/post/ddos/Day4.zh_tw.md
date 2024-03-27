---
author: Sunny Tsai
title: "[Day 4] 閑的沒事 - 來源端口（踩坑）"
date: 2023-09-17
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
# 發送封包
昨天建立一個TCP SYN packet
TCP Header建立好了，TCP Data暫時不建立。缺了什麼？缺了srcPort...

使用net建立packet的時候，go通常不會幫我們選擇未使用的來源port。它期望我們明確指定來源port。因此，我們要自己設定一個未使用的來源port。

機器要丟一個封包出去，首先就是在1~65535的port隨機找一個還沒被服務站用的port

## 來源PORT
建立隨機數字1~65535，當作來源地址發送的PORT
因為PORT被佔用就不能發送封包，所以先把容易被佔用的PORT剔除，這邊列出nmap top 100作為容易佔用的PORT
```go
func randomPort() int {
	// Nmap top ports
	nmapTopPorts := []int{
		21, 22, 23, 25, 53, 80, 110, 111, 135, 139,
		143, 443, 445, 993, 995, 1723, 3306, 3389, 5900, 8080,
		8443, 8888, 9090, 1433, 2000, 5432,
	}

	rand := rand.New(rand.NewSource(time.Now().UnixNano()))

	var srcPort int
	for {
		srcPort = rand.Intn(65535) + 1
		if !slices.Contains(nmapTopPorts, srcPort) && isPortAvailable(uint16(srcPort)) {
			break
		}
	}

	fmt.Println("Selected source port:", srcPort)

	return srcPort
}
```

測試PORT除了在nmap top 100之外，是否有被佔用
```go
func isPortAvailable(port uint16) bool {
	addr := fmt.Sprintf(":%d", port)
	listener, err := net.Listen("tcp", addr)
	if err != nil {
		// 端口已被占用
		return false
	}
	listener.Close()
	return true
}

```


# 接收封包
發送封包
```
$ go run
> Selected source port: 35413
> SYN packet sent successfully
```


在靶機上執行tcpdump攔截封包
注：tcpdump預設情況下是給root使用的
```shell
sudo tcpdump -i eth0 host 10.211.55.10 and port 12345 -v
```

攔截封包
```shell
16:13:20.913177 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto TCP (6), length 64)
    10.211.55.2.64027 > ubuntu-linux-20-04-desktop.12345: Flags [SEW], cksum 0x4d64 (correct), seq 947175497, win 65535, options [mss 1460,nop,wscale 6,nop,nop,TS val 393620125 ecr 0,sackOK,eol], length 0
16:13:20.913578 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto TCP (6), length 60)
...
```

先不提每一筆封包的詳細內容，看第二行10.211.55.2.64027 > ubuntu-linux-20-04-desktop.12345...
但是terminal輸出表示tcpheader裡面source port應該是35413，跟攔截封包上面的64027對不上

## 為什麼
因為net.Dial建立時已經完成TCP的三次握手連接，在```conn.Write(tcpHeader)```只是重寫一個tcp header到已連接連線中，代表只是在TCP連接的資料流中加入一些byte資料，而這些資料不會被當作header處理，只會被當作一般資料。


但是tcp header還是要寫，明天來寫。
