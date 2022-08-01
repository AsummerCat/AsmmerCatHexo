---
title: Java数据结构与算法基础-数组
date: 2019-11-25 11:48:52
tags: [数据结构与算法,java]
---

# Java数据结构与算法基础-数组

## 数组



  数组是应用最广泛的数据存储结构，它被植入到大部分编程语言中。由于数组十分易懂，所以它被用来介绍数据结构的起点。 数组分为2种：**无序数组**与**有序数组**。有序数组就是无序数组经过排序后结果

​     关于数组，大部分读者都已经非常熟悉了，不过需要注意的是，在数据结构与算法中，我们在讨论数组的时候，有一些特别要注意的地方

1. 我们通常假设数组中是没有空洞的
2. 当删除数组中一个元素时，这个数组中之后所有的元素位置都会前移一个位置
3. 如果是无序数组的话，添加一个元素时，总是添加到数组的最后位置；
4. 如果是有序数组的话，添加元素到某个位置时，当前位置的元素与之后的元素都要往后移动一个位置
5. 数组维护了当前元素的数量<!--more-->

之所以有这些要求，主要是从操作方便的角度考虑，例如我有一个大小为5的数组，其中有3个元素，如下所示：

![](/img/2019-11-25/1.png)

###  **1 我们通常假设数组中是没有空元素的**

   当我们想在数组查找某个元素时，当所有元素都查过了之后，依然没有查到，就说明数组中不包含此元素。那么我们如何知道所有的元素都已经查过了呢？按照规则1，只要我们保证数组所有非空元素，都排在数组的前面，那么当我们遇到第一个空元素时，就说明所有的元素都查找完了。

### **2 当删除数组中一个元素时，这个数组中之后所有的元素位置都会前移一个位置**

   例如我们删除了第二个元素，在java中，就是移除数组对这个对象的引用，只要将对应位置设为null即可，但是在这里我们却不能这样做，因为根据规则1，我们查找的时候，遇到第一个为null的元素的时候，就认为所有的元素都查找完了，例如现在查找5，那么就会查不到，因此删除必须将后面所有的元素都前移一个位置

### **3 如果是无序数组的话，添加一个元素时，总是添加到数组的最后位置；**

   插入操作同样要满足，插入后，数组中依然不能存在空洞，否则查找依然会出现问题，例如现在还有2个位置可以插入，如果我们插入在最后一个位置上，根据规则1，之后在查找的时候又找不到这个元素了。

### **4、我们通常假设数组中没有相同的元素**

在查找的时候，如果有相同的元素，那么可能会有多个匹配值，那么到底返回哪个呢？还是全部都返回？为了方便，我们通常假设数组中没有相同的元素，因此只需要返回第一个匹配上的值即可。

下面的代码是按照上述要求实现的数组

```
public class Array<V> {
    private Object[] elements;
    private int size=0;//数组中元素的数量
    private int capacity;//数组的容量
 
    /**
     * 数组的容量
     * @param capacity
     */
    public Array(int capacity) {
        this.capacity = capacity;
        if(capacity<=0){
            throw new IllegalArgumentException("capacity must > 0");
        }
        elements=new Object[capacity];
    }
 
    public void insert(V v){
        if(size==capacity-1){//达到容量限制
            throw new IndexOutOfBoundsException();
        }
        elements[size++]=v;//插入元素
    }
 
    public boolean remove(V v){
        for (int i = 0; i < size; i++) {
            if(elements[i].equals(v)){
                elements[i]=null;//删除
                moveUp(i,size);//将后面的所有数据项都前移一个位置
                size--;//元素数量-1
                return true;//找到第一个要删除的项，返回true
            }
        }
        return false;//所有元素都查找过了，依然没有找到要删除的项，犯规false
    }
 
    public V find(V v){
        for (int i = 0; i < size; i++) {
            if(elements[i].equals(v)){
              return (V) elements[i];
            }
        }
        return null;
    }
 
    private void moveUp(int i, int size) {
        while (i<size-1){
            elements[i]=elements[++i];
        }
        elements[size-1]=null;//最后一个元素置位null
    }
 
    /**
     * 返回指定下标的元素
     * @param index
     * @return
     */
    public V get(int index){
        if(index>capacity-1){
            throw new IndexOutOfBoundsException();
        }
        return (V) elements[index];
    }
 
    /**
     * 返回数组中元素的数量
     * @return
     */
    public int size(){
        return size;
    }
    /**
     * 显示所有元素
     */
    public void display(String prefix){
        System.out.print(prefix);
        for (int i = 0; i < elements.length; i++) {
            if(i<size){
                System.out.print(elements[i]+"    ");
            }else{
                System.out.print("null"+"    ");
            }
        }
        System.out.println();
    }
 
    public static void main(String[] args) {
        Array<Integer> array=new Array<Integer>(5);
        array.insert(1);
        array.insert(5);
        array.insert(3);
        array.display("初始 1、5、3 ： ");
        array.insert(4);
        array.display("添加 4       ： ");
        array.remove(3);
        array.display("删除 3       ： ");
       System.out.println("查找 4："+array.find(4) );
       System.out.println("查找 3："+array.find(3) );
    }
}
```

运行程序，输出：

```
初始 1、5、3 ： 1    5    3    null    null   

添加 4       ： 1    5    3    4    null   

删除 3       ： 1    5    4    null    null   

查找 4：4

查找 3：null 
```

## **数组的效率**

   在数据结构与算法中，衡量算法的效率是通过时间复杂度和空间复杂度来进行的，后面我们会有专门的讲解，下面是一个简单的介绍

| 操作 | 时间复杂度 |
| ---- | ---------- |
| 插入 | O(1)       |
| 删除 | O(N)       |
| 查找 | O(N)       |

  其中：

###  **O(1)**表示，此操作不受数组元素个数的影响，不论数组中现有多少元素，插入操作总是1步就可以完成

 ### **O(N)**表示此操作受到数据元素个数的影响，最直观的感受是，我们可以看到删除和查找操作，里面都有一个for循环来迭代数组中的所有元素，假设数组中有N个元素，我们随机查找或者删除一个数字，运气好的情况下，可能1次就查找了，运气不好，可能所有的N个元素迭代完了，还是没有找到，根据概率，平均可能需要进行N/2次循环，由于时间复杂度是忽略常数的，因此删除和查找操作的平均时间复杂度是O(N)

## **关于查找的说明**

   在数据结构与算法中，查找指的并不是根据数组的下标进行查找(上例中的get方法)，根据下标进行查找的时间复杂度总是O(1)，而是根据关键字进行查找(上例中的find方法) ，需要迭代数组中的每一个元素进行查找，有一个专门的术语，称之为线性查找

这里我们往数组中存入对象，来说明根据关键字查找的含义

```
public class ArrayQueryTest {
    public static class User {
        private int id;
        private String name;
 
        public User(int id, String name) {
            this.id = id;
            this.name = name;
        }
 
        @Override
        public String toString() {
            return "User{" +
                    "id=" + id +
                    ", name='" + name + '\'' +
                    '}';
        }
    }
 
    public static void main(String[] args) {
        Array<User> userArray=new Array<User>(5);
        userArray.insert(new User(1,"tianshouzhi"));
        userArray.insert(new User(2,"wangxiaoxiao"));
        userArray.insert(new User(3,"huhuamin"));
 
        //根据user的id进行查找
        int queryKeyWord=2;
 
        User result=null;//查询结果
        for (int i = 0; i < userArray.size(); i++) {
            User user=userArray.get(i);
            if(queryKeyWord==user.id){
                System.out.println(user);
                break;
            }
        }
    }
}
```
