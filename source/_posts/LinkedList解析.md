---
title: LinkedList解析
date: 2019-01-30 14:18:05
tags: [jdk源码解析]
---

# LinkedList链表集合

**主要注意的是 如果没有实现迭代器 那么在debug中不会出现内容**

###  LinkedList介绍 双向链表

**LinkedList简介**

* `LinkedList` 是一个继承于`AbstractSequentialList`的双向链表。它也可以被当作堆栈、队列或双端队列进行操作。
* `LinkedList` 实现` List` 接口，能对它进行队列操作。<AbstractSequentialList中继承AbstractList <-实现list接口> 
* `LinkedList` 实现 `Deque` 接口，即能将LinkedList当作双端队列使用。
* `LinkedList` 实现了`Cloneable`接口，即覆盖了函数clone()，能克隆。
* `LinkedList` 实现java.io.Serializable接口，这意味着LinkedList支持序列化，能通过序列化去传输。
* LinkedList 是非同步的。 

<!--more-->

# 实现自定义LinkedList

 这边也是根据node节点来保存的 

实现前驱 后继 +本地节点

### 默认参数

```
    //node节点数量
    transient int size = 0;

    // 前驱节点
    transient Node<E> first;

    //后继节点 (这边是标记为最后一个节点)
    transient Node<E> last;

    protected transient int modCount = 0;
```



### 构造函数

```
 public MyLinkList() {
    }
```



### node节点相关

```
 private static class Node<E> {
        E item;
        //后继
        Node<E> next;
        //前驱
        Node<E> prev;

        //构造函数
        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }

    //根据下标 去遍历
    Node<E> getNode(int index) {

        //如果index小于 节点数量一半 从0遍历++ 遍历到 index节点
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++) {
                x = x.next;
            }
            return x;
        } else {
            //如果index大于 节点数量一半 从size遍历-- 遍历到 index节点
            Node<E> x = last;
            for (int i = size - 1; i > index; i--) {
                x = x.prev;
            }
            return x;
        }
    }


    //将当前节点分离出去
    E unlink(Node<E> x) {
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        //如果为头节点  前驱处理
        if (prev == null) {
            //将全局变量的头节点向后走一步
            first = next;
        } else {
            //否则 头节点的后继节点为 当前节点的后继节点
            prev.next = next;
            //当前节点 前驱节点 置空
            x.prev = null;
        }

        //如果为尾节点  后继处理
        if (next == null) {
            //将全局变量的尾节点向前走一步
            last = prev;
        } else {
            //否则 原节点的后继节点的前驱节点为当前节点的前驱节点
            next.prev = prev;
            //置空当前节点
            x.next = null;
        }

        //全置空了 帮助GC null
        x.item = null;
        //计数器-1
        size--;
        //修改次数+1
        modCount++;
        return element;
    }

```



###  对外方法 集合的





### 内部方法 集合的

