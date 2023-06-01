---
title: Spring中bean的创建
date: 2023-04-17 20:23:00
categories:
  - SSM
  - Spring
---

# Spring 概述

## 模块划分

![spring](spring-overview.png)

- Test: Spring 的单元测试

- Core Container：核心容器（IOC），黑色代表所需要的 jar 包

  > spring-beans-[version].jar
  >
  > spring-core-[version].jar
  >
  > spring-context-[version].jar
  >
  > spring-express-[version].jar

- AOP+Aspect：面向切面编程

  > spring-aop-[version].jar

- Data Access：数据访问，ORM（Object Relation Mapping）

  > spring-jdbc-[version].jar
  >
  > spring-orm-[version].jar
  >
  > spring-tx-[version].jar

- Web：开发 Web 应用

  > spring-web-[version].jar，和原生 web 相关（servlet）
  >
  > spring-webmvc-[version].jar

maven 依赖：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>5.2.23.RELEASE</version>
</dependency>
<!--
	spring-context依赖于spring-core, spring-express, spring-aop及spring-beans,
 	spring-context依赖的jar包会被自动导入
-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.2.23.RELEASE</version>
</dependency>
<!--
	spring-orm依赖于spring-jdbc和spring-tx
-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-orm</artifactId>
    <version>5.2.23.RELEASE</version>
</dependency>
```

## IOC 和 DI

IOC (Inversion Of Control)：控制反转，将主动通过 new 关键字获取的资源转为被动的从容器接收资源。

DI (Dependency Injection)：依赖注入，容器能知道组件运行的时候需要哪些类，通过反射的方式将所需的对象注入到当前组件中。

# 组件注册

## 基本组件注册

1. 定义 Person 类

   ```java
   public class Person {
       private String lastName;
       private Integer age;
       private String gender;
       private String email;

       public Person() {}

       public Person(String lastName, Integer age, String gender, String email) {
           this.lastName = lastName;
           this.age = age;
           this.gender = gender;
           this.email = email;
       }

       public String getLastName() {
           return lastName;
       }

       public void setLastName(String lastName) {
           this.lastName = lastName;
       }

       public Integer getAge() {
           return age;
       }

       public void setAge(Integer age) {
           this.age = age;
       }

       public String getGender() {
           return gender;
       }

       public void setGender(String gender) {
           this.gender = gender;
       }

       public String getEmail() {
           return email;
       }

       public void setEmail(String email) {
           this.email = email;
       }

       @Override
       public String toString() {
           return "Person{" +
                   "lastName='" + lastName + '\'' +
                   ", age=" + age +
                   ", gender='" + gender + '\'' +
                   ", email='" + email + '\'' +
                   '}';
       }
   }
   ```

2. 在 xml 文件中注册需要容器管理的类

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

       <!-- 注册一个Person对象，Spring会自动创建这个Person对象 -->
       <!--
       一个Bean标签可以注册一个组件
       class：组件的全限定类名
       id：对象唯一标识
        -->
       <bean id="person01" class="com.study.bean.Person">
           <!-- 使用property标签为对象的属性赋值 -->
           <property name="lastName" value="张三" />
           <property name="age" value="18" />
       </bean>
   </beans>
   ```

3. 测试代码

   ```java
   public class IOCTest {
       // 代表ioc容器, 配置文件在类路径下
       // 如果配置文件在文件路径，可以使用 FileSystemXmlApplicationContext("F://ioc.xml")
       ApplicationContext ioc = new ClassPathXmlApplicationContext("ioc.xml");

       @Test
       public void test01() {
           Person bean = ioc.getBean("person01", Person.class);
           System.out.println(bean);
       }


       @Test
       public void test02() {
           // 根据bean的类型从IOC容器中获取实例
           // 如果有多个这个类型的bean，查找就会报错
           Person bean = ioc.getBean(Person.class);
           System.out.println(bean);
       }
   }
   ```

总结：

