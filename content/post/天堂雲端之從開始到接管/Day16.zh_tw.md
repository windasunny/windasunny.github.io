---
author: Sunny Tsai
title: "[Day 16] 天堂雲端 - AWS S3 Enumerate POC"
date: 2023-09-29
math: true
tags: [
    "cloud",
    "s3",
    "enumerate",
	"poc"
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-天堂雲端之從開始到接管"]
---
# Enumerate
## S3 Web endpoint

S3 web endpoint組成：https://{BUCKET_NAME}.s3.amazonaws.com/
enumerate（枚舉）所有可能的名詞組合建立出來的S3 bucket是否是公開存取的？
```
// set wordlist to enumerate all s3-buckets

package main

import (
	"bufio"
	"fmt"
	"net/http"
	"os"
)

func main() {
	// open wordlist file
	file, err := os.Open("S3/wordlist.txt")   # File here!
	if err != nil {
		fmt.Println("Failed to open wordlist file:", err)
		return
	}
	defer file.Close()

	scanner := bufio.NewScanner(file)
	for scanner.Scan() {
		// get wordlist
		word := scanner.Text()

		// build s3 web endpoint
		url := "https://" + word + ".s3.amazonaws.com/"

		// send http request
		response, err := http.Get(url)
		if err != nil {
			fmt.Println("Http request err:", err)
			return
		}
		defer response.Body.Close()

		// Get status code
		if response.StatusCode == http.StatusOK {
			fmt.Printf("%s: S3 bucket is exist!\n", url)
		} else if response.StatusCode == http.StatusNotFound {
			fmt.Printf("%s: S3 bucket is not exist\n", url)
		} else {
			fmt.Printf("%s: Unknown status code %d\n", url, response.StatusCode)
		}
	}

	if err := scanner.Err(); err != nil {
		fmt.Println("scanner err:", err)
	}
}
```
