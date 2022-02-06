---
title: List分片取余
date: 2019-03-05 20:07:38
tags: java
---

# javaList分片

<!--more-->
```
package com.syswin.systoon.edusyncmq.utils;

import com.google.common.collect.Lists;

import java.util.ArrayList;
import java.util.List;

/**
 *集合分片工具类
 *
 * @author cxc
 * @create 2021年12月3日18:11:15
 * @since 1.0.0
 */
public class ListUtil {
    /**
     * list分片取余
      * @param list 源数据
     * @param blockSize 每页数量
     * @param <T> 数据类型
     * @return
     */
    public static <T> List<List<T>> splitList(List<T> list, int blockSize)   {
        List<List<T>> lists = new ArrayList<List<T>>();
        if(blockSize == 1){
            lists.add(list);
            return lists;
        }
        if (list != null && blockSize > 0) {
            int listSize = list.size();
            if(listSize<=blockSize){
                lists.add(list);
                return lists;
            }
            int batchSize = listSize / blockSize;
            int remain = listSize % blockSize;
            for (int i = 0; i < batchSize; i++) {
                int fromIndex = i * blockSize;
                int toIndex = fromIndex + blockSize;
                System.out.println("fromIndex=" + fromIndex + ", toIndex=" + toIndex);
                lists.add(list.subList(fromIndex, toIndex));
            }
            if(remain>0){
                System.out.println("fromIndex=" + (listSize-remain) + ", toIndex=" + (listSize));
                lists.add(list.subList(listSize-remain, listSize));
            }
        }
        return lists;
    }

    /**
     * 使用Guava分片
     * @param list 源数据
     * @param groupSize 每页数量
     * @return
     */
    public static List<List<String>> splitListByGuava(List<String> list , int groupSize){
        return  Lists.partition(list, groupSize); // 使用guava
    }

     /**
     * 平分list成n份 数据量尽可能相等
     * @param list 需要平分的list
     * @param n    平分成n分
     * @return
     */
    public static <T> List<List<T>> splitListByGroupNum(List<T> list, int n) {
        List<List<T>> strList = new ArrayList<>();
        if (list == null) return strList;
        int size = list.size();
        int quotient = size / n; // 商数
        int remainder = size % n; // 余数
        int offset = 0; // 偏移量
        int len = quotient > 0 ? n : remainder; // 循环长度
        int start = 0;    // 起始下标
        int end = 0;    // 结束下标
        List<T> tempList = null;
        for (int i = 0; i < len; i++) {
            if (remainder != 0) {
                remainder--;
                offset = 1;
            } else {
                offset = 0;
            }
            end = start + quotient + offset;
            tempList = list.subList(start, end);
            start = end;
            strList.add(tempList);
        }
        return strList;
    }

}
```
