---
title: python-collections模块
date: 2020-01-04 23:13:10
tags: [python]
---

# 集合模块collections

# tuple 不可变数组

### 拆包

```python
user_tuple=('小明',16,170)
name,age,height=user_tuple
name,*other=user_tuple
```

<!--more-->

# namedtuple  类似使用类(常用)

好处: 省空间 ,不用新建类 ,使用tuple的拆包等特性

源码: 内置生成限制使用的类

### 导入

```python
from collections import namedtuple
```

### 语法构造 

```python
User=namedtuple("User",['name','age','height'])
```

### 测试

```python
from collections import namedtuple
User=namedtuple("User",['name','age','height'])
user=User(name='boddy',age=29,height=175)
print(user.age,user.name,user.height)

# 映射另外一个 传入tuple
user_tuple=('boddy',29,175)
user_1=User(*user_tuple )
## 同意思 必须创建完整参数
user_1=User._make(user_tuple)

user_tuple=('boddy',29)
user_1=User(*user_tuple,174)


# 映射 传入dict
user_dict={
  'name':'boddy',
  'age': 29,
 'height':175
}
## 同意思 必须创建完整参数
user_2=User._make(user_dict)
user_2=(**user_dict)
user_dict={
  'name':'boddy',
  'age': 29
}
user_2=(**user_dict,height=175)

```

### 普通映射

```python
from collections import namedtuple
User=namedtuple("User",['name','age','height'])
user=User(name='boddy',age=29,height=175)
# 映射另外一个 传入tuple
user_tuple=('boddy',29,175)
user_1=User(*user_tuple )
user_tuple=('boddy',29)
user_1=User(*user_tuple,174)


# 映射 传入dict
user_dict={
  'name':'boddy',
  'age': 29,
 'height':175
}
user_2=(**user_dict)
user_dict={
  'name':'boddy',
  'age': 29
}
user_2=(**user_dict,height=175)
```

### 内置方法    _make()->映射

必须存在完整的参数

```python
## 同意思 必须创建完整参数
user_tuple=('boddy',29,175)
user_1=User._make(user_tuple)

# 映射 传入dict
user_dict={
  'name':'boddy',
  'age': 29,
 'height':175
}
## 同意思 必须创建完整参数
user_2=User._make(user_dict)
```

### 内置方法 _asdict() ->转换为orderDict 有排序的dict

```python
user_tuple=('boddy',29,175)
user_1=User._make(user_tuple)
# 转换为orderDict
user_1._asdict()   
```

### 也能使用拆包

```python
user=User('小明',16,170)
name,*other=user
```

# defaultDict 含有默认值的dict(常用)

用处:键不存在 设置一个默认值

### 导入

```python
from collections import defaultDict
```

### 测试

```python
#  参数传入定义的对象类型 比如int 如果键不存在 默认创建键的value=传入的对象类型   如果是int 默认为0
default_dict=defaultDict(list)

default_dict['boddy']


```

### 自定义对象生成结构

```python
# 构建一个自定义的结构返回方法 ==自定义对象
def gen_default():
   return {
   "name":"",
   "age":0
   }

default_dict=defaultDict(gen_default)

default_dict['boddy']

# 返回默认的结构
```



# deque双端队列

有点: 线程安全 GTL保护

### 导入

```python
from collections import deque
```

##  语法构造

传入一个可迭代的对象 deque()

```python
user_list=deque(['boddy1','boddy2'])
或者
user_list=deque=(('boddy1','boddy2'))
或者 key值初始化
user_list=deque=({
'boddy1':28,
'boddy2':29
})

## 返回值都是:

deque(['boddy1','boddy2'])

```

### 内置函数 appendleft() 往头部加入数据

加入数据进入队列头部

```python
user_list=deque(['boddy1','boddy2'])
user_list.appendleft('boddy8')
```

### 内置函数 clear() 清空函数

```python
user_list.clear()
```

### 内置函数 copy() 浅拷贝

