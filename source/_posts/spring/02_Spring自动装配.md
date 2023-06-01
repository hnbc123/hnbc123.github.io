---
title: Spring自动装配
date: 2023-04-17 20:23:00
categories:
  - SSM
  - Spring
---

# 基于xml的自动装配

在xml文件中添加配置：
```xml
<bean id="car" class="com.study.bean.Car" />

<!--
    为Person里的自定义类型属性赋值
    autowire="default/no"：不自动装配
    autowire="byName"：以属性名作为id去容器中找到这个组件，找不到就装配null
    autowire="byType"：以属性的类型去容器中找到这个组件
    autowire="constructor"：按照构造器进行赋值（不会报错）
        1） 先按照有参构造器参数的类型进行装配，成功就赋值，没有就直接装配null
        2） 如果按照类型找到了多个，参数名作为id继续匹配，成功就赋值，没有返回null

-->
<bean id="person" class="com.study.bean.Person" autowire="constructor" />
```

如果使用的是构造器赋值，添加有参构造器：
```java
public Person(Car car) {
    this.car = car;
}
```





# 基于注解的自动装配

## 基本使用

通过给bean上添加某些注解，可以快速地将bean加入到ioc容器中。
Spring有四个注解：
* @Controller：推荐给控制器层组件添加
* @Service：推荐给业务逻辑层的组件添加
* @Repository：给数据库层的组件添加
* @Component：给不属于以上几层的组件添加

Spring底层不会验证这个组件与注解是否匹配，可以随意选一个注解使用，但推荐各自层添加各自注解。

需要在xml中配置自动组件扫描：
在声明中添加：
```
xmlns:context="http://www.springframework.org/schema/context"

// 添加在xsi:schemaLocation中
http://www.springframework.org/schema/context
http://www.springframework.org/schema/context/spring-context.xsd
```


开启自动装配：
```xml
<!--
    base-package：指定扫描的基础包，把基础包及下面的所有包加了注解的类自动添加进ioc容器
-->
<context:component-scan base-package="com.study"/>
```


使用注解将组件加入容器中的要点：
* 给要添加的组件上标四个注解的任何一个
* 告诉Spring自动扫描加了注解的组件，依赖context名称空间
* 需要添加aop相关依赖
* 组件的id默认是组件的类名首字母小写，可在组件后添加名称来自定义id：@Repository("myBook")
* 作用域默认是单例的，可通过注解@Scope("prototype")指定为多例




## 指定扫描时排除的组件

```xml
<context:component-scan base-package="com.study">
    <!--
        扫描时排除一些不要的组件
        type="annotation"：指定排除规则, expression=""：注解的全类名
        type="assignable"：指定排除某个具体的类， expression=""：类的全类名
        type="aspectj"：aspectj表达式
        type="custom"：自定义TypeFilter决定哪些使用
        type="regex"：正则表达式
    -->
    <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
```




## 指定扫描时包含的组件

```xml
<context:component-scan base-package="com.study" use-default-filters="false">
    <!-- 指定扫描时包含的类，默认全部扫描，需要关闭默认的过滤规则 -->
    <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
```




## @Autowired实现自动装配

标有@Autowired注解的属性，Spring会自动为其赋值，去容器中找到这个属性对应的组件。

```java
@Controller
public class BookController {
    @Autowired
    private BookService bookService;

    public void saveBook() {
        bookService.save();
    }
}
```

@Autowired原理：
先按照类型去容器中找到对应的组件：
* 找到一个，直接赋值
* 没找到，抛出异常
* 找到多个，按照变量名作为id继续匹配，找到则赋值，否则抛异常


补充：
* 可以使用@Qualifier指定id名称，而不使用变量名。
* 将Autowired注解的required属性设为false，找不到属性时赋值为null而不是报错。
* 在方法上加@Autowired注解，方法上的每个参数都会自动注入值





# 泛型依赖注入

DAO层：
```java
public abstract class BaseDao<T> {
    public abstract void save();
}

@Repository
public class BookDao extends BaseDao<Book> {
    @Override
    public void save() {
        System.out.println("保存图书");
    }
}

@Repository
public class UserDao extends BaseDao<User>{
    @Override
    public void save() {
        System.out.println("保存用户");
    }
}
```


Service层：
```java
public class BaseService<T> {
    @Autowired
    private BaseDao<T> baseDao;

    public void save() {
        baseDao.save();
    }
}

@Service
public class BookService extends BaseService<Book> {}

@Service
public class UserService extends BaseService<User> {}
```


测试：
```java
@Test
public void test0() {
    BookService bookService = ioc.getBean(BookService.class);
    UserService userService = ioc.getBean(UserService.class);

    bookService.save();
    userService.save();
}
```