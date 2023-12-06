---
title: Windows写sh脚本的问题
date: 2023-12-06 16:00:36
tags: [linux]
---

## Windows写sh脚本的问题 
```
sh脚本提示:

start.sh: line 2: $'\r': command not found

这个是因为win的换行符\r\n和linux的换行符\n不一样 导致报错
```
可以使用脚本 或者命令处理上传后的数据
<!--more-->
### 方式1. 手动处理
`sed -i 's/\r//' start.sh`

### 方式2. 使用 dos2unix 进行转换
```
yum install -y dos2unix


dos2unix start.sh

```