如果结构中有 可变的对象 调用的同一个代码区域 例如 list里面的list    基础类型除外

```python
user_list1=user_list.copy()
```

## python内置的深拷贝

```python
import copy
user_list2=copy.deepcopy(user_list)
```

### 内置函数 count() 计算元素数量

```python
user_list.count()
```

### 内置函数 extend() 合并deque 动态扩容 没有返回值

```python
user_list=deque(['boddy1','boddy2'])
user_list1=deque(['boddy3','boddy4'])

# 合并 追加到前面一个
user_list.extend(user_list1)
print(user_list) 
```

# Counter  统计

直接统计传入对象的个数

排序 由大到小

### 导入

```python
from collections import Counter 
```

### 语法构造

传入一个可迭代的对象 或者字符串

```python
user_counter=Counter(users)
```

### 测试

```python
users=['boddy1','boddy2','boddy3','boddy4','boddy5']
user_counter=Counter(users)
```

### 内置函数 .update() 合并统计

```python
users=['boddy1','boddy2','boddy3','boddy4','boddy5']
user_counter=Counter(users)
users2=['boddy6','boddy7','boddy8','boddy9','boddy10']
user_counter.update(users2)

也能传入Counter
```

### 内置函数 .most_common() 出现次数最多的前n个元素 (常用)     -> top n

```python
users=['boddy1','boddy2','boddy3','boddy4','boddy5']
user_counter=Counter(users)
# 出现次数最多的前两个
print(user_counter.most_common(2))
```

# OrderedDIct 可排序的dict

继承了dict 

保证添加的顺序

### 导入

```python
from collections import OrderedDIct  
```

### 语法构造

```python
user_dict=OrderedDict()
```

### 测试

```python
user_dict=OrderedDict()
user_dict["b"]='boddy2'
user_dict["a"]='boddy1'
user_dict["c"]='boddy3'
# 保证添加的顺序 python2的dict是无序的 python3的dict默认有序 等于OrderedDict()
print(user_dict)

```

### 内置函数  .popitem()  弹出最后一个item

```python
user_dict=OrderedDict()
user_dict["b"]='boddy2'
user_dict["a"]='boddy1'
user_dict["c"]='boddy3'
# 弹出最后一个键值对
user_dict.popitem()
## 如果是使用pop()必须带入一个key
```

### 内置函数 .move_to_end(self,key,last) 移动指定元素到最后

```python
user_dict=OrderedDict()
user_dict["b"]='boddy2'
user_dict["a"]='boddy1'
user_dict["c"]='boddy3'

 # 移动指定元素到最后
user_dict.move_to_end('b')

```

# ChainMap   连接数据合并进行遍历

### 导入

```python
from collections import OrderedDIct 
```

### 语法构造

```python
new_dict=ChainMap()
```

### 测试

```python
user_dict1={'a':'boddy1','b':'boddy2'}
user_dict2={'a':'boddy1','b':'boddy2'}
new_dict=ChainMap(user_dict1,user_dict2)
for key,value in new_dict.items():
    print(key,value)
```

### 注意 

```python
如果两个dict 中存在同名的key 遍历的时候只会保留前一个
```

### 内置函数 .new_child() 动态添加   不带参数添加一个null的

```python
user_dict1={'a':'boddy1','b':'boddy2'}
user_dict2={'a':'boddy1','b':'boddy2'}
new_dict=ChainMap(user_dict1,user_dict2)
# 动态添加
new_dict.new_child({'e':'boddy5','f':'boddy6'})
for key,value in new_dict.items():
    print(key,value)
```

### 内置函数  maps ( )      把ChainMap对象转化为list对象，可以被访问和修改。

```python
dict1={"hello":1,"world":2}
    dict2={"hello":3,"java":3}
    dict4={"hello":5,"java":5}
    dict3=ChainMap(dict1,dict2)
    print(dict3)
    # maps:把ChainMap对象转化为list对象，可以被访问和修改。
    print(dict3.maps)
```

