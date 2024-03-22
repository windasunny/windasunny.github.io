---
author: Sunny Tsai
title: "[Day 18] 天堂雲端 - S3 takeover POC"
date: 2023-10-01
math: true
tags: [
    "cloud",
    "S3",
    "takeover",
    "poc"
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-天堂雲端之從開始到接管"]
---
# S3 takeover POC
第一步：確認target ip
第二步：掃描target ip 下符合條件的子域
第三步：確認這個S3 bucket web endpoint是不存在的
第四步：takeover it!

### 第一步
確認標的：example.com  # 我沒有域名，很窮QQ

-----


### 第二部
找出所有子域

#### 1. 使用dig查詢DNS record
```shell
# 列出 authority section
> dig example.com

...
;; AUTHORITY SECTION:
example.com.		71232	IN	NS	a.iana-servers.net.
example.com.		71232	IN	NS	b.iana-servers.net.
...
```


```shell
# 找出所有subdomain
> dig @a.iana-servers.net example.com axfr
```

如果有找到subdomain會輸出類似以下數據
```shell
mail.example.com.	1800	IN	A	1.2.3.4
www.example.com.	1800	IN	A	5.6.7.8
```

查找CNAME指向子域名的s3 web endpoint
```shell
> dig sub.example.com
```

#### 2. 使用Shodan或Fofa等搜尋banner
找出banner有以下關鍵字
```json
<Code>NoSuchBucket</Code>
```

-----

### 第三步：確認這個S3 bucket不存在
Response:
```json
<Error>
<Code>NoSuchBucket</Code>
<Message>The specified bucket does not exist</Message>
<BucketName>...</BucketName>
</Error>
```


-----


### 第四部：takeover！
創建一個名字一樣地域一樣的s3 bucket，再次訪問subdomain，如果成功印出上傳的網頁，算是接管成功！
