# Spring5

## Spring框架概述

两大核心：IOC AOP

- IOC:控制反转,把创建对象过程交给spring进行。
- Aop:面向切面,不修改源代码进行功能增强。

Spring特点

1. 方便解耦，简化开发
2. AOP编程支持
3. 方便程序测试
4. 方便和其他框架进行整合
5. 方便进行事务操作
6. 降低API开发难度

- Spring5系统架构

![00](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/01/00.png)

ioc基本包：commons-logging spring-beans spring-context spring-core spring-expression

**入门案例**

```java
 1.加载spring配置文件
        ClassPathXmlApplicationContext context =
                new ClassPathXmlApplicationContext("bean1.xml");
 2.获取配置创建的对象
        User user = context.getBean("user", User.class);

        System.out.println(user);
        user.add();

xml文件：
    <?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--配置User对象相关的创建-->
    <bean id="user" class="com.company.User"></bean>
</beans>

```

## Spring核心控制反转

## IOC 概念和原理

### 什么是IOC

1. 控制反转，把对象创建和对象之间的调用过程，交给Spring进行管理
2. 使用IOC目的：为了降低耦合度

### IOC 底层原理

1. xml解析、工厂模式、反射

- 原始方式与工厂模式对比

![01](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/02/01.png)

### 图解IOC底层原理

![02](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/02/02.png)

## IOC 接口(BeanFactory)

1. **IOC思想基于IOC容器完成，IOC容器底层就是对象工厂**

2. Spring提供IOC容器实现两种方式:(两个接口)

   - BeanFactory:IOC容器基本实现，是Spring内部的使用接口，不提供开发人员进行使用

     > 特点:加载配置文件时候不会创建对象，在获取对象(或者使用对象)才去创建对象

   - ApplicationContext:BeanFactory接口的子接口，提供更多更强大的功能，一般由开发人人员进行调用

     > 特点:加载配置文件时候就会把在配置文件对象进行创建

3. ApplicationContext接口有实现类

![02](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/02/03.png)

- FileSystemXmlApplicationContext

  `configLocation:要写上配置文件在系统盘(某个盘)里的路径`

- ClassPathXmlApplicationContext

  `configLocation:要写上类路径`

# IOC操作Bean管理

## 概念

1. 什么是Bean管理
   - Bean管理指的是两个操作
     1. Spring 创建对象
     2. Spring 注入属性
2. Bean管理操作有两种方式
   1. 基于xml配置文件方式实现
   2. 基于注解方式实现

## IOC操作Bean管理-基于xml方式

1. 基于xml方式创建对象

   

   ```xml
   <!--配置User对象创建-->
   <bean id="user" class="com.frx01.spring5.User"></bean>
   ```

   1. **在Spring配置文件中，使用bean标签，标签里面添加对应属性，就可以实现对象创建**
   2. 在bean标签有很多属性，介绍常用的属性
      - id属性:**唯一标识**
      - class属性:**类全路径(包和类路径)**
   3. **创建对象时候，默认也是执行无参构造方法**

2. 基于xml方式注入属性

   **DI:依赖注入,就是注入属性**

> DI是IOC一种具体实现，表示依赖注入，注入属性是在创建对象的基础之上进行完成

### 第一种注入方式:**使用set方法进行注入**

1. 创建类，定义属性和对应的set方法

```java
 /**
   * 演示set方法注入属性
   */
  public class Book {
  
      //创建属性
      private String bname;
      private String bauthor;
  
      //创建属性对应set方法
      public void setBauthor(String bauthor) {
          this.bauthor = bauthor;
      }
      public void setBname(String bname) {
          this.bname = bname;
      }
  }
```

2. 在Spring配置文件对象创建，配置属性注入

```xml
 <!--set方法注入属性-->
      <bean id="book" class="com.frx01.spring5.testdemo.Book">
          <!--使用property完成属性注入
              name：类里面属性名称
              value：向属性注入的值-->
          <property name="bname" value="易筋经"></property>
          <property name="bauthor" value="达摩老祖"></property>
  
      </bean>
```

### 第二种注入方式:**使用有参数构造方法进行注入**

1. 创建类，定义属性，创建属性对应有参构造方法

```java
/**
 *使用有参数构造注入
 */
public class Orders {

    private String oname;
    private String address;

    public Orders(String oname, String address) {
        this.oname = oname;
        this.address = address;
    }
}
```

2.在Spring配置文件中进行配置

```xml
<!--用有参构造注入属性-->
    <bean id="orders" class="com.frx01.spring5.testdemo.Orders">
        <constructor-arg name="oname" value="computer"></constructor-arg>
        <constructor-arg name="address"value="China"></constructor-arg>
    </bean>
```

### p名称空间注入(了解)

1. 使用p名称空间注入，可以简化基于xml配置方式
   1. 添加p名称空间在配置文件中

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd"></beans>
```

2. 进行属性注入，在bean标签里面进行操作

```xml
 <!--set方法注入属性-->
        <bean id="book" class="com.frx01.spring5.testdemo.Book" p:bname="九阳神功" p:bauthor="无名氏">
        </bean>
```

## xml注入其他类型属性

### 字面量

1. **null值**

```xml
     <!--null值-->
        <property name="address">
            <null/>
        </property>
```

2. **属性值包含特殊符号**

```xml
    <!--属性值包含特殊符号
         1. 把<>进行转义 &lt;&gt;
         2. 把带特殊符号内容写到CDATA
		-->
        <property name="address">
            <value><![CDATA[<<南京>>]]></value>
        </property>
```

### 注入属性-外部bean

1. 创建两个类service类和dao类
2. 在service调用dao里面的方法

```java
package com.frx01.service;

import com.frx01.dao.UserDao;
import com.frx01.dao.UserDaoImpl;

/**
 * @author frx
 * @version 1.0
 * @date 2021/12/2  21:31
 */
public class UserService {

    private UserDao userDao;

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void add(){
        System.out.println("service add.................");
        userDao.update();


    }
}
```

3. 在spring配置文件中进行配置

```xml
    <!--1 配置service和dao对象相关的创建-->
    <bean id="userService" class="com.frx01.spring5.service.UserService">
	<!--注入userDao对象
        name属性值:类里面属性名称
        ref属性:创建userDao对象bean标签id值-->
        <property name="userDao" ref="userDaoImpl"></property>
    </bean>
    <bean id="UserDao" class="com.frx01.spring5.dao.UserDaoImpl"></bean>
	<bean id="userDaoImpl" class="com.frx01.spring5.dao.UserDaoImpl"/></bean>
