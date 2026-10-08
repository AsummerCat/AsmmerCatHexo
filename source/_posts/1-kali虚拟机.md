---
title: 1.kali虚拟机
date: 2026-10-08 17:20:23
tags: [网络安全]
---
# 1.kali 下载
www.kali.org/
初始账号密码: kali kali
基于debian的linux发现版,面向各种信息安全任务,例如渗透测试\安全研究\计算机取证和逆向工程
## 1.1转换中文系统
```
<!--more-->


终端输入 sudo dpkg-reconfigure locales 
选择 zh_CN.UTF-8
```
## 1.2 美化->普通win桌面
```
终端输入
sudo apt update
sudo apt install kali-desktop-gnome

然后选择 gdm3模式

安装彻底完成后，记得在终端执行 `sudo reboot` 重启系统。重启后，在登录界面输入密码前，点击右下角的齿轮图标，选择 **GNOME** 会话即可进入全新的桌面环境。
```
## 1.3 更新所有依赖
```
sudo apt full-upgrade -y 

这是 Kali 官方推荐的标准更新方式。
```


    ```
    systemctl is-enabled lightdm

    sudo systemctl disable lightdm

    sudo systemctl enable gdm3

    sudo reboot 重启一次系统
    ```
# 2.vm虚拟机安装

[https://www.vmware.com/products/desktop-hypervisor/workstation-and-fusion]()
[https://support.broadcom.com/group/ecx/productdownloads?subfamily=VMware%20Workstation%20Pro&freeDownloads=true]()

## 2.1 汉化包
汉化包在夸克上
```
文件名称后最需要加上
完整示例："...\\vmware.exe" --locale zh_CN
```


[https://files06.tchspt.com/down/VMware-Workstation-Full-26H1-25388281.exe](https://files06.tchspt.com/down/VMware-Workstation-Full-26H1-25388281.exe)

