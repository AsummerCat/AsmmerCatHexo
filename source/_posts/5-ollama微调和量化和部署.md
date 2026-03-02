---
title: 5.ollama微调和量化和部署
date: 2026-03-02 10:26:15
tags: [AI,ollama]
---
# 1.在huggingface.co 上获取需要的模型

## 2.指令微调 使用peft

可以在`huggingface`下载`peft`
或者使用`llama-factory`进行微调

执行`run_clm_sft_with_peft`,所需参数

```
--model_name_or_path
输入模型完整路径
--tokenizer_name_or_path
输入分词器完整路径
--dataset_dir
输入完整的训练集路径
--per_device_train_batch_size
1
--per_device_eval_batch_size
1
--do_train
1
--do_eval
1


```
<!--more-->

## 3.合并lora模型

这是微调过的模型

## 4.进行量化

将原来的模型大小进行压缩
使用`llama-cpp`项目去进行量化操作

#### 4.1. 安装依赖

```
需要先安装好 https//cmake.org/download

然后 
git clone https://github.com/ggerganov/llama.cpp

cd llama.cpp

pip install -r requirements/requirements-convert-hf-to-gguf.txt

cmake -B build

cmake --build build --config Release

```

#### 4.2. 先进行微调模型的格式转换

在项目文件里找到需要转换的模型
首先先进行格式转换
这里的f16表示 模型的字节大小

    convert-hf-to-gguf.py 
    微调的模型地址 --outtype f16 --outfile
    xxxx.gguf

#### 4.3.

```
quantize.exe 将转换后的xxxx.gguf
量化的模型地址.gguf q4_0
 
```

## 5.部署

直接使用ollama部署
需要构建一个Modelfile的文件

```
ollama create 名字 -f Modelfile

```

