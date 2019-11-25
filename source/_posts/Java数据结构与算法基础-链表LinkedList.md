---
title: Java数据结构与算法基础-链表LinkedList
date: 2019-11-25 14:52:27
tags: [数据结构与算法,java]
---

在本节中，我们将看到一种新的数据存储结构，它可以解决上面的一些问题。这种数据存储结构就是`链表`。**链表可能是继数组之后第二种使用最广泛的通用存储结构。**

​    在本章中，我们将学习：

- ​    单链表
- ​    双端链表
- ​    有序链表
- ​    双向列表
- ​    有迭代器的列表

链表与数组一样，都作为数据的基本存储结构，但是在存储原理上二者是不同的。在数组中，数据是存储在一段连续的内存空间中，我们可以通过下标来访问数组中的元素；而在链表中，元素是存储在不同的内存空间中，前一个元素的位置维护了后一个元素在内存中的地址，在Java中，就是前一个元素维护了后一个元素的引用。在本教程我们，我们将链表中的每个元素称之为一个节点(`Node`)。对比数组， 链表的数据结构可以用下图表示

![](/img/2019-11-25/2.png)

<!--more-->

这张图显示了一个链表的数据结构，链表中的**每个Node都维护2个信息**：一个是这个Node自身存储的数据`Data`，另一个是下一个Node的引用，图中用`Next`表示。对于最后一个Node，因为没有下一个元素了，所以其并没有引用其他元素，在图中用紫色框来表示。

这张图主要显示的是链表中Node的内部结构和Node之间的关系。一般情况下，我们在链表中还要维护第一个Node的引用，原因是在链表中访问数据必须通过前一个元素才能访问下一个元素，如果不知道第一个Node的话，后面的Node都不可以访问。事实上，对链表中元素的访问，都是从第一个Node中开始的，第一个Node是整个链表的入口；而在数组中，我们可以通过下标进行访问元素。

### **Node.Java：**

```
public class Node {
    //Node中维护的数据
       private Object data;
       //下一个元素的引用
       private Node next;
    
    // setters and getters
}
```

## **一、单链表Java实现**

本节介绍单链表的Java实现，我们用`SingleLinkList`来表示。

分析：

**1、SingleLinkList中要维护的信息：**维护第一个节点(`firstNode`)的引用，作为整个链表的入口；

**2、插入操作分析**：基于链表的特性，插入到链表的第一个位置是非常快的，因为只要改变fisrtNode的引用即可。因此对于单链表，我们会提供`addFirst`方法。

**3、查找操作分析：**从链表的fisrtNode开始进行查找，如果确定Node中维护的data就是我们要查找的数据，即返回，如果不是，根据next获取下一个节点，重复这些步骤，直到找到最后一个元素，如果最后一个都没找到，返回null。

**4、删除操作分析**

​     首先查找到要删除的元素节点，同时将这个节点的上一个节点和下一个节点也要记录下来，只要将上一个节点的next引用直接指向下一个节点即可，这就相当于 删除了这个节点。如果要删除的是第一个节点，直接将LinkList的firstNode指向第二个节点即可。如果删除的是最后一个节点，只要将上一个节 点的next引用置为null即可。上述分析，可以删除任意节点，具有通用性但是效率较低。通常情况下，我们还会提供一个`removeFirst`方法，因为这个方法效率较高，同样只要改变fisrtNode的引用即可。  

此外，根据情况而定，可以选择是否要维护链表中元素的数量`size`，不过这不是实现一个链表必须的核心特性。

SingleLinkList.java

