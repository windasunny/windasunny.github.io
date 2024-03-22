---
author: Sunny Tsai
title: "[Day 28] 天堂雲端 - EC2權限升級"
date: 2023-10-11
math: true
tags: [
    "aws",
    "ec2",
    "iam"
]
categories: [
    "2023鐵人賽"
]
series: ["2023鐵人賽-天堂雲端之從開始到接管"]
---
# 權限升級
在AWS雲端環境中，權限升級是透過IAM完成升級的，攻擊者需要找到哪個帳戶可以做到這件事。

## Shadow admin
[這裡](https://docs.cyberark.com/CEM/Latest/en/Content/CloudAdmin/kd_shadow-admin-per-platform.htm)有詳細列出各大雲端服務的shadow admin。這裡先列出幾個特別的：

### iam-PassRole & ec2-AssociateIamInstanceProfile
[AWS官方文件](https://docs.aws.amazon.com/zh_tw/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)
iam-PassRole用於控制哪些身份可以將IAM角色分配給其他AWS服務或資源。例如：具有該權限才可以將角色分配給EC2 intance。
ec2-AssociateIamInstanceProfile權限用於管理EC2 instance與IAM Instance Profile的關聯，確保EC2 instance有獲取所需的權限。
如果攻擊者可以存去擁有這兩個權限的帳戶，可以將特權instance profile附加在感染的EC2 instance上。

### iam-PassRole & iam-CreateInstanceProfile & iam-AddRoleToInstanceProfile
[AWS官方文件](https://docs.aws.amazon.com/zh_tw/emr/latest/ManagementGuide/emr-iam-roles-create-permissions.html)
iam-CreateInstanceProfile權限允許用戶或角色建立新的IAM Instance Profile，例如可以使用EC2 instance訪問DynamoDB；iam-AddRoleToInstanceProfile權限允許用戶或角色添加到現有的IAM Instance Profile中。
iam-CreateInstanceProfile跟iam-AddRoleToInstanceProfile通常一起使用，所以通常用戶或角色裡面兩個會同時存在。攻擊者如果可存取這兩個權限的帳戶，可以建立特權instance profile到受感染的instance。

### iam-PassRole & iam-RemoveRoleFromInstanceProfile &iam-AddRoleToInstanceProfile
iam-RemoveRoleFromInstanceProfile權限允許刪除IAM配置文件中的用戶或角色。在AWS中，通常一個instance只能對應一個角色，但一個角色可以對應多個instance，以此降低權限管理的複雜性。
例如說一個Lamdba Function只能跟一個角色關聯，該角色定義了這個Lamdba函數在執行期間的權限。一個AWS Glue Job只能與一個角色關聯，該角色定義了Glue Job在執行期間的權限。
如果一個instance已經有角色了，攻擊者可以利用iam-RemoveRoleFromInstanceProfile刪除原本角色，添加特權instance profile到受感染的instance。

### iam-SetDefaultPolicyVersion & （ iam:CreatePolicyVersion or iam:PutUserPolicy）
[AWS官方文件](https://docs.aws.amazon.com/zh_tw/IAM/latest/UserGuide/access_policies_managed-versioning.html)
iam:CreatePolicyVersion跟iam:PutUserPolicy同樣都是允許IAM account為一個已存在的管理式策略創建新的版本。例如說把Action改成" * "，Resource也改成" * "。或是直接替換成有這些權限的role。兩者的差異主要是`iam:PutUserPolicy`修改用戶的行內策略，而 `iam:CreatePolicyVersion` 為管理式策略創建新版本。
iam-SetDefaultPolicyVersion權限允許將特定權限版本設定為默認版本。通常版本是按順序創建的，新版本會取代舊版本。
通過設定默認版本，攻擊者可以修改客戶管理的policy，將非特權instance提升為特權instance


### ec2:ModifyInstanceAttribute
除了可以用於建立reverse shell之外，可以改變任何使用者的數據。例如說建立EC2 instance到ECS叢集中，這樣可以滲透這些服務(ex docker)，並竊取role。

修改了ECS agent的配置文件（範例）
```bash
#!/bin/bash
echo ECS_CLUSTER=<cluster-name> >> /etc/ecs/ecs.config;echo ECS_BACKEND_HOST= >> /etc/ecs/ecs.config;
```

# 權限維持
對於雲端環境來說，創建一個具有完整存取權限的新用戶是最簡單的進入後門的方法（但很容易被發現）

### 新用戶
```shell
> aws iam add-user-to-group --user-name BlackdoorUser --group-name AdminGroup
```



-----


# AWS提權

## 1. 取得Role policy
AWS Role Policy分兩種
![](https://imgur.com/Io1BiqQ.png)
- inline policy
    - 管理員創建管理的策略，可以直接與AWS indentity(account、role、group)相關聯，並附加在這個identity上。通常有這個意味著有好料。
- attack policy
    - AWS 托管策略（AWS Managed Policies）
    - Group/Role/Instance Policy


```go
func main() {
	sess, err := session.NewSession(&aws.Config{
		Credentials: credentials.NewStaticCredentials(access_key_id, secret_access_key, token),
		Region:      aws.String(region),
	})
	if err != nil {
		fmt.Println("Error creating session:", err)
		return
	}

	svc := iam.New(sess)

	roleName := "ROLE"  # 填寫角色名稱

	// Get Policy

	// Inline Policy
	inlineInput := &iam.ListRolePoliciesInput{
		RoleName: aws.String(roleName),
	}

	inlineResult, err := svc.ListRolePolicies(inlineInput)
	if err != nil {
		fmt.Println("Error listing role policies:", err)
		return
	}

	fmt.Println("Inline Policies:")
	for _, policyName := range inlineResult.PolicyNames {

		getPolicyInput := &iam.GetRolePolicyInput{
			RoleName:   aws.String(roleName),
			PolicyName: policyName,
		}

		policyResult, err := svc.GetRolePolicy(getPolicyInput)
		if err != nil {
			fmt.Println("Error getting role policy:", err)
			return
		}

		policyDocument, err := url.QueryUnescape(*policyResult.PolicyDocument)
		if err != nil {
			fmt.Println("Error decoding policy document:", err)
			return
		}

		fmt.Println("Policy Name:", *policyResult.PolicyName)
		fmt.Println("Policy Document:", policyDocument)
	}
	fmt.Println("--------------------------------")
	fmt.Println("Managed Policies:")

	// Managed Policy
	managerInput := &iam.ListAttachedRolePoliciesInput{
		RoleName: aws.String(roleName),
	}

	managerResult, err := svc.ListAttachedRolePolicies(managerInput)
	if err != nil {
		fmt.Println("Error listing role policies:", err)
		return
	}
	for _, policy := range managerResult.AttachedPolicies {
		fmt.Println("Policy ARN:", *policy.PolicyArn)
	}
}
```

## 2. 找到Admin & Shadow Admin