```

### 注入属性-内部bean

1. 一对多关系，部门和员工

> 一个部门有多个员工，一个员工属于一个部门，部门是一，员工是多。

2. 在实体类之间表示一对多的关系,员工所属的部门，使用对象类型属性进行表示

```java
public class Dept {

    private String dname;

    public void setDname(String dname) {
        this.dname = dname;
    }
}
```

```java
//员工类
public class Emp {
    private String name;
    private String gender;

    //员工属于某一个部门,使用对象表示
    private Dept dept;

    public void setName(String name) {
        this.name = name;
    }

    public void setGender(String gender) {
        this.gender = gender;
    }
}
```

3. 在Spring配置文件

```xml
    <!--内部bean-->
    <bean id="Emp" class="com.frx01.spring5.bean.Emp">
    <!--设置两个普通属性-->
        <property name="name" value="lucy"></property>
        <property name="gender" value="女"></property>
    <!--设置对象类型的属性-->
        <property name="dept">
            <bean id="dept" class="com.frx01.spring5.bean.Dept">
                <property name="dname" value="保安部"></property>
            </bean>

        </property>
    </bean>
```

### 注入属性-级联赋值

1. 第一种写法

```xml
   <!--内部bean-->
    <bean id="Emp" class="com.frx01.spring5.bean.Emp">
    <!--设置两个普通属性-->
        <property name="name" value="lucy"></property>
        <property name="gender" value="女"></property>
        <!--级联赋值-->
        <property name="dept" ref="dept"></property>
    </bean>
    <bean id="dept" class="com.frx01.spring5.bean.Dept">
        <property name="dname" value="财务部"></property>
    </bean>
```

2. 第二种写法

在Emp类中添加属性dept的getDept方法

```xml
   <!--内部bean-->
    <bean id="Emp" class="com.frx01.spring5.bean.Emp">
    <!--设置两个普通属性-->
        <property name="name" value="lucy"></property>
        <property name="gender" value="女"></property>
        <!--级联赋值-->
        <property name="dept" ref="dept"></property>
        <property name="dept.dname" value="技术部"></property>
    </bean>
    <bean id="dept" class="com.frx01.spring5.bean.Dept">
        <property name="dname" value="财务部"></property>
    </bean>
```

### xml方式注入集合属性

1. 注入数组类型属性
2. 注入List集合类型属性
3. 注入Map集合类型属性

第一步，创建类，定义数组，list,map,set类型属性，生成对应set方法

```java
public class Stu {
//    1.数组类型的属性
    private String[] course;

//    2.List集合类型
    private List<String> list;

//    3.Map集合类型
    private Map<String,String> map;

//    4.set集合类型
    private Set<String> set;
    public void setCourse(String[] course) {
        this.course = course;
    }

    public void setList(List<String> list) {
        this.list = list;
    }

    public void setSet(Set<String> set) {
        this.set = set;
    }

    public void setMap(Map<String, String> map) {
        this.map = map;
    }
}
```

第二步，在Spring配置文件进行配置

```xml
<!--完成集合类型属性的注入-->
    <bean id="stu" class="com.frx01.spring5.collectiontype.Stu">
        <!--数组类型的属性注入-->
        <property name="courses">
            <array>
                <value>java课程</value>
                <value>数据库课程</value>
            </array>
        </property>
        <!--list属性注入-->
        <property name="list">
            <list>
                <value>张三</value>
                <value>李四</value>
            </list>
        </property>
        <!--Map类型注入-->
        <property name="map">
            <map>
                <entry key="JAVA" value="java" ></entry>
                <entry key="PHP" value="php"></entry>
            </map>
        </property>
        <!--set集合注入-->
        <property name="set">
            <set>
                <value>MySQL</value>
                <value>redis</value>
            </set>
        </property>
    </bean>
```

- 在集合里面设置对象类型的值

```xml
     <!--注入list集合类型，值是对象-->
        <property name="courseList">
            <list>
                <ref bean="course1"></ref>
                <ref bean="course2"></ref>
            </list>
        </property>
    </bean>
<!--    创建多个course对象-->
    <bean id="course1" class="com.frx01.spring5.collectiontype.Course">
        <property name="cname" value="Spring5框架"></property>
    </bean>
    <bean id="course2" class="com.frx01.spring5.collectiontype.Course">
        <property name="cname" value="Mybatis框架"></property>
    </bean>
```

- 在集合里面设置对象类型的值



```xml
     <!--注入list集合类型，值是对象-->
        <property name="courseList">
            <list>
                <ref bean="course1"></ref>
                <ref bean="course2"></ref>
            </list>
        </property>
    </bean>
<!--    创建多个course对象-->
    <bean id="course1" class="com.frx01.spring5.collectiontype.Course">
        <property name="cname" value="Spring5框架"></property>
    </bean>
    <bean id="course2" class="com.frx01.spring5.collectiontype.Course">
        <property name="cname" value="Mybatis框架"></property>
    </bean>
```

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15

- 把集合注入部分提取出来

1. 在Spring配置文件中引入名称空间util



```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/util/spring-beans.xsd
                            http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd">
        <!--把集合注入部分提取出来-->
</beans>
```

2. 用util标签完成list集合注入提取

```xml
       <!--1.把提取list集合类型属性注入-->
    <util:list id="bookList">
        <value>易筋经</value>
        <value>九阳神功</value>
        <value>九阴真经</value>
    </util:list>

        <!--2.提取list集合类型属性注入使用-->
    <bean id="book" class="com.frx01.spring5.collectiontype.Book">
        <property name="list" ref="bookList"></property>
    </bean>
```

## FactoryBean

1. **Spring有两种类型bean,一种普通bean,另一种工厂bean(FactoryBean)**
2. **普通bean:在配置文件中定义bean类型就是返回类型**
3. **工厂bean:在配置文件中定义bean类型可以和返回类型不一样**

第一步，创建类，让这个类作为工厂bean，实现接口FactoryBean

```java
/**
 * @author frx
 * @version 1.0
 * @date 2021/12/4  17:42
 */
public class MyBean implements FactoryBean<Course> {

    //定义返回bean
    @Override
    public Course getObject() throws Exception {
        Course course = new Course();
        String cou[]={"math,english"};
        course.setCname(cou);
        return course;
    }

    @Override
    public Class<?> getObjectType() {
        return null;
    }

