---
title: 基于jdk1.8HashMap理解
date: 2019-01-01 16:45:47
tags: [java,hashMap]
---

#  简单实现MyHashMap(持续更新)



![hashMap](/img/2019-1-2/HashMap.jpeg)



## 暂时没有实现迭代器 只有get,set,扩容

ps: 目前只做到了 对于链表的扩容 

红黑树 后面继续看看

<!--more-->

```java
package MyHashMap;

import java.io.Serializable;
import java.util.*;

/**
 * 基于jdk1.8的hashMap (忽略红黑树) 链表插入方式 尾插
 *
 * @author cxc
 * @date 2018/12/31 14:29
 */
public class MyHashMap<K, V> extends AbstractMap<K, V> implements Map<K, V>, Cloneable, Serializable {
    /**
     * 序列化
     */
    private static final long serialVersionUID = 4318938644099875728L;


    /**
     * ########################################################################################################
     *                              >>>>>>>>>>>>>>>Node节点信息
     * ########################################################################################################
     */
    /**
     * 静态内部类
     * Map Node节点  ->槽点 用来存放key value
     */
    static class Node<K, V> implements Map.Entry<K, V> {
        /**
         * 用来计算hash
         */
        final int hash;
        /**
         * 我们输入的key
         */
        final K key;
        /**
         * 我们输入的value
         */
        V value;
        /**
         * 下一个节点
         */
        MyHashMap.Node<K, V> next;

        /**
         * 节点的构造方法
         */
        Node(int hash, K key, V value, MyHashMap.Node<K, V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        @Override
        public K getKey() {
            return key;
        }

        @Override
        public V getValue() {
            return value;
        }

        /**
         * 赋值方法
         *
         * @param newValue
         * @return
         */
        @Override
        public V setValue(V newValue) {
            //如果有原值
            V oldValue = value;
            //新值赋值给value;
            value = newValue;
            //最后返回旧的value
            return oldValue;
        }

        /**
         * 重写hashCode散列值算法
         *
         * @return
         */
        @Override
        public int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        /**
         * 重写equals方法
         *
         * @param obj
         * @return
         */
        @Override
        public boolean equals(Object obj) {
            //如果传入的obj对象=原来的 返回相等
            if (obj == this) {
                return true;
            }
            //如果Obj 也是Entry类型   继续校验  否则 返回false;
            if (obj instanceof Map.Entry) {
                Map.Entry<?, ?> e = (Map.Entry<?, ?>) obj;
                //如果obj的key==key 并且 obj的value==value 表示 这两个node节点相等
                if (Objects.equals(key, e.getKey()) &&
                        Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }

        /**
         * 重写节点的toString方法
         *
         * @return
         */
        @Override
        public String toString() {
            return "Node{" +
                    "hash=" + hash +
                    ", key=" + key +
                    ", value=" + value +
                    ", next=" + next +
                    '}';
        }
    }

    /**
     * 创建一个非TREE的节点
     *
     * @param hash
     * @param key
     * @param value
     * @param next
     * @return
     */
    private Node<K, V> newNode(int hash, K key, V value, Node<K, V> next) {
        return new Node<>(hash, key, value, next);
    }

    /**
     * 实现map.get及相关方法
     *
     * @param hash 值
     * @param key  键
     * @ 返回节点，如果没有则为空
     */
    final Node<K, V> getNode(int hash, Object key) {
        //赋值操作
        //数组
        Node<K, V>[] tab;
        //hash对应的节点
        Node<K, V> first;
        //节点信息中链表的后继节点
        Node<K, V> e;
        //数组长度
        int n;
        //键名
        K k;

        //如果table初始化了而且数组里有内容 并且这个hash值在table中不为空
        if ((tab = table) != null && (n = tab.length) > 0 && (first = tab[(n - 1) & hash]) != null) {

            //如果这个位置的hash值等于传入的hash key=key 或者key不等于null 传入的key=k 直接返回这个节点
            if (first.hash == hash && ((k = first.key) == key || (key != null && key.equals(k)))) {
                return first;
            }
            //如果当前节点的next节点不为空
            if ((e = first.next) != null) {
                //todo 红黑树 暂不实现
                //链表方式
                //直接递归查询 while 查询下一个节点 查询到直接返回
                do {
                    if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) {
                        return e;
                    }
                } while ((e = e.next) != null);
            }
        }
        return null;
    }

    /**
     * ########################################################################################################
     *                                              Node节点信息<<<<<<<<<<<<<<<<<<
     * ########################################################################################################
     */

    /**
     * ########################################################################################################
     *                              >>>>>>>>>>>>>>>默认初始化参数
     * ########################################################################################################
     */
    /**
     * 默认的数组大小16
     */
    private static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
    /**
     * 默认负载系数
     */
    private static final float DEFAULT_LOAD_FACTOR = 0.75f;
    /**
     * Map最大容量
     */
    private static final int MAXIMUM_CAPACITY = 1 << 30;
/**
 * ########################################################################################################
 *                                              默认初始化参数<<<<<<<<<<<<<
 * ########################################################################################################
 */

    /**
     * ########################################################################################################
     *                              >>>>>>>>>>>>>>>初始化参数
     * ########################################################################################################
     */
    /**
     * 阀值/默认“初始”容量。
     * 如果未分配初始大小 则表示初始容量
     * 当前数组内的key-value映射如果大于这个阀值 就可以开始扩容
     * HashMap的扩容阈值，在HashMap中存储的Node键值对超过这个数量时，自动扩容容量为原来的二倍。
     */
    private int threshold;
    /**
     * 当前HashMap的负载系数
     * 也就是说如果hashMap的数组的使用达到数组X系数的大小的话 用来扩容数组的
     */
    private final float loadFactor;
    /**
     * 表示当前Map的数组
     * HashMap的哈希桶数组，非常重要的存储结构，用于存放表示键值对数据的Node元素。
     */
    transient Node<K, V>[] table;

    /**
     * //HashMap中实际存在的Node数量，
     * 注意这个数量不等于table的长度，甚至可能大于它，因为在table的每个节点上是一个链表（或RBT）结构，可能不止有一个Node元素存在。
     */
    private transient int size;

    /**
     * 此哈希图在结构上被修改的次数 结构修改是指更改 hashmap或以其他方式修改其内部结构
     * （例如， 重新整理。此字段用于使集合视图上的迭代器
     */
    private transient int modCount;

    /**
     * ########################################################################################################
     *                                             初始化参数<<<<<<<<<<<<<<<<<<
     * ########################################################################################################
     */


    /**
     * ########################################################################################################
     *                              >>>>>>>>>>>>>>>构造方法
     * ########################################################################################################
     */

    /**
     * 默认构造方法 ->初始化负载系数
     * 在put的时候默认初始化数组长度为16
     */
    public MyHashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
    }

    /**
     * 构造方法 ->传入扩容系数
     * 调用MyHashMap(int threshold, float loadFactor)
     * 修改默认数组大小 ->会自动根据传入的值进行处理 换成2的次幂
     *
     * @param threshold
     */
    public MyHashMap(int threshold) {
        this(threshold, DEFAULT_LOAD_FACTOR);
    }

    /**
     * 创建一个空的hashMap
     * 构造方法 ->传入 初始的大小,负载系数
     * 进行初始化
     *
     * @param initialCapacity
     * @param loadFactor
     */
    public MyHashMap(int initialCapacity, float loadFactor) {
        //参数校验
        if (initialCapacity < 0) {
            throw new IllegalArgumentException("初始容量大小不为小于0" + "传入参数为:" + initialCapacity);
        }
        //如果初始化容量>最大容量
        if (initialCapacity > MAXIMUM_CAPACITY) {
            initialCapacity = MAXIMUM_CAPACITY;
        }
        //判断传入的负载因子 isNaN 防止类似0.0f/0.0f  结果:NaN  这边情况出现
        if (loadFactor <= 0 || Float.isNaN(loadFactor)) {
            throw new IllegalArgumentException("初始化负载因子错误" + "传入的参数为" + loadFactor);
        }

        //这边会处理初始化阀值(属于优化部分)  变成2的次幂
        this.threshold = tableSizeFor(initialCapacity);
        System.out.println("初始化容器大小为:" + threshold);
        //初始化 负载因数
        this.loadFactor = loadFactor;
    }

    /**
     * ########################################################################################################
     *                                                 构造方法<<<<<<<<<<<<<<<<<
     * ########################################################################################################
     */


    /**
     * ########################################################################################################
     *                                 >>>>>>>>>>>>>>>普通方法 put,get之类
     * ########################################################################################################
     */


    /**
     * 如果计算的hash相等 去查看数组的槽点是否为空
     * 为空就直接插入
     * 不为空判断key的值是否相等
     * 不相等的就新增一个
     * 相等的就更新该节点
     * 如果当前size大于我们的阀值 就调用扩容方法进行 重新new 一个两倍大小的数组 替换 重新计算每一个的hash值 进行插入
     *
     * @param key
     * @param value
     * @return
     */
    @Override
    public V put(K key, V value) {
        //实际操作方法
        return putVal(hash(key), key, value, false, true);
    }

    /**
     * 具体put操作
     *
     * @param hash         计算hash
     * @param key
     * @param value
     * @param onlyIfAbsent 如果为true，则不更改现有值
     * @param evict        如果为false，则表处于创建模式。
     * @return todo 最后一个参数没看懂什么意思 看了下是关于LinkedHashMap ????
     */
    private V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
        //用来标记原数组;
        Node<K, V>[] tab;
        //用来标记操作的数组
        Node<K, V> p;
        //n=数组长度
        int n, i;

        //判断我们的map是否初始化了 没有初始化 调用 resize方法初始化一下;
        if ((tab = table) == null || (n = tab.length) == 0) {
            n = (tab = reSize()).length;
        }
        //如果这个key的hash值不在tab数组的位置中 则直接新增一个Node节点
        if ((p = tab[i = (n - 1) & hash]) == null) {
            tab[i] = newNode(hash, key, value, null);
        } else {
            //否则 开始计算节点内容 添加至链表或者替换
            //新node节点
            Node<K, V> e;
            //标记key
            K k;
            //如果 hash相等 key 也相等 直接替换
            if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k)))) {
                //将p的引用指向e
                e = p;
            }
            //else{
            //    //todo 如果是红黑树形式 另外忽略
            //}
            else {
                //如果当前数组中hash值存在 加入到链表后一个节点上 尾插 1.7之前头插入
                //死循环遍历
                for (int binCount = 0; ; ++binCount) {
                    // 知道某一个链表节点的后继节点为null 插入 跳出循环
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        //todo 1.8需要转红黑树 链表长度不能超过8
                        break;
                    }
                    //如果查找到的next节点上 key hash=传入的值 说明找到存在的node节点信息了 跳出循环
                    if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) {
                        break;
                    }
                    //如果p.next不为空 将e的值赋给p继续下一次循环
                    p = e;
                }
            }
            //如果e的值不为空  这里的e的地址已经被指向了p 或者p.next
            //1.说明e节点存在   新起一个node节点   或者找到node节点修改value
            if (e != null) {
                //如果是e.value 复制给一个临时变量
                V oldValue = e.value;
                //这里onlyIfAbsent值的是 是否修改值  false 为修改  然后将value修改为我们设置的value
                if (!onlyIfAbsent || oldValue == null) {
                    e.value = value;
                }
                //直接返回不当做结构修改次数
                return oldValue;
            }
        }
        //添加修改次数
        ++modCount;
        //如果新增成功 ++key-value映射数量 如果大与阀值 则扩容
        if (++size > threshold) {
            //todo 扩容
            reSize();
        }
        //这里的返回值 是如果节点被修改了返回一oldValue
        return null;
    }


    /**
     * 获取节点信息如果找不到则返回null
     *
     * @param key
     * @return
     */
    @Override
    public V get(Object key) {
        //赋值
        Node<K, V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }

    @Override
    public V remove(Object key) {
        return super.remove(key);
    }

    @Override
    public boolean isEmpty() {
        return size == 0;
    }

    /**
     * 返回map下的 key-value映射数量
     *
     * @return
     */
    @Override
    public int size() {
        return size;
    }


    /**
     * ########################################################################################################
     * 普通方法 put,get之类<<<<<<<<<<
     * ########################################################################################################
     */


    @Override
    public Set<Entry<K, V>> entrySet() {
        return null;
    }


//todo 以下部分为工具方法
    /**
     * ########################################################################################################
     *                              >>>>>>>>>>>>>>>工具方法
     * ########################################################################################################
     */
    /**
     * 返回给定目标容量的二次幂。
     */
    private static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }

    /**
     * 用来计算 该key的hash值
     */
    private static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }

    /**
     * 初始化数组或者扩容数组
     */
    final Node<K, V>[] reSize() {
        //原来的数组
        Node<K, V>[] oldTab = table;
        //原数组容量  如果没初始化的话 则为0
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        //原阀值
        int oldThr = threshold;
        //扩容操作 新数组参数->新数组长度;
        int newCap = 0;
        //新阀值
        int newThr = 0;

        //如果原table初始化了 进入的肯定是扩容方法 否则进入else分支 根据初始化条件初始化数组

        //这里表明map的数组初始化了 扩容
        if (oldCap > 0) {
            //参数校验 如果 已经是map可存放的数组最大长度 直接返回
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            //如果如果原数组长度位运算扩大1倍 如果小于数组最大长度
            // 阀值并且大于等于默认的数组长度 1<<4 16  新阀值=旧X2 否则根据数组长度计算阀值
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                    oldCap >= DEFAULT_INITIAL_CAPACITY) {
                newThr = oldThr << 1;
            }
        }
        //创建初始化方法
        //如原数组阀值大于0 初始化新数组容量为原阀值 否则使用默认的值 创建一个阀值 也就是16*0.75f
        else if (oldThr > 0) {
            //
            newCap = oldThr;
        } else {
            //进入这里表示 new hashMap(); 所有阀值为null
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int) (DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }

        //如果新阀值大小还没有设定
        if (newThr == 0) {
            float ft = (float) newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float) MAXIMUM_CAPACITY ?
                    (int) ft : Integer.MAX_VALUE);
        }

        //将新阀值设置map的阀值
        threshold = newThr;

        //将新数组设置map的数组
        Node<K, V>[] newTab = (Node<K, V>[]) new Node[newCap];
        //将table的地址指向新起的Table;
        table = newTab;

        //接下来就是扩容操作了 移动旧table内容遍历 重新计算hash值塞给newTable
        if (oldTab != null) {
            //根据旧数组长度遍历
            for (int j = 0; j < oldCap; ++j) {
                //创建标记当前操作的node节点
                Node<K, V> e;
                //如果当前节点不为空 继续操作 否则直接略过
                if ((e = oldTab[j]) != null) {
                    //标记旧节点完毕后 移除旧节点中的内容
                    oldTab[j] = null;
                    //判断e这个node中是否有后继节点
                    if (e.next == null) {
                        //不存在后继 重新计算hash后 插入newTable
                        newTab[e.hash & (newCap - 1)] = e;
                    }
                    //todo 如果是红黑树类型暂时忽略
                    //else if(){
                    //
                    //}

                    else {
                        //如果后继有值 需要处理链表
                        Node<K, V> loHead = null, loTail = null;
                        Node<K, V> hiHead = null, hiTail = null;
                        //当前操作的节点
                        Node<K, V> next;
                        do {
                            // 注意：不是(e.hash & (oldCap-1));而是(e.hash & oldCap)

                            // (e.hash & oldCap) 得到的是 元素的在数组中的位置是否需要移动,示例如下
                            // 示例1：
                            // e.hash=10 0000 1010
                            // oldCap=16 0001 0000
                            //   &   =0  0000 0000       比较高位的第一位 0
                            //结论：元素位置在扩容后数组中的位置没有发生改变

                            // 示例2：
                            // e.hash=17 0001 0001
                            // oldCap=16 0001 0000
                            //   &   =1  0001 0000      比较高位的第一位   1
                            //结论：元素位置在扩容后数组中的位置发生了改变，新的下标位置是原下标位置+原数组长度

                            // (e.hash & (oldCap-1)) 得到的是下标位置,示例如下
                            //   e.hash=10 0000 1010
                            // oldCap-1=15 0000 1111
                            //      &  =10 0000 1010

                            //   e.hash=17 0001 0001
                            // oldCap-1=15 0000 1111
                            //      &  =1  0000 0001

                            //新下标位置
                            //   e.hash=17 0001 0001
                            // newCap-1=31 0001 1111    newCap=32
                            //      &  =17 0001 0001    1+oldCap = 1+16

                            //元素在重新计算hash之后，因为n变为2倍，那么n-1的mask范围在高位多1bit(红色)，因此新的index就会发生这样的变化：
                            // 0000 0001->0001 0001

                            next = e.next;
                            //元素的在数组中的位置是否需要移动
                            if ((e.hash & oldCap) == 0) {
                                // 如果原元素位置没有发生变化
                                if (loTail == null) {
                                    // 确定首元素
                                    loHead = e;
                                    // 第一次进入时     e   -> aa  ; loHead-> aa
                                } else {
                                    loTail.next = e;
                                }
                                //第二次进入时        loTail-> aa  ;    e  -> bb ;  loTail.next -> bb;而loHead和loTail是指向同一块内存的，所以loHead.next 地址为 bb
                                //第三次进入时        loTail-> bb  ;    e  -> cc ;  loTail.next 地址为 cc;loHead.next.next = cc
                                loTail = e;
                                // 第一次进入时         e   -> aa  ; loTail-> aa loTail指向了和  loHead相同的内存空间
                                // 第二次进入时               e   -> bb  ; loTail-> bb loTail指向了和  loTail.next（loHead.next）相同的内存空间   loTail=loTail.next
                                // 第三次进入时               e   -> cc  ; loTail-> cc loTail指向了和  loTail.next(loHead.next.next)相同的内存
                            } else {
                                //同理
                                if (hiTail == null) {
                                    hiHead = e;
                                } else {
                                    hiTail.next = e;
                                }
                                hiTail = e;
                            }


                        }
                        while ((e = next) != null);//这一步就是链表迁移的过程 重新维护关系
                        //总结：1.8中 旧链表迁移新链表    链表元素相对位置没有变化; 实际是对对象的内存地址进行操作
                        //在1.7中  旧链表迁移新链表        如果在新表的数组索引位置相同，则链表元素会倒置

                        //这里是元素位置没有被改变
                        if (loTail != null) {
                            //将最后一个e.next 设置为null
                            loTail.next = null;
                            //原地址不变
                            newTab[j] = loHead;
                        }
                        //改变位置
                        if (hiTail != null) {
                            //将最后一个e.next 设置为null
                            hiTail.next = null;
                            //重新构建地址
                            newTab[j + oldCap] = hiHead;
                        }

                    }

                }
            }
        }
        return newTab;
    }


    /**
     * ########################################################################################################
     * 工具方法<<<<<<<<<<<<<<<<<<
     * ########################################################################################################
     */
    @Override
    public String toString() {
        return "MyHashMap{" +
                "threshold=" + threshold +
                ", loadFactor=" + loadFactor +
                ", table=" + Arrays.toString(table) +
                ", size=" + size +
                ", modCount=" + modCount +
                '}';
    }
}

```

