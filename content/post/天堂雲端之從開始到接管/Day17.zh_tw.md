---
author: Sunny Tsai
title: "[Day 17] 天堂雲端 - AWS S3 Misconfiguration&POC"
date: 2023-09-30
math: true
tags: [
    "cloud",
    "s3",
    "misconfiguration",
	"poc"
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-天堂雲端之從開始到接管"]
---
# Misconfiguration
配置錯誤

## 常見的S3 bucket 配置錯誤

#### 透過ACL公開存取權限
![](https://imgur.com/JRfo5WK.png)

-----


#### 透過策略公開存取權限
![](https://imgur.com/S7IQEBb.png)

-----
注：ACL（Access Control List）為S3物件（Object）提供簡單的權限控制，包括私有、公開讀取及公開讀寫。策略（Policy）適用於整個S3 bucket，允許更複雜靈活的方式定義訪問權限，通常以JSON格式創建自定義權限。



-----




#### 伺服器端無加密檔案
AWS S3預設加密隨時隨地都保持啟用狀態，而且至少採用SSE-S3進行伺服器加密，SSE-S3是免費的。在AWS S3物件上傳前加密，下載前解密。確定保持加密狀態已符合大部分資安規範框架。
![](https://imgur.com/8gj1EX7.png)

-----


#### S3無啟用日誌紀錄
沒有日誌紀錄就無法追蹤事件歷史紀錄，日誌紀錄在所有合規性框架/標準中都是必不可少的。
![](https://imgur.com/Eq7ce0d.png)
-----


# POC
An AWS S3 misconfiguration simple POC (Proof of Concept) for checking public access, read access, and write access.
```
package main

import (
	"fmt"
	"io"
	"net/http"
	"os"
	"strings"

	"github.com/aws/aws-sdk-go/aws"
	"github.com/aws/aws-sdk-go/aws/session"
	"github.com/aws/aws-sdk-go/service/s3"
)

var response http.Response

func main() {
	// s3 bucket name
	bucketName := "YOUR_S3_BUCKET_NAME"

	// Create a Session
	sess := session.Must(session.NewSessionWithOptions(session.Options{
		SharedConfigState: session.SharedConfigEnable,
	}))

	// S3 client
	svc := s3.New(sess)

	// Check if s3 bucket is public
	url := "https://" + bucketName + ".s3.amazonaws.com/"

	// send http request
	response, err := http.Get(url)

	if err != nil {
		fmt.Println("Http request err:", err)
		os.Exit(1)
	}
	defer response.Body.Close()

	// Get status code
	if response.StatusCode == http.StatusOK {
		fmt.Printf("This s3 bucket %s is public!\n", bucketName)
	} else if response.StatusCode == http.StatusNotFound {
		fmt.Printf("This s3 bucket %s is public!\n", url)
	} else {
		fmt.Printf("%s: Unknown status code %d\n", url, response.StatusCode)
	}

	// Check if s3 bucket can list object
	if listObjectsInBucket(response) {
		fmt.Printf("This s3 bucket %s can list object!\n", bucketName)
	} else {
		fmt.Printf("This s3 bucket %s can not list object!\n", bucketName)
		os.Exit(1)
	}

	// Check if s3 bucket can upload object
	if canModifyObject(svc, bucketName, "S3/test.txt") {
		fmt.Printf("This s3 bucket %s can upload object!\n", bucketName)
	} else {
		fmt.Printf("This s3 bucket %s can not upload object!\n", bucketName)
		os.Exit(1)
	}

}

// check s3 bucket have ListObject
func listObjectsInBucket(response *http.Response) bool {
	body, err := io.ReadAll(response.Body)
	if err != nil {
		fmt.Println("ReadAll err:", err)
	}
	if strings.Contains(string(body), "ListBucketResult") {
		return true
	} else {
		return false
	}
}

// check s3 bucket have WriteObject (can upload or modify files)
func canModifyObject(svc *s3.S3, bucketName string, filePath string) bool {

	file, err := os.Open(filePath)
	if err != nil {
		fmt.Println("Open file err:", err)
		return false
	}
	defer file.Close()

	input := &s3.PutObjectInput{
		Body:   file,
		Bucket: aws.String(bucketName),
		Key:    aws.String("test"),
	}

	input.Body = file

	_, err = svc.PutObject(input)
	if err != nil {
		fmt.Println("Failed to upload file.")
		return false
	}

	return true
}

```
