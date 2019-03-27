---
title: 从list中随机抽取不同的指定数量的元素组成新list
date: 2019-03-24 20:37:35
tags: [java,工具类]
---

# 从list中随机抽取不同的指定数量的元素组成新list

## 代码

```
 /**  
     * 从list中随机抽取元素 判断并获取新药随机样本数量  
     * @param list  
     * @param randomQuantity 抽取样本数量   (需要抽取的数量)
     * @return  
     */  
    private static List<PRdrugReviewSampleEntity>  createRandomList(List<PRdrugReviewSampleEntity> list, int randomQuantity) {  
        Integer size = list.size();  
        Map map = new HashMap<>();  
        List<PRdrugReviewSampleEntity> listNew = new ArrayList<>();  
        if (randomQuantity >= size) {  
            return list;  
        }else{  
            while(map.size()<randomQuantity){  
                int random = (int) (Math.random() * list.size());  
                if (!map.containsKey(random)) {  
                    map.put(random, "");  
                    listNew.add(list.get(random));  
                }  
            }  
            return listNew;  
        }  
    }  
```

