---
title: Oozie定时任务调度介绍一
date: 2022-08-02 09:19:56
tags: [大数据,Oozie]
---
# Oozie定时任务调度
一个基于工作流引擎的开源框架;
提供对 Hadoop MapReduce , Pig Jobs 或shell的任务调度与协调;
Oozie需要部署到java Servlet容器中运行.
主要用于定时调度任务,多任务可以按照执行的逻辑顺序调度;

# 1.Oozie的模块介绍

## 1.1模块
三个配置文件 对应三个模块
#### 1) Workflow   (工作流流程图)
顺序执行流程节点,支持fork(分支多个节点),join(合并多个节点为一个)

#### 2) Coordinator   (定时器)
定时触发Workflow

#### 3) Bundle Job  (任务)
绑定多个Coordinator

<!--more-->
## 1.2常用节点

#### 1) 控制流节点(Control Flow Nodes)
控制流节点一般都是定义在工作流开始或者结束的位置,比如start,end,kill等.
以及提供工作流的执行路径机制,如decision,fork,join等.

#### 2) 动作节点(Action Nodes)
负责具体执行动作的节点,比如: 拷贝文件,执行某个shell脚本等等.