```java
public class SingleLinkList<T> {
    //链表中第一个节点
    protected Node firstNode=null;
 
    //链表中维护的节点总量
    protected int size;
 
    /**
     * 添加到链表的最前面
     * @param element
     */
    public void addFirst(T element){
        Node node=new Node();
        node.setData(element);
        Node currentFirst=firstNode;
        node.setNext(currentFirst);
        firstNode=node;
        size++;
    }
 
    /**
     * 如果链表中包含要删除的元素，删除第一个匹配上的要删除的元素，并且返回true;
     * 如果没有找到要删除的元素，返回false
     * @param element
     */
    public boolean remove(T element){
        if(size==0){
            return false;
        }
        if(size==1){
            firstNode=null;
            size--;
        }
 
        Node pre=firstNode;
        Node current=firstNode.getNext();
        while(current!=null){
            /*如果当前节点中维护的值就是要删除的值，
            直接将上一个节点pre的next应用指向当前节点current的下一个节点接口*/
            if((current.getData()==null&&element==null)
                    ||(current.getData().equals(element))){
                pre.setNext(current.getNext());
                size--;
                return true;
            }
 
            //如果当前元素不是要删除的元素，继续循环
            pre=current;
            current=current.getNext();
        }
        return false;
    }
 
    /**
     * 如果包含返回true，如果不包含，返回false
     * @param element
     * @return
     */
    public boolean contains(Object element){
        if(size==0){
            return false;
        }
        Node current=firstNode;
        while(current!=null){
            if((current.getData()==null&&element==null)
                    ||(current.getData().equals(element))){
                return true;
            }
 
            //如果当前元素不是要删除的元素，继续循环
            current=current.getNext();
        }
        return false;
    }
 
    public boolean isEmpty(){
        return size==0;
    }
 
    public int size(){
        return size;
    }
 
    /**
     * 打印出所有的元素
     */
    public void display(){
        if(!isEmpty()){
            Node current=firstNode;
            while(current!=null){
                System.out.print(current.getData()+"\t");
                current=current.getNext();
            }
        }
    }
    /**
     * 删除第一个元素
     */
    public T removeFisrt() {
        Node result=null;
        if(size!=0) {
            result = firstNode.getNext();
            firstNode= result;
            return (T) result.getData();
        }
       return null;
    }
 
    public T getFirst() {
        return (T) firstNode.getData();
    }
}
```

### **测试添加addFirst**

```
@Test
public void testAddFisrt() {
    SingleLinkList<Integer> linkList=new SingleLinkList<Integer>();
    for (int i = 0; i < 10; i++) {
        linkList.addFirst(i);
    }
    linkList.display();
}
```

控制台输出：

9    8    7    6    5    4    3    2    1    0

因为总是添加到最前面，因此时降序的。

需要注意的是：在本案例中，不能同时调用addFirst，addLast。因为我们在addFirst方法中并没有维护lastNode的信息，因此同时使用这两种方法可能会出错，有待继续完善。

### **测试删除任意元素**

```.
@Test
public void testRemove() {
    SingleLinkList<Integer> linkList=new SingleLinkList<Integer>();
    for (int i = 0; i < 10; i++) {
        linkList.addFirst(i);
    }
    if(!linkList.isEmpty()){
        linkList.remove(5);
    }
    linkList.display();
}
```

控制要输出：

0    1    2    3    4    6    7    8    9

可以看到5的确没有了


  ### **测试删除第一个元素**

```
@Test
public void testRemoveFisrt() {
    SingleLinkList<Integer> linkList=new SingleLinkList<Integer>();
    for (int i = 0; i < 10; i++) {
        linkList.addFirst(i);
    }
    linkList.removeFisrt();
    linkList.display();
}
```

控制台输出：

1    2    3    4    5    6    7    8    9

**测试包含：**

```java
@Test
public void testContains() {
    SingleLinkList<Integer> linkList=new SingleLinkList<Integer>();
    for (int i = 0; i < 10; i++) {
        linkList.addFirst(i);
    }
    System.out.println(linkList.contains(5));
    System.out.println(linkList.contains(10));
}
```

控制台输出：

true
false

结果显示，包含5，不包含10

##  **双端链表Java实现**

本节介绍双端链表的Java实现，我们用`DoubleLinkJava`来表示。

