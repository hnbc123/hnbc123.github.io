---
title: Spring AOP
date: 2023-04-17 20:23:00
categories:
  - SSM
  - Spring
---

# 基本使用

基础概念：
![AOP](AOP概念.png)

使用步骤：

1. 添加依赖

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>5.2.23.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aop</artifactId>
    <version>5.2.23.RELEASE</version>
</dependency>
```

2. 将目标类和切面类（封装了通知方法，即在目标方法执行前后执行的方法）加入 ioc 容器中

```java
public interface Calculator {
    int add(int i, int j);
    int sub(int i, int j);
    int multiply(int i, int j);
    int divided(int i, int j);
}

// 目标类
@Component
public class MyCalculator implements Calculator {
    @Override
    public int add(int i, int j) {
        return i + j;
    }

    @Override
    public int sub(int i, int j) {
        return i - j;
    }

    @Override
    public int multiply(int i, int j) {
        return i * j;
    }

    @Override
    public int divided(int i, int j) {
        return i / j;
    }
}

// 切面类
@Component
public class LogUtils {
    public static void logStart() {
        System.out.println("方法开始执行");
    }

    public static void logReturn() {
        System.out.println("方法正常执行完成");
    }

    public static void logException() {
        System.out.println("方法出现异常");
    }

    public static void logEnd() {
        System.out.println("方法最终结束");
    }
}
```

3. 在切面类上添加@Aspect 注解，告诉 Spring 哪个是切面类

```java
@Aspect
@Component
public class LogUtils {
// 内容略
}
```

4. 声明切面方法在什么时候执行

- @Before：前置通知，在目标方法之前执行
- @After：后置通知，在目标方法结束之后执行
- @AfterReturning：返回通知，在目标方法正常返回之后执行
- @AfterThrowing：异常通知，在目标方法抛出异常之后执行
- @Around：环绕通知

```java
@Aspect
@Component
public class LogUtils {
    // execution(访问权限符 返回值类型 方法签名)
    @Before("execution(public int com.study.impl.MyMathCalculator.*(int, int))")
    public static void logStart() {
        System.out.println("方法开始执行");
    }

    @AfterReturning("execution(public int com.study.impl.MyMathCalculator.*(int, int))")
    public static void logReturn() {
        System.out.println("方法正常执行完成");
    }

    @AfterThrowing("execution(public int com.study.impl.MyMathCalculator.*(int, int))")
    public static void logException() {
        System.out.println("方法出现异常");
    }

    @After("execution(public int com.study.impl.MyMathCalculator.*(int, int))")
    public static void logEnd() {
        System.out.println("方法最终结束");
    }
}
```

1. 开启基于注解的 AOP 模式

xml 文件添加声明：

```xml
xmlns:aop="http://www.springframework.org/schema/aop"

