---
title: UnSafe类使用CAS无锁化操作如ConcurrentHashMap
date: 2020-02-01 22:57:58
tags: [java,jvm]
---

# Unsafe类操作

```java
    public static void main(String[] args) {
        GwyuTest gwyuTest = new GwyuTest();
        gwyuTest.setId(10);
//		gwyuTest.id
        try {
            Unsafe unsafe = null;
//			这里是原生调用方法  测试调用无法使用 Unsafe unsafe = Unsafe.getUnsafe();
            // 获取 Unsafe 内部的私有的实例化单例对象
            Field field = Unsafe.class.getDeclaredField("theUnsafe");
            // 无视权限
            field.setAccessible(true);
            unsafe = (Unsafe) field.get(null);

            //获取偏移量
            long I_OFFSET = unsafe.objectFieldOffset(GwyuTest.class.getDeclaredField("id"));
            //CAS操作修改     对象 ,偏移量 原值 修改值 返回是否成功
            boolean flag = unsafe.compareAndSwapLong(gwyuTest, I_OFFSET, gwyuTest.getId(), gwyuTest.id + 1);
            System.out.println("是否修改成功" + flag);
            //这里就是直接从内存中获取值 而不会从线程副本里获取值
            System.out.println(unsafe.getIntVolatile(gwyuTest, I_OFFSET));

        } catch (Exception e) {
            System.out.println(e.getMessage());
        }
    }
```

<!--more-->

### 如果需要获取修改String数组中的内容

```java
            //数组中存储的对象头大小
            int ns=unsafe.arrayIndexScale(String[].class);
            //数组中第一个元素的起始位置
            int base=unsafe.arrayBaseOffset(String[].class);
            //获取第五个元素 这边4表示原数组下标
            System.out.println(unsafe.getObject(gwyuTest.id,base+4*ns));
```

