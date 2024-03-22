---
author: Sunny Tsai
title: "[Day 10] 天堂雲端 - 雲端曾經出現過的漏洞"
date: 2023-09-23
math: true
tags: [
    "cloud",
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-天堂雲端之從開始到接管"]
---
雲端技術雖然帶來了便利和效能，但並非銅牆鐵壁。它不僅有潛在的硬體問題，也可能因供應商或使用者的不當操作而產生安全漏洞。

# 硬體造成的漏洞（掃到颱風尾）
雲供應商使用的硬體造成的漏洞，影響到雲端服務。
## AWS
- [CVE-2023-20569](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-20569)
    - AMD CPUs

---

## Azure

- [CVE-2022-21123](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-21123)
    - Intel(R) Processors

---

## GCP

- [CVE-2023-20593](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-20593)
    - Zen 2 CPUs

-----

# 雲端供應商提供的服務漏洞
服務漏洞
## AWS
- AppSync
    - 混淆代理人（confused deputy）：一個權限較低的（如攻擊者）使用權限較高的（如AppSync）執行行為
- WorkSpaces
    - RCE 漏洞

## Azure

- Azure API Management、Azure Functions、Azure Machine Learning 和 Azure Digital Twins服務存有漏洞
    - 容易受到SSRF攻擊
- Power Platform
    - 自定義程式碼竊取敏感資訊

## GCP

- Cloud Asset Inventory
    - 資產密鑰竊取漏洞

# 使用者造成的漏洞
由使用者配置不當、弱憑證弱密碼、第三方元件漏洞等造成。

- 未經授權訪問
    - 當使用者未經授權地訪問資源時，可能會產生潛在的安全風險，這通常源於不當的配置設定。
- takeover
    - 錯誤的配置容易被攻擊者接管
- 錯誤的配置
* 由於疏忽或缺乏專業知識，使用者有時會錯誤地配置安全設置，依照前幾天的文章[天堂雲端 - 安全性由誰負責？](https://ithelp.ithome.com.tw/articles/10315977)，使用者必須負責相應區塊的安全性。
