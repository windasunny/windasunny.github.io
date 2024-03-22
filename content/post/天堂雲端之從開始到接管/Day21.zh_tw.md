---
author: Sunny Tsai
title: "[Day 21] 天堂雲端 - AWS Metadata Service (IMDS)"
date: 2023-10-04
math: true
tags: [
    "aws",
    "ec2",
    "imds"
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-天堂雲端之從開始到接管"]
---
在接下去寫EC2 POC之前，先來解釋AWS IMDS是啥
# AWS Metadata Service (IMDS)
[IMDS](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html)是有關實例的元數據（Metadata），可用於設定或管理正在執行的實例（例如EC2 instance）。這些元數據包含有關實例的重要資訊，例如實例的ID、IAM角色、IP地址、實例類型、安全組等等。IMDS是一個RESTful API，通常在實例內部運行，用於提供對這些數據的安全訪問。

注：IMDS僅在實例內部可用，不能通過網絡從外部訪問。

# IMDS RESTful API
### 查詢EC2 instance ID
```shell
> curl http://169.254.169.254/latest/meta-data/instance-id
```

### IAM Role
IAM role關聯的名稱
```shell
> curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
```
如果沒有關聯的role則返回404


檢索credential
```shell
> curl http://169.254.169.254/latest/meta-data/iam/security-credentials/{role_name}
```

### 安全組
```shell
> curl http://169.254.169.254/latest/meta-data/security-groups
```

### IP Address
私有IP地址
```shell
> curl http://169.254.169.254/latest/meta-data/local-ipv4
```

公有IP地址(如果有)
```shell
> http://169.254.169.254/latest/meta-data/public-ipv4
```

主機名（包括公有DNS和私有DNS）
```shell
> http://169.254.169.254/latest/meta-data/public-hostname
```

```shell
> http://169.254.169.254/latest/meta-data/local-hostname
```
