---
title: ArrayList解析
date: 2019-01-28 14:44:05
tags: [jdk源码解析]
---





# 介绍

* `List<E>接口`：我们会出现这样一个疑问，在查看了ArrayList的父类AbstractList也实现了List<E>接口，那为什么子类ArrayList还是去实现一遍呢？这是想不通的地方，所以我就去查资料，有的人说是为了查看代码方便，使观看者一目了然，说法不一，但每一个让我感觉合理的，但是在stackOverFlow中找到了答案，这里其实很有趣，网址贴出来 <http://stackoverflow.com/questions/2165204/why-does-linkedhashsete-extend-hashsete-and-implement-sete>开发这个collection 的作者Josh说。这其实是一个mistake，因为他写这代码的时候觉得这个会有用处，但是其实并没什么用，但因为没什么影响，就一直留到了现在。    

* ArrayList 继承了`AbstractList`，实现了List。它是一个数组队列，提供了相关的添加、删除、修改、遍历等功能。  

* ArrayList 实现了`RandmoAccess`接口，即提供了随机访问功能。RandmoAccess是java中用来被List实现，为List提供快速访问功能的。在ArrayList中，我们即可以通过元素的序号快速获取元素对象；这就是快速随机访问。稍后，我们会比较List的“快速随机访问”和“通过Iterator迭代器访问”的效率。  

* ArrayList 实现了`Cloneable`接口，即覆盖了函数clone()，能被克隆。  
* ArrayList 实现`java.io.Serializable`接口，这意味着ArrayList支持序列化，能通过序列化去传输。  

和Vector不同，ArrayList中的操作不是线程安全的！

<!--more-->



#  构造方法

### 默认

```
// 默认空参
    public MyArrayList() {
        //数组大小 为空实例
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
```



### 传入初始容量 

```
    // 传入初始容量 构建默认数组
    public MyArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            //如果==0 就直接调用空数组
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("非法容量" + initialCapacity);
        }
    }
```



### 拷贝数组

```
//拷贝数组 构造
    public MyArrayList(Collection<? extends E> c) {
        //转换为数组
        elementData = c.toArray();
        //当前数组大小
        size = elementData.length;
        if (size != 0) {
            if (elementData.getClass() != Object[].class) {
                elementData = Arrays.copyOf(elementData, size, Object[].class);
            }
        } else {
            //替换为空数组
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```

# 默认参数

```
   //初始化容量
    private static final int DEFAULT_CAPACITY = 10;

    //空实例的默认数组
    private static final Object[] EMPTY_ELEMENTDATA = {};

    //用于默认大小的空实例
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};


    //最大数组容量  尝试分配较大的数组可能会导致OutOfMemoryError:请求的数组大小超过了VM限制
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

```



# 初始参数

```
  //数组内容
    transient Object[] elementData;

    //数组列表的大小（它包含的元素数)
    private int size;

```



# 这边照旧创建一个自定义的list

实现了简单的 add get

