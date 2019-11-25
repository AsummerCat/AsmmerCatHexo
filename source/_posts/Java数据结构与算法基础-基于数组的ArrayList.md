---
title: Java数据结构与算法基础-基于数组的ArrayList
date: 2019-11-25 11:59:26
tags: [数据结构与算法,java]
---

# Java数据结构与算法基础-基于数组的ArrayList

ArrayList是Java中我们最常使用的List接口的实现类，其是内部就是通过维护一个**无序数组**来实现的。因此ArrayList具备无须数组拥有的所有优点和缺点：

```java
操作	时间复杂度
插入	O(1)
删除	O(N)
查找	O(N)
```

需要注意的是：

* 1、**ArrayList总是将元素加入数组中的第一个非空位置**： 当我们往ArrayList中添加一个元素时，为了可以保证以O(1)时间插入元素，ArrayList总是将元素加入数组中的第一个非空位置，这是通过维护size变量实现的，size表示的是数组中已经添加的元素的数量，当我们插入一个数据时，直接在数组size+1的位置上加入这个元素即可。

* 2、**ArrayList中维护的数组中是没有空元素的**。这意味着 当删除数组中一个元素时，这个数组中之后所有的元素位置都会前移一个位置。当我们删除一个元素，size变为size-1 ，而如果这个元素不是数组中最后一个元素，意味着虽然只有size-1个元素，但是在0到size的中间有一个位置元素是空的，而size位置上是有元素的。当下一次插入元素时，又在size-1基础上+1，也就是在size位置上插入元素，就会将原来的size位置上元素覆盖掉。

* 3、**ArrayList中维护的数组需要动态扩容**。由于数组一旦创建，大小就是固定的。因此当ArrayList中维护的数组容量大小达到限度时，就要将数组拷贝到一个更大的数组中。

  <!--more-->

## **ArrayList源码分析**

ArrayList的源码中维护了2个重要的变量

```java
transient Object[] elementData; // 用于存放元素的数据，数组的大小就是ArrayList的容量capacity
private int size;//数组中已经存放的元素的数量
```

### **添加元素分析**

添加一个元素通过add方法实现

```
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // 确保elementData数组中还有空间插入新的元素
    elementData[size++] = e;//在数组的最后一个插入元素
    return true;
}
```

ensureCapacityInternal方法确保elementData数组中还有空间插入新的元素，也就是当前elementData.length>size，如果elementData.length==size，说明需要数组需要动态扩容。

扩容是通过调用grow方法实现：

```java
private void grow(int minCapacity) {//minCapacity表示的是需要扩容最小容量
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);//默认扩容为1.5倍
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)//这个用于保证数组的容量最大不会超过2的30次方-1
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    //使用计算出需要扩大到的新的容量创建一个新数组，并将elementData[]的数组中元素拷贝到新的数组中，再重新赋值给elementData[]。
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

minCapacity表示的是需要扩容的最小容量。例如假设当前elementData[]数组的长度来capacity，那么添加一个元素的时候，理论上只需要将数组容量扩大为capacity+1即可，那么此时minCapacity=capacity+1。不过由于创建一个新的数组之后都需要将旧的数组中的内容进行拷贝，拷贝的操作是非常消耗资源的。如果扩容后容量只在当前基础上+1，那么下一次添加1个元素，又要扩容，又要进行数组拷贝。为了避免这种情况下的出现，会有一个默认的扩容时扩容比例，就是代码中的newCapacity，从代码中可以看出是扩容1.5倍。如果需要扩容的newCapacity>minCapacity，就会使用newCapacity作为新数组的容量。

### **删除操作remove方法分析**

```java
public E remove(int index) {
    rangeCheck(index);
 
    modCount++;
    E oldValue = elementData(index);//获取要删除的元素的值
   //因为删除一个元素之后，要将后继的元素往前移动一个位置，所以计算从哪个位置开始移动
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);//移动元素
    elementData[--size] = null; //删除size位置上的元素，同时将size-1，再将这个位置上的元素置为null以便垃圾回收
 
    return oldValue;//返回删除的元素的值。
}
```

### **查找操作分析**

如果要获取指定位置上的元素，那么调用get(int index)方法即可，这个方法的时间复杂度是O(1)。但是我们这里所有的查找，指的是并不是知道某个元素在哪个位置上，因此只能使用线程查找的方式进行。因此我们需要遍历ArrayList时，假设有N个元素，当我们遍历时，根据经验，平均需要遍历N/2次，因此时间复杂度是O(N)。

ArrayList的indexOf和lastIndexOf方法都是通过遍历的方法查找一个对象在ArrayList中的位置。

```
public int indexOf(Object o) {
    if (o == null) {//如果对象是null，返回elementData[]中第一个元素值为null的下标
        for (int i = 0; i < size; i++)
            if (elementData[i]==null)
                return i;
    } else {////如果对象不是null，返回elementData[]中第一个equals方法相等的index
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;//查找不到返回-1
}
```

类似的，lastIndexof(Object obj)是从后往前查。之所以有这两种查找方式是因为ArrayList中是可以添加重复的元素。