```
  //获取链表内部node数量
    @Override
    public int size() {
        return size;
    }

    /**
     * linkedList 最结尾追加一个节点
     */
    private void linkLast(E e) {
        //这里 l变量 表示最后一个节点的位置
        final Node<E> l = last;
        //构造函数 将最后一个节点 变成新添加节点的前驱节点
        final Node<E> newNode = new Node<>(l, e, null);

        //修改全局变量 将是最后一个节点改为最后新添加的这个节点
        last = newNode;

        //最后在进行判断一下  如果是第一次添加 全局第一节点 为null 就将新添加的node节点 修改为frist节点 否最后一个节点的后继节点就是新创建的这node节点
        if (l == null) {
            first = newNode;
        } else {
            l.next = newNode;
        }
        //追加list的node节点数量
        size++;
        modCount++;
    }


    //在非空节点之前添加节点
    void linkBefore(E e, Node<E> succ) {
        //将原节点的前驱节点 拎出来
        final Node<E> pred = succ.prev;
        //创建一个新节点    前驱为 原节点的前驱 后继为原节点
        final Node<E> newNode = new Node<>(pred, e, succ);
        //修改源节点的前驱为 新节点
        succ.prev = newNode;

        //这里进行判断是否是头节点 否的话 就修改原前驱节点的后继节点为 新节点
        if (pred == null) {
            first = newNode;
        } else {
            pred.next = newNode;
        }

        //最后计数器+1 修改次数+1
        size++;
        modCount++;
    }


    //查看传入的下标是否超出了list最大节点数量
    private boolean isPositionIndex(int index) {
        return index >= 0 && index <= size;
    }

    //如果下表大于list节点数量 抛出异常 下标超出
    private void checkPositionIndex(int index) {
        if (!isPositionIndex(index)) {
            throw new IndexOutOfBoundsException("Index: " + index + ", Size: " + size());
        }
    }

 /**
     * 分离第一个非空节点
     */
    private E unlinkFirst(Node<E> f) {
        //首节点的数据
        final E element = f.item;
        //后一个节点
        final Node<E> next = f.next;
        //设置为空
        f.item = null;
        f.next = null;
        //帮助GC
        //首节点为next
        first = next;
        //如果next为空 最后一个节点也是空
        if (next == null) {
            last = null;
        } else {
            //不为空那么修改前驱为空
            next.prev = null;
        }
        //数量--
        size--;
        modCount++;
        return element;
    }

    /**
     * 分离最后一个非空节点
     */
    private E unlinkLast(Node<E> l) {
        //获取节点的数据
        final E element = l.item;
        //获取前驱节点
        final Node<E> prev = l.prev;
        //当前节点设置为空
        l.item = null;
        l.prev = null;
        //帮助GC

        //前驱节点设置为尾节点
        last = prev;
        //如果前驱节点为空 那么说明没有元素存在
        if (prev == null) {
            first = null;
        } else {
            //否则尾节点设置为null
            prev.next = null;
        }
        //数量--
        size--;
        modCount++;
        return element;
    }
```





### 对外方法 队列

```


    @Override
    public void addFirst(E e) {
        linkFirst(e);
    }

    @Override
    public void addLast(E e) {
        linkLast(e);
    }

    //插入头部
    @Override
    public boolean offerFirst(E e) {
        addFirst(e);
        return true;
    }

    //插入尾部
    @Override
    public boolean offerLast(E e) {
        linkLast(e);
        return true;
    }

    @Override
    public E removeFirst() {
        final Node<E> f = first;
        if (f == null) {
            throw new NoSuchElementException();
        }
        return unlinkFirst(f);
    }

    @Override
    public E removeLast() {
        final Node<E> f = last;
        if (f == null) {
            throw new NoSuchElementException();
        }
        return unlinkLast(f);
    }

    //不会报错的删除
    @Override
    public E pollFirst() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }

    //不会报错的删除
    @Override
    public E pollLast() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkLast(f);
    }


    @Override
    public E getFirst() {
        final Node<E> f = first;
        if (f == null) {
            throw new NoSuchElementException();
        }
        return f.item;
    }

    @Override
    public E getLast() {
        final Node<E> f = last;
        if (f == null) {
            throw new NoSuchElementException();
        }
        return f.item;
    }


    @Override
    public E peekFirst() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
    }

    @Override
    public E peekLast() {
        final Node<E> f = last;
        return (f == null) ? null : f.item;
    }

    //删除列表中该元素第一次出现
    @Override
    public boolean removeFirstOccurrence(Object o) {
        return remove(o);
    }

    //删除该元素最后一次出现
    @Override
    public boolean removeLastOccurrence(Object o) {
        if (o == null) {
            for (Node<E> x = last; x != null; x = x.prev) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = last; x != null; x = x.prev) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }


    //检索但不删除此列表的头（第一个元素）。  为空会报错
    @Override
    public E element() {
        return getFirst();
    }


    //检索但不删除此列表的头（第一个元素） 不会报错
    @Override
    public E peek() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
    }

    @Override
    public boolean offer(E e) {
        return add(e);
    }

    //检索删除此列表的头（第一个元素）。
    @Override
    public E remove() {
        return removeFirst();
    }


    //分离第一个元素不报错
    @Override
    public E poll() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }
    
    //插入头部
    @Override
    public void push(E e) {
        addFirst(e);
    }

    //*从该列表表示的堆栈中弹出一个元素。删除并返回此列表的第一个元素。 空值会报错
    @Override
    public E pop() {
        return removeFirst();
    }

    @Override
    public Iterator<E> descendingIterator() {
        return new DescendingIterator();
    }

    /**
     * 通过listitr.previous提供降序迭代器的适配器
     */
    private class DescendingIterator implements Iterator<E> {
        private final MyLinkList.ListItr itr = new MyLinkList.ListItr(size());

        @Override
        public boolean hasNext() {
            return itr.hasPrevious();
        }

        @Override
        public E next() {
            return (E) itr.previous();
        }

        @Override
        public void remove() {
            itr.remove();
        }
    }

```



