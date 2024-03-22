---
author: Sunny Tsai
title: "[Day 5] 天堂雲端 - 雲端架構"
date: 2023-09-18
math: true
tags: [
    "cloud",
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-天堂雲端之從開始到接管"]
---
# 雲端架構

AWS版本的分布式雲端架構
![](https://imgur.com/Vmxwmgr.png)
[來源](https://www.youtube.com/watch?v=_WCjqokBGbg)
搜尋aws分布式雲端架構image第一個就出現這張圖，如有不妥我再刪掉自己畫，因為我不太擅長畫這個~~我畫的都很醜~~，所以一般我都是在網路上找好看的圖。

雲端架構都會比較偏向分布式架構，畢竟都上雲了，就不會把db, cache, web app都塞進一個EC2裡面，這樣效益不高成本也高，所以使用者會偏向使用雲端供應商提供的服務。
先來講講架構

### Web APP
* EC2
    以AWS為例，AWS提供可擴展的IaaS-EC2，供使用者快速開發及部署。類似在Vmware起一台虛擬機
多台虛擬機同時運行同一個網站服務
* ELB（Elastic Load Balancing）
    可以使用Load Balance(ELB)負載平衡，將流量分發到多台EC2。
* ASG（Auto Scaling）
    如果有即時監測Web APP效能的需求，AWS提供ASG服務自動增加受限資源的容量，維持高品質服務

### Cache
* ElasticCache
    AWS提供ElasticCache支援Memcached和Redis快取引擎，依照使用需求選擇適合的快取引擎，全受管 Redis 和 MemCached 相容服務。

### 資料庫服務
* AWS RDS
    AWS提供的託管關聯式資料庫服務，具有自動備份跟修復跟自動擴展與高可用性等功能。多種常用的引擎支持，例如MySQL、PostgreSQL、MSSQL等。

* AWS Athena
AWS提供的無服務器互動式分析服務，以流量計費。建立在Trino跟Presto引擎和Apache Spark框架上，查詢服務支援包括ORC、JSON、CSV、Parquet等資料格式，可以將數據作為靜態檔案存放在S3中。

-----
從這樣看起來，雲端架構是不是很簡單呢？連接相應的IaaS跟PaaS並組合成想要的架構。諸如無伺服器架構(Serverless framework)在開發中可以解決很多問題；諸如S3等的Object Storage在數據處理等方面更優於Block Storage。使用者可以使用相應的服務還不需要大量的建置與維護的成本，這也是雲端越來越熱門的原因。
