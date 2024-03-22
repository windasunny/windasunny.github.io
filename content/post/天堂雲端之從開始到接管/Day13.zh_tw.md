---
author: Sunny Tsai
title: "[Day 13] 天堂雲端 - S3 Bucket Enumeration"
date: 2023-09-26
math: true
tags: [
    "cloud",
    "S3",
    "Enumeration"
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-天堂雲端之從開始到接管"]
---
S3 Bucket是一個Object Storage Service，當物件(Object)存入Bucket的時候，這個物件(Object)會包含以下三個部分
* Key
    * 檔案名稱（唯一值）
* Value
    * 物件(Object)內容
* Metadata
    * 紀錄物件(Object)相關訊息的資料(ex 上傳時間、修改時間等)

其中，Key代表著獲取檔案URL一個重要關鍵字。上傳後的物件(object)若是可以進行公開存取則訪問的URL會是長這樣：https://{S3_BUCKET_NAME}.s3.amazonaws.com/{KEY}


-----

## Bucket Object訪問

### 公開存取檔案
如果是設定成可以公眾讀取的
![](https://imgur.com/Db9zEVE.png)
開啟物件(object)URL可以正常訪問到檔案。

### 非公開存取檔案
如果物件(object)沒有設定公眾權限
![](https://imgur.com/S7zinH5.png)
訪問URL的時候會出現以下錯誤
```
This XML file does not appear to have any style information associated with it. The document tree is shown below.
<Error>
<Code>AccessDenied</Code>
<Message>Access Denied</Message>
<RequestId>XXXXXXX</RequestId>
<HostId>XXXXX=</HostId>
</Error>
```


需要有token、credential跟signature才可以登入
```
https://{S3_BUCKET_NAME}.s3.{REGION}.amazonaws.com/{KEY}?response-content-disposition=inline&X-Amz-Security-Token=XXX&X-Amz-Algorithm=XXXXXXXX&X-Amz-Date=DATE&X-Amz-SignedHeaders=host&X-Amz-Expires=300&X-Amz-Credential=XXXXXus-east-1%2Fs3%2Faws4_request&X-Amz-Signature=XXX
```

### 不存在此檔案
如果輸入一個不存在此檔案的URL，輸出如下
```
This XML file does not appear to have any style information associated with it. The document tree is shown below.
<Error>
<Code>AccessDenied</Code>
<Message>Access Denied</Message>
<RequestId>XXXXXX</RequestId>
<HostId>XXXXXXXXX=</HostId>
</Error>
```

昨天提到**隱晦式安全（Security through obscurity）**。只要S3 Object配置是公開存取，只有在攻擊者知道S3 bucket URL可訪問。那理論上來說，只要key值越複雜，在某種程度上這個檔案是安全的，不會被攻擊者搜索到；等同於密碼設置越複雜、就越不可能被爆破。但還是不建議這樣做，科技日新月異，保護密碼都進階到2FA、MFA了。爆破S3 bucket下的公開檔案也不是不可能。


-----

## S3 Bucket Enumeration

### 訪問已存在的S3 bucket，可公開列出所有Object
輸出如下：
```
<ListBucketResult xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
<Name>XXXXXXXX</Name>
<Prefix/>
<Marker/>
<MaxKeys>1000</MaxKeys>
<IsTruncated>false</IsTruncated>
<Contents>
<Key>XXXXXXXXX</Key>
<LastModified>XXXXX</LastModified>
<ETag>"XXXXXXXXX"</ETag>
<Size>...</Size>
<StorageClass>STANDARD</StorageClass>
</Contents>
<Contents>
<Key>XXXXXXXXXX</Key>
<LastModified>XXXXX</LastModified>
<ETag>"XXXXXXXXX"</ETag>
<Size>...</Size>
<StorageClass>STANDARD</StorageClass>
</Contents>
</ListBucketResult>
```

### 訪問已存在的S3 bucket，但沒有公眾存取權
輸出如下：
```
This XML file does not appear to have any style information associated with it. The document tree is shown below.
<Error>
<Code>AccessDenied</Code>
<Message>Access Denied</Message>
<RequestId>XXXXXXXXX</RequestId>
<HostId>XXXXXXXXx=</HostId>
</Error>
```

### 訪問不存在的S3 bucket
輸出如下
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

依照Response的不同，依據字典檔進行S3 bucket爆破，爆破S3 bucket後，爆破key也就更可行了。
