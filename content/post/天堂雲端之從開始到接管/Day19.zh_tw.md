---
author: Sunny Tsai
title: "[Day 19] 天堂雲端 - AWS Elastic Beanstalk"
date: 2023-10-02
math: true
tags: [
    "aws",
    "ec2",
    "elastic beanstalk"
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-天堂雲端之從開始到接管"]
---
# AWS Elastic Beanstalk
[AWS Elastic Beanstalk](https://aws.amazon.com/tw/elasticbeanstalk/)是AWS提供一個全受管的PaaS服務，用於建構、部署Web應用程式，只要使用者上傳code，Elastic Beanstalk會自動部署、負載平衡、自動調整等功能。~~看起來就像在薛不懂infra的人。~~

## 建立新環境
![](https://imgur.com/3JwQhra.png)
建立好後去EC2看會看到beanstalk幫你建ㄧ台EC2機器。雖然本身beanstalk不收費，但這個服務就是建好infra的工作。

## 常見的Elastic Beanstalk漏洞

### 1. Takeover
beanstalk web endpoint：http://[name].-[region].elasticbeanstalk.com
跟S3 takeover一樣，如果找到有DNS record存在一個被砍掉的beanstalk域名，攻擊者接管這個域名


### 2. SSRF/XSS/SQL Injection/
部署的程式碼本身含有SSRF、XSS、SQL Injection等漏洞，攻擊者擷取到Metadata Service、IAM等資料。
