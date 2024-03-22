---
author: Sunny Tsai
title: "[Day 12] 天堂雲端 - S3之為什麼要打開公有存取權？"
date: 2023-09-25
math: true
tags: [
    "cloud",
    "S3"
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-天堂雲端之從開始到接管"]
---
# AWS S3
是什麼？

[AWS Simple Storage Service (S3)](https://aws.amazon.com/tw/s3/)是一個object storage service，主要用於在雲端環境中的存儲與任何數量的數據檢索。高可用性、高耐久性、及整合AWS生態系統使得此項服務廣為人知。

也就是說，文件、圖檔、靜態庫（javascript庫等）、用戶上傳內容、靜態網站、資料庫資料、日誌、映像檔都可以放在S3裡面，而S3也提供災難復原及備份還原等功能。


# 所以為什麼會設定公開權限？
AWS S3在建立一個新的S3 bucket的時候，AWS都會提醒使用者，盡量不要使用**公開存取權限**
![](https://imgur.com/DpKNX8W.png) <- 新建bucket的時候，會有AWS的友善提醒

但為什麼還是很多人會打開公開存取權限呢？

### 架設靜態網站
S3有提供架設靜態網站的功能，只要上傳HTML靜態檔到S3，就可以架設靜態網站。
不過也需要將S3設定成**公開讀取權限**，畢竟HTML靜態檔要設定成公眾都可以讀取，不然客戶端要怎麼讀取到HTML檔案的內容呢。
一但將S3設定成公開讀取權限，就要小心有沒有上傳不該上傳的檔案，例如.git、.env、config.yaml等。

#### 如果沒有公開存取權限
設定S3，在這個S3 bucket上傳一個靜態檔案，並設定靜態檔案託管，會得到一個這個靜態檔案的URI跟URL，URL格式通常是這樣的：https://{YOUR_S3_BUCKET_NAME}.s3.amazonaws.com/{FILE_NAME}.html
連接此URL，如果沒有公開權限，會得到以下錯誤訊息
```
<Error>
    <Code>AccessDenied</Code>
    <Message>Access Denied</Message>
    <RequestId>XXXXXXXXXXXX</RequestId>
    <HostId>XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX=</HostId>
</Error>
```

#### 為了使公眾能觀看靜態檔案內容，只好打開公開存取權限
打開了就可以正常訪問這個放在S3的靜態檔案了。


-----


### 日誌、備份檔、映像檔、靜態庫
S3一個常見的用途，就是存放一些主機日誌、應用程式備份檔、部署容器的映像檔、程式語言靜態庫等，因為S3高耐久性與高可用性很適合長期存放一些檔案內容。但由於為了一時方便，很多時候讀取靜態庫跟映像檔會設定成公開存取權限
；傳送日誌及備份檔的時候也會因為方便，將bucket設定成公開讀取好方便傳送。這就會造成潛在的洩漏風險。

-----

### 臨時修改
開發團隊常常遇到一種情境，上午開會說緊急支援，下午就得上production環境，這種時候當天都快過不下去了就會發生兩個潛在的風險：
1. 權限配置錯誤 - 不管了全開再說
2. 開發環境變成正式環境 - 不管了先上再說

以上兩種，我都親身體驗過，都快爆炸了還管什麼安不安全。但事後並不會回去補...

-----


### 隱晦式安全（Security through obscurity）
> 大家都看不見所以很安全

[Security through obscurity](https://zh.wikipedia.org/zh-tw/%E9%9A%B1%E6%99%A6%E5%BC%8F%E5%AE%89%E5%85%A8)指的就是攻擊者猜不到目標URL，所以使用者相信，就算S3設定是公開存取也很安全。

實際上是，攻擊者也猜不到subdomain呢，但其實爆破subdomain的工具有很多；同理，S3 bucket name跟gmail還有domain name一樣是有唯一性的，所以並不會發生兩個名字一樣的bucket衝突到的情況，而因為這樣使用者以為別人不會知道我的s3 bucket名稱是什麼，所以就理所當然的開啟公開權限。但現在攻擊者會盡可能地猜出名稱，包含常見的字串型態或是使用AI工具猜測常見的名稱（就跟猜密碼一樣）。所以說S3 bucket還是很有可能被爆破的，就連Github上面有很多專門暴力搜尋S3 bucket名稱的工具。
