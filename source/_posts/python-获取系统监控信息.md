---
title: python-获取系统监控信息
date: 2020-01-02 14:14:08
tags: [python]
---

#  python-获取系统监控信息

[demo地址](https://github.com/AsummerCat/other_demo/blob/master/sys_message.py)

<!--more-->

```python
# -*- coding:utf-8 -*-
import psutil
import time


# 获取系统监控信息
def get_sys_message():
    # 获取当前运行的pid
    # p1 = psutil.Process(os.getpid())

    while True:
        a = psutil.virtual_memory().percent  # 内存占用率

        b = psutil.cpu_percent(interval=1.0)  # cpu占用率
        c = get_disk_info()  # 磁盘使用率

        json_body = [
            {
                "measurement": "cpu_load_short",
                "tags": {
                    "host": "server01",
                    "region": "us-west"
                },
                # "time": "2009-11-10T23:00:00Z",
                "fields": {
                    "mem": a,
                    "cpu": b,
                    "disk": c

                }
            }
        ]
        print(json_body)
        time.sleep(2)


# 获取本机磁盘使用率和剩余空间G信息
def get_disk_info():
    # 循环磁盘分区
    content = ""
    for disk in psutil.disk_partitions():
        # 读写方式 光盘 or 有效磁盘类型
        if 'cdrom' in disk.opts or disk.fstype == '':
            continue
        disk_name_arr = disk.device.split(':')
        disk_name = disk_name_arr[0]
        disk_info = psutil.disk_usage(disk.device)
        # 磁盘剩余空间，单位G
        free_disk_size = disk_info.free // 1024 // 1024 // 1024
        # 当前磁盘使用率和剩余空间G信息
        info = "%s盘使用率：%s%%， 剩余空间：%iG" % (disk_name, str(disk_info.percent), free_disk_size)
        # 拼接多个磁盘的信息
        content += info
    return content


if __name__ == '__main__':
    get_sys_message()

```

