---
author: Sunny Tsai
title: "[Day 29] 天堂雲端 - Systems Manager"
date: 2023-10-12
math: true
tags: [
    "aws",
    "ec2",
    "ssm"
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-天堂雲端之從開始到接管"]
---
# SSM
[AWS Systems Manager (SSM)](https://docs.aws.amazon.com/zh_tw/systems-manager/latest/userguide/what-is-systems-manager.html) 是一個AWS服務，提供一個集中式的解決方案，用於管理和遠程操作AWS環境中的多個instance。

## 主要功能
* 遠程執行命令
    * 使用SSM控制EC2 instance，而無需進行SSH或RDP連接。這有助於提高安全性和簡化操作。
* 系統監控
    * SSM提供了關於EC2 instance的性能和運行狀況的詳細資訊，可以監控指標、設定警報並自動回應事件。
* 安全性
    * SSM 使用IAM來控制對instance的訪問權限，可以更精準的控制誰可以進行什麼操作。

## 設定Session Manager
在EC2上設定好Session Manager後即可使用(需要注意role有沒有設定到)
![](https://imgur.com/Y6rXkF1.png)
Session Manager是SSM的一部分，允許使用AWS控制台或CLI直接從執行主機啟動會話。

## 訪問
Remote Access:
```shell
> aws ssm start-session --target {EC2_INSTANCE_ID}
```

會出現現在是由哪個account遠程訪問EC2 instance
```shell
Starting session with SessionId: XXXXXXXXXX
```