- 容器中对象的创建在容器创建完成的时候就已经创建好了
- 默认同一个组件在 ioc 容器中是单实例的
- 获取组件时，容器中如果没有对应组件，会报 NoSuchBeanDefinitionException
- ioc 容器在创建组件对象的时候，会利用 setter 方法为 javaBean 的属性进行赋值
- javaBean 的属性名由 getter/setter 方法决定，去掉 set 后面的字母小写就是属性名

## 通过构造器赋值

xml 文件配置

```xml
<bean id="person02" class="com.study.bean.Person">
   <!--
        通过有参构造器创建对象并赋值
        如果省略name属性，顺序必须和构造器参数位置一致，或用index指定索引,index从0开始
        如果有重载的情况，可通过type执行类型
		<constructor-arg value="15" index="2" type="java.lang.Integer"/>
    -->
    <constructor-arg name="lastName" value="lucy" />
    <constructor-arg name="age" value="15"/>
    <constructor-arg name="email" value="lucy@qq.com"/>
    <constructor-arg name="gender" value="0" />
</bean>
```

测试代码

```java
@Test
public void test02() {
    Person bean = ioc.getBean("person02", Person.class);
    System.out.println(bean);
}
```

## 通过 p 名称空间为 bean 赋值

在 xml 中名称空间是用来防止标签重复的。

使用 p 名称空间需要在 beans 标签内添加声明：

```
xmlns:p="http://www.springframework.org/schema/p"
```

使用 p 名称空间赋值：

```xml
<bean id="person04" class="com.study.bean.Person"
      p:age="18" p:email="lili@qq.com" p:lastName="lili" p:gender="0">
</bean>
```

## 复杂属性赋值

在 Person 类中添加属性：

```java
private Car car;
private List<Book> books;
private Map<String, Object> maps;
private Properties properties;

// 对应的getset方法、Car和Book类此处省略
```

在 xml 文件中添加配置：

```xml
<bean id="car01" class="com.study.bean.Car">
    <property name="carName" value="宝马" />
    <property name="color" value="blue" />
</bean>

<bean id="book01" class="com.study.bean.Book">
    <property name="bookName" value="西游记"/>
</bean>

<bean id="person05" class="com.study.bean.Person">
    <!-- lastName = null -->
    <property name="lastName"><null /></property>
    <!-- ref：引用外面的一个值 -->
    <!--<property name="car" ref="car01" />-->
    <property name="car">
        <!-- 应用内部bean，创建一个新的对象 -->
        <bean class="com.study.bean.Car">
            <property name="carName" value="自行车" />
        </bean>
    </property>

    <!-- 为list类型赋值 -->
    <property name="books">
        <!-- books = new ArrayList<Book>(); -->
        <list>
            <ref bean="book01" />
            <bean id="book02" class="com.study.bean.Book" p:bookName="水浒传" />
        </list>
    </property>

    <!-- Map<String, Object> maps -->
    <property name="maps">
        <!-- maps = new LinkedHashMap<>(); -->
        <map>
            <entry key="name" value="lucy"/>
            <entry key="age" value="12" />
            <entry key="book" value-ref="book01" />
            <entry key="car">
                <bean class="com.study.bean.Car">
                    <property name="carName" value="宝马" />
                </bean>
            </entry>
        </map>
    </property>

    <!-- private Properties properties -->
    <property name="properties">
        <!-- properties = new Properties(); 所有k=v都是String -->
        <props>
            <!-- k=v都是字符串，值直接写在标签体中 -->
            <prop key="username">root</prop>
            <prop key="password">123456</prop>
        </props>
    </property>
</bean>

<!-- 级联属性赋值 -->
<bean id="person06" class="com.study.bean.Person">
    <property name="car" ref="car01" />
    <!-- 原来的bean的属性也会被修改 -->
    <property name="car.price" value="90000" />
</bean>
```

## util 名称空间

在标签中添加声明：

```
xmlns:p="http://www.springframework.org/schema/util"

xsi:schemaLocation="http://www.springframework.org/schema/beans 			http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.0.xsd"
```

