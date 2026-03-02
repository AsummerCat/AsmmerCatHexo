---
title: 32.深度学习-RAG文本切割工具
date: 2026-03-02 14:44:18
tags: [机器学习]
---

1. 简单文本 jieba分词 或者 LangChian的RecursiveCharacterTextSplitter
   2.复杂文本:
- 基于NPL篇章分析(discoures parsing)工具
  提取段落之间的主要关系,把所有包含主从关系的段落合并成一段
- 基于BERT中的NSP( next sentence prediction) 训练任务
  设置相似度阀值t: 从前往后依次判断相邻两个段落的相似度分数是否大于t,大于则合并,否则断开
