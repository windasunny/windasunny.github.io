---
author: Sunny Tsai
title: "[Day 20] 天堂雲端 - EC2之建立靶機"
date: 2023-10-03
math: true
tags: [
    "aws",
    "ec2"
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-天堂雲端之從開始到接管"]
---
# AWS EC2
Amazon Elastic Compute Cloud (AWS EC2)是AWS提供的IaaS服務，等同於在AWS上開啟虛擬機。


# 建立EC2
首先，先建立一台EC2靶機。記得安裝openssh
![](https://imgur.com/w9qvCmN.png)

註：因為是當成靶機的關係，我是用SSH的方式連線雲端主機，如果是production環境的話可以換成System Manager快速設定ssm。~~因為吃流量計費所以我就先不用了~~

快速建立一個Flask Web app，放到這台EC2靶機中
```
from flask import Flask

app = Flask(__name__)


@app.route("/")
def index():
    return "Hello World!"


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8000)

```

# 編輯安全組規則
把flask佔用的PORT加到安全組規則中，不然外網吃不到。


# 測試
外網連接公有 IPv4 DNS，通常長這樣 - ec2-{IP}.compute-1.amazonaws.com，確認可以看到Hello World就可以接著後續的測試。