### 迭代器方法

```
 @Override
    public ListIterator<E> listIterator(int index) {
        checkPositionIndex(index);
        return new ListItr(index);
    }

    private class ListItr implements ListIterator<E> {
        private Node<E> lastReturned;
        private Node<E> next;
        private int nextIndex;
        private int expectedModCount = modCount;

        ListItr(int index) {
            // assert isPositionIndex(index);
            next = (index == size) ? null : getNode(index);
            nextIndex = index;
        }

        @Override
        public boolean hasNext() {
            return nextIndex < size;
        }

        @Override
        public E next() {
            checkForComodification();
            if (!hasNext()) {
                throw new NoSuchElementException();
            }
            lastReturned = next;
            next = next.next;
            nextIndex++;
            return lastReturned.item;
        }

        @Override
        public boolean hasPrevious() {
            return nextIndex > 0;
        }

        @Override
        public E previous() {
            checkForComodification();
            if (!hasPrevious()) {
                throw new NoSuchElementException();
            }

            lastReturned = next = (next == null) ? last : next.prev;
            nextIndex--;
            return lastReturned.item;
        }

        @Override
        public int nextIndex() {
            return nextIndex;
        }

        @Override
        public int previousIndex() {
            return nextIndex - 1;
        }

        @Override
        public void remove() {
            checkForComodification();
            if (lastReturned == null) {
                throw new IllegalStateException();
            }

            Node<E> lastNext = lastReturned.next;
            unlink(lastReturned);
            if (next == lastReturned) {
                next = lastNext;
            } else {
                nextIndex--;
            }
            lastReturned = null;
            expectedModCount++;
        }

        @Override
        public void set(E e) {
            if (lastReturned == null) {
                throw new IllegalStateException();
            }
            checkForComodification();
            lastReturned.item = e;
        }

        @Override
        public void add(E e) {
            checkForComodification();
            lastReturned = null;
            if (next == null) {
                linkLast(e);
            } else {
                linkBefore(e, next);
            }
            nextIndex++;
            expectedModCount++;
        }

        @Override
        public void forEachRemaining(Consumer<? super E> action) {
            Objects.requireNonNull(action);
            while (modCount == expectedModCount && nextIndex < size) {
                action.accept(next.item);
                lastReturned = next;
                next = next.next;
                nextIndex++;
            }
            checkForComodification();
        }

        final void checkForComodification() {
            if (modCount != expectedModCount) {
                throw new ConcurrentModificationException();
            }
        }
    }
```



# 完整

