---
title: 2.基于Streamlit框架实现页面
date: 2026-03-02 10:23:40
tags: [AI]
---
# Steamlit框架

主要用来生成demo使用

*   <https://steamlit.io>
*   Steamlit更快构建和共享数据应用的方法
*   Steamlit能够在几分钟内将数据脚本转换为可共享的web应用程序
*   全程使用纯Python开发,无需前端经验

<!--more-->

## 演示案例

    import steamlit as st
    import pandas as pd

    st.title("定义窗口标题")

    //输入文字显示
    st.write("""
    # 这是我的程序
    hall
    """)


    //定义左侧边栏内容 里面使用mk语法来书写
    with st.sidebar:
       st.markdown(f"""
       
       
       """,unsafe_allow_html=True)

    //定义多行输入框
    system_message=st.text_area("名称","文字预输入内容")


    //读取csv文件
    df= pd.read_csv("my_data.csv")
    //生成直线图
    st.line_chart(df)

### 运行

    steamlit run app.py

可以直接使用md语法编写
