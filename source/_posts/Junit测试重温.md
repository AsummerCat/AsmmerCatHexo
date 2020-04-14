---
title: Junit测试重温
date: 2020-04-13 17:15:27
tags: [junit,java]
---

# junit4 测试用例



## @beforeClass

```
被@BeforeClass注解的方法会是：
只被执行一次
运行junit测试类时第一个被执行的方法
这样的方法被用作执行计算代价很大的任务，如打开数据库连接。被@BeforeClass 注解的方法应该是静态的（即 static类型的）.
```

## @AfterClass注解

```
被@AfterClass注解的方法应是：

只被执行一次
运行junit测试类是最后一个被执行的方法
该类型的方法被用作执行类似关闭数据库连接的任务。被@AfterClass 注解的方法应该是静态的（即 static类型的）.
```

<!--more-->

## @Before注解

```
被@Before 注解的方法应是：
junit测试类中的任意一个测试方法执行 前 都会执行此方法
该类型的方法可以被用来为测试方法初始化所需的资源。
```

## @After注解

```
被@After注解的方法应是：
junit测试类中的任意一个测试方法执行后 都会执行此方法, 即使被@Test 或 @Before修饰的测试方法抛出异常
该类型的方法被用来关闭由@Before注解修饰的测试方法打开的资源。
```

## @Test 注解

```
被@Test注解的测试方法包含了真正的测试代码，并且会被Junit应用为要测试的方法。@Test注解有两个可选的参数：
expected 表示此测试方法执行后应该抛出的异常，（值是异常名）
timeout 检测测试方法的执行时间
```

## 断言

```
assertNull(java.lang.Object object)	                       检查对象是否为空
assertNotNull(java.lang.Object object)	                   检查对象是否不为空
assertEquals(long expected, long actual)	               检查long类型的值是否相等
assertEquals(double expected, double actual, double delta)  检查指定精度的double值是否相等
assertFalse(boolean condition)                           	检查条件是否为假
assertTrue(boolean condition)	                            检查条件是否为真
assertSame(java.lang.Object expected, java.lang.Object actual)	检查两个对象引用是否引用同一对象（即对象是否相等）
assertNotSame(java.lang.Object unexpected, java.lang.Object actual)	
检查两个对象引用是否不引用统一对象(即对象不等)
```

## 参数化测试

```
1.对测试类添加注解 @RunWith(Parameterized.class)
2.将需要使用变化范围参数值测试的参数定义为私有变量
3.使用上一步骤声明的私有变量作为入参，创建构造函数
4.创建一个使用@Parameters注解的公共静态方法，它将需要测试的各种变量值通过集合的形式返回。
5.使用定义的私有变量定义测试方法
```

```java
package in.co.javatutorials;
 
import static org.junit.Assert.*;
 
import java.util.Arrays;
import java.util.Collection;
 
import org.junit.Test;
import org.junit.runner.RunWith;
import org.junit.runners.Parameterized;
import org.junit.runners.Parameterized.Parameters;
 
/**
* @author javatutorials.co.in
*/
// Step 1
@RunWith(Parameterized.class)
public class EvenNumberCheckerTest {
 
    // Step 2: variables to be used in test method of Step 5
    private int inputNumber;
    private boolean isEven;
 
    // Step 3: parameterized constructor
    public EvenNumberCheckerTest(int inputNumber, boolean isEven) {
        super();
        this.inputNumber = inputNumber;
        this.isEven = isEven;
    }
 
    // Step 4: data set of variable values
    @Parameters
    public static Collection<Object[]> data() {
        Object[][] data = new Object[][] {
                { 2, true },
                { 5, false },
                { 10, false }
        };
        return Arrays.asList(data);
    }
 
    @Test
    public void test() {
        System.out.println("inputNumber: " + inputNumber + "; isEven: " + isEven);
        EvenNumberChecker evenNumberChecker = new EvenNumberChecker();
        // Step 5: use variables in test code
        boolean actualResult = evenNumberChecker.isEven(inputNumber);
        assertEquals(isEven, actualResult);
    }
}
```

## 测试套件 批量执行

```java
Junit 4允许通过使用测试套件类批量运行测试类 . 为一套测试类创建一个测试套件，要为测试类添加以下注解：
@RunWith(Suite.class)
@SuiteClasses(TestClass1.class, TestClass2.class)
当运行时，所有包含在@SuiteClasses注解内的所有测试类都会被执行。
```

## 忽略测试

```
添加@ignore注解
1.添加到测试方法上 忽略该测试方法
2.添加到测试类上 忽略该类的测试方法
```

## 超时测试

```
Junit 4超时测试（Timeout test）可以被用来测试方法的执行时间。 Junit 4 超时测试可以被用在:
在测试类的方法上使用 @Timeout 注解
测试类的所有方法应用 Timeout规则
```

```
1.直接使用@Timeout

@Test(timeout = 200)
    public void testTimeout() {
        while (true);
    }

```

```
2. 使用规则
 @Rule
    public Timeout timeout = new Timeout(1000);
    注意使用规则的话 是全局的 所有方法都不能超过该规则的时间
```