<!-- xsi:schemaLocation中添加：-->
http://www.springframework.org/schema/aop
http://www.springframework.org/schema/aop/spring-aop.xsd
```

添加配置：

```xml
<aop:aspectj-autoproxy/>
```

测试：

```java
@Test
public void test05() {
    // 从ioc容器中获取目标对象。注意：一定要用接口类型，不要用本类
    Calculator bean = ioc.getBean(Calculator.class);
    int add = bean.add(5, 1);
    System.out.println(add);
}
```

# AOP 细节

## IOC 容器中保存的是代理对象

从 ioc 容器中获取目标对象时，容器中保存的是代理对象。
AOP 的底层是动态代理，代理类实现接口的情况下，用本类的类型无法获取对象。ioc 容器保存的对象类型为 com.sun.proxy.$Proxy12，所以用类型获取需要使用接口类型。

如果代理类没有实现接口，保存的对象就是本类类型，是 cglib 创建的代理对象：MyMathCalculator&#36;&#36;EnhancerBySpringCGLIB&#36;&#36;7218ba5d
6

## 切入点表达式写法

固定格式：execution(访问权限符 返回值类型 方法全类名(参数表))

通配符：

- \*:
  - 匹配一个或多个字符:execution(public int com.study.impl.MyMat\*r.\*(int, int))
  - 匹配任意一个参数：execution(public int com.study.impl.MyMathCalculator.\*(int, \*))
  - 匹配一层路径：execution(public int com._.impl.MyMathCalculator.\*(int, _))
  - 匹配任意返回值，但不能匹配权限，权限直接省略即可
- ..:
  - 匹配任意多个参数，任意类型参数：execution(public int com.study.impl.MyMath\*.\*(..))
  - 匹配任意多层路径：execution(public int com.study..MyMathCalculator.\*(int, \*))

逻辑运算符：

- &&：切入的位置满足所有表达式
- ||：切入的位置满足任意一个表达式

总结：

- 最精确：execution(public int com.study.impl.MyMathCalculator.add(int, int))
- 最省略：execution(\* \*(..))

## 通知方法执行顺序

```java
try {
  @Before
  method.invoke(obj, args);
  @AfterReturning
} catch () {
  @AfterThrowing
} finally {
  @After
}
```

Spring5 之前后置通知在返回通知之前运行，Spring5 开始后置通知最后运行

## 获取目标方法信息

在通知方法的参数列表添加 JoinPoint 获取方法信息：

```java
@Before("execution(public int com.atguigu.impl.MyMathCalculator.*(int, int))")
public static void logStart(JoinPoint joinPoint) {
    Object[] args = joinPoint.getArgs();
    Signature signature = joinPoint.getSignature();
    String name = signature.getName();
    System.out.println(name + "方法开始执行，参数：" + Arrays.asList(args));
}
```

切面方法参数列表的每个参数都需要 Spring 进行管理，添加参数需要在注解中通过属性指定。

在注解中加入 returning 属性，指定用哪个值来接受返回值

```java
@AfterReturning(value = "execution(public int com.atguigu.impl.MyMathCalculator.*(int, int))", returning = "result")
public static void logReturn(JoinPoint joinPoint, Object result) {
    Object[] args = joinPoint.getArgs();
    Signature signature = joinPoint.getSignature();
    String name = signature.getName();
    System.out.println(name + "方法正常执行完成，计算结果为" + result);
}
```

使用 throwing 属性指定用哪个参数接受异常：

```java
@AfterThrowing(value = "execution(public int com.atguigu.impl.MyMathCalculator.*(int, int))", throwing = "exception")
public static void logException(JoinPoint joinPoint, Exception exception) {
    Signature signature = joinPoint.getSignature();
    String name = signature.getName();
    System.out.println(name + "方法出现异常，信息为：" + exception.getMessage());
}
```

## 抽取可重用的切入点表达式

声明一个没有实现的 void 类型空方法，在方法上标注@Pointcut 注解

```java
@Pointcut("execution(public int com.study.impl.MyMathCalculator.*(int, int))")
public void MyPoint(){}
```

在切面方法上使用方法名称代替切入点表达式

```java
@Before("MyPoint()")
```

# 环绕通知

## 基本使用

```java
@Around("MyPoint()")
public Object myAround(ProceedingJoinPoint point) throws Throwable {
    Object[] args = point.getArgs();
    Object proceed = null;
    try {
        System.out.println("前置通知");
        // 利用反射调用目标方法，就是method.invoke(obj, args)
        proceed = point.proceed(args);
        System.out.println("后置通知");
    } catch (Exception e) {
        System.out.println("异常通知");
    } finally {
        System.out.println("返回通知");
    }

    // 反射调用后的返回值也要返回出去
    return proceed;
}
```

## 环绕通知执行顺序

环绕通知优先于普通通知执行，执行顺序：

- 【环绕前置】
- 【普通前置】
- 【环绕执行】：目标方法执行
- 【普通后置】
- 【普通返回】
- 【环绕后置】
- 【环绕返回】

出现异常时执行顺序：

- 【环绕前置】
- 【普通前置】
- 【环绕执行】：目标方法执行
- 【普通异常】
- 【普通返回】
- 【环绕异常】
- 【环绕返回】

有多个切面类处理同一方法时，按切面类字母排序，顺序靠前的包裹在外。
可以通过@Order()改变执行顺序，序号小的在外