xml 文件添加配置：

```xml
<bean id="person07" class="com.study.bean.Person">
    <property name="maps" ref="myMap"/>
</bean>

<!-- 相当于new LinkedHashMap<>() -->
<util:map id="myMap">
    <entry key="name" value="mary" />
</util:map>
```



# 配置的继承和依赖

## bean 配置的继承

```xml
<!-- parent：指定当前bean的配置信息继承于哪个配置 -->
<bean id="person08" class="com.study.bean.Person" parent="person05">
    <property name="lastName" value="sam" />
</bean>

<!-- abstract="true"表示这个bean的配置是抽象的，不能获取实例，只能用来继承 -->
<bean id="person00" class="com.study.bean.Person" abstract="true">
    <property name="lastName" value="sam" />
</bean>
```

## bean 之间的依赖

```xml
<!-- 原来是按照配置顺序创建bean -->
<!-- 改变bean的创建顺序 -->
<bean id="car" class="com.study.bean.Car" depends-on="person,book" />
<bean id="person" class="com.study.bean.Person" />
<bean id="book" class="com.study.bean.Book" />
```

## bean 的作用域

```xml
 <!--
     bean作用域：指定bean是否单实例，默认为单实例
     prototype：多实例
         容器启动默认不会去创建多实例bean
         获取的时候创建这个bean
         每次获取都会创建一个新的对象
     singleton：单实例
         在容器启动完成之前就已经创建好对象，保存在容器中
         任何时候获取都是获取之前创建好的对象
     request：在web环境下，同一次请求创建一个bean实例
     session：在web环境下，同一次会话创建一个bean实例
-->
<bean id="book" class="com.study.bean.Book" scope="prototype"/>
```



# 自定义bean的创建过程

## 工厂模式

工厂模式：一个专门创建对象的类，即工厂，用来创建对象。

- 静态工厂：工厂本身不用创建对象，通过静态方法调用。对象 = 工厂类.工厂方法名();

- 实例工厂：工厂本身需要创建对象。

  工厂类 工厂对象 = new 工厂类(); 工厂对象.setName("name");

新建一个需要工厂创建的对象：

```java
public class Article {
    private String name;
    private String title;
    private String content;
    private String author;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }

    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    @Override
    public String toString() {
        return "Article{" +
                "name='" + name + '\'' +
                ", title='" + title + '\'' +
                ", content='" + content + '\'' +
                ", author='" + author + '\'' +
                '}';
    }
}
```

静态工厂类：

```java
public class ArticleStaticFactory {
    // ArticleStaticFactory.getArticle()
    public static Article getArticle(String name) {
        Article article = new Article();
        article.setName(name);
        article.setAuthor("lucy");
        article.setContent("hello");
        article.setTitle("title");
        return article;
    }
}
```

实例工厂类：

```java
public class ArticleInstanceFactory {
    // new ArticleInstanceFactory().getArticle()
    public Article getArticle(String name) {
        Article article = new Article();
        article.setName(name);
        article.setAuthor("lucy");
        article.setContent("hello");
        article.setTitle("title");
        return article;
    }
}
```

配置文件：

```xml
<!--
    静态工厂（不需要创建工厂本身）
    class：指定静态工厂全类名
    factory-method：指定工厂方法
    constructor-arg：可以为方法传参
-->
<bean id="article01" class="com.study.factory.ArticleStaticFactory"
      factory-method="getArticle">
    <!-- 可以为方法指定参数 -->
    <constructor-arg value="world" />
</bean>

<!-- 实例工厂使用 -->
<bean id="articleInstanceFactory" class="com.study.factory.ArticleInstanceFactory" />
<!--
    factory-bean：指定当前对象使用那个工厂创建
    factory-method：指定这个实例工厂中哪个方法是工厂方法
-->
<bean id="article02" class="com.study.bean.Article"
      factory-bean="articleInstanceFactory" factory-method="getArticle">
    <constructor-arg value="world" />
</bean>
```

