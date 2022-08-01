---
title: python基础学习
date: 2019-12-13 11:37:03
tags: [python]
---

# 基础学习

[demo地址](https://github.com/AsummerCat/PythonBasic)

# 结构

```python
目录只有包含一个叫做 __init__.py 的文件才会被认作是一个包，主要是为了避免一些滥俗的名字（比如叫做 string）不小心的影响搜索路径中的有效模块。
```

## 注释

```

# 单行注释

'''
多行注释
1
2
3
'''

"""
多行注释
1
2
3
"""

```

<!--more-->

## 輸出語句

```python
print("输出语句")
```

## 多行语句

### +\

```python
"""
多行语句  +\  这个标记
"""
a ="这是一个String类型多行拼接"+\
   "111"+\
    "拼接成功";
print(a)
```

###  """ 

```python
"""
字符串多行 拼接 : 三个引号
"""
b="""111
1111"""
print(b)
```

## 控制台输入

```
"""
等待用户输入
以上代码中 ，"\n\n"在结果输出前会输出两个新的空行。一旦用户按下 enter 键时，程序将退出。
"""
input("\n\n按下enter键后退出.")

print("end")
    
    
'''
读取键盘输入
'''
str = input("请输入：");
print ("你输入的内容是: ", str)
```

## 字符串截取

```cython
"""
字符串截取 
字符串截取的语法格式如下: 变量[头下标:尾下标:步长]
"""

str="linjingc"

print(str) #原字符串
print(str[0:-1]) #输出第一个到倒数第二个的所有字符
print(str[0])  # 输出字符串第一个字符
print(str[2:5])  # 输出从第三个开始到第五个的字符
print(str[2:])  # 输出从第三个开始后的所有字符
print(str * 2)  # 输出字符串两次
print(str + '你好')  # 连接字符串

print('------------------------------')

print('hello\nrunoob')  # 使用反斜杠(\)+n转义特殊字符
print(r'hello\nrunoob')  # 在字符串前面添加一个 r，表示原始字符串，不会发生转义
```

## 字符串格式化类

## %

```python
'''
Python字符串格式化

'''
print ("我叫 %s 今年 %d 岁!" % ('小明', 10))

'''
f-string 新格式化语句 3.6 -3.8
'''
test='测试哦'
print(f'hello{test}')

```

### str.format()

```python
'''
str.format() 的基本使用如下
'''
print('{}网址： "{}!"'.format('学习使我快乐', 'www.linjingc.top'))
# 如果在 format() 中使用了关键字参数, 那么它们的值会指向使用该名字的参数。
print('{name}网址： {site}'.format(name='学习使我快乐', site='www.linjingc.top'))
# 位置及关键字参数可以任意的结合:
print('站点列表 {0}, {1}, 和 {other}。'.format('Google', 'Runoob', other='Taobao'))

print('{0:5} ==> {1:10d}'.format(1, 2))
```



## 字符串填充

### 右填充 rjust();  左填充 ljust();

```python
'''
这个例子展示了字符串对象的 rjust() 方法, 它可以将字符串靠右, 并在左边填充空格。
'''
for x in range(1, 11):
    print(repr(x).rjust(2), repr(x * x).rjust(3), end=' ')
    # 注意前一行 'end' 的使用
    print(repr(x * x * x).rjust(4))
    
    # 左填充类似
```

### zfill() 会在数值左边填充0

```
print('12'.zfill(5))
```

## 转义特殊字符

```
''''
repr() 函数可以转义字符串中的特殊字符  repr() 的参数可以是 Python 的任何对象
'''
c = repr("hello\n")
print(c)
```

## 断言

```python
# -*- coding: utf-8 -*-
# 断言测试
import sys


assert ('linux' in sys.platform), "改代码只能在 Linux 下执行"
```

## if判断

需要注意  0表示false 其他表示true

```python
"""
if判断
"""
if True:
    print("True")
else:
    print("false")
    print("嘿嘿")

"""
多条件判断
"""
a =1
if a==1:
    print("a=1")
elif a==2:
    print("a=2")
else:
    print("a为默认")

```



## while循环

```
'''
end关键字 用于输出一行 后面加上拼接关键字
end=','
'''

a, b = 0, 1
while b < 1000:
    print(b, end=',')
    a, b = b, a + b
else:
    print("退出了循环")
```

# 导包操作

```python
"""
在 python 用 import 或者 from...import 来导入相应的模块。

将整个模块(somemodule)导入，格式为： import somemodule

从某个模块中导入某个函数,格式为： from somemodule import somefunction

从某个模块中导入多个函数,格式为： from somemodule import firstfunc, secondfunc, thirdfunc

将某个模块中的全部函数导入，格式为： from somemodule import *
"""
```

## 测试

```python
import sys

print('================Python import mode==========================')
print('命令行参数为:')
for i in sys.argv:
    print(i)
print('\n python 路径为', sys.path)

from sys import path  # 导入特定的成员

print('================python from import===================================')
print('path:', path)  # 因为已经导入path成员，所以此处引用时不需要加sys.path

'''
调用其他模块
'''
import function

# from function import add 指定函数调用

print(function.add(1, 2))

'''
如果在当前页常用还可以将函数赋值给var
'''
addTest = function.add
print(addTest(1, 2))

'''
__name__属性 来判断是否在当前模块运行
'''
if __name__ == '__main__':
    print('程序自身在运行')
else:
    print('我来自另一模块')

'''
dir()  内置的函数 dir() 可以找到模块内定义的所有名称。以一个字符串列表的形式返回
'''
print(dir(sys))
'''
dir() 不给参数能获取到当前模块的参数
'''
a=5
print(dir())
del a
print(dir())
```

## 调用主函数

```python

if __name__ == '__main__':
  mian()
else:
```

# 类class

```
class ClassName:
    <statement-1>
    .
    .
    .
    <statement-N>
'''
```

### 测试类

```python
class MyClass:
    """测试实例"""
    i = 123456

    def __init__(self):
        print("MyClass构造函数")

    # 类的方法与普通的函数只有一个特别的区别——它们必须有一个额外的第一个参数名称, 按照惯例它的名称是 self。
    def f(self):
        #  'self表示当前对象'
        print(self.i)
        return 'hello world'

```

### 实例化类 调用方法

```python

# 实例化类
x = MyClass()
# 访问类的属性和方法
print("MyClass 类的属性 i 为：", x.i)
print("MyClass 类的方法 f 输出为：", x.f())
```

### 私有属性

```python
'''
私有属性 private 对应 属性名称 __开头 两个下划线开头
私有方法 private 对应 属性名称也是 __开头
'''
# 类定义
class people:
    # 定义基本属性
    name = ''
    age = 0
    # 定义私有属性,私有属性在类外部无法直接进行访问
    __weight = 0

    # 定义构造方法
    def __init__(self, n, a, w):
        self.name = n
        self.age = a
        self.__weight = w

    def speak(self):
        print("%s 说: 我 %d 岁。" % (self.name, self.age))

```

### 类继承

#### 单继承

```
'''
类继承
语法
class DerivedClassName(BaseClassName1):
    <statement-1>
    .
    .
    .
    <statement-N>
'''


class ChildMyClass(MyClass):
    grade = ''

    def __init__(self, g):
        print("ChildMyClass构造函数")
        self.grade = g
        # 覆写父类的方法

    def f(self):
        return "子类方法哦"


x = ChildMyClass("测试")
print(x.f())
```

#### 多继承

```python
'''
支持
多继承
class DerivedClassName(Base1, Base2, Base3):
    <statement-1>
    .
    .
    .
    <statement-N>
    需要注意圆括号中父类的顺序，若是父类中有相同的方法名，而在子类使用时未指定，python从左至右搜索 即方法在子类中未找到时，从左到右查找父类中是否包含方法。
    方法名同，默认调用的是在括号中排前地父类的方法
'''

```

## 内部作用域 修改全局变量

global 关键字声明

```python
'''
内部作用域无法修改全局变量 给予方法的全是副本 
如果需要修改全局遍历需要使用 global 关键字声明
'''
def test1():
    # 声明修改全局变量
    global num
    print("修改前", num)
    # num = num - 1
    num -= 1
    print("修改后", num)


print("全局变量", num)
test1()
print("全局变量", num)
test1()
print("全局变量", num)
test1()

```

## 嵌套作用域 修改上级作用域内容   nonlocal关键字声明 

```python

def outer():
    num = 10

    def inner():
        nonlocal num  # nonlocal关键字声明
        num = 100
        print(num)

    inner()
    print(num)


outer()

```

# 文件流输入输出

```python
'''

open() 将会返回一个 file 对象，基本语法格式如下:
open(filename, mode)
ilename：包含了你要访问的文件名称的字符串值。
mode：决定了打开文件的模式：只读，写入，追加等。所有可取值见如下的完全列表。这个参数是非强制的，默认文件访问模式为只读(r)。
不同模式打开文件的完全列表：
https://www.runoob.com/python3/python3-inputoutput.html
'''
```

## 导入模块

```python
# 需要导入模块
import codecs
```

## 写入文件

```python
# 写入文件
f = codecs.open("D:/pythonDdemo/basic/fileTest.txt", "w+", encoding='utf-8')
wText = "输出语句转换UTF-8\n"
f.write(wText)
f.writelines("下一行\n")
f.writelines("下一行")
f.close()
```

## 写入文件 返回写入的字符数

```python
# 写入返回写入的字符数
f = codecs.open("D:/pythonDdemo/basic/fileTest.txt","w",encoding='utf-8')
num = f.write( "Python 是一个非常好的语言。\n是的，的确非常好!!\n" )
print(num)
# 关闭打开的文件
f.close()
```



## 读取文件

```python

# 读取文件 read()
f = codecs.open("D:/pythonDdemo/basic/fileTest.txt","r",encoding='utf-8')
 ## 一次性读取
str=f.read()
print(str)
f.close()
```

## 读取文件 读取单行

```python

# 读取文件 读取单行 f.readline() 换行符为 '\n   readline() 如果返回一个空字符串, 说明已经已经读取到最后一行
f = codecs.open("D:/pythonDdemo/basic/fileTest.txt","r",encoding='utf-8')
 ## 一次性读取
str=f.readline()
print(str)
f.close()
```

## 读取文件 读取所有行

```python
# f.readlines() 将返回该文件中包含的所有行。
f = codecs.open("D:/pythonDdemo/basic/fileTest.txt","r",encoding='utf-8')
str=f.readlines()
print(str)
f.close()
```

## 读取文件 遍历

```python
# 遍历
f = codecs.open("D:/pythonDdemo/basic/fileTest.txt","r",encoding='utf-8')
for line in f:
    print(line, end='')
f.close()
```

## 获取当前目录下的文件列表

```python
import os

print(os.path)
print(os.pipe())
# 获取当前目录下的文件列表
print(os.listdir("D:/pythonDdemo/basic"))
```

# 自定义函数

## 定义

```python
'''
自定义函数定义
def
'''

```

## 基础形式

```python
def add(a, b):
    return a + b


print(add(1, 2))
```

## 不定长函数

```python
'''
不定长参数 如元组
*参数 表示元组
**参数 表示字典(map)

'''


# 可写函数说明
def printinfo(arg1, *vartuple):
    # "打印任何传入的参数"
    print("输出: ")
    print(arg1)
    print(vartuple)


# 调用printinfo 函数
printinfo(70, 60, 50)
```

## lambda表达式

```python
'''
lambda表达式 
# 可写函数说明
sum = lambda arg1, arg2: arg1 + arg2
'''
# 可写函数说明
sum = lambda arg1, arg2: arg1 + arg2

# 调用sum函数
print("相加后的值为 : ", sum(10, 20))
print("相加后的值为 : ", sum(20, 20))

```









# 遍历操作

## 列表当做堆栈 后进先出

```python
'''
将列表当做堆栈  后进先出
'''
stack = [3, 4, 5]
print(stack.pop())
print(stack.pop())
print(stack.pop())

```

## 将列表当做队列

```python
'''
将列表当做队列
'''
from collections import deque

queue = deque(["Eric", "John", "Michael"])
print(queue.popleft())
print(queue.popleft())
print(queue.popleft())

```

## 列表推导公式

```python
'''
列表推导公式
'''
vec = [2, 4, 6]
print([3 * x for x in vec])
```

## 同时遍历两个或更多的序列，可以使用 zip() 组合：

```python
'''
同时遍历两个或更多的序列，可以使用 zip() 组合：
'''
questions = ['name', 'quest', 'favorite color']
answers = ['lancelot', 'the holy grail', 'blue']
for q, a in zip(questions, answers):
    result1 = 'What is your {0}?  It is {1}.'.format(q, a)
    print(result1)

```

## 迭代器

### 基础数据

```python
list = [1, 2, 3, 4]
it = iter(list)  # 创建迭代器对象
print(next(it))  # 输出迭代器的下一个元素
print(next(it))
```

### 迭代器遍历普通遍历

```python
list = [1, 2, 3, 4]
it = iter(list)  # 创建迭代器对象
for x in list:
    print(x)
```

### 使用next 获取下一个值 进行遍历

```python
import sys  # 引入 sys 模块

list = [1, 2, 3, 4]
it = iter(list)  # 创建迭代器对象
while True:
    try:
        print(next(it))
    except StopIteration:
        sys.exit()

```



# json和python数据结构转换

`import json`

## Python 字典类型转换为 JSON 对象

```python
# Python 字典类型转换为 JSON 对象
data = {
    'no': 1,
    'name': 'Runoob',
    'url': 'http://www.runoob.com'
}

json_str = json.dumps(data)
print("Python 原始数据：", repr(data))
print("JSON 对象：", json_str)
```

##  Python 字典类型转换为 JSON 对象

```python
# Python 字典类型转换为 JSON 对象
data1 = {
    'no': 1,
    'name': 'Runoob',
    'url': 'http://www.runoob.com'
}

json_str = json.dumps(data1)
print("Python 原始数据：", repr(data1))
print("JSON 对象：", json_str)

# 将 JSON 对象转换为 Python 字典
data2 = json.loads(json_str)
print("data2['name']: ", data2['name'])
print("data2['url']: ", data2['url'])
```

##  son.dump() 和 json.load()

```python
'''
如果你要处理的是文件而不是字符串，你可以使用 json.dump() 和 json.load() 来编码和解码JSON数据。例如：
'''
# 写入 JSON 数据
with open('data.json', 'w') as f:
    json.dump(data, f)

# 读取数据
with open('data.json', 'r') as f:
    data = json.load(f)

```

# 时间模块的使用

`import time;  # 引入time模块`

## 获取时间戳

```python
ticks = time.time()
print("当前时间戳为:", ticks)
```

## 获取当前时间

```python
# 获取当前时间
# 从返回浮点数的时间戳方式向时间元组转换，只要将浮点数传递给如localtime之类的函数。
localtime = time.localtime(time.time())
print("本地时间为 :", localtime)
```

## 获取格式化的时间

```python
# 获取格式化的时间
localtime = time.asctime(time.localtime(time.time()))
print("本地时间为 :", localtime)
```

## 格式化时间

```python

'''
格式化时间
time.strftime(format[, t])
'''
# 格式化成2016-03-20 11:45:39形式
print(time.strftime("%Y-%m-%d %H:%M:%S", time.localtime()))

# 格式化成Sat Mar 28 22:24:24 2016形式
print(time.strftime("%a %b %d %H:%M:%S %Y", time.localtime()))

# 将格式字符串转换为时间戳
a = "Sat Mar 28 22:24:24 2016"
print(time.mktime(time.strptime(a, "%a %b %d %H:%M:%S %Y")))
```

## Calendar模块 

```python
'''
Calendar模块有很广泛的方法用来处理年历和月历，例如打印某月的月历：
'''
# 获取某月日历
import calendar

cal = calendar.month(2016, 1)
print("以下输出2016年1月份的日历:")
print(cal)
```

# 异常处理 try catch

## 定义

```
异常处理语句
try:

:except:

'''
```

## 单异常

```python
import sys

while True:
    try:
        x = int(input("请输入一个数字:"))
        break
    except:
        print("您输入的不是数字，请再次尝试输入！")
    finally:
        print("结束输出")

```

## 多异常

```python
'''
一个except子句可以同时处理多个异常，这些异常将被放在一个括号里成为一个元组，例如

except (RuntimeError, TypeError, NameError,ValueError):
'''
```

## try catch else

```python
'''
try/except...else

try/except 语句还有一个可选的 else 子句，如果使用这个子句，那么必须放在所有的 except 子句之后。

else 子句将在 try 子句没有发生任何异常的时候执行。
'''
for arg in sys.argv[1:]:
    try:
        f = open(arg, 'r')
    except IOError:
        print('cannot open', arg)
    else:
        print(arg, 'has', len(f.readlines()), 'lines')
        f.close()

```

## 异常别名

```python
'''
给予异常别名
'''


def this_fails():
    x = 1 / 0
    return x


try:
    this_fails()
except ZeroDivisionError as err:
    print('Handling run-time error:', err)
```

## 抛出异常

```python
'''
抛出异常
raise [Exception [, args [, traceback]]]
'''
x = 10
if x > 5:
    raise Exception('x 不能大于 5。x 的值为: {}'.format(x))
```

## 抛出异常不处理

```python
'''
抛出异常不处理
'''
try:
    raise NameError('HiThere')
except NameError:
    print('An exception flew by!')
    raise


```

# 基本类型

## 定义

```python
"""
Python3 中有六个标准的数据类型：

Number（数字）
String（字符串）
List（列表）
Tuple（元组）
Set（集合）
Dictionary（字典）
Python3 的六个标准数据类型中：

不可变数据（3 个）：Number（数字）、String（字符串）、Tuple格式:()（元组）；
可变数据（3 个）：List格式:[]（列表）、Dictionary（字典）、Set格式:{}（集合）。
"""
```

## list列表

```python
list = ['abcd', 786, 2.23, 'runoob', 70.2]
tinylist = [123, 'runoob']
# 新增
list.append("cc")
print(list)
# 删除 1
list.remove(786)
# 删除 2
del list[3]
print("删除")
print(list)

# 拼接
print(list + tinylist)

# 反转
list.reverse()
print(list)

# 改变元素
list[0] = 9
print(list)
list[2:5] = [13, 14, 15]
print(list)

print(list)  # 输出完整列表
print(list[0])  # 输出列表第一个元素
print(list[1:3])  # 从第二个开始输出到第三个元素
print(list[2:])  # 输出从第三个元素开始的所有元素
print(tinylist * 2)  # 输出两次列表
print(list + tinylist)  # 连接列表

```

## 元组=数组

```
# 元组=数组
"""
元组（tuple）与列表类似，不同之处在于元组的元素不能修改。元组写在小括号 () 里，元素之间用逗号隔开。
"""
tuple = ('abcd', 786, 2.23, 'runoob', 70.2)
tinytuple = (123, 'runoob')

print(tuple)  # 输出完整元组
print(tuple[0])  # 输出元组的第一个元素
print(tuple[1:3])  # 输出从第二个元素开始到第三个元素
print(tuple[2:])  # 输出从第三个元素开始的所有元素
print(tinytuple * 2)  # 输出两次元组
print(tuple + tinytuple)  # 连接元组

```

## 集合 set 自动去重

```python
"""
可以使用大括号 { } 或者 set() 函数创建集合，注意：创建一个空集合必须用 set() 而不是 { }，因为 { } 是用来创建一个空字典。

创建格式：

parame = {value01,value02,...}
或者
set(value)
"""
parame = set()
# or
parame = {}

student = {'Tom', 'Jim', 'Mary', 'Tom', 'Jack', 'Rose'}

print(student)  # 输出集合，重复的元素被自动去掉

# 成员测试
if 'Rose' in student:
    print('Rose 在集合中')
else:
    print('Rose 不在集合中')

# set可以进行集合运算
a = set('abracadabra')
b = set('alacazam')

print(a)

print(a - b)  # a 和 b 的差集

print(a | b)  # a 和 b 的并集

print(a & b)  # a 和 b 的交集

print(a ^ b)  # a 和 b 中不同时存在的元素
print("===========================================================")
```



## 字典 ->hashMap

```python
# java的hashMap=Dictionary（字典）
"""
字典（dictionary）是Python中另一个非常有用的内置数据类型。

列表是有序的对象集合，字典是无序的对象集合。两者之间的区别在于：字典当中的元素是通过键来存取的，而不是通过偏移存取。

字典是一种映射类型，字典用 { } 标识，它是一个无序的 键(key) : 值(value) 的集合。

键(key)必须使用不可变类型。

在同一个字典中，键(key)必须是唯一的。
"""
dict = {}
dict['1'] = "1-第一个value"
dict['2'] = "第二个value"
for key in dict.keys():
    print(key + ':' + dict[key])

# 方式一：
for key in dict:
    print(key + ':' + dict[key])
# 方式二：
for key in dict.keys():
    print(key + ':' + dict[key])
# 方式三：
for key, value in dict.items():
    print(key + ':' + value)
# 方式四：
for (key, value) in dict.items():
    print(key + ':' + value)

# 遍历字典项
for kv in dict.items():
    print(kv)


'''
遍历list 获取下标 函数enumerate
'''
list = ['abcd', 786, 2.23, 'runoob', 70.2]
for se,ai in enumerate(list):
    print(se,ai)
```

# 发送邮件

## 简单实例

```python
import smtplib
from email.header import Header
from email.mime.text import MIMEText

sender = 'from@runoob.com'
receivers = ['429240967@qq.com']  # 接收邮件，可设置为你的QQ邮箱或者其他邮箱

# 三个参数：第一个为文本内容，第二个 plain 设置文本格式，第三个 utf-8 设置编码
message = MIMEText('Python 邮件发送测试...', 'plain', 'utf-8')
message['From'] = Header("菜鸟教程", 'utf-8')  # 发送者
message['To'] = Header("测试", 'utf-8')  # 接收者

subject = 'Python SMTP 邮件测试'
message['Subject'] = Header(subject, 'utf-8')

try:
    smtpObj = smtplib.SMTP('localhost')
    smtpObj.sendmail(sender, receivers, message.as_string())
    print("邮件发送成功")
except smtplib.SMTPException:
    print("Error: 无法发送邮件")

print("=====================================================================================")
```

## 发送html格式的邮件

```python

import smtplib
from email.mime.text import MIMEText
from email.header import Header

sender = 'from@runoob.com'
receivers = ['429240967@qq.com']  # 接收邮件，可设置为你的QQ邮箱或者其他邮箱

mail_msg = """
<p>Python 邮件发送测试...</p>
<p><a href="http://www.runoob.com">这是一个链接</a></p>
"""
message = MIMEText(mail_msg, 'html', 'utf-8')
message['From'] = Header("菜鸟教程", 'utf-8')
message['To'] = Header("测试", 'utf-8')

subject = 'Python SMTP 邮件测试'
message['Subject'] = Header(subject, 'utf-8')

try:
    smtpObj = smtplib.SMTP('localhost')
    smtpObj.sendmail(sender, receivers, message.as_string())
    print("邮件发送成功")
except smtplib.SMTPException:
    print("Error: 无法发送邮件")

print("=====================================================================================")
```

## Python 发送带附件的邮件

```python
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from email.header import Header

sender = 'from@runoob.com'
receivers = ['429240967@qq.com']  # 接收邮件，可设置为你的QQ邮箱或者其他邮箱

# 创建一个带附件的实例
message = MIMEMultipart()
message['From'] = Header("菜鸟教程", 'utf-8')
message['To'] = Header("测试", 'utf-8')
subject = 'Python SMTP 邮件测试'
message['Subject'] = Header(subject, 'utf-8')

# 邮件正文内容
message.attach(MIMEText('这是菜鸟教程Python 邮件发送测试……', 'plain', 'utf-8'))

# 构造附件1，传送当前目录下的 test.txt 文件
att1 = MIMEText(open('test.txt', 'rb').read(), 'base64', 'utf-8')
att1["Content-Type"] = 'application/octet-stream'
# 这里的filename可以任意写，写什么名字，邮件中显示什么名字
att1["Content-Disposition"] = 'attachment; filename="test.txt"'
message.attach(att1)

# 构造附件2，传送当前目录下的 runoob.txt 文件
att2 = MIMEText(open('runoob.txt', 'rb').read(), 'base64', 'utf-8')
att2["Content-Type"] = 'application/octet-stream'
att2["Content-Disposition"] = 'attachment; filename="runoob.txt"'
message.attach(att2)

try:
    smtpObj = smtplib.SMTP('localhost')
    smtpObj.sendmail(sender, receivers, message.as_string())
    print("邮件发送成功")
except smtplib.SMTPException:
    print("Error: 无法发送邮件")

print("=====================================================================================")
```

## 在 HTML 文本中添加图片

```python
import smtplib
from email.mime.image import MIMEImage
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from email.header import Header

sender = 'from@runoob.com'
receivers = ['429240967@qq.com']  # 接收邮件，可设置为你的QQ邮箱或者其他邮箱

msgRoot = MIMEMultipart('related')
msgRoot['From'] = Header("菜鸟教程", 'utf-8')
msgRoot['To'] = Header("测试", 'utf-8')
subject = 'Python SMTP 邮件测试'
msgRoot['Subject'] = Header(subject, 'utf-8')

msgAlternative = MIMEMultipart('alternative')
msgRoot.attach(msgAlternative)

mail_msg = """
<p>Python 邮件发送测试...</p>
<p><a href="http://www.runoob.com">菜鸟教程链接</a></p>
<p>图片演示：</p>
<p><img src="cid:image1"></p>
"""
msgAlternative.attach(MIMEText(mail_msg, 'html', 'utf-8'))

# 指定图片为当前目录
fp = open('test.png', 'rb')
msgImage = MIMEImage(fp.read())
fp.close()

# 定义图片 ID，在 HTML 文本中引用
msgImage.add_header('Content-ID', '<image1>')
msgRoot.attach(msgImage)

try:
    smtpObj = smtplib.SMTP('localhost')
    smtpObj.sendmail(sender, receivers, msgRoot.as_string())
    print("邮件发送成功")
except smtplib.SMTPException:
    print("Error: 无法发送邮件")
```

# 基础库

# 操作系统接口 os

`import os`

```python

# 操作cmd 命令
 print(os.popen('start.'))

# 返回当前的工作目录
import time

print(os.getcwd())

# 修改当前的工作目录
 os.chdir('/server/accesslogs')

# 执行系统命令 mkdir
 os.system('mkdir today')

# 创建目录
os.mkdir(r"d:\a")

# 返回目录下的所有文件 相当于ls
os.listdir(r"d:\a")

# 删除空的目录
os.rmdir(r"d:\a")

# 递归删除目录和文件
 os.removedirs(path)

# 删除path指向的文件
 os.remove(path)

# 重命名 重命名文件，src和dst为两个路径，分别表示重命名之前和之后的路径。
 os.rename(src, dst)

# 修改权限 相当于chmod
 os.chmod(path, mode)

```

## 针对日常的文件和目录管理任务 shutil模块

`import shutil`

```python

# 拷贝文件1
 shutil.copyfile('data.db', 'archive.db')

# 重命名
  shutil.move('/build/executables', 'installdir')

# 仅copy权限，不更改文件内容，组和用户。
 shutil.copymode(src,dst)

# 复制所有的状态信息，包括权限，组，用户，时间等
 shutil.copystat(src,dst)

# 复制文件的内容以及权限，先copyfile后copymode
 shutil.copy(src,dst)

# 复制文件的内容以及文件的所有状态信息。先copyfile后copystat
 shutil.copy2(src,dst)

# 递归的复制文件内容及状态信息
  shutil.copytree(src, dst, symlinks=False, ignore=None, copy_function=copy2,ignore_dangling_symlinks=False)

# 递归删除文件
 shutil.rmtree(path, ignore_errors=False, οnerrοr=None)

# 递归移动文件
 shutil.move(src, dst)

# 压缩打包
# base_name：    压缩打包后的文件名或者路径名
# format：          压缩或者打包格式    "zip", "tar", "bztar"or "gztar"
# root_dir :         将哪个目录或者文件打包（也就是源文件）
# shutil.make_archive('tarball','gztar',root_dir='copytree_test')

```

## 文件通配符 查找文件列表    glob模块

`import glob`

```python

# 查询指定目录下的文件  *通配符
print(glob.glob(r'D:\work\pass\src\main\java\com\jeecg\demo\controller\*.java'))

# 查询指定目录下带?的文件  通配符?单个字符
print(glob.glob(r'D:\work\pass\src\main\java\com\jeecg\demo\controller\JeecgDemoExcelControlle?.java'))

# 查询指定目录下 特定字符的范围 通配符
print(glob.glob(r'D:\work\pass\src\main\java\com\jeecg\demo\controller\*[A-Z].java'))

print("==================================================================")
```

## 命令行参数

`import sys`

```python
import sys

print(sys.argv)

# 输出警告
sys.stderr.write('Warning, log file not found starting a new one\n')

sys.version  # 获取Python解释程器的版本信息
sys.maxsize  # 最大的Int值，64位平台是2**63 - 1
sys.path  # 返回模块的搜索路径，初始化时使用PYTHONPATH环境变量的值
sys.platform  # 返回操作系统平台名称
sys.stdin  # 输入相关
sys.stdout  # 输出相关
sys.stderr  # 错误相关
sys.exc_info()  # 返回异常信息三元元组
sys.getdefaultencoding()  # 获取系统当前编码，默认为utf-8
sys.setdefaultencoding()  # 设置系统的默认编码
sys.getfilesystemencoding()  # 获取文件系统使用编码方式，默认是utf-8
sys.modules  # 以字典的形式返回所有当前Python环境中已经导入的模块
sys.builtin_module_names  # 返回一个列表，包含所有已经编译到Python解释器里的模块的名字
sys.copyright  # 当前Python的版权信息
sys.flags  # 命令行标识状态信息列表。只读。
sys.getrefcount(object)  # 返回对象的引用数量
sys.getrecursionlimit()  # 返回Python最大递归深度，默认1000
sys.getswitchinterval()  # 返回线程切换时间间隔，默认0.005秒
sys.getwindowsversion()  # 返回当前windwos系统的版本信息
sys.hash_info  # 返回Python默认的哈希方法的参数
sys.implementation  # 当前正在运行的Python解释器的具体实现，比如CPython
sys.thread_info  # 当前线程信息
# 程序终止
 sys.exit()
```

## 加密包  hashlib模块

`import hashlib`

```python
# 散列加密  最后两个参数 salt 和加密次数 不可逆
dk = hashlib.pbkdf2_hmac('sha256', b'password', b'salt', 100000)
dkhex = dk.hex()
print(dk.hex())
dk1 = hashlib.pbkdf2_hmac('sha256', b'password', b'salt', 100000)
dk1hex = dk1.hex()
print(dk1.hex())
if dk1hex.__eq__(dkhex):
    print("比对成功加密匹配正确")
else:
    print("比对成功加密匹配错误")

```

## 正则匹配  re模块

`import re`

```python
import re

# 搜索字符串,以列表类型返回全部能匹配的子串
print(re.findall(r'\bf[a-z]*', 'which foot or hand fell fastest'))

# 在一个字符中替换所有能匹配正则的子串,返回替换的后的字符串  最后一个5表示能替换的最大次数
print(re.sub(r'(\b[a-z]+) \1', r'替换咯', 'cat in the the hat', 5))

print('tea for too'.replace('too', 'two'))

# search 的作用是 查找后面字符中，存在前面字符的情况
m = re.search('[0-9]', "abcd4ef5")
print(m.group(0))

# re.match函数 re.match 尝试从字符串的起始位置匹配一个模式，如果不是起始位置匹配成功的话，match()就返回none。
print(re.match('www', 'www.runoob.com').span())  # 在起始位置匹配
print(re.match('com', 'www.runoob.com'))  # 不在起始位置匹配

```

## 数学函数  math模块

`import math`

```python
import math

print(math.cos(math.pi / 4))
print(math.log(1024, 2))
```



## 随机数 random模块

```python
import random

# 根据数组随机获取一个
print(random.choice(['apple', 'pear', 'banana']))

# 随机获取 指定指定个数
print(random.sample(range(100), 10))

# 返回float
print(random.random())

# 随机获取 限制范围
print(random.randrange(4, 6))
print(random.randrange(6))  # 0-6

```

## 访问互联网 hhtp模块

### get请求

```python
# 处理get请求，不传data，则为get请求

import urllib
from urllib.request import urlopen
from urllib.parse import urlencode

url = 'http://www.xxx.com/login'
data = {"username": "admin", "password": 123456}
req_data = urlencode(data)  # 将字典类型的请求数据转变为url编码
res = urlopen(url + '?' + req_data)  # 通过urlopen方法访问拼接好的url
res = res.read().decode()  # read()方法是读取返回数据内容，decode是转换返回数据的bytes格式为str

print(res)
# 处理post请求,如果传了data，则为post请求

import urllib
from urllib.request import Request
from urllib.parse import urlencode

url = 'http://www.xxx.com/login'
data = {"username": "admin", "password": 123456}
data = urlencode(data)  # 将字典类型的请求数据转变为url编码
data = data.encode('ascii')  # 将url编码类型的请求数据转变为bytes类型
req_data = Request(url, data)  # 将url和请求数据处理为一个Request对象，供urlopen调用
with urlopen(req_data) as res:
    res = res.read().decode()  # read()方法是读取返回数据内容，decode是转换返回数据的bytes格式为str

print(res)

```

## 时间和日期 date模块

`from datetime import date`

```python

now = date.today()
print(now)

# 格式化
print(now.strftime("%m-%d-%y. %d %b %Y is a %A on the %d day of %B."))

birthday = date(1964, 7, 31)
age = now - birthday
print(age.days)

# 今天
today = datetime.date.today()
print(today)
# 昨天
yesterday = today - datetime.timedelta(days=1)
print(yesterday)
# 上个月
last_month = today.month - 1 if today.month - 1 else 12
print(last_month)
# 当前时间戳
time_stamp = time.time()
print(time_stamp)
# 时间戳转datetime
print(datetime.datetime.fromtimestamp(time_stamp))
# datetime转时间戳
print(int(time.mktime(today.timetuple())))
# datetime转字符串
today_str = today.strftime("%Y-%m-%d")
print(today_str)
# 字符串转datetime
today = datetime.datetime.strptime(today_str, "%Y-%m-%d")
print(today)
# 补时差
print(today + datetime.timedelta(hours=8))
```

## 数据压缩 zlib模块

`import zlib`

```python
ompress(message)
decompressed = zlib.decompress(compressed)

print('original:', repr(message))
print('compressed:', repr(compressed))
print('decompressed:', repr(decompressed))


# 压缩和解压文件
# zlib.compress用于压缩流数据。参数string指定了要压缩的数据流，参数level指定了压缩的级别，它的取值范围是1到9。压缩速度与压缩率成反比，1表示压缩速度最快，而压缩率最低，而9则表示压缩速度最慢但压缩率最高
## 压缩
def compress(infile, dst, level=9):
    infile = open(infile, 'rb')
    dst = open(dst, 'wb')
    compress = zlib.compressobj(level)
    data = infile.read(1024)
    while data:
        dst.write(compress.compress(data))
        data = infile.read(1024)
    dst.write(compress.flush())

## 解压
def decompress(infile, dst):
    infile = open(infile, 'rb')
    dst = open(dst, 'wb')
    decompress = zlib.decompressobj()
    data = infile.read(1024)
    while data:
        dst.write(decompress.decompress(data))
        data = infile.read(1024)
    dst.write(decompress.flush())


if __name__ == "__main__":
    compress('in.txt', 'out.txt')
    decompress('out.txt', 'out_decompress.txt')

```

## html转义

`import html`

```python
import html

# 输出原文
dataHtmlValue = html.escape("<h1>hello world</h1>")
# 转义
html.unescape(dataHtmlValue)

```



# mysql的驱动

`import mysql.connector`

```python
# -*- coding: utf-8 -*-
# mysql-connector 测试
# 需要安装提供的驱动  python -m pip install mysql-connector
import mysql.connector

# 连接
mydb = mysql.connector.connect(
    host="localhost",  # 数据库主机地址
    user="yourusername",  # 数据库用户名
    passwd="yourpassword"  # 数据库密码
    # ,database="runoob_db" #库名 可指定
)

print(mydb)

# 创建库
mycursor = mydb.cursor()

mycursor.execute("CREATE DATABASE runoob_db")

print("===============================================================================================")

# 显示所有库

mycursor.execute("SHOW DATABASES")

for x in mycursor:
    print(x)

    print("===============================================================================================")

# 创建表

mycursor.execute("CREATE TABLE sites (name VARCHAR(255), url VARCHAR(255))")

print("===============================================================================================")

# 插入数据
sql = "INSERT INTO sites (name, url) VALUES (%s, %s)"
val = ("RUNOOB", "https://www.runoob.com")
mycursor.execute(sql, val)
mydb.commit()  # 数据表内容有更新，必须使用到该语句
print(mycursor.rowcount, "记录插入成功。")
print("1 条记录已插入, ID:", mycursor.lastrowid)

print("===============================================================================================")

# 批量插入
sql = "INSERT INTO sites (name, url) VALUES (%s, %s)"
val = [
    ('Google', 'https://www.google.com'),
    ('Github', 'https://www.github.com'),
    ('Taobao', 'https://www.taobao.com'),
    ('stackoverflow', 'https://www.stackoverflow.com/')
]

mycursor.executemany(sql, val)
mydb.commit()  # 数据表内容有更新，必须使用到该语句
print(mycursor.rowcount, "记录插入成功。")

print("===============================================================================================")

# 查询数据
mycursor.execute("SELECT * FROM sites")

myresult = mycursor.fetchall()  # fetchall() 获取所有记录

for x in myresult:
    print(x)

print("===============================================================================================")

# 读取指定字段数据
mycursor.execute("SELECT name, url FROM sites")

myresult = mycursor.fetchall()

for x in myresult:
    print(x)

# 只想读取一条数据，可以使用 fetchone()
mycursor.execute("SELECT * FROM sites")

myresult = mycursor.fetchone()

print(myresult)

print("===============================================================================================")

# where 条件语句
sql = "SELECT * FROM sites WHERE name ='RUNOOB'"

mycursor.execute(sql)

myresult = mycursor.fetchall()

for x in myresult:
    print(x)

# 占位符查询
sql = "SELECT * FROM sites WHERE name = %s"
na = ("RUNOOB",)

mycursor.execute(sql, na)

myresult = mycursor.fetchall()

for x in myresult:
    print(x)

print("===============================================================================================")

# 删除语句
sql = "DELETE FROM sites WHERE name = 'stackoverflow'"

mycursor.execute(sql)

mydb.commit()

print(mycursor.rowcount, " 条记录删除")
print("===============================================================================================")

# 更新语句
sql = "UPDATE sites SET name = %s WHERE name = %s"
val = ("Zhihu", "ZH")

mycursor.execute(sql, val)

mydb.commit()

print(mycursor.rowcount, " 条记录被修改")
print("===============================================================================================")

# 删除表
sql = "DROP TABLE IF EXISTS sites"  # 删除数据表 sites

mycursor.execute(sql)

print("===============================================================================================")

```

# mysql客户端  pymysql模块

```python
# -*- coding: utf-8 -*-
# PyMySQL 测试
# 需要安装提供的驱动  python -m pip install PyMySQL
# PyMySQL 遵循 Python 数据库 API v2.0 规范，并包含了 pure-Python MySQL 客户端库。

import pymysql

# 打开数据库连接
db = pymysql.connect("localhost", "testuser", "test123", "TESTDB")

# 使用 cursor() 方法创建一个游标对象 cursor
cursor = db.cursor()

# 使用 execute()  方法执行 SQL 查询
cursor.execute("SELECT VERSION()")

# 使用 fetchone() 方法获取单条数据.
data = cursor.fetchone()

print("Database version : %s " % data)

# 关闭数据库连接
db.close()

print("===============================================================================================")
'''
创建数据库表
'''

# 打开数据库连接
db = pymysql.connect("localhost", "testuser", "test123", "TESTDB")
# 使用 cursor() 方法创建一个游标对象 cursor
cursor = db.cursor()

# 使用 execute() 方法执行 SQL，如果表存在则删除
cursor.execute("DROP TABLE IF EXISTS EMPLOYEE")

# 使用预处理语句创建表
sql = """CREATE TABLE EMPLOYEE (
         FIRST_NAME  CHAR(20) NOT NULL,
         LAST_NAME  CHAR(20),
         AGE INT,  
         SEX CHAR(1),
         INCOME FLOAT )"""

cursor.execute(sql)

# 关闭数据库连接
db.close()
print("===============================================================================================")

'''
数据库插入操作
'''
# 打开数据库连接
db = pymysql.connect("localhost", "testuser", "test123", "TESTDB")
# 使用cursor()方法获取操作游标
cursor = db.cursor()

# SQL 插入语句
sql = """INSERT INTO EMPLOYEE(FIRST_NAME,
         LAST_NAME, AGE, SEX, INCOME)
         VALUES ('Mac', 'Mohan', 20, 'M', 2000)"""
try:
    # 执行sql语句
    cursor.execute(sql)
    # 提交到数据库执行
    db.commit()
except:
    # 如果发生错误则回滚
    db.rollback()

# 关闭数据库连接
db.close()

# 插入占位符操作
sql = "INSERT INTO EMPLOYEE(FIRST_NAME, \
       LAST_NAME, AGE, SEX, INCOME) \
       VALUES ('%s', '%s',  %s,  '%s',  %s)" % \
      ('Mac', 'Mohan', 20, 'M', 2000)

print("====================================================================================")

'''
数据库查询操作
fetchone(): 该方法获取下一个查询结果集。结果集是一个对象
fetchall(): 接收全部的返回结果行.
rowcount: 这是一个只读属性，并返回执行execute()方法后影响的行数。
'''
# 打开数据库连接
db = pymysql.connect("localhost", "testuser", "test123", "TESTDB")

# 使用cursor()方法获取操作游标
cursor = db.cursor()

# SQL 查询语句
sql = "SELECT * FROM EMPLOYEE \
       WHERE INCOME > %s" % (1000)
try:
    # 执行SQL语句
    cursor.execute(sql)
    # 获取所有记录列表
    results = cursor.fetchall()
    for row in results:
        fname = row[0]
        lname = row[1]
        age = row[2]
        sex = row[3]
        income = row[4]
        # 打印结果
        print("fname=%s,lname=%s,age=%s,sex=%s,income=%s" % \
              (fname, lname, age, sex, income))
except:
    print("Error: unable to fetch data")

# 关闭数据库连接
db.close()

print("====================================================================")

'''
数据库更新操作
'''
# 打开数据库连接
db = pymysql.connect("localhost", "testuser", "test123", "TESTDB")

# 使用cursor()方法获取操作游标
cursor = db.cursor()

# SQL 更新语句
sql = "UPDATE EMPLOYEE SET AGE = AGE + 1 WHERE SEX = '%c'" % ('M')
try:
    # 执行SQL语句
    cursor.execute(sql)
    # 提交到数据库执行
    db.commit()
except:
    # 发生错误时回滚
    db.rollback()

# 关闭数据库连接
db.close()

print("=========================================================================")

'''
删除操作
'''
# 打开数据库连接
db = pymysql.connect("localhost", "testuser", "test123", "TESTDB")

# 使用cursor()方法获取操作游标
cursor = db.cursor()

# SQL 删除语句
sql = "DELETE FROM EMPLOYEE WHERE AGE > %s" % (20)
try:
    # 执行SQL语句
    cursor.execute(sql)
    # 提交修改
    db.commit()
except:
    # 发生错误时回滚
    db.rollback()

# 关闭连接
db.close()

print("==============================================================================")

```