    @Override
    public boolean isSingleton() {
        return false;
    }
}
```

第二步，配置Spring配置文件

```xml
<bean id="myBean" class="com.frx01.spring5.factorybean.MyBean"></bean>
```

## bean 作用域

1. 在Spring里面，可以设置bean实例是单实例还是多实例
2. 在Spring里面，默认情况下，bean是单实例对象

```java
    @Test
    public void testCollection2(){
        ApplicationContext context=new ClassPathXmlApplicationContext("bean2.xml");
        Book book =context.getBean("book", Book.class);
        Book book1 =context.getBean("book", Book.class);
        System.out.println(book);
        System.out.println(book1);

    }
```

![02](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/02/04.png)

1. 如何设置单实例还是多实例

   1. **在Spring配置文件bean标签里面有属性(scope)用于设置单实例还是多实例**
   2. scope属性值

   第一个值默认值，**singleton**，表示单实例对象

   第二个值**prototype**，表示是多实例对象

```xml
    <!--2 提取list集合类型属性注入使用-->
    <bean id="myBean" class="com.frx01.spring5.factorybean.MyBean" scope="prototype"></bean>
</beans>
```

```java
    @Test
    public void testCollection2(){
        ApplicationContext context=new ClassPathXmlApplicationContext("bean2.xml");
        Book book =context.getBean("book", Book.class);
        Book book1 =context.getBean("book", Book.class);
        System.out.println(book);
        System.out.println(book1);

    }
```

![02](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/02/05.png)

- singleton和prototype区别

1. **singleton表示单实例，prototype表示多实例**
2. 设置scope值singleton时候，加载spring配置文件就会创建一个单实例对象
   - **设置scope值是prototype时候，不是在加载spring配置文件时候创建对象，在调用getBean方法时候创建多实例对象**

## bean 生命周期

1. 生命周期
   - **从对象创建到对象销毁的过程**
2. bean生命周期
   1. **通过构造器创建bean实例(无参数构造)**
   2. **为bean的属性设置值和对其他bean引用(调用set方法)**
   3. **调用bean初始化的方法(需要进行配置初始化方法)**
   4. **bean可以使用了(对象获取到了)**
   5. **当容器关闭时候，调用bean的销毁的方法(需要进行配置销毁的方法)**
3. 演示bean生命周期

```java
public class Orders {

//  无参数构造
    public Orders() {
        System.out.println("第一步 执行无参构造创建bean实例");
    }

    private String oname;

    public void setOname(String oname) {
        this.oname = oname;
        System.out.println("第二步 调用set方法设置属性的值");
    }

//    创建执行的初始化方法
    public void initMethod(){
        System.out.println("第三步 执行初始化方法");
    }

    //    创建执行的销毁方法
    public void destroyMethod(){
        System.out.println("第五步 执行销毁方法");
    }

}
```

```java
    @Test
    public void testCollection4() {
//        ApplicationContext context =
//                new ClassPathXmlApplicationContext("bean4.xml");
        ClassPathXmlApplicationContext context =
                new ClassPathXmlApplicationContext("bean4.xml");
        Orders orders = context.getBean("orders", Orders.class);
        System.out.println("第四步 获取创建bean实例对象");
        System.out.println(orders);

//        手动让bean实例销毁
        context.close();

    }
```



```xml
    <bean id="orders" class="com.frx01.spring5.bean.Orders" init-method="initMethod" destroy-method="destroyMethod">
        <property name="oname" value="手机"></property>
    </bean>