也可以使用 Spring 的工厂类：

```java
/**
 * 实现了FactoryBean接口的类是Spring可以认识的工厂类
 * Spring会自动地调用工厂方法创建实例
 */
public class MyFactoryBeanImpl implements FactoryBean<Article> {
    /**
     * 工厂方法
     * @return 创建的对象
     */
    @Override
    public Article getObject() throws Exception {
        Article article = new Article();
        article.setName("factory bean");
        return article;
    }

    /**
     * Spring会自动调用这个方法来确认创建的对象是什么类型
     * @return 创建对象的类型
     */
    @Override
    public Class<?> getObjectType() {
        return Article.class;
    }

    @Override
    public boolean isSingleton() {
        return false;
    }
}
```

配置文件：

```xml
<!-- 实际返回的是工厂类创建的对象，即Article对象 -->
<!-- ioc容器启动的时候不会创建实例，即使设为单例, 只有获取的时候才创建 -->
<bean id="myFactoryBeanImpl" class="com.study.factory.MyFactoryBeanImpl"/>
```


## 生命周期方法

```xml
<!--
   生命周期：bean的创建到销毁
   ioc容器中注册的bean：
       1)单实例bean：容器启动的时候就会创建好，容器关闭也会销毁创建的bean
       2)多实例bean：获取的时候才创建
   可以为bean自定义生命周期方法，spring在创建或销毁的时候就会调用指定方法
   自定义初始化和销毁方法不能有参数
-->
<bean id="book03" class="com.study.bean.Book"
      destroy-method="myDestory" init-method="myInit"/>
```



## 后置处理器

创建处理器类：
```java
public class MyBeanPostProcessor implements BeanPostProcessor {
    /**
     * 初始化方法之前调用
     * @param bean 创建的bean
     * @param beanName bean在xml中配置的id
     * @return 初始化后返回的bean，返回的是什么，容器中保存的就是什么
     * @throws BeansException BeansException
     */
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println(beanName + "将要初始化");
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println(beanName + "初始化完成");
        return bean;
     }
}
```

添加xml配置：
```xml
<!--
    Spring的BeanPostProcessor接口可以在bean的初始化前后调用方法
-->
<bean id="beanPostProcessor" class="com.study.bean.MyBeanPostProcessor" />
```



## 引用外部文件

创建需要引用的properties文件：
```
#dbconfig.properties
jdbc.username=root
jdbc.password=123456
jdbc.jdbcUrl=jdbc:mysql://localhost:3306/test
jdbc.driverClass=com.mysql.jdbc.Driver
```

xml文件声明添加：
```xml
xmlns:context="http://www.springframework.org/schema/context"
```

xml文件添加配置：
```xml
<!-- 加载外部配置文件 -->
<context:property-placeholder location="classpath:dbconfig.properties"/>

<!-- username是Spring的key中的一个关键字，为防止冲突建议加前缀 -->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="username" value="${jdbc.username}" />
    <property name="password" value="${jdbc.password}" />
    <property name="jdbcUrl" value="${jdbc.jdbcUrl}" />
    <property name="driverClass" value="${jdbc.driverClass}" />
</bean>
```



## SpEL表达式

SpEL(Spring Expression Language)：Spring表达式语言

```xml
<bean id="person" class="com.atguigu.bean.Person">
    <!-- 字面量 -->
    <property name="salary" value="#{10020 * 12}" />
    <!-- 引用其他bean的某个属性值 -->
    <property name="lastName" value="#{book.bookName}" />
    <!-- 引用其他bean -->
    <property name="car" value="#{car}" />
    <!-- 调用静态方法：#{T(全类名).静态方法名(参数)} -->
    <property name="email" value="#{T(java.util.UUID).randomUUID().toString().substring(0,5)}" />
    <!-- 调用非静态方法：#{对象.方法名} -->
    <property name="gender" value="#{book.getBookName()}" />
</bean>
```