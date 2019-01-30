---
title: LinkedList解析
date: 2019-01-30 14:18:05
tags: jdk源码解析
---

#LinkedList链表集合

**主要注意的是 如果没有实现迭代器 那么在debug中不会出现内容**

###  LinkedList介绍 双向链表

**LinkedList简介**

* `LinkedList` 是一个继承于`AbstractSequentialList`的双向链表。它也可以被当作堆栈、队列或双端队列进行操作。
* `LinkedList` 实现` List` 接口，能对它进行队列操作。<AbstractSequentialList中继承AbstractList <-实现list接口> 
* `LinkedList` 实现 `Deque` 接口，即能将LinkedList当作双端队列使用。
* `LinkedList` 实现了`Cloneable`接口，即覆盖了函数clone()，能克隆。
* `LinkedList` 实现java.io.Serializable接口，这意味着LinkedList支持序列化，能通过序列化去传输。
* LinkedList 是非同步的。 



# 实现自定义LinkedList

 这边也是根据node节点来保存的 

实现前驱 后继 +本地节点

### 默认参数

```java
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

```java
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
```





### 对外方法 队列



### 对内方法  队列





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

