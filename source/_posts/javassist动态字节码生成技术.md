---
title: javassist动态字节码生成技术
date: 2019-11-22 17:34:01
tags: [javassist]
---

# javassist动态字节码生成技术

[demo地址](https://github.com/AsummerCat/javassistdemo)

`Javassist`是一个开源的分析、编辑和创建Java字节码的类库。是由东京工业大学的数学和计算机科学系的 Shigeru Chiba （千叶 滋）所创建的。它已加入了开放源代码JBoss 应用服务器项目,通过使用Javassist对字节码操作为JBoss实现动态"AOP"框架。

Javassist的官方网站：http://jboss-javassist.github.io/javassist/

 通过javasssit，我们可以：

- 动态创建新类或新接口的二进制字节码
- 动态扩展现有类或接口的二进制字节码(AOP)

## **1.动态创建新类或新接口的二进制字节码**

假设我们需要生成一个User类：

```
package com.linjingc.reflecttest;

public class User {
	private String name;

	public User(String name) {
		this.name = name;
	}

	public User() {
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	@Override
	public String toString() {
		return "name=" + name;
	}
}
```

<!--more-->

## javassist创建代码如下：

```
/**
 * 动态创建新类或新接口的二进制字节码
 */
public class UserGenerator {
	public static void main(String[] args) throws Exception {
		ClassPool classPool = ClassPool.getDefault();
		//定义User类
		CtClass ctClassUser = classPool.makeClass("com.linjingc.reflecttest.User");

		//定义name字段
		CtClass fieldType = classPool.get("java.lang.String");//字段类型
		String name = "name";//字段名称
		CtField ctFieldName = new CtField(fieldType, name, ctClassUser);
		ctFieldName.setModifiers(Modifier.PRIVATE);//设置访问修饰符
		ctClassUser.addField(ctFieldName, CtField.Initializer.constant("javasssit"));//添加name字段，赋值为javassist

		//定义构造方法
		CtClass[] parameters = new CtClass[]{classPool.get("java.lang.String")};//构造方法参数
		CtConstructor constructor = new CtConstructor(parameters, ctClassUser);
		String body = "{this.name=$1;}";//方法体 $1表示的第一个参数
		constructor.setBody(body);
		ctClassUser.addConstructor(constructor);

		//setName getName方法
		ctClassUser.addMethod(CtNewMethod.setter("setName", ctFieldName));
		ctClassUser.addMethod(CtNewMethod.getter("getName", ctFieldName));

		//toString方法
		CtClass returnType = classPool.get("java.lang.String");
		String methodName = "toString";
		CtMethod toStringMethod = new CtMethod(returnType, methodName, null, ctClassUser);
		toStringMethod.setModifiers(Modifier.PUBLIC);
		String methodBody = "{return \"name=\"+$0.name;}";//$0表示的是this
		toStringMethod.setBody(methodBody);
		ctClassUser.addMethod(toStringMethod);

		//代表class文件的CtClass创建完成，现在将其转换成class对象
		Class clazz = ctClassUser.toClass();
		Constructor cons = clazz.getConstructor(String.class);
		Object user = cons.newInstance("嘿嘿");
		Method toString = clazz.getMethod("toString");
		System.out.println(toString.invoke(user));

		ctClassUser.writeFile(".");//在当前目录下，生成com/linjingc/reflecttest/User.class文件
	}
}
```

运行程序后输出：

```
name=嘿嘿
```

## 动态扩展现有类或接口的二进制字节码(AOP)

假设我们现在有如下一个类

```
public class Looper {
	public void loop() {
		try {
			System.out.println("Looper.loop() invoked");
			Thread.sleep(1000L);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}
```

现在我们想统计Looper的loop方法的耗时时间。最简单的思路是使用javassist修改loop方法的源码：

在最前加入：long start=System.currentTimeMillis();

在最后插入：System.out.println("耗时:"+(System.currentTimeMillis()-start)+"ms");

### 错误案例

如下：

```
public class Looper {
 
    public void loop(){
 
        //记录方法调用的开始时间
 
        long start=System.currentTimeMillis();
 
        try {
 
            System.out.println("Looper.loop() invoked");
 
            Thread.sleep(1000L);
 
        } catch (InterruptedException e) {
 
            e.printStackTrace();
 
        }
 
        //方法结束时打印耗时
 
        System.out.println("耗时:"+(System.currentTimeMillis()-start)+"ms");
 
    }
 
}
```

javassist的CtClass方法提供的insertBefore和insertAfter方法，允许我们在一个方法开始和结束添加自己的代码。

```
/**
 * 错误案例
 * aop切入 环绕
 */
public class JavassisTimingWrong {

	public static void main(String[] args) throws Exception {

		//需要修改的已有的类名和方法名

		String className = "com.linjingc.aoptest.Looper";

		String methodName = "loop";

		ClassPool classPool = ClassPool.getDefault();

		CtClass clazz = classPool.get(className);

		CtMethod method = clazz.getDeclaredMethod(methodName);

		method.insertBefore("long start=System.currentTimeMillis();");

		method.insertAfter("System.out.println(\"耗时:\"+(System.currentTimeMillis()-start)+\"ms\");");

		//调用修改的Looper类的loop方法

		Looper looper = (Looper) clazz.toClass().newInstance();

		looper.loop();

	}

}
```

此时：

因此运行时，会爆出类似以下的错

![javassist异常](/img/2019-11-22/javassist1.png)

这是因为，javassist插入的代码片段中，每次插入操作的代码，称之为一个插入代码块，后面的插入块不能使用前面的插入块定义的局部变量，而言且也不能使用方法中的原有局部变量。而上述代码中，我们分表调用了insertBefore和insertAfter插入了两个代码块，而后面的插入块不能使用前面的插入块定义的局部变量start，因此爆出了上面的错。

而如果代码片段都位于一个插入块中，则局部变量是可以引用的。因此考虑使用如下的方法实现：

具体的思路是：将原有的loop方法名改为loop$impl，然后再定义一个loop方法，新的loop方法内部会调用loop$impl，在调用之前和调用之后分别加入上述的代码片段。

### 正确案例

实现如下：

```
/**
 * 正确案例
 * aop切入 环绕
 */
public class JavassisTiming {
	public static void main(String[] args) throws Exception {
		//需要修改的已有的类名和方法名
		String className = "com.linjingc.aoptest.Looper";
		String methodName = "loop";

		//修改为原有类的方法名为loop$impl
		CtClass clazz = ClassPool.getDefault().get(className);
		CtMethod method = clazz.getDeclaredMethod(methodName);
		String newname = methodName + "$impl";
		method.setName(newname);

		//使用原始方法名loop，定义一个新方法，在这个方法内部调用loop$impl
		CtMethod newMethod = CtNewMethod.make("public void " + methodName + "(){" +
						"long start=System.currentTimeMillis();" +
						"" + newname + "();" +//调用loop$impl
						"System.out.println(\"耗时:\"+(System.currentTimeMillis()-start)+\"ms\");" +
						"}"
				, clazz);
		clazz.addMethod(newMethod);

		//调用修改的Looper类的loop方法
		Looper looper = (Looper) clazz.toClass().newInstance();
		looper.loop();
	}
}
```

输出：

```
Looper.loop() invoked耗时:1000ms
```

### 动态代理

此外还有一种更加简单的方式

```
/**
 * 动态代理
 */
public class JavassistAop {
	public static void main(String[] args) throws IllegalAccessException, InstantiationException {
		ProxyFactory factory = new ProxyFactory();
		//设置父类，ProxyFactory将会动态生成一个类，继承该父类
		factory.setSuperclass(Looper.class);
		factory.setFilter(new MethodFilter() {
			@Override
			public boolean isHandled(Method m) {
				if (m.getName().equals("loop")) {
					return true;
				}
				return false;
			}
		});
		//设置拦截处理
		factory.setHandler(new MethodHandler() {
			@Override
			public Object invoke(Object self, Method thisMethod, Method proceed,
			                     Object[] args) throws Throwable {
				long start = System.currentTimeMillis();
				Object result = proceed.invoke(self, args);
				System.out.println("耗时:" + (System.currentTimeMillis() - start) + "ms");
				return result;
			}
		});
		Class<?> c = factory.createClass();
		Looper object = (Looper) c.newInstance();
		object.loop();
	}
}
```