```
package myLinkList;

import java.util.*;
import java.util.function.Consumer;

/**
 * @author cxc
 * @date 2019/1/30 14:09
 */
public class MyLinkList<E> extends AbstractSequentialList<E> implements Cloneable, Deque<E>, java.io.Serializable {

    /**
     * 序列化
     */
    private static final long serialVersionUID = 876323262645176324L;


    /**
     * ########################################################################################################
     * 默认参数
     * ########################################################################################################
     */

    //node节点数量
    transient int size = 0;

    // 前驱节点
    transient Node<E> first;

    //后继节点 (这边是标记为最后一个节点)
    transient Node<E> last;

    protected transient int modCount = 0;


    /**
     * ########################################################################################################
     * 构造函数 (简单 因为 这边是 根据 节点信息保存的 所以初始化就简单了 初始化的时候 会使用默认的前驱节点)
     * ########################################################################################################
     */

    public MyLinkList() {
    }


    /**
     * ########################################################################################################
     * node节点相关
     * ########################################################################################################
     */

    private static class Node<E> {
        E item;
        //后继
        Node<E> next;
        //前驱
        Node<E> prev;

        //构造函数
        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }

    //根据下标 去遍历
    Node<E> getNode(int index) {

        //如果index小于 节点数量一半 从0遍历++ 遍历到 index节点
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++) {
                x = x.next;
            }
            return x;
        } else {
            //如果index大于 节点数量一半 从size遍历-- 遍历到 index节点
            Node<E> x = last;
            for (int i = size - 1; i > index; i--) {
                x = x.prev;
            }
            return x;
        }
    }


    //将当前节点分离出去
    E unlink(Node<E> x) {
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        //如果为头节点  前驱处理
        if (prev == null) {
            //将全局变量的头节点向后走一步
            first = next;
        } else {
            //否则 头节点的后继节点为 当前节点的后继节点
            prev.next = next;
            //当前节点 前驱节点 置空
            x.prev = null;
        }

        //如果为尾节点  后继处理
        if (next == null) {
            //将全局变量的尾节点向前走一步
            last = prev;
        } else {
            //否则 原节点的后继节点的前驱节点为当前节点的前驱节点
            next.prev = prev;
            //置空当前节点
            x.next = null;
        }

        //全置空了 帮助GC null
        x.item = null;
        //计数器-1
        size--;
        //修改次数+1
        modCount++;
        return element;
    }


    /**
     * ########################################################################################################
     * 对外方法   集合的方法
     * ########################################################################################################
     */
    @Override
    public boolean add(E e) {
        //因为是链表结构 这边添加就等于在 最后一个节点的后继上加入一个值
        linkLast(e);
        return true;
    }


    @Override
    public E get(int index) {
        //这边照旧进行判断数组溢出的问题
        checkPositionIndex(index);
        //这边因为是链表形式 我们可以用分两段进行查询  (这里为什么不用 二分查询 一直切一半 去循环)
        return getNode(index).item;
    }

    //添加到指定节点下标后
    @Override
    public void add(int index, E element) {
        //这个简单理解  创建一个节点 为就是原节点的后继的节点新节点 再去修改原后继节点的前驱节点改为新节点

        //首先照旧判断是否溢出
        checkPositionIndex(index);

//判断 如果该下标为最后一个存在node节点 那直接追加到最后 类似add();
        if (index == size) {
            linkLast(element);
        } else {
            //否则进行添加到前面
            linkBefore(element, getNode(index));
        }
    }


    @Override
    public void clear() {
        //清除节点 前后节点
        for (Node<E> x = first; x != null; ) {
            Node<E> next = x.next;
            x.item = null;
            x.next = null;
            x.prev = null;
            x = next;
        }
        first = last = null;
        size = 0;
        modCount++;
    }


    @Override
    public E remove(int index) {
        //判断溢出
        checkPositionIndex(index);
        //将当前节点抽出
        return unlink(getNode(index));
    }


    //在尾节点加入all
    @Override
    public boolean addAll(Collection<? extends E> c) {
        return addAll(size, c);
    }

    //在指定index前加入all
    @Override
    public boolean addAll(int index, Collection<? extends E> c) {
        //照旧检查是否溢出
        checkPositionIndex(index);

        //转成数组
        Object[] a = c.toArray();
        //获取数组长度
        int numNew = a.length;
        //如果长度为0 就直接结束了 因为不需要增加了
        if (numNew == 0) {
            return false;
        }

        //设置两个变量 前驱节点 后继节点
        Node<E> pred, succ;
        //如果 当前节点为尾节点 后继节点为null 前驱节点为last
        if (index == size) {
            succ = null;
            pred = last;
        } else {
            //否则 后继节点为index  前驱节点为当前节点的前驱节点
            succ = getNode(index);
            pred = succ.prev;
        }

        //这里开始遍历赋值
        for (Object o : a) {
            //值
            @SuppressWarnings("unchecked") E e = (E) o;
            //创建一个新节点
            Node<E> newNode = new Node<>(pred, e, null);

            //如果前驱节点为null 说明是头节点
            if (pred == null) {
                //直接赋值给头节点
                first = newNode;
            } else {
                //否则前驱节点后next
                //这里暂时先不考虑原节点 最后再进行中间插入
                pred.next = newNode;
            }
            //这边就是继续遍历
            pred = newNode;
        }

        //到了这里all 全部都已经排好了队

        //如果原节点为空 那为节点就是 尾节点就是最后处理的节点
        if (succ == null) {
            last = pred;
        } else {
            //否则 在尾巴加入原节点 绑定前驱后继
            pred.next = succ;
            succ.prev = pred;
        }

        //计数器+数组的长度
        size += numNew;
        //操作数+1
        modCount++;
        return true;


    }


    //转换数组
    @Override
    public Object[] toArray() {
        //创建一个满载的数组
        Object[] result = new Object[size];
        int i = 0;
        //遍历 直到遍历到最后一个next为空的
        for (Node<E> x = first; x != null; x = x.next) {
            result[i++] = x.item;
        }
        return result;
    }

    //转换给指定数组
    @Override
    public <T> T[] toArray(T[] a) {
        //如果需要传入的数组大小不够存放
        if (a.length < size) {
            //改变数组长度 new一个
            a = (T[]) java.lang.reflect.Array.newInstance(a.getClass().getComponentType(), size);
        }
        int i = 0;
        Object[] result = a;
        //照旧遍历
        for (Node<E> x = first; x != null; x = x.next) {
            result[i++] = x.item;
        }

        //这边另外一个种情况  a的大小 大于list长度
        if (a.length > size) {
            a[size] = null;
        }
        return a;
    }


    //删除指定节点
    @Override
    public boolean remove(Object o) {
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null) {
                    //将查询到的节点剔除
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item)) {
                    //将查询到的节点剔除
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }


    //拷贝
    @SuppressWarnings("unchecked")
    @Override
    protected Object clone() throws CloneNotSupportedException {
        MyLinkList<E> clone = (MyLinkList<E>) super.clone();

        clone.first = clone.last = null;
        clone.size = 0;
        clone.modCount = 0;
        //递归拷贝
        for (Node<E> x = first; x != null; x = x.next) {
            clone.add(x.item);
        }
        return clone;
    }

    /**
     * ########################################################################################################
     * 对外方法   队列的方法
     * ########################################################################################################
     */

    @Override
    public void addFirst(E e) {
        linkFirst(e);
    }

    @Override
    public void addLast(E e) {
        linkLast(e);
    }

    //插入头部
    @Override
    public boolean offerFirst(E e) {
        addFirst(e);
        return true;
    }

    //插入尾部
    @Override
    public boolean offerLast(E e) {
        linkLast(e);
        return true;
    }

    @Override
    public E removeFirst() {
        final Node<E> f = first;
        if (f == null) {
            throw new NoSuchElementException();
        }
        return unlinkFirst(f);
    }

    @Override
    public E removeLast() {
        final Node<E> f = last;
        if (f == null) {
            throw new NoSuchElementException();
        }
        return unlinkLast(f);
    }

    //不会报错的删除
    @Override
    public E pollFirst() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }

    //不会报错的删除
    @Override
    public E pollLast() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkLast(f);
    }


    @Override
    public E getFirst() {
        final Node<E> f = first;
        if (f == null) {
            throw new NoSuchElementException();
        }
        return f.item;
    }

    @Override
    public E getLast() {
        final Node<E> f = last;
        if (f == null) {
            throw new NoSuchElementException();
        }
        return f.item;
    }


    @Override
    public E peekFirst() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
    }

    @Override
    public E peekLast() {
        final Node<E> f = last;
        return (f == null) ? null : f.item;
    }

    //删除列表中该元素第一次出现
    @Override
    public boolean removeFirstOccurrence(Object o) {
        return remove(o);
    }

    //删除该元素最后一次出现
    @Override
    public boolean removeLastOccurrence(Object o) {
        if (o == null) {
            for (Node<E> x = last; x != null; x = x.prev) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = last; x != null; x = x.prev) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }


    //检索但不删除此列表的头（第一个元素）。  为空会报错
    @Override
    public E element() {
        return getFirst();
    }


    //检索但不删除此列表的头（第一个元素） 不会报错
    @Override
    public E peek() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
    }

    @Override
    public boolean offer(E e) {
        return add(e);
    }

    //检索删除此列表的头（第一个元素）。
    @Override
    public E remove() {
        return removeFirst();
    }


    //分离第一个元素不报错
    @Override
    public E poll() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }

    //插入头部
    @Override
    public void push(E e) {
        addFirst(e);
    }

    //*从该列表表示的堆栈中弹出一个元素。删除并返回此列表的第一个元素。 空值会报错
    @Override
    public E pop() {
        return removeFirst();
    }

    @Override
    public Iterator<E> descendingIterator() {
        return new DescendingIterator();
    }

    /**
     * 通过listitr.previous提供降序迭代器的适配器
     */
    private class DescendingIterator implements Iterator<E> {
        private final MyLinkList.ListItr itr = new MyLinkList.ListItr(size());

        @Override
        public boolean hasNext() {
            return itr.hasPrevious();
        }

        @Override
        public E next() {
            return (E) itr.previous();
        }

        @Override
        public void remove() {
            itr.remove();
        }
    }

    /**
     * ########################################################################################################
     * 迭代器方法
     * ########################################################################################################
     */

    @Override
    public ListIterator<E> listIterator(int index) {
        checkPositionIndex(index);
        return new ListItr(index);
    }

    private class ListItr implements ListIterator<E> {
        private Node<E> lastReturned;
        private Node<E> next;
        private int nextIndex;
        private int expectedModCount = modCount;

        ListItr(int index) {
            // assert isPositionIndex(index);
            next = (index == size) ? null : getNode(index);
            nextIndex = index;
        }

        @Override
        public boolean hasNext() {
            return nextIndex < size;
        }

        @Override
        public E next() {
            checkForComodification();
            if (!hasNext()) {
                throw new NoSuchElementException();
            }
            lastReturned = next;
            next = next.next;
            nextIndex++;
            return lastReturned.item;
        }

        @Override
        public boolean hasPrevious() {
            return nextIndex > 0;
        }

        @Override
        public E previous() {
            checkForComodification();
            if (!hasPrevious()) {
                throw new NoSuchElementException();
            }

            lastReturned = next = (next == null) ? last : next.prev;
            nextIndex--;
            return lastReturned.item;
        }

        @Override
        public int nextIndex() {
            return nextIndex;
        }

        @Override
        public int previousIndex() {
            return nextIndex - 1;
        }

        @Override
        public void remove() {
            checkForComodification();
            if (lastReturned == null) {
                throw new IllegalStateException();
            }

            Node<E> lastNext = lastReturned.next;
            unlink(lastReturned);
            if (next == lastReturned) {
                next = lastNext;
            } else {
                nextIndex--;
            }
            lastReturned = null;
            expectedModCount++;
        }

        @Override
        public void set(E e) {
            if (lastReturned == null) {
                throw new IllegalStateException();
            }
            checkForComodification();
            lastReturned.item = e;
        }

        @Override
        public void add(E e) {
            checkForComodification();
            lastReturned = null;
            if (next == null) {
                linkLast(e);
            } else {
                linkBefore(e, next);
            }
            nextIndex++;
            expectedModCount++;
        }

        @Override
        public void forEachRemaining(Consumer<? super E> action) {
            Objects.requireNonNull(action);
            while (modCount == expectedModCount && nextIndex < size) {
                action.accept(next.item);
                lastReturned = next;
                next = next.next;
                nextIndex++;
            }
            checkForComodification();
        }

        final void checkForComodification() {
            if (modCount != expectedModCount) {
                throw new ConcurrentModificationException();
            }
        }
    }

    /**
     * ########################################################################################################
     * 内部方法
     * ########################################################################################################
     */

    //获取链表内部node数量
    @Override
    public int size() {
        return size;
    }

    /**
     * 在头部添加一个元素
     *
     * @param e
     */
    private void linkFirst(E e) {
        //获取头部元素
        final Node<E> f = first;
        //创建一个新元素 后继为旧头部
        final Node<E> newNode = new Node<>(null, e, f);
        //修改头部元素为新元素
        first = newNode;
        //判断 f是否初次
        if (f == null) {
            last = newNode;
        } else {
            //如果存在值那么 修改原头部的前驱为新节点
            f.prev = newNode;
        }
        //++
        size++;
        modCount++;
    }

    /**
     * linkedList 最结尾追加一个节点
     */
    private void linkLast(E e) {
        //这里 l变量 表示最后一个节点的位置
        final Node<E> l = last;
        //构造函数 将最后一个节点 变成新添加节点的前驱节点
        final Node<E> newNode = new Node<>(l, e, null);

        //修改全局变量 将是最后一个节点改为最后新添加的这个节点
        last = newNode;

        //最后在进行判断一下  如果是第一次添加 全局第一节点 为null 就将新添加的node节点 修改为frist节点 否最后一个节点的后继节点就是新创建的这node节点
        if (l == null) {
            first = newNode;
        } else {
            l.next = newNode;
        }
        //追加list的node节点数量
        size++;
        modCount++;
    }


    //在非空节点之前添加节点
    void linkBefore(E e, Node<E> succ) {
        //将原节点的前驱节点 拎出来
        final Node<E> pred = succ.prev;
        //创建一个新节点    前驱为 原节点的前驱 后继为原节点
        final Node<E> newNode = new Node<>(pred, e, succ);
        //修改源节点的前驱为 新节点
        succ.prev = newNode;

        //这里进行判断是否是头节点 否的话 就修改原前驱节点的后继节点为 新节点
        if (pred == null) {
            first = newNode;
        } else {
            pred.next = newNode;
        }

        //最后计数器+1 修改次数+1
        size++;
        modCount++;
    }


    //查看传入的下标是否超出了list最大节点数量
    private boolean isPositionIndex(int index) {
        return index >= 0 && index <= size;
    }

    //如果下表大于list节点数量 抛出异常 下标超出
    private void checkPositionIndex(int index) {
        if (!isPositionIndex(index)) {
            throw new IndexOutOfBoundsException("Index: " + index + ", Size: " + size());
        }
    }

    /**
     * 分离第一个非空节点
     */
    private E unlinkFirst(Node<E> f) {
        //首节点的数据
        final E element = f.item;
        //后一个节点
        final Node<E> next = f.next;
        //设置为空
        f.item = null;
        f.next = null;
        //帮助GC
        //首节点为next
        first = next;
        //如果next为空 最后一个节点也是空
        if (next == null) {
            last = null;
        } else {
            //不为空那么修改前驱为空
            next.prev = null;
        }
        //数量--
        size--;
        modCount++;
        return element;
    }

    /**
     * 分离最后一个非空节点
     */
    private E unlinkLast(Node<E> l) {
        //获取节点的数据
        final E element = l.item;
        //获取前驱节点
        final Node<E> prev = l.prev;
        //当前节点设置为空
        l.item = null;
        l.prev = null;
        //帮助GC

        //前驱节点设置为尾节点
        last = prev;
        //如果前驱节点为空 那么说明没有元素存在
        if (prev == null) {
            first = null;
        } else {
            //否则尾节点设置为null
            prev.next = null;
        }
        //数量--
        size--;
        modCount++;
        return element;
    }
}

```