```
package myList;

import java.io.Serializable;
import java.util.*;
import java.util.function.Consumer;
import java.util.function.Predicate;
import java.util.function.UnaryOperator;
import java.util.stream.Stream;

/**
 * ArrayList
 *
 * @author cxc
 * @date 2019/1/23 16:17
 */
public class MyArrayList<E> extends AbstractList<E> implements Cloneable, Serializable {


    @Override
    public E set(int index, E element) {
        return super.set(index, element);
    }

    @Override
    public void add(int index, E element) {
        super.add(index, element);
    }


    @Override
    public Iterator iterator() {
        return super.iterator();
    }

    @Override
    public ListIterator listIterator() {
        return super.listIterator();
    }

    @Override
    public ListIterator listIterator(int index) {
        return super.listIterator(index);
    }

    @Override
    public List subList(int fromIndex, int toIndex) {
        return super.subList(fromIndex, toIndex);
    }

    @Override
    public boolean equals(Object o) {
        return super.equals(o);
    }

    @Override
    public int hashCode() {
        return super.hashCode();
    }

    @Override
    protected void removeRange(int fromIndex, int toIndex) {
        super.removeRange(fromIndex, toIndex);
    }


    @Override
    public boolean containsAll(Collection c) {
        return super.containsAll(c);
    }

    @Override
    public boolean addAll(Collection c) {
        return super.addAll(c);
    }

    @Override
    public boolean removeAll(Collection c) {
        return super.removeAll(c);
    }

    @Override
    public boolean retainAll(Collection c) {
        return super.retainAll(c);
    }

    @Override
    public String toString() {
        return super.toString();
    }

    @Override
    public void replaceAll(UnaryOperator operator) {

    }

    @Override
    public void sort(Comparator c) {

    }

    @Override
    public Spliterator spliterator() {
        return null;
    }

    @Override
    public boolean removeIf(Predicate filter) {
        return false;
    }

    @Override
    public Stream stream() {
        return null;
    }

    @Override
    public Stream parallelStream() {
        return null;
    }

    @Override
    public void forEach(Consumer action) {

    }


    @Override
    protected void finalize() throws Throwable {
        super.finalize();
    }


    /**
     * ########################################################################################################
     * 默认参数
     * ########################################################################################################
     */

    //初始化容量
    private static final int DEFAULT_CAPACITY = 10;

    //空实例的默认数组
    private static final Object[] EMPTY_ELEMENTDATA = {};

    //用于默认大小的空实例
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};


    //最大数组容量  尝试分配较大的数组可能会导致OutOfMemoryError:请求的数组大小超过了VM限制
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;


    /**
     * ########################################################################################################
     * 初始化参数
     * ########################################################################################################
     */

    //数组内容
    transient Object[] elementData;

    //数组列表的大小（它包含的元素数)
    private int size;


    /**
     * ########################################################################################################
     * 构造函数
     * ########################################################################################################
     */

    // 默认空参
    public MyArrayList() {
        //数组大小 为空实例
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }


    // 传入初始容量 构建默认数组
    public MyArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            //如果==0 就直接调用空数组
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("非法容量" + initialCapacity);
        }
    }


    //拷贝数组 构造
    public MyArrayList(Collection<? extends E> c) {
        //转换为数组
        elementData = c.toArray();
        //当前数组大小
        size = elementData.length;
        if (size != 0) {
            if (elementData.getClass() != Object[].class) {
                elementData = Arrays.copyOf(elementData, size, Object[].class);
            }
        } else {
            //替换为空数组
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }


    /**
     * ########################################################################################################
     * 常规方法
     * ########################################################################################################
     */
    @Override
    public boolean add(E e) {
        //首先肯定要判断下大小 然后在添加 最后计数器+1
        //初始化和扩容都在这里操作 包装了一下
        ensureCapacityInternal(size + 1);

        //直接赋值给当前位置当前
        elementData[size++] = e;
        return true;
    }


    @Override
    public E get(int index) {
        //首先判断是否在数组中有这个位置 //源码中是有针对不情况下 进行校验的 我们这边统一使用一个
        rangeCheckForAdd(index);

        //如果存在那么直接返回就是了
        //调用方法
        return elementData(index);
    }

    //整合了一个位置返回的通用操作 忽略unchecked
    @SuppressWarnings("unchecked")
    private E elementData(int index) {
        return (E) elementData[index];
    }

    //根据位置删除值
    @Override
    public E remove(int index) {
        rangeCheckForAdd(index);
        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        //如果大于0 表示在size范围内 不在结尾 将大于index的值拷贝到前面
        if (numMoved > 0) {
            System.arraycopy(elementData, index + 1, elementData, index, numMoved);
        }
        //最后将size大小扣1 然后置空 主动让jvm gc
        elementData[--size] = null;

        return oldValue;
    }

    //根据该内容 删除位置
    @Override
    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++) {
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
            }
        } else {
            for (int index = 0; index < size; index++) {
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
            }
        }
        return false;
    }

    //跳过边界检查但不跳过边界检查的私有移除方法 返回删除的值
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0) {
            System.arraycopy(elementData, index + 1, elementData, index, numMoved);
        }
        elementData[--size] = null; // clear to let GC do its work
    }

    //清空
    @Override
    public void clear() {
        modCount++;
        for (int i = 0; i < size; i++) {
            elementData[i] = null;
        }
        size = 0;
    }

    @Override
    public boolean addAll(int index, Collection c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);
        //追加内容getelementData
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }


    @Override
    public int size() {
        return size;
    }

    /**
     * ########################################################################################################
     * 内部工具方法
     * ########################################################################################################
     */

    //初始化 + 扩容的 修饰方法
    private void ensureCapacityInternal(int minCapacity) {
        //这里获取最大的容量
        int maxCapacity = calculateCapacity(elementData, minCapacity);
        //判断传入的值 是否超过容量 是否扩容
        ensureExplicitCapacity(maxCapacity);
    }

    private static int calculateCapacity(Object[] elementData, int minCapacity) {
        //如果初始化为默认的 那就 进行传入的size 与默认的容量做比较取最大
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }

    //判断是否需要扩容
    private void ensureExplicitCapacity(int minCapacity) {
        //操作记录+1 这边因为是不安全的 所以加入一个这个 在迭代器的时候 如果不一致会报错
        modCount++;
        //判断 如果当前的数量 大于 当前list的容量 进行扩容
        if (minCapacity - elementData.length > 0) {
            //扩容
            grow(minCapacity);
        }
    }

    //扩容操作
    private void grow(int minCapacity) {
        //将原数组长度赋值给old
        int oldCapacity = elementData.length;
        //新数组长度为 扩容1.5倍   位运算
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        //这里进行判断 是否超过的int最大值

        //如果扩容后的大小还小于当前存入的数量 直接将当前数量的长度 赋值给为扩容后的长度
        if (newCapacity - minCapacity < 0) {
            newCapacity = minCapacity;
        }
        //如果扩容后的大小超出了默认数组最大容量的话 那么我们尝试使用当前存入的数量进行扩容
        if (newCapacity - MAX_ARRAY_SIZE > 0) {
            newCapacity = hugeCapacity(minCapacity);
        }
        //最后使用Arrays工具类进行拷贝数组 给新的长度
        elementData = Arrays.copyOf(elementData, newCapacity);
    }


    //扩容后大小超出默认最大容量
    private static int hugeCapacity(int minCapacity) {
        //如果存入的数量小于0; 内存溢出
        if (minCapacity < 0) {
            throw new OutOfMemoryError();
        }
        //否则的话 进行判断 如果存的数量大小 也是超过了最大数组可用默认容量 取int最大值 否则就是为默认容量
        return (minCapacity > MAX_ARRAY_SIZE) ? Integer.MAX_VALUE : MAX_ARRAY_SIZE;
    }


    //在获取当前数组位置数据的时候 判断是否超出了数组存放的内容(不是数组长度 是指有数据的那一部分 用size来记录)
    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0) {
            throw new IndexOutOfBoundsException("Index: " + index + ", Size: " + size);
        }
    }


    /**
     * ########################################################################################################
     * 外部工具方法
     * ########################################################################################################
     */


//缩小数组大小 列表的当前大小 使其容量满载
    public void trimToSize() {
        modCount++;
        if (size < elementData.length) {
            elementData = (size == 0) ? EMPTY_ELEMENTDATA : Arrays.copyOf(elementData, size);
        }
    }


    //判断当前list是否存在内容
    @Override
    public boolean isEmpty() {
        return size == 0;
    }

    //判断是否存在这个值
    @Override
    public boolean contains(Object o) {
        return indexOf(o) >= 0;
    }


    //查询值的在数组中第一次存在位置 以i++ 通过遍历获取 存在返回i 不存在返回-1
    @Override
    public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++) {
                if (elementData[i] == null) {
                    return i;
                }
            }
        } else {
            for (int i = 0; i < size; i++) {
                if (o.equals(elementData[i])) {
                    return i;
                }
            }
        }
        return -1;
    }


    //查询值的在数组中最后一次存在位置 以i-- 通过遍历获取 存在返回i 不存在返回-1
    @Override
    public int lastIndexOf(Object o) {
        if (o == null) {
            for (int i = size - 1; i >= 0; i--) {
                if (elementData[i] == null) {
                    return i;
                }
            }
        } else {
            for (int i = size - 1; i >= 0; i--) {
                if (o.equals(elementData[i])) {
                    return i;
                }
            }
        }
        return -1;
    }


    //克隆集合
    @Override
    public Object clone() throws CloneNotSupportedException {
        try {
            MyArrayList<?> v = (MyArrayList<?>) super.clone();
            v.elementData = Arrays.copyOf(elementData, size);
            v.modCount = 0;
            return v;
        } catch (CloneNotSupportedException e) {
            throw new InternalError(e);
        }
    }

    //转换为数组 大小为size
    @Override
    public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }

    @Override

    //传入一个数组 将list的内容复制给数组
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        //如果传入的数组大小 小于 list的内容 则 返回复制到size 返回
        if (a.length < size) {
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
        }
        //如果传入的数组大于 list的内容 则 返回拷贝0,size 返回
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size) {
            a[size] = null;
        }
        return a;
    }
}

```





# *// c.toArray might (incorrectly) not return Object[] (see 6260652)*

出现在构造方法  public ArrayLsit(Collection<? extends E> c)中

因为这个方法是其他list 来构造当前数组 

这边会调用一个Arrays.toArray();

因为不同的类通过toArray方法返回的不一定是Object类型（如下），这样会导致在调用ArrayList增加Object元素的方法的时候出现错误。