双端链表与传统的链表非常类似，但是它有一个新增的特性：即对链表中最后一个节点的引用`lastNode`。我们可以像在单链表中在表头插入一个元素一样，在链表的尾端插入元素。如果不维护对最后一个节点的引用，我们必须要迭代整个链表才能得到最后一个节点，然后再插入，效率很低。因此我们在双链表中添加一个`addLast`方法，用于添加节点到末尾。

addLast方法分析：直接将链表中维护的lastNode的next引用指向新的节点，再将lastNode的引用指向新的节点即可。



因为单链表中，大部分的代码在双端链表中都可以重用，所以此处我们编写的DoubleLinkList只要继承SingleLinkList，添加必要的属性和方法支持从尾部操作即可。



DoubleLinkList.java

```java
package com.tianshouzhi.algrithm.list;
 
public class DoubleLinkList<T> extends SingleLinkList<T>{
    //链表中的最后一个节点
    protected Node lastNode=null;
     /**
     * 添加到链表的最后
     * @param element
     */
    public void addLast(T element){
        Node node=new Node();
        node.setData(element);
        
        if(size==0){//说明没有任何元素，说明第一个元素
            firstNode=node;
        }else{//如果有元素，将最后一个节点的next指向新的节点即可
            /*这里有一个要注意的地方：
                当size=1的时候，firstNode和lastNode指向同一个引用
                因此lastNode.setNext时，fisrtNode的next引用也会改变;
                当size!=1的时候，lastNode的next的改变与firstNode无关*/
            lastNode.setNext(node);
        }
        
        //将lastNode引用指向新node
        lastNode=node;
        size++;
        
    }
    
    /**
     * 当链表中没有元素时，清空lastNode引用
     */
    @Override
    public boolean remove(T element) {
        boolean result=super.remove(element);
        if(size==0){
            lastNode=null;
        }
        return result;
    }
 
    /**
     * 因为在SingleLinkList中并没有维护lastNode的信息，我们要自己维护
     */
    @Override
    public Node addFirst(T element) {
        Node node=super.addFirst(element);
        if(size==1){//如果链表为size为1，将lastNode指向当前节点
            lastNode=node;
        }
        return node;
    }
    
    
}
```

测试addLast

```java
@Test
    public void testAddFisrt() {
        DoubleLinkList<Integer> linkList=new DoubleLinkList<Integer>();
        for (int i = 0; i < 5; i++) {
            linkList.addFirst(i);
        }
        for (int i = 0; i < 5; i++) {
            linkList.addLast(i);
        }
        linkList.display();
    }
```

控制台输出：

4    3    2    1    0    0    1    2    3    4

从输出中我们可以到，前五个元素因为是addFirst添加的，所以是降序的，而后面五个元素是addLast添加的，所以是升序的。

## **三、有序链表Java实现**

本节讲解有序链表，使用SortedLinkList表示。

所谓有序链表，就是链表中Node节点之间的引用关系是根据Node中维护的数据data的某个字段为key值进行排序的。

为了在一个有序链表中插入，算法必须首先搜索链表，直到找到合适的位置：它恰好在第一个比它大的数据项前面。

​    当算法找到了要插入的数据项的位置，用通常的方式插入数据项：把新的节点Node指向下一个节点，然后把前一个节点Node的next字段改为指向新的节点。然而，需要考虑一些特殊情况，连接点有可能插入在表头或者表尾。



在本例中，我们创建一个类Person表示插入的数据，我们希望链表中数据是按照Person的enName属性升序排列的。  

**三、Java中的双端链表实现LinkedList**

java中已经提供了双端链表的实现，java.util.LinkedList，以下是这个类部分方法摘要，相信不需要介绍，根据方法名字，你就可以知道其含义：

```
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
public E getFirst() ;
public E getLast() ;
public E removeFirst();
public E removeLast();
public void addFirst(E e);
public void addLast(E e);
public boolean contains(Object o);
public int size();
public boolean add(E e);//等价于addLast
public boolean remove(Object o);
public void clear() ;
....
}
```

