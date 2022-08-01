---
title: java基础-枚举
date: 2018-10-09 16:56:03
tags: [java基础]
---

那么必须在enum实例序列的最后添加一个分号.

# 第一种枚举

<!--more-->

枚举类:

```
/**
 * @author cxc
 */
public enum Color {
  RED, GREEN, BLANK, YELLOW;
}
```
遍历:

```
/**
 * @author cxc
 * @date 2018/10/9 16:53
 */
public class red {
    public static void main(String[] args){
         for(Color ye: Color.values()){
             System.out.println(ye.name()+"---"+ye.toString());
         }
    }
}

```

结果:  

```
RED---RED  
GREEN---GREEN  
BLANK---BLANK  
YELLOW---YELLOW  

Process finished with exit code 0  
```

---
---
---


# 第二种枚举
枚举类

```
/**
 * @author cxc
 */
public enum Color {
    //定义这种枚举需要添加构造方法
    RED("红色",1), GREEN("绿色", 2), BLANK("白色", 3), YELLOW("黄色", 4);
    // 成员变量
    private String name;
    private int index;
    

    // 构造方法
    private Color(String name, int index) {
        this.name = name;
        this.index = index;
    }

    public String getName() {
        return name;
    }

    public int getIndex() {
        return index;
    }
}
```

遍历:

```
/**
 * @author cxc
 * @date 2018/10/9 16:53
 */
public class red {
    public static void main(String[] args){
         for(Color ye: Color.values()){
             System.out.println(ye.name()+"---"+ye.getName()+"---"+ye.getIndex());
         }
    }
}

```

结果:

```
RED---红色---1
GREEN---绿色---2
BLANK---白色---3
YELLOW---黄色---4

Process finished with exit code 0
```

可以创建普通静态方法 在 枚举类中调用   
例如:

```
    //根据index 获取name
public static String getName(int index){
        for(Color color:values()){
             if(color.getIndex()==index){
                 return color.getName();
             }
        }
        return null;
    }
    
    
    //根据name 获取index
    public static String getindex(String name){
        for(Color color:values()){
            if(color.getName().equals(name)){
                return String.valueOf(color.getIndex());
            }
        }
        return null;
    }

```

---

# 注意事项

# 枚举类遍历方式 value()

枚举类下会有一个方法value();
伪代码如下:

```
for(Color color:values()){
     color.get属性
           
           }
```

# 也可以写个静态方法去获取值什么的

例如:

```
//根据name 获取index
    public static String getindex(String name){
        for(Color color:values()){
            if(color.getName().equals(name)){
                return String.valueOf(color.getIndex());
            }
        }
        return null;
    }
```
 