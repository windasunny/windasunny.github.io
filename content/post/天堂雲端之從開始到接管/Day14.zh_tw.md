---
author: Sunny Tsai
title: "[Day 14] 天堂雲端 - S3 takeover"
date: 2023-09-27
math: true
tags: [
    "cloud",
    "s3",
    "takeover",
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-天堂雲端之從開始到接管"]
---
前天寫道每個S3 bucket object都有唯一的URL訪問路口，格式會長得像這樣：
https://{S3_BUCKET_NAME}.s3.amazonaws.com/{FILE_NAME}

如果在S3 bucket裡面設定了靜態託管服務
![](https://imgur.com/ZitOzXO.png)

設定靜態託管後，會在下方找到S3提供的web endpoint
![](https://imgur.com/bAJy3La.png)
通常會有兩種形式其中之一：
- s3-網站破折號 (-)
    - 區域‐http://{S3_BUCKET_NAME}.s3-website-{Region}.amazonaws.com

- s3-網站點 (.)
    - 區域‐http://{S3_BUCKET_NAME}.s3-website.{Region}.amazonaws.com

這個S3提供的Web endpoint會導倒配置的靜態文件。

# 建立靜態網址
S3 bucket只是託管了靜態檔案，通常不會把這個URL直接提供給公眾用戶，會用DNS Record CNAME將域名指向S3 Web endpoint。

## 自訂網域與S3 Web endpoint
把這個Web endpoint指向自定義的DNS
通常DNS Record會長這樣
```
CNAME  subdomain.example.com  http://S3_BUCKET_WEB_ENDPOINT
```

## 突然有一天，這個S3 bucket不用了
不要問我為什麼，不用了就是不用了，刪掉這個S3 bucket還可以省錢呢。
然後就把S3 bucket刪掉。咦好像少了什麼？

## DNS Record還留有S3 bucket web point紀錄
S3 bucket刪除了，但DNS Record還留有S3 bucket的web endpoint紀錄
但因為DNS Record也不是都有人在上面看，所以說不定還沒有人知道？


-----


# Takeover

## 攻擊者掃描整個DNS Record
使用工具（不管是暴力破解還是非暴力破解還是直接掃描DNS Record）找subdomain，獲取整個DNS Record，看看有沒有無人認領的S3 bucket web endpoint。

如果有找到疑似沒有人在用的S3 bucket web endpoint，訪問web endpoint得到這樣的輸出
```
This XML file does not appear to have any style information associated with it. The document tree is shown below.
<Error>
<Code>NoSuchBucket</Code>
<Message>The specified bucket does not exist</Message>
<BucketName>XXX</BucketName>
<RequestId>XXXXXXXXXXXXX</RequestId>
<HostId>XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX=</HostId>
</Error>
```

代表說攻擊者可以建立一個新的bucket，名稱跟DNS Record上面的一模一樣，完成接管(takeover)，可以用來進行各式各樣的攻擊。
