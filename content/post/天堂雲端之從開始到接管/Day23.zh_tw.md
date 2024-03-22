---
author: Sunny Tsai
title: "[Day 23] 天堂雲端 - AWS Account ID Enumeration"
date: 2023-10-06
math: true
tags: [
    "aws",
    "ec2",
    "enumeration",
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-天堂雲端之從開始到接管"]
---
# Account ID & ARN
昨天提到可以用SSRF等技術取得API KEY等資訊。
如果說已取得API KEY，可以用AWS CLI提供的get-access-key-info取得Account ID
```shell
aws sts get-access-key-info --access-key-id=XXXXXXXXXXXX
{
    "Account": "ACCOUNT_ID"
}
```

# 目標
在不存取任何AWS帳戶的情況下，要怎麼枚舉有效的AWS帳戶名稱？

## Amazon Resource Name(ARN)
Amazon Resource Name(ARN)結構如下：
```shell
arn:partition:service:region:account-id:resource-type/resource
```
用於唯一識別和管理資源的標識符。每個資源都有一個，包括IAM使用者和IAM組。

## 使用ARN枚舉AWS Account ID和ARN

### 第一種方法：UpdateAssumeRolePolicy
先使用ARN枚舉帳戶名稱
UpdateAssumeRolePolicy是AWS提供的一個更新IAM角色的AssumeRole的策略。

#### 創建不相關的AWS帳戶
建立一個不相關的AWS帳戶，接著建立一個角色。

#### 枚舉
在角色的**編輯信任政策**中加入以下內容
```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Effect": "Deny",
			"Principal": {
				"AWS": "arn:aws:iam::ACCOUNT_ID:user/USER_NAME"
			},
			"Action": "sts:AssumeRole"
		}
	]
}
```
如果正確枚舉到USER_NAME，網頁會回傳`信任政策已更新。`等代表執行成功的回應。如果是錯誤的USER_NAME，網頁會回傳`無法更新信任政策。Invalid principal in policy: "AWS":"arn:aws:iam::ACCOUNT_ID:user/XXXXX"`等資訊。

如果大量枚舉需要做成自動化會類似以下的code
```go
package main

import (
	"fmt"

	"github.com/aws/aws-sdk-go/aws"
	"github.com/aws/aws-sdk-go/aws/credentials"
	"github.com/aws/aws-sdk-go/aws/session"
	"github.com/aws/aws-sdk-go/service/iam"
)

func main() {
	sess, err := session.NewSession(&aws.Config{
		Region:      aws.String("us-east-1"),
		Credentials: credentials.NewStaticCredentials("XXXXx", "XXXXXXXXXX", ""),
	})

	if err != nil {
		fmt.Println("Error creating session:", err)
		return
	}

	svc := iam.New(sess)

	roleName := "Role Name"

	newPolicyDocument := `{
		"Version": "2012-10-17",
		"Statement": [
			{
				"Effect": "Deny",
				"Principal": {
					"AWS": "arn:aws:iam::ACCOUNT_ID:user/NAME"
				},
				"Action": "sts:AssumeRole"
			}
		]
	}`

	_, err = svc.UpdateAssumeRolePolicy(&iam.UpdateAssumeRolePolicyInput{
		RoleName:       &roleName,
		PolicyDocument: &newPolicyDocument,
	})

	if err != nil {
		fmt.Println("Error updating AssumeRole policy:", err)
		return
	}

	fmt.Printf("AssumeRole policy for role %s updated successfully.\n", roleName)
}

```

### 第二種方法：在自己環境的AWS服務上指派政策
與UpdateAssumeRolePolicy一樣。建立一個新的S3 bucket，在政策下測試是否可以正確地新增policy。
