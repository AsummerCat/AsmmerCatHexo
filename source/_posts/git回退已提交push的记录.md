---
title: git回退已提交push的记录
date: 2020-06-19 17:28:38
tags: [git]
---

idea 操作 gitHub还原本地线上版本 操作步骤

git reset --mixed：此为默认方式，不带任何参数的git reset，即时这种方式，它回退到某个版本，只保留源码，回退commit和index信息  git reset --soft：回退到某个版本，只回退了commit的信息，不会恢复到index file一级。如果还要提交，直接commit即可  git reset --hard：彻底回退到某个版本，本地的源码也会变为上一个版本的内容

1.0 还原分支不保留

git reset --soft  ****  (****为上一个commit的id)

2.0 把线上的还原 git push origin hongjie.zhang-tccrm --force