```

![02](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/02/06.png)

4. bean的后置处理器，bean生命周期有七步

1. 创建一个类，实现接口BeanPostProcessor，创建后处理器

```java
public class MyBeanPost implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("在初始化之前执行的方法");
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("在初始化之后执行的方法");
        return bean;
    }

}
```

```java
    @Test
    public void testCollection4() {
//        ApplicationContext context =
//                new ClassPathXmlApplicationContext("bean4.xml");
        ClassPathXmlApplicationContext context =
                new ClassPathXmlApplicationContext("bean4.xml");
        Orders orders = context.getBean("orders", Orders.class);
        System.out.println("第四步 获取创建bean实例对象");
        System.out.println(orders);

//        手动让bean实例销毁
        context.close();
```

```xml
<!--    配置后置处理器-->
    <bean id="MyBeanPost" class="com.frx01.spring5.bean.MyBeanPost"></bean>
</beans>
```

![02](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/02/07.png)

## xml 自动装配

> 要求类中有相应的这个属性，才能进行自动装配
>
> 例如：public class TextEditor {
>     private SpellChecker spellChecker;  有这个属性
>     private String name;
>
>     public void spellCheck() {
>         spellChecker.checkSpelling();
>     }
> }
>

1. 什么是自动装配
   - **根据指定装配规则(属性名称或者属性类型)，Spring自动将匹配的属性值进行注入**
2. 演示自动装配过程

- 根据属性名称自动注入

```xml
<!--    实现自动装配
        bean标签属性autowire,配置自动装配
        autowire属性常用两个值:
            byName根据属性名称注入,注入值bean的id值和类属性名称一样
            byType根据属性类型注入
       -->
    <bean id="emp" class="com.frx01.spring5.autowire.Emp" autowire="byName">
<!--        <property name="dept" ref="dept"></property>-->
    </bean>
    <bean id="dept" class="com.frx01.spring5.autowire.Dept"></bean>
```

- 根据属性类型自动注入

```xml
    <bean id="emp" class="com.frx01.spring5.autowire.Emp" autowire="byType">
<!--        <property name="dept" ref="dept"></property>-->
    </bean>
    <bean id="dept" class="com.frx01.spring5.autowire.Dept"></bean>
```

## 引入外部属性文件

1. 直接配置数据库信息
   1. 配置德鲁伊连接池
   2. 引入德鲁伊连接池依赖jar包

```xml
<!--直接配置连接池-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
    <property name="url" value="jdbc:mysql://localhost:3306/userDb"></property>
    <property name="username" value="root"></property>
    <property name="password" value="root"></property>
    </bean>
```

1. 引入外部属性文件配置数据库连接池

- 创建外部属性文件，properteis格式文件，写数据库信息

![02](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/02/08.png)

- 把外部properties属性文件引入到spring配置文件中

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:util="http://www.springframework.org/schema/util"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                            http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd
                            http://www.springframework.org/schema/context  http://www.springframework.org/schema/util/spring-util.context.xsd">
```

- 在Spring配置文件使用标签引入外部属性文件

```xml
<!--    引入外部属性文件-->
    <context:property-placeholder location="classpath*:jdbc.properties"/>
    <!--配置连接池-->
        <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${prop.driverClass}"></property>
        <property name="url" value="${prop.url}"></property>
        <property name="username" value="${prop.userName}"></property>
        <property name="password" value="${prop.passwd}"></property>
        </bean>
```

## IOC操作Bean管理-基于注解方式

1. 什么是注解
   1. 注解是代码特殊标记，格式:@注解名称(属性名称=属性值，属性名称=属性值)
   2. 使用注解，注解作用在类上面，方法上面，属性上面
   3. 使用注解目的:简化xml配置
2. Spring针对Bean管理中创建对象提供注解
   1. @Component
   2. @Service
   3. @Controller
   4. @repository

- 上面四个注解功能是一样的，都可以用来创建bean实例

### 基于注解方式实现对象创建

第一步 引入依赖

第二步 开启组件扫描

```xml
    <!--开启组件扫描
        1 如果扫描多个包 使用逗号隔开
        2 扫描包上层目录-->
    <context:component-scan base-package="com.frx01.spring5"></context:component-scan>
```

第三步创建类，在类上面添加创建对象注解

```java
//在注解里面value属性值可以省略不写
//默认值是类名称，首字母小写
//  UserService -- userService
@Component(value = "userService")  //<bean id="userService" class=".."/>
public class UserService {

    public void add(){
        System.out.println("service add......");
    }
}
```

- 开启组件扫描细节配置

```xml
    <!--示例1
        use-default-filters="false 表示现在不使用默认filter,不扫描全部，自己配置filter
        context:include-filler,设置扫描那些内容-->
    <context:component-scan base-package="com.frx01" use-default-filters="false">
        <context:include-filter type="annotation"
                                expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
    <!--示例2
        下面配置扫描包所有内容
        context:exclude-filter：设置那些内容不进行扫描-->
    <context:component-scan base-package="com.frx01">
        <context:exclude-filter type="annotation"
                                expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
```

### 基于注解方式实现属性注入

1. **@AutoWired：根据属性类型自动装配**

第一步把service和dao对象创建，在service和dao类添加创建对象注解

第二步在service注入dao对象，在service类添加dao类型属性，在属性上面使用注解

```java
@Service
public class UserService {

//    定义dao类型的属性
//    不需要添加set方法
//    添加注入属性注解
    @Autowired
    private UserDao userDao;


    public void add(){
        System.out.println("service add......");
        userDao.add();
    }
}
```

1. **@Qualifier：根据属性名称注入**

这个@Qualifier注解的使用，和上面@Autowired一起使用

> 因为Spring的依赖注入是基于类型的，但也会考虑bean的名字。在Spring的上下文中，**接口的实现类**被当作一个bean来管理，并且这个bean的类型是接口的实现类。但是，由于Java的多态性，你可以将这个实现类的实例赋值给一个接口类型的变量。

```java
@Repository(value = "userDaoImpl1")
public class UserDaoImpl implements UserDao{

    @Override
    public void add() {
        System.out.println("dao add......");

    }
}
```

```java
public class UserService {

//    定义dao类型的属性
//    不需要添加set方法
//    添加注入属性注解
    @Autowired  //根据类型进行注入
    @Qualifier(value = "userDaoImpl1")
    private UserDao userDao;


    public void add(){
        System.out.println("service add......");
        userDao.add();
    }
}
```

3. @Resource：可以根据类型注入，可以根据名称注入

```java
//    @Resource  //根据类型进行注入
    @Resource(name="userDaoImpl1") //根据名称进行注入
    private UserDao userDao;
```

4. @Value：注入普通类型属性

```java
    @Value(value = "abc")
    private String name;
```

### 完全注解开发

1. 创建配置类，替代xml配置文件

```java
@Configuration //作为配置类，替代xml配置文件
@ComponentScan(basePackages = "com.frx01")
public class SpringConfig {
}
```

2. 编写测试类

```java
   	@Test
    public void testService2(){
//        加载配置类
        ApplicationContext context =
                new AnnotationConfigApplicationContext(SpringConfig.class);
        UserService userService = context.getBean("userService", UserService.class);
        System.out.println(userService);
        userService.add();
    }
```

# Spring核心之面向切面编程AOP

## 什么是AOP

1. 向切面编程（方面），利用 AOP 可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。
2. 通俗描述:不通过修改源代码方式，在主干功能里面添加新功能
3. 使用登陆例子说明AOP

![01](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/03/01.png)

## AOP底层原理

### AOP底层使用动态代理

1. **有接口的情况,使用JDK动态代理**

- 创建接口实现类代理对象，增强类的方法

1. **没有接口情况，使用CGLIB动态代理**

- 创建子类的动态对象，增强类的方法

![01](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/03/02.png)

## AOP(JDK动态代理)

1. 使用JDK动态代理，使用Proxy类里面的方法创建代理对象

> java.lang.reflect
>
> **Class Proxy**
>
> java.lang.Object
>
> > java.lang.reflect.Proxy

- 调用newProxyInstance方法

![01](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/03/04.png)

方法有三个参数:

第一参数，类加载器

第二参数，增强方法所在的类，这个类实现的接口，支持多个接口

第三参数，实现这个接口InvocationHandler,创建代理对象，写增强的部分

1. 编写JDK动态代理代码

   创建接口，定义方法

```java
public interface UserDao {
    public int add(int a,int b);

    public String update(String id);
}
```

 创建接口实现类，实现方法

```java
public class UserDaoImpl implements UserDao{
    @Override
    public int add(int a, int b) {
        return a+b;
    }

    @Override
    public String update(String id) {
        return id;
    }
}
```

 使用Proxy类创建接口代理对象

```java
package com.frx01.spring5;

import java.lang.reflect.Array;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.util.Arrays;

/**
 * @author frx
 * @version 1.0
 * @date 2021/12/29  16:06
 */
public class JDKProxy {
    public static void main(String[] args) {
        //创建接口实现类代理对象
        Class[] interfaces={UserDao.class};
//        Proxy.newProxyInstance(JDKProxy.class.getClassLoader(), interfaces, new InvocationHandler() {
//            @Override
//            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
//                return null;
//            }
//        });
        UserDaoImpl userDao=new UserDaoImpl();
        UserDao dao=(UserDao) Proxy.newProxyInstance(JDKProxy.class.getClassLoader(), interfaces, new UserDaoProxy(userDao));
        int result = dao.add(1, 2);
        System.out.println("result"+result);



    }
}

//创建代理对象代码
class UserDaoProxy implements InvocationHandler{

    //1 把创建的是谁的代理对象 把谁传递过来、
    //有参数构造器
    private Object obj;
    public UserDaoProxy(Object obj){
        this.obj=obj;


    }

    //增强的逻辑
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        //方法之前
        System.out.println("方法之前执行......"+method.getName()+" :传递的参数..."+ Arrays.toString(args));

        //被增强的方法执行
        Object res= method.invoke(obj,args);

        //方法之后
        System.out.println("方法之后执行......"+obj);
        return res;
    }
}
```

## AOP(术语)

- **连接点（Jointpoint）**：表示需要在程序中插入横切关注点的扩展点，**连接点可能是类初始化、方法执行、方法调用、字段调用或处理异常等等**，Spring只支持方法执行连接点，在AOP中表示为**在哪里干**；
- **切入点（Pointcut）**： 选择一组相关连接点的模式，即可以认为连接点的集合，Spring支持perl5正则表达式和AspectJ切入点模式，Spring默认使用AspectJ语法，在AOP中表示为**在哪里干的集合**；
- **通知（Advice）**：在连接点上执行的行为，通知提供了在AOP中需要在切入点所选择的连接点处进行扩展现有行为的手段；包括前置通知（before advice）、后置通知(after advice)、环绕通知（around advice），在Spring中通过代理模式实现AOP，并通过拦截器模式以环绕连接点的拦截器链织入通知；在AOP中表示为**干什么**；
- **方面/切面（Aspect）**：横切关注点的模块化，比如上边提到的日志组件。可以认为是通知、引入和切入点的组合；在Spring中可以使用Schema和@AspectJ方式进行组织实现；在AOP中表示为**在哪干和干什么集合**；

![01](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/03/03.png)

## AOP操作(准备工作)

1. Spring框架一般基于AspectJ实现AOP操作

   - 什么是AspectJ

   AspectJ不是Spring组成部分，独立AOP框架，一般把AspectJ和Spring框架一起使用，进行AOP操作

2. 基于AspectJ实现AOP操作

   1. **基于xml配置文件实现**
   2. **基于注解方式实现(使用)**

3. 在项目工程里面引入AOP相关依赖

![01](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/03/05.png)

1. 切入点表达式

   1. 切入点表达式的作用，知道对哪个类型里面的哪个方法进行增强
   2. 语法结构:

   **execution([权限修饰符] [返回类型] [类全路径] [方法名称] [参数列表])**

举例1:对com.frx01.spring5.dao.BookDao类里面的add进行增强

execution(* com.frx01.spring5.dao.BookDao.add(..))

举例2:对com.frx01.spring5.dao.BookDao类里面的所有方法进行增强

execution(* com.frx01.spring5.dao.BookDao.*(..))

举例2:对com.frx01.spring5.dao包里面所有类,类里面的所有方法进行增强

execution(* com.frx01.spring5.dao*.*(..))

## AOP操作(AspectJ)注解

1. 创建类，在类里面定义方法

```java
public class User {
    public void add(){
        System.out.println("add..............");
    }
}
```

2. 建增强类(编写增强逻辑)

- 在增强类里面，创建方法，让不同方法代表不同通知类型

```java
public class UserProxy {

    //前置通知
    public void before(){
        System.out.println("before.......");
    }
}
```

3. 进行通知的配置

1. 在Spring配置文件中，开启注解扫描

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xmlns:aop="http://www.springframework.org/schema/aop"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                               http://www.springframework.org/schema/context http://www.springframework.org/schema/beans/spring-context.xsd
                               http://www.springframework.org/schema/aop http://www.springframework.org/schema/beans/spring-aop.xsd">
   <!--    开启全盘扫描-->
   <context:component-scan base-package="com.frx01.spring5.aopanno"></context:component-scan>
   </beans>
   ```

2. 使用注解创建User和UserProxy对象

   ```java
   @Component
   public class User {
   }
   @Component
   //被增强的类
   public class UserProxy {
   }
   ```

3. 在增强类上面添加注解@Aspect

   ```java
   @Component
   @Aspect //生成代理对象
   //被增强的类
   public class UserProxy {
   }
   ```

4. 在spring配置文件中开启生成代理对象

```xml
<!--    开启Aspect生成代理对象-->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```

4. 配置不同类型的通知

1. 在增强类里面，在作为通知方法上面添加通知类型注解，使用切入点表达式配置

   ```java
   package com.frx01.spring5.aopanno;
   
   import org.aspectj.lang.ProceedingJoinPoint;
   import org.aspectj.lang.annotation.*;
   import org.springframework.stereotype.Component;
   
   /**
    * @author frx
    * @version 1.0
    * @date 2021/12/29  19:54
    */
   @Component
   @Aspect //生成代理对象
   //被增强的类
   public class UserProxy {
   
       //前置通知
       //@Before注解表示作为前置通知
       @Before(value = "execution(* com.frx01.spring5.aopanno.User.add(..))")
       public void before(){
           System.out.println("before.......");
       }
       //后置通知(返回通知)
       @AfterReturning(value = "execution(* com.frx01.spring5.aopanno.User.add(..))")
       public void afterReturning(){
           System.out.println("afterReturning.....");
       }
   
   
       //最终通知
       @After(value = "execution(* com.frx01.spring5.aopanno.User.add(..))")
       public void after(){
           System.out.println("afterg.....");
       }
   
       //异常通知
       @AfterThrowing(value = "execution(* com.frx01.spring5.aopanno.User.add(..))")
       public void afterThrowing(){
           System.out.println("afterThrowing.....");
       }
   
       //环绕通知
       @Around(value = "execution(* com.frx01.spring5.aopanno.User.add(..))")
       public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable{
           System.out.println("环绕之前.......");
           //被增强的方法执行
   
           proceedingJoinPoint.proceed();
           System.out.println("环绕之后.......");
       }
   }
   ```

- 测试

```java
public class TestAop {
    @Test
    public void testAopAnno(){
        ApplicationContext context =
                new ClassPathXmlApplicationContext("bean1.xml");

        User user = context.getBean("user", User.class);
        user.add();

    }
}
```

![01](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/03/06.png)

5. 相同切入点抽取



```java
    //相同切入点抽取
    @Pointcut(value ="execution(* com.frx01.spring5.aopanno.User.add(..))")
    public void pointdemo(){

    }

    //前置通知
    //@Before注解表示作为前置通知
    @Before(value = "pointdemo()")
    public void before(){
        System.out.println("before.......");
    }
```

6. 有多个增强类多同一个方法进行增强，设置增强类优先级

1. 在增强类上面添加注解@Order(数字类型值),数字类型值越小优先级越高

   ```java
   @Component
   @Aspect
   @Order(1)
   public class PersonProxy {
   
       //后置通知(返回通知)
       @Before(value = "execution(* com.frx01.spring5.aopanno.User.add(..))")
       public void afterThrowing(){
           System.out.println("PersonBefore.....");
       }
   
   }
   ```

7. 完全使用注解开发

1. 创建配置类，不需要创建xml配置文件

   ```java
   @Configuration
   @ComponentScan(basePackages = {"com.frx01"})
   @EnableAspectJAutoProxy(proxyTargetClass = true)
   public class ConfigAop {
   }
   ```

## AOP操作(AspectJ配置文件)

1. 创建两个类，增强类和被增强类，创建方法

2. 在Spring配置文件中创建两个类对象

   ```xml
       <!--创建对象-->
       <bean id="book" class="com.frx01.spring5.aopxml.Book"></bean>
       <bean id="bookProxy" class="com.frx01.spring5.aopxml.BookProxy"></bean>
   ```

3. 在Spring配置文件中配置切入点

```xml
    <!--配置aop增强-->
    <aop:config>
        <!--切入点-->
        <aop:pointcut id="p" expression="execution(* com.frx01.spring5.aopxml.Book.buy(..))"/>
        <!--配置切面-->
        <aop:aspect ref="bookProxy">
        <!--配置增强作用在哪个方法上-->
            <aop:before method="before" pointcut-ref="p"/>
        </aop:aspect>
    </aop:config>
</beans>
```

# Jdbc Template

## Jdbc Template(概念和准备)

1. 什么是JdbcTemplate
   1. **Spring框架对JDBC进行封装，使用JdbcTemplate方便实现对数据库操作**
2. 准备工作

- 创建数据库和表

```sql
CREATE DATABASE user_db
CREATE TABLE t_book(
	userId BIGINT PRIMARY KEY,
	username VARCHAR(100) NOT NULL,
	ustatus VARCHAR(50) NOT NULL)
```

- 引入相关的jar包

![01](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/04/01.png)

- 在Spring文件中配置数据库的连接池

```xml
    <!-- 数据库连接池 -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource"
          destroy-method="close">
        <property name="url" value="jdbc:mysql:///user_db" />
        <property name="username" value="root" />
        <property name="password" value="hsp" />
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />
    </bean>
```

- 配置JdbcTemplate对象，注入DataSource

```xml
 <!--创建jdbcTemplate对象-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <!--注入DataSource-->
        <property name="dataSource" ref="dataSource"></property>
    </bean>
```

- 创建service类，创建dao,在dao注入jdbcTemplate对象

第一部分

```xml
    <!--开启组件扫描-->
    <context:component-scan base-package="com.frx01"></context:component-scan>
```

第二部分

- Service

```java
@Service
public class BookService {

    //注入dao
    @Autowired
    private BookDao bookDao;
}
```

- Dao

```java
@Repository
public class BookDaoImpl implements BookDao{

    //注入JdbcTemplate
    @Autowired
    private JdbcTemplate jdbcTemplate;
}
```

## Jdbc Template操作数据库(添加)

1. 对应数据库创建实体类

   ```java
   public class User {
       private String userId;
       private String username;
       private String ustatus;
   
       public String getUserId() {
           return userId;
       }
   
       public String getUsername() {
           return username;
       }
   
       public String getUstatus() {
           return ustatus;
       }
   
       public void setUserId(String userId) {
           this.userId = userId;
       }
   
       public void setUsername(String username) {
           this.username = username;
       }
   
       public void setUstatus(String ustatus) {
           this.ustatus = ustatus;
       }
   }
   ```

2. 编写service和dao

   1. 在dao进行数据库添加操作

   2. 调用JdbcTemplate对象里面update方法实现添加操作

      ```java
      update(String sql,Object... args)
      ```

   - 有两个参数

     - 第一个参数:sql语句

     - 第二个参数:可变参数，设置sql语句值

       ```java
       @Repository
       public class BookDaoImpl implements BookDao{
       
       
           //注入JdbcTemplate
           @Autowired
           private JdbcTemplate jdbcTemplate;
       
           //添加方法
           @Override
           public void add(Book book) {
       
               //1.创建sql语句
               String sql="insert into t_book values(?,?,?)";
               //2.调用方法实现
               Object[] args={ book.getUserId(), book.getUsername(), book.getUstatus()};
               int update = jdbcTemplate.update(sql,args);
               System.out.println(update);
       
           }
       
       
       }
       ```

   - 测试

   ```java
   public class TestBook {
       @Test
       public void testJdbcTemplate(){
   
           ApplicationContext context =
                   new ClassPathXmlApplicationContext("bean1.xml");
   
           BookService bookService = context.getBean("bookService", BookService.class);
           Book book = new Book();
           book.setUserId("1");
           book.setUsername("java");
           book.setUstatus("A");
           bookService.addBook(book);
   
   
       }
   ```

   ![01](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/04/02.png)

   ![01](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/04/03.png)

## Jdbc Template数据库操作数据库(修改和删除)

- 在BookDaoImpl编写修改和删除操作

  ```java
    //修改
      @Override
      public void update(Book book) {
          String sql="update t_book set username=?,ustate=? where userId=?";
          Object[] args={ book.getUsername(), book.getUstatus(), book.getUserId()};
          int update = jdbcTemplate.update(sql,args);
          System.out.println(update>0?"修改成功":"修改失败");
  
      }
  
      //删除
      @Override
      public void delete(String id) {
          String sql="delete from t_book where userId=?)";
          int update = jdbcTemplate.update(sql, id);
          System.out.println(update>0?"删除成功":"删除失败");
  
      }
  ```

- 测试类测试

  ```java
    //修改
  //        Book book = new Book();
  //        book.setUserId("1");
  //        book.setUsername("javaup");
  //        book.setUstatus("AAA");
  //        bookService.updateBook(book);
  
          //删除
          Book book = new Book();
          book.setUserId("1");
          bookService.deleteBook("1");
  ```

  ![01](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/04/04.png)

  ![01](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/04/05.png)

- 删除结果

![01](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/04/06.png)

![01](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/04/07.png)

## Jdbc Template数据库操作数据库(查询返回某个值)

1. 查询表里面有多少条记录，返回是某个值

2. 使用JdbcTemplate对面里面queryForOcject方法实现查询操作

   ```java
   queryForObject(String sql,Class<T> requiredType)
   ```

   - 有两个参数

     - 第一个参数:sql语句
     - 第二个参数:返回类型Class

     ```java
       //查询表记录数
         @Override
         public int selectCount(){
             String sql="select count(*) from t_book";
             Integer count = jdbcTemplate.queryForObject(sql, Integer.class);
             return count;
     
         }
     ```

   - 测试

     ![01](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/04/08.png)

## Jdbc Template数据库操作数据库(查询返回某个对象)

1. 场景:查询图书详情

2. JdbcTemplate实现查询返回对象

   ```java
   queryForObject(String sql,RowMapper<T> rowMapper,Object... args)
   ```

   - 有三个参数

     - 第一个参数:sql语句
     - 第二个参数:RowMapper,是接口，返回不同类型的数据，使用这个接口里面实现类完成数据封装
     - 第三个参数:sql语句值

     ```java
      //查询返回对象
         @Override
         public Book findBookInfo(String id) {
             String sql="select * from t_book where userId=?";
             //调用方法
             Book book = jdbcTemplate.queryForObject(sql, new BeanPropertyRowMapper<Book>(Book.class),id);
             return book;
         }
     ```

   - 测试

     ![01](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/04/09.png)


## 调用JdbcTemplate方法实现查询返回集合

- 场景:查询图书列表分页

```java
query(String sql,RowMapper<T> rowMapper,Object... args)
```

- 有三个参数

  - 第一个参数:sql语句
  - 第二个参数:RowMapper是接口，针对返回不同类型数据，使用这个接口里面实现类完成数据封装
  - 第三个参数:sql语句

  ```java
      //查询返回集合
      @Override
      public List<Book> findAllBook() {
          String sql="select * from t_book";
          //调用方法
          List<Book> bookList = jdbcTemplate.query(sql, new BeanPropertyRowMapper<Book>(Book.class));
  
          return  bookList;
  
      }
  ```

- 测试

  ![01](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/04/10.png)

## Jdbc Template数据库操作数据库(批量操作)

1. 批量操作:操作表里面多条记录

2. JdbcTemplate实现批量添加操作

   ```java
   batchUpdate(String sql,List<Object[]> batchArgs)
   ```

   - 有两个参数

     - 第一个参数:sql语句
     - 第二个参数:List集合，添加多条记录数据

     ```java
       //批量添加
         @Override
         public void batchAddBook(List<Object[]> batchArgs) {
             String sql="insert into t_book values(?,?,?)";
             int[] ints=jdbcTemplate.batchUpdate(sql,batchArgs);
             System.out.println(Arrays.toString(ints));
         }
     ```

   - 测试

     ```java
      //批量添加
             List<Object[]> batchArgs=new ArrayList<>();
             Object[] o1={"3","java","a"};
             Object[] o2={"4","c++","b"};
             Object[] o3={"5","MySQL","e"};
             batchArgs.add(o1);
             batchArgs.add(o2);
             batchArgs.add(o3);
             //调用批量添加
             bookService.batchAdd(batchArgs);
     ```

   ![01](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/04/11.png)

3. JdbcTemplate实现批量修改操作

   ```java
    //批量修改
       @Override
       public void batchUpdateBook(List<Object[]> batchArgs) {
           String sql="update t_book set username=?,ustatus=? where userId=?";
           int[] ints = jdbcTemplate.batchUpdate(sql, batchArgs);
           System.out.println(Arrays.toString(ints));
   
       }
   ```

- 测试

  ```java
    //批量修改
          List<Object[]> batchArgs=new ArrayList<>();
          Object[] obj1={"java111","a3","3"};
          Object[] obj2={"c++1010","b4","4"};
          Object[] obj3={"MySQL11","c5","5"};
          batchArgs.add(obj1);
          batchArgs.add(obj2);
          batchArgs.add(obj3);
          //调用方法
          bookService.batchUpdate(batchArgs);
  
      }
  ```

  ![01](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/04/12.png)

1. JdbcTemplate实现批量删除操作

   ```java
    //批量删除
       @Override
       public void batchDeleteBook(List<Object[]> batchArgs) {
           String sql="delete from t_book where userId=?";
           int[] ints = jdbcTemplate.batchUpdate(sql, batchArgs);
           System.out.println(Arrays.toString(ints));
       }
   ```

- 测试

  ```java
    //批量修改
          List<Object[]> batchArgs=new ArrayList<>();
          Object[] obj1={"3"};
          Object[] obj2={"4"};
          batchArgs.add(obj1);
          batchArgs.add(obj2);
          //调用方法实现批量删除
          bookService.batchDelete(batchArgs);
  ```

  ![01](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/04/13.png)

# Spring5 事务	

## 事务概念

1. 什么是事务

   1. 事务是数据库操作最基本的单元，逻辑上一组操作，要么都成功，如果有一个失败所有操作都失败
   2. 典型场景:银行转账

   - lucy转账100元给mary
   - lucy少100,mary多100

2. 事务四个特性(ACID)

   1. 原子性
   2. 一致性
   3. 隔离性
   4. 持久性

   问题：银行转账问题！

1. 上面问题如何解决呢

   - 使用事务进行解决

2. 事务操作

   ```java
       //转账的方法
       public  void  accountMoney(){
           try {
   
               //第一步 开启事务
   
               //第二步进行业务操作
               //lucy少100
               userDao.reduceMoney();
   
               //模拟异常
               int i = 10 / 0;
   
               //mary多100
               userDao.addMoney();
   
               //第三步 没有发生异常，提交事务
           }catch (Exception e){
   
               //第四步 出现异常，事务回滚
   
           }
   ```

## Spring 事务管理介绍

1. **事务添加到JavaEE三层结构里面Service层(业务逻辑层)**

2. 在Spring进行事务管理操作

   1. 有两种方式:编程式事务管理和声明式事务管理(使用)

3. 声明式事务管理

   1. 基于注解方式(使用)
   2. 基于xml配置文件方式

4. 在Spring进行声明式事务管理，底层使用AOP

5. Spring事务管理API

   1. 提供一个接口，代表事务管理器，这个接口针对不同的框架提供不同的实现类

   ![01](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/05/02.png)

## 注解声明式事务管理

1. 在Spring:配置文件配置事务管理器

   ```xml
    <!--创建事务管理器-->
       <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
       <!--注入数据源-->
           <property name="dataSource" ref="dataSource"></property>
       </bean>
   ```

2. 在Spring配置文件，开启事务注解

   1. 在Spring配置文件引入名称空间tx

      ```xml
      <beans xmlns="http://www.springframework.org/schema/beans"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xmlns:context="http://www.springframework.org/schema/context"
             xmlns:aop="http://www.springframework.org/schema/aop"
             xmlns:tx="http://www.springframework.org/schema/tx"
             xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                                  http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                                  http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
                                  http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">
      ```

   2. 开启事务注解

      ```xml
          <!--开启事务注解-->
          <tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>
      ```

   3. 在service类上面(获取service类里面方法上面)添加事务注解

      1. @Transactional,这个注解添加到类上面，也可以添加到方法上面
      2. 如果把这个注解添加到类上面，这个类里面的所有的方法都添加事务
      3. 如果把这个注解添加到方法上面，为这个方法添加事务

   ```java
   @Service
   @Transactional
   public class UserService {
   }
   ```

## 注解声明式事务管理参数配置

1. 在service类上面添加注解@Transactiona，在这个注解里面可以配置事务相关参数

   ![01](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/05/03.png)

### propagation：事务传播行为

1. 多事务方法直接进行调用，这个过程中事务是如何进行管理的

   ![01](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/05/04.png)

事务的传播行为可以由传播属性指定。Spring定义了7种类传播行为

| 传播属性     | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| REQUIRED     | 如果有事务在运行，当前的方法就在这个事务内运行，否则，就启动一个新的事物，并在自己的事务内运行 |
| REQUIRED_NEW | 当前的方法必须启动新事物，并在它自己的事务内运行，如果有事务正在运行，应该将它挂起 |
| SUPPORTS     | 如果有事务在运行，当前的方法就在这个事务内运行，否则它可以不运行在事务中 |
| NOT_SUPPROTS | 当前的方法不应该运行在事务中，如果有运行的事务，将它挂起     |
| MANDATORY    | 当前的方法必须运行在事物内部，如果没有正在运行的事务，就抛出异常 |
| NEVER        | 当前的方法不应该运行在事务中，如果有运行的事务，就抛出异常   |
| NESTED       | 如果有事务在运行，当前的方法就应该在这个事务的嵌套事务内运行。否则，就启动一个新的事务，并在它自己的事务内运行 |

```java
@Service
@Transactional(propagation = Propagation.REQUIRED)
public class UserService {
}
```

### isolation：事务隔离级别

1. 事务有特性称为隔离性，多事务操作之间不会产生影响。不考虑隔离性产生很多问题

2. 有三个读的问题:脏读、不可重复读、幻读

   1. 脏读：一个未提交事务读取到另一个 未提交事务的数据

      ![01](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/05/05.png)

   2. 不可重复读

      ![01](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/Spring5/images/05/06.png)

   3. 幻读:一个未提交事务读取到另一个提交事务添加数据

3. 通过设置事务隔离性，解决读问题

   |                            | 脏读 | 不可重复度 | 幻读 |
   | :------------------------: | :--: | :--------: | :--: |
   | READ UNCOMMITTED(读未提交) |  有  |     有     |  有  |
   |  READ COMMITTED(读已提交)  |  无  |     有     |  有  |
   | REPEATABLE READ(可重复读)  |  无  |     无     |  有  |
   |    SERIALIZABLE(串行化)    |  无  |     无     |  无  |

   ```java
   @Service
   @Transactional(propagation = Propagation.REQUIRED,isolation = Isolation.REPEATABLE_READ)
   public class UserService {
   }
   ```

### timeout：超时时间

1. 事务需要在一定时间内提交，如果不提交进行回滚
2. 默认值是-1,设置时间以秒为单位进行计算

```java
@Service
@Transactional(timeout = -1,propagation = Propagation.REQUIRED,isolation = Isolation.REPEATABLE_READ)
public class UserService {
```

### readOnly：是否只读

1. 读:查询操作，写:添加修改删除操作
2. readOnly默认值false,表示可以查询，可以添加修改删除操作
3. 设置readOnly值是true,设置成true之后，只能查询

```java
@Service
@Transactional(readOnly = true,timeout = -1,propagation = Propagation.REQUIRED,isolation = Isolation.REPEATABLE_READ)
public class UserService {
}
```

### rolibackFor：回滚

- 设置出现哪些异常进行事务回滚

### noRollbackFor：不回滚

- 设置出现哪些异常不进行事务回滚

## XML声明式事务管理(了解)

1. 在Spring配置文件中进行配置

   1. 配置事务管理器
   2. 配置通知
   3. 配置切入点和切面

   ```xml
     <!--1 创建事务管理器-->
       <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
       <!--注入数据源-->
           <property name="dataSource" ref="dataSource"></property>
       </bean>
   
       <!--2 配置通知-->
       <tx:advice id="txadvice">
       <!--配置事务参数-->
           <tx:attributes>
               <!--指定哪种规则的方法上面添加事务-->
               <tx:method name="accountMoney" propagation="REQUIRED"/>
               <!--<tx:method name="account*"/>-->
           </tx:attributes>
       </tx:advice>
       <!--3 配置切入点和切面-->
       <aop:config>
           <!--配置切入点-->
           <aop:pointcut id="pt" expression="execution(* com.frx01.spring5.service.UserService.*(..))"/>
           <!--配置切面-->
           <aop:advisor advice-ref="txadvice" pointcut-ref="pt"></aop:advisor>
       </aop:config>
   ```

## 完全注解开发

1. 创建配置类，使用配置类代替xml配置文件

   ```java
   /**
    * @author frx
    * @version 1.0
    * @date 2022/1/4  18:50
    */
   @Configuration//配置类
   @ComponentScan(basePackages = "com.frx01")//组件扫描
   @EnableTransactionManagement//开启事务
   public class TxConfig {
   
       //创建数据库连接池
       @Bean
       public DruidDataSource getDruidDataSource(){
   
           DruidDataSource dataSource = new DruidDataSource();
           dataSource.setDriverClassName("com.mysql.jdbc.Driver");
           dataSource.setUrl("jdbc:mysql:///user_db");
           dataSource.setUsername("root");
           dataSource.setPassword("hsp");
           return dataSource;
       }
       //Jdbc模板对象
       @Bean
       public JdbcTemplate getJdbcTemplate(DataSource dataSource){
           //到ioc容器中根据类型找到dataSource
           JdbcTemplate jdbcTemplate = new JdbcTemplate();
           //注入dataSource
           jdbcTemplate.setDataSource(dataSource);
           return  jdbcTemplate;
       }
       //创建事务事务管理器
       @Bean
       public DataSourceTransactionManager getDataSourceTransactionManager(DataSource dataSource){
           DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();
           transactionManager.setDataSource(dataSource);
           return transactionManager;
   
       }
   }
   ```

   - 测试

     ```java
         @Test
         public void  testAccount3(){
             ApplicationContext context =
                     new AnnotationConfigApplicationContext(TxConfig.class);
     
             UserService userService = context.getBean("userService", UserService.class);
             userService.accountMoney();
     
         }
     ```

# Spring5 新功能



# Spring5 Webflux


