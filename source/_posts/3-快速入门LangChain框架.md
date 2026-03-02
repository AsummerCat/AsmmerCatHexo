---
title: 3.快速入门LangChain框架
date: 2026-03-02 10:24:19
tags: [AI,python,LangChain]
---
# 快速入门LangChain框架

高效开发大模型应用,并且可以使用aysnc异步流式处理

## 1.ChatPromptTemplate: 提示词模板

```
from langchain_core.prompts import ChatPromptTemplate
prompt_tempalte=ChatPromptTemplate.from_message([
   ("system","你是一个小丑"),
   ("user","给我讲一个笑话{topic}")
])
result=prompt_tempalte.invoke({"topic":"笑话"})

```

<!--more-->


## 2.MessagePlaceholder :消息模板

效果: 就是给Ai发送多段话 ,主要作用是替换系统消息 ,给角色提供多个详细定义

```
from langchain_core.prompts import ChatPromptTemplate ,MessagePlaceholder 
from langchain_core.messages import HumanMessage  

prompt_tempalte=ChatPromptTemplate.from_message([
   ("system","你是一个小丑"),
   MessagePlaceholder("msg")
])
result=prompt_tempalte.invoke({"msg":[HumanMessage(context="hi"),HumanMessage(context="hi1")]})


方式二:
from langchain_core.prompts import ChatPromptTemplate ,MessagePlaceholder 
from langchain_core.messages import HumanMessage  

prompt_tempalte=ChatPromptTemplate.from_message([
   ("system","你是一个小丑"),
   ("placeholder","{msg}")
])
result=prompt_tempalte.invoke({"msg":[HumanMessage(context="hi"),HumanMessage(context="hi1")]})

```

## 3.Output parsers输出解析器

大模型输出给服务之后 需要对他返回的格式进行二次处理

```
from langchian_openai import ChatOpenAI
from langchain_core.messages import HumanMessage,SystemMessage  
from langchain_core.output_parpers import StrOutParper

model=ChatOpenAI(model=gpt-4)

message=[
 SystemMessage(context="将以下内容翻译为中文"),
 HumanMessage(context="a person")
]

# 使用文本解析器
parper=StrOutParper()
result=model.invoke(message)
# 使用parper 来处理返回结果
response=parper.invoke(result)
print(respone)

```

## 4.Chains 链式调用

可以用`|`运算符 轻松将多个元素组合在一起

```
from langchian_openai import ChatOpenAI
from langchain_core.messages import HumanMessage,SystemMessage  
from langchain_core.output_parpers import StrOutParper

model=ChatOpenAI(model=gpt-4)

message=[
 SystemMessage(context="将以下内容翻译为中文"),
 HumanMessage(context="a person")
]

# 使用文本解析器
parper=StrOutParper()
#result=model.invoke(message)
# 使用parper 来处理返回结果
#response=parper.invoke(result)
#print(respone)

# 使用chain 轻松组合链式流程
chain=model|parper
response=chain.invoke(message)
print(respone)

```

## 5.Steam Events(事件流)

可以在处理中的任意环节加上事件流做监控 类似AOP

```
event=[]
async for event in model.astream_events("hello",version="v2"):
  event.append(event)
prin(event)
# 输出当前流所有的事件流

```

## 6.同步异步 流

    import asyncio
    from langchian_openai import ChatOpenAI
    from langchain_core.messages import HumanMessage,SystemMessage  
    from langchain_core.output_parpers import StrOutParper

    model=ChatOpenAI(model=gpt-4)
    # 使用文本解析器
    chain=(model|JsonOutParper())

    async def async_stream():
       asnyc for text chain.astream(
       "以json形式输出法国/日本人口",
       "每个国家都应该有name和population"
       );
       print(text,flush=True)   
       
    # 执行异步处理
    syncio.run(async_stream());

## 7.debuger模式

`1,2 类似打log`
1.详细模式(Verbose Mode):
(为你的链中的'重要'事件输出打印日志)
不支持-> 可视化,持久化
支持->查看本次运行

2.调试模式(Debug Mode):
(为你的链中的'所有'事件输出打印日志)
不支持-> 可视化,持久化
支持  -> 查看本次运行
3.LangSmith跟踪: (5000次免费调用额度)
(将事件记录到LangSmith进行可视化)
支持-> 可视化,持久化
不支持查看本次运行

#### 7.1 详细模式日志打印(Verbose Mode)

代码中新增

```
from langchian.globals from set_verbose

#开启全局日志
set_verbose(True)

```

#### 7.2 调试模式日志打印(Debug Mode)

代码中新增

```
from langchian.globals from set_debug

#开启调试日志
set_debug(True)

```

# 1
