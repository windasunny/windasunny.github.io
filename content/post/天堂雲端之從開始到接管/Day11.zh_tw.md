---
author: Sunny Tsai
title: "[Day 11] 天堂雲端 - 曾經的0day AppSync"
date: 2023-09-24
math: true
tags: [
    "cloud",
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-天堂雲端之從開始到接管"]
---
![](https://imgur.com/lVApQ1d.png)

# AWS AppSync
AppSync是一個Serverless服務，可建立GraphQL或是Pub/Sub API，後端資料來源可以為多個來源合併成一個。


### 同場加映 - Pub/Sub API
Pub/Sub (Publish-Subscribe)是一種消息傳送的模式，由發布者(Publish)發送消息，訂閱者(Subscribe)接收消息。發布者(Publish)將消息發送到通道(channel)中，訂閱者(Subscribe)訂閱通道(Channel)。訂閱者(Subscribe)可以同時訂閱多個通道(Channel)。Pub/Sub模式可以實現各種分布式系統及應用。


### 同場加映 - GraphQL
目前API查詢分3種：RESTful API, GraphQL API, GRPC

GraphQL是由FB開發的API查詢語言，提供一種更靈活高效的方式請求或操作數據，解決傳統RESTful API中的查詢問題。
GraphQL可以精準的取得數據，避免了RESTful API提取整個資源的問題，同時也避免地對於資料庫過度IO的問題。

# AWS AppSync未經授權訪問
## 混淆代理人(confused deputy)
一個權限較低的（如攻擊者）使用權限較高的（如AppSync）執行行為


### 實例
![](https://imgur.com/UvYCheD.png)
以儲存庫為S3為例，使用者使用AppSync建立GraphQL API。當GraphQL API被呼叫時，由AppSync執行AWS API呼叫並執行結果。
原本AWS會驗證Amazon Resource Name (ARN)防止AppSync濫用行為，ARN是AWS的獨立辨識代碼。在執行命令時，AppSync建立的GraphQL API 下的serviceRoleArn會包含此帳號的ARN。AppSync會檢查呼叫的ARN是不是同一個AWS帳號，若不是同一個帳號，AppSync會拋出錯誤代碼。
而API的serviceRoleArn參數如果以不同大小寫組合，例如servicerolearn，就能繞過驗證。使不同帳號的人可以建立API到其他AWS帳號的AppSync資料源。意思是說，只要攻擊者能取得目標底下任一角色，最後都可以取得目標公司資源的控制權。

註：Datadog在2022年9月1日通報AWS後，9月6日AWS已修補了AppSync服務的漏洞


### 碎碎念
AppSync是一個很好的0day漏洞實例，未經授權的漏洞不管是在地端還是在雲端都很常見。而且他已經被修補好了，不能測了。
