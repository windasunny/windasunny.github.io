---
author: Sunny Tsai
title: "[Day 24] 天堂雲端 - 取得EC2快照(snapshot)"
date: 2023-10-07
math: true
tags: [
    "aws",
    "ec2",
    "snapshot"
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-天堂雲端之從開始到接管"]
---
# AWS EBS
[AWS Elastic Block Store(EBS)](https://aws.amazon.com/tw/ebs/)是AWS提供的可擴展的高效能區塊儲存服務。通常EC2 instance上面的數據（包括操作系統、應用程序數據等）通常儲存在EBS上。而EC2的快照(snapshot)是EC2 instance的狀態的定期副本，通常是基於EBS volumes的，快照捕捉了EBS volumes上的數據的狀態。

注：EC2實例本身具有暫時性存儲，稱為實例存儲（Instance Store），但該存儲通常用於臨時數據，並且不具備持久性。

注：EBS是區塊儲存服務(Block Storage Service)，跟S3(Object Storage Service)不同。EBS主要儲存數據，操作系統，應用程序等；S3用於儲存和檢索任何類型的數據，並提供高度可擴展的存儲解決方案。

# Public & Private
EBS快照分為公開跟私有。當快照設定成公有時，任何人都可以存取這張快照。
![](https://imgur.com/ttM5T1X.png)

# 取得Account下的快照
取得帳號下的所有快照
```shell
>  aws ec2 describe-snapshots --owner-ids {ACCOUNT_ID}
```

Output:
```json
{
    "Snapshots": [
        {
            "Description": "20231007-snapshot",
            "Encrypted": false,    # 沒有設定加密
            "OwnerId": "{ACCOUNT_ID}",
                "Progress": "100%",    # 100%代表快照建立完成
            "SnapshotId": "snap-XXXXXXX",
            "StartTime": "{Date}",
            "State": "completed",
            "VolumeId": "vol-XXXXXXXXXX",
            "VolumeSize": X,
            "StorageTier": "standard"
        }
    ]
}
```

取得帳號下的公開快照
```shell
>  aws ec2 describe-snapshots --restorable-by-user-ids all --owner-ids {ACCOUNT_ID}
```


# 取得所有公開快照
```shell
> aws ec2 describe-snapshots --restorable-by-user-ids all
```

Output
```json
{
    "Snapshots": [
        {
            "Description": "hvm-ssd/ubuntu-trusty-amd64-server-20170202.1",
            "Encrypted": false,
            "OwnerId": "099720109477",
            "Progress": "100%",
            "SnapshotId": "snap-046281ab24d756c50",
            "StartTime": "2017-02-02T23:57:19+00:00",
            "State": "completed",
            "VolumeId": "vol-033ca269aeedb3521",
            "VolumeSize": 8,
            "OwnerAlias": "amazon",
            "StorageTier": "standard"
        },
        {
            "Description": "hvm/ubuntu-trusty-amd64-server-20170202.1",
            "Encrypted": false,
            "OwnerId": "099720109477",
            "Progress": "100%",
            "SnapshotId": "snap-00de7e12fd08987c4",
            "StartTime": "2017-02-02T23:44:49+00:00",
            "State": "completed",
            "VolumeId": "vol-0bf7e5b6270473ac6",
            "VolumeSize": 8,
            "OwnerAlias": "amazon",
            "StorageTier": "standard"
        },
....
```
