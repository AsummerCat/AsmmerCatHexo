---
title: 在docker中部署Ollama体验AI模型
date: 2024-05-28 23:24:03
tags: [docker,Ollama,AI,ChatGPT]
---
# docker部署ollama

注意需要挂载到物理硬盘 才好下载模型

## CPU模式

    docker run -d -v /opt/ai/ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama

比如我的硬盘是E:

    docker run -d -v E://ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama

<!--more-->
## GPU模式（需要有NVIDIA显卡支持）

安装英伟达容器工具包（以Ubuntu22.04为例）
其他系统请参考：英伟达官方文档

GPU support in Docker Desktop（可选，如果本地有GPU，则需要安装）：<https://docs.docker.com/desktop/gpu/>

### linux

    # 1.配置apt源
    curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
      && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
        sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
        sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
    # 2.更新源
    sudo apt-get update
    # 3.安装工具包
    sudo apt-get install -y nvidia-container-toolkit



### win

下载地址:<https://developer.nvidia.com/cuda-toolkit-archive>

##### 1.打开本地的NVIDIA控制面板->帮助 ->系统信息->组件->查看NVCUDA64.DLL的版本号 (产品名称那一块)

##### 2.下载地址中选择windows->X86\_64 ->local版本

路径C:\Users\ADMINI\~1\AppData\Local\Temp\CUDA为其安装时的临时目录，我们可以换一个，免得增加C盘负担。
F:\CUDAtemp

    自定义安装
    只要安装CUDA即可

需要注意验证是否安装成功 `nvcc --version`
如果不存在手动添加环境变量
安装目录: `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.2`

```
C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.2\bin
C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.2\libnvvp

```

##### 3. 安装cuDNN Archive

官网链接：<https://developer.nvidia.com/rdp/cudnn-archive>
找到与CUDA相对应的版本，我的CUDA版本为CUDA12
cudnn-windows-x86\_64-8.9.7.29\_cuda12-archive

    将
    cudnn-windows-x86_64-8.9.7.29_cuda12-archive\bin
    中的文件复制到
    C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.2\bin


    将
    cudnn-windows-x86_64-8.9.7.29_cuda12-archive\include
    中的文件复制到
    C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.2\include

    将
    cudnn-windows-x86_64-8.9.7.29_cuda12-archive\lib
    中的文件复制到
    C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.2\x64

这样安装就完成了,最好重启机子

#### 4. 查看GPU持久化是否开启

    执行: nvidia-smi -a
     查询自己的 Persistence Mode 是否开启
     
     在Attached GPUs
       ->Persistence Mode
       
       强制开启: nvidia-smi -pm ENABLED

### docker使用GPU运行ollama

    docker run --gpus all -d -v E://ollama:/root/.ollama -p 11434:11434 --name ollama_gpu ollama/ollama

## docker部署ollama web ui

    docker run -d -p 8080:8080 --add-host=host.docker.internal:host-gateway --name ollama-webui --restart always ghcr.io/ollama-webui/ollama-webui:main

## 使用docker中的ollama下载并运行AI模型（示例为阿里通义千问4b-chat）

    docker exec -it ollama ollama run qwen:4b-chat

## ollama模型仓库

    https://ollama.com/library

## win需要在linux子系统上运行

输入

`wsl --update`
安装

`wsl --install`

并且在应用中安装centos

然后在docker中开启使用wsl2



## 最后需要注意的是
大数据模型的大小概念 