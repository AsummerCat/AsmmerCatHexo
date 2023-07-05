---
title: java9到17新特性
date: 2023-07-05 16:15:46
tags: [java]
---
# java9到17新特性

# <font color="red">JAVA9新特性</font>

## 1.包,模块

主要就是为了减少依赖
不然引入第三方包就是全量引用

//模块名称可以随便起,注意必须是唯一的,已经模块内的包名也得是唯一的,即使模块不同

那么JDK被拆为了哪些模块呢？打开终端执行`java --list-modules`查看。
<!--more-->
根目录创建`module-info.java`
```
    module-info.java

    module demo {
        requires java.logging;
        requires spring.boot;
        requires spring.boot.autoconfigure; //除了JDK的一些常用包之外,只有我们明确需要的模块才会导入依赖库
        //当然如果要导入JAVA SE 所有的依赖的, 直接requires java.se; 即可;
    //    requires java.se;
    }
```
ps:
```
    java.lang.IllegalAccessException-->module demo does not open com.linjingc.demo to unnamed module @69379752
     
    如果出现这个错误 请检查JDK是否为17
    是的话 需要添加
    VM参数:
    --add-opens java.base/java.lang=ALL-UNNAMED
```
需要注意的是结构的话 module-info应该放在最外层

```
-java
--自定义文件夹
---对应java类
--module-info.java

```

如果maven子工程要使用我们的内容需要这么操作
比如B模块要用到A模块
```
    需要在A模块的module-info中
    添加
    exports com.xxx

    如果需要传递依赖的话->则需要在原本的A依赖中加上
    requires java.logging;
    修改为
    requires transitive java.logging;
    这样就是传递依赖了
    B就可以不引用会直接使用logging

```

```
B模块中requests  A的module名称
```

这样就是实现了引用

## 2.接口中允许私有化方法
```
    public interface test1 {
        default void test() {
            System.out.println("helloword");
        }

        private String test11() {
            System.out.println("heloo");
            return null;
        }
    }
```
## 3.可以快速创建 不可变集合
```
           //注意 这个是一个不可变集合
            Map<String, Integer> aaa = Map.of("AAA", 1, "BBB", 2);
            List<String> a = List.of("A", "B", "C", "D");
            Set<String> a1 = Set.of("A", "B", "C", "D");
```
## 4. String底层数据存储结构修改

对字符串采用更节省空间的内部表示,
将原有char 修改为byte 节省一半空间

ps: 字符串是堆使用的主要组成部分

# <font color="red">JAVA10新特性</font>

## 1.局部变量类型推断 可以直接用var了

    //注意只能用于局部变量 ,不能当做全局变量

    var a="A";
    var b=1;

# <font color="red">JAVA11新特性</font>

## 1.新增对于String的静态方法

```
    String a=" ABCD ";
        // 静态方法:重复拼接 repeat
        String repeat = a.repeat(2);
        System.out.println(repeat);

        //静态方法:快速去空格
        String b=" A B C D ";
        
        System.out.println(b.trim());
        
        //trim()可以去除字符串前后的半角空白字符
        //strip()可以去除字符串前后的全角和半角空白字符
        //去掉前后空格 strip
        System.out.println(b.strip());
        
        //去掉头部空格 stripLeading
        System.out.println(b.stripLeading());
        
        //去掉尾部空格 stripTrailing
        System.out.println(b.stripTrailing());


```

## 2.新增全新的HttpClient

全新HTTP Client API 支持最新的HTTP2和WebSocket协议
```
    import java.net.URI;
    import java.net.http.HttpClient;
    import java.net.http.HttpRequest;
    import java.net.http.HttpResponse;

            //全新HTTP Client API 支持最新的HTTP2和WebSocket协议
            HttpClient httpClient = HttpClient.newHttpClient();
            //请求正文
            HttpRequest httpRequest = HttpRequest.newBuilder().uri(new URI("https://www.baidu.com")).GET().header("token", "A").build();
            //发送 并且返回body,而且是String形式
            String body = httpClient.send(httpRequest, HttpResponse.BodyHandlers.ofString()).body();
            System.out.println(body);
```
# <font color="red">JAVA12-16新特性</font>

## 1.switch表达式优化 可使用xx

可以携带返回值

//新特性 可以直接使用 case xx -> "xx"; 这种形式 不用写break ,如果需要有逻辑处理 则使用yield返回

```
        var a = 100 / 2;
        //新特性 可以直接使用 case xx -> "xx"; 这种形式 不用写break ,如果需要有逻辑处理 则使用yield返回
        String grade = switch (a) {
            case 10, 9 -> "优秀";
            case 6 -> "不及格";
            case 50 -> {
                yield "优秀1";
            }
            default -> "不及格";
        };

        System.out.println(grade);

```

## 2.文字块

使用三引号 可以不用写转义符,注意空格不会移除
仅仅是不用做转义, 其他操作可以参考字符串的静态方法;

## 3. instanceof语法的改造

类型判断后 可以直接返回转换后的类型

```
    private static boolean ins(Object a) {
        if (a instanceof test b) {
            System.out.println(b.name);
            return true;
        }
        return false;
    }
        
    public static void main(String[] args) throws Exception {
        Object a = new test("小明");
        ins(a);
    }

```

## 4.空指针的优化

会显示更详细的异常信息

## 5.新增记录类型  \*.Record   (常用 类似lombok注解的概念)

可以简单理解为一个普通类
编译时,会自动编译出 get hashcode 等方法,
要注意的是 只会自动生成get方法,没有set方法;
```
    可以将字段写在括号内
    public record A(String name, String age) { }


    使用:

     A a1 = new A("XIAO", "A");
```
# <font color="red">JAVA17新特性</font>

## 1.类的 密封类型  (有点类似继承extends关键字)

可以实现 指定类可以继承(final)该类,其他类不允许继承
语法: public sealed A permits G
表示: A这个类,仅仅允许G继承A,生成后的G是final类型的 不可在继承

non-sealed 表示重新打开,允许所有类继承

```
//创建一个密封类,仅允许 G继承 
public sealed class C permits G {}

// G 继承了A的内容 并且变成的final
public non-sealed class G extends C { }

```

# <font color="red">JAVA18新特性</font>

## 1.默认是用UTF-8字符编码

# <font color="red">JAVA19新特性</font>

## 1.虚拟线程

### 1.1 通过Thread.startVirtualThread直接创建一个虚拟线程
```
    Runable task =()->{
        System.out.println("创建虚拟线程");
    }

    Thread.startVirtualThread(task);
```
### 1.2 Thread.ofVirtual 创建一个虚拟线程
```
    Thread vt1=Thread.ofVirtual.name("虚拟线程").unstarted(task); //创建虚拟线程

    Thread vt1=Thread.ofPlatform.name("平台线程").unstarted(task); //创建平台线程

    vt.start();//启动虚拟线程
```
### 1.3 使用Executors.newVirtualThreadPerTaskExecutor() 创建虚拟线程

### 1.4 判断是否是虚拟线程还是平台线程
```
    vt1.isVirtual();

    true 表示是虚拟线程
```
