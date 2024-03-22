---
author: Sunny Tsai
title: "[Day 7] 天堂雲端 - 安全性由誰負責？"
date: 2023-09-20
math: true
tags: [
    "cloud",
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-天堂雲端之從開始到接管"]
---
昨天有講到雲端攻擊大多在於使用者設定不當，那來談談關於雲端供應商對於資訊安全方面做了哪些？需要使用者注意哪些？

# AWS
## Shared Responsibility Model
![](https://imgur.com/O66zSJJ.png)
[來源](https://aws.amazon.com/tw/compliance/shared-responsibility-model/)

AWS負責的是**雲端本身的安全**
* AWS保護的IaaS、PaaS、SaaS，包含使用的軟硬體、聯網、設施
* 意思是說AWS只負責關於他們提供的服務本身是否有漏洞

使用者負責**雲端內部的安全**
包含
* 數據
* IAM
* 在執行個體上安裝的APP或是公用程式
* 網路配置，如VPC等
* 個體提供的防火牆組態
都是需要使用者自行設定

# Azure

## Shared responsibility in the cloud
![](https://learn.microsoft.com/en-us/azure/security/fundamentals/media/shared-responsibility/shared-responsibility.svg)
[來源](https://learn.microsoft.com/en-us/azure/security/fundamentals/shared-responsibility)

上面明確標示了，使用者需要完全負責
* 數據（Data）
* 端點（Endpoints）
* 帳戶（Account）
* 權限管理（Access management）


# GCP
## Shared responsibilities and shared fate on Google Cloud
![](https://cloud.google.com/static/docs/security/overview/resources/shared_responsibilities.svg)
[來源](https://cloud.google.com/architecture/framework/security/shared-responsibility-shared-fate)

GCP也不例外，使用者同樣要對
* 數據（Data）
* 訪問決策（Access Policy）
* 帳戶（Account）
* 部署（Deployment）

等負責
