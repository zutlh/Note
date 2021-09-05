​	

# 1、Spring

## 1、1简介

## 1、2优点

Spring是一个开源的免费框架

Spring是一个轻量级的，非入侵式的框架

控制反转(IOC)、面向切面编程(AOP)

支持事务的处理，对框架集合的支持

## 1、3组成

![654A94D9-9DB4-4B2C-A4D0-18BAA69677A6](/Users/apple/Library/Containers/com.tencent.qq/Data/Library/Application Support/QQ/Users/781936292/QQ/Temp.db/654A94D9-9DB4-4B2C-A4D0-18BAA69677A6.png)

## 1、4拓展

Spring Boot：

一个快速开发的脚手架

基于SpringBoot可以快速的开发单个微服务

约定大于配置

Spring Cloud

Spring Cloud是基于SpringBoot实现的

# 2、IOC理论推导

1、UserDao接口

```java
package com.liu.dao;

public interface UserDao {
    public void getUser();
}

```



2、UserDaoImpl 实现类

```java
package com.liu.dao;

public class UserDaoImpl implements UserDao {
    public void getUser() {
        System.out.println("获取用户");
    }
}

```



3、UserService 业务接口

```java
package com.liu.Service;

public interface UserService {
    public void getUser();
}

```



4、UserServiceImpl 业务实现类

```java
package com.liu.Service;

import com.liu.dao.UserDao;
import com.liu.dao.UserDaoImpl;

public class UserServiceImpl implements UserService{
    private UserDao userDao = new UserDaoImpl();
    public void getUser() {
        userDao.getUser();
    }
}

```

5、test测试类

```java
import com.liu.Service.UserService;
import com.liu.Service.UserServiceImpl;

public class Mytest {
    public static void main(String[] args) {
        UserService userService = new UserServiceImpl();
        userService.getUser();
    }
}

```

在我们之前的业务中，用户的需求可能会影响我们原来的代码，我们需要根据用户的需求去修改原代码，如果代码量十分大，修改一次的成本代价十分昂贵



我们使用一个Set接口实现，已经发生了革命性的变化

```java
    //利用set进行动态实现值的注入
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }
```

之前，程序是主动创建对象，控制权在程序员手上

使用set注入之后，程序不再具有主动性，而是变成了被动的接受对象



这种思想，从本质上解决了问题，程序员不用再去管理对象的创建了，系统的耦合性大大降低，可以更加专注在业务的实现上，这是IOC的原型。

### IOC本质

控制反转IoC（Inversion of Control），是一种设计思想，DI（依赖注入）是实现IoC的一种方法，也有人认为DI只是IoC的另一种说法。没有IoC的程序中，我们使用面向对象编程，对象的创建与对象间的依赖关系完全硬编码在程序中，对象的创建由程序自己控制，控制反转后将对象的创建转移给第三方，个人认为所谓控制反转就是：获得依赖对象的方式反转了。

采用XML方式配置Bean的时候，Bean的定义信息是和实现分离的，而采用注解的方式可以把两者合为一体，Bean的定义信息直接以注解的形式定义在实现类中，从而达到了零配置的目的。
**控制反转是一种通过描述（XML或注解）并通过第三方去生产或获取特定对象的方式。在Spring中实现控制反转的是IoC容器，其实现方法是依赖注入（Dependency Injection，DI）。**


# 3、HelloSpring

1·、创建一个Hello实体类

```java
package com.liu.pojo;

public class Hello {
    private String str;

    public String getStr() {
        return str;
    }

    public void setStr(String str) {
        this.str = str;
    }

    @Override
    public String toString() {
        return "Hello{" +
                "str='" + str + '\'' +
                '}';
    }
}

```

2、在resources文件夹下创建beans.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
<!--使用spring来创建对象，在spring这些都称为Bean-->
    <!--类型 变量名 = new 类型-->
    <!--Hello hello =new Hello-->
    <!--id = 变量名 class = new的对象
    property相当于给对象中的属性赋值
    -->
    <bean id="hello" class="com.liu.pojo.Hello">
        <property name="str" value="Spring"/>
    </bean>
</beans>
```

3、创建测试类Test

```java
import com.liu.pojo.Hello;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class MyTest {
    public static void main(String[] args) {
        //获取spring的上下文对象
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        //我们的对象现在都在spring中管理，我们要使用，直接去里面取出来就可以
        Hello hello = (Hello) context.getBean("hello");
        System.out.println(hello.toString());

    }
}

```

4、控制台输出

```java
Hello{str='Spring'}
```

思考问题

Hello对象是谁创建的？Hello对象由Spring创建

Hello对象的属性是怎么设置的？ Hello对象的属性由Spring容器设置

这个过程就叫控制反转

控制：谁来控制对象的创建，传统应用程序的对象是由程序本身控制创建，使用Spring后，对象由Spring来创建

反转：程序本身不创建对象，而变成被动的接收对象

依赖注入：就是利用set方法进行注入

IOC是一种编程思想，由主动的编程变成被动的接收

可以通过new ClassPathXmlApplicationContext浏览底层源代码

# 4、对之前例子的优化

1、在resources文件夹下创建beans.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="mysqlImpl" class="com.liu.dao.UserDaoMysqlImpl"></bean>
    <bean id="oracleImpl" class="com.liu.dao.UserDaoOracleImpl"></bean>
    <bean id="UserServiceImpl" class="com.liu.Service.UserServiceImpl">
        <!--ref:引用spring中创建好的对象-->
        <!--value:具体的值，基本数据类型-->
        <property name="userDao" ref="mysqlImpl"/>
    </bean>
</beans>
```

2、修改测试类

```java
import com.liu.Service.UserService;
import com.liu.Service.UserServiceImpl;
import com.liu.dao.UserDaoImpl;
import com.sun.tools.internal.xjc.reader.xmlschema.bindinfo.BIConversion;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Mytest {
    public static void main(String[] args) {
//        UserServiceImpl userService = new UserServiceImpl();
//        userService.setUserDao(new UserDaoImpl());
//
//        userService.getUser();
        //获取ApplicationContext
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        UserService userServiceImpl = (UserService) context.getBean("UserServiceImpl");
        userServiceImpl.getUser();

    }
}

```

控制台输出

```java
mysql
```

之前创建对象需要在代码中改动，用了spring之后只需要改xml文件

# 5、IOC创建对象的方式

1、使用无参构造创建对象，默认

2、假设我们要使用有参构造创建对象

​	1、下标赋值

```xml
<!--       第一种，下标赋值-->
    <bean id="user" class="com.liu.pojo.User">
        <constructor-arg index="0" value="liuhao">
    </bean>
```

2、通过类型赋值

```xml
<!--    第二种，通过类型创建-->
    <bean id="user" class="com.liu.pojo.User">
        <constructor-arg type="java.lang.String" value="liuhao"/>
    </bean>
```

3、直接通过参数名创建

```xml
<!--    第三种，直接通过参数名创建-->
    <bean id="user" class="com.liu.pojo.User">
        <constructor-arg name="name" value="liuhao"/>
    </bean>

```

总结：在配置文件加载的时候，容器中管理的对象就已经初始化了，对象的无参构造被调用

# 6.Spring配置

## 6.1别名

```xml
    <!--别名，如果添加了别名，我们也可以使用别名获取到这个对象-->
    <alias name="user" alias="user2"/>

```



## 6.2Bean的配置

```xml
<!--    id:bean 的唯一标识符，相当于我们的对象名
        class：bean的对象所对应的全限定名：包名+类型
        name：也是别名，而且name可以同时取多个别名
-->
    <bean id="userT" class="com.liu.pojo.UserT" name="user3,u2">
			<property name ="name" value="西部开源"/>
		</bean>

```



## 6.3 import

这个import，一般用于团队开发使用,他可以将多个配置文件，导入合并为一个

假设，现在项目中有多个人开发，这三个人负责不同的类开发，不同的类需要注册在不同的bean中，我们可以利用import将所有人的beans.xml合并为一个总的！

张三

李四

王五

applicationContext.xml

```xml
    <import resource="beans.xml"/>
    <import resource="beans2.xml"/>
    <import resource="beans3.xml"/>
```

使用的时候，直接使用总的配置就可以了

# 7、依赖注入

## 7.1、构造器注入

前面已经说过了

## 7.2、Set方式注入(重点)

依赖注入：本质是Set注入！

依赖：bean对象的创建依赖于容器

注入：bean对象中的所有属性，由容器来注入！

【环境搭建】

1、复杂类型

```java
package com.liu.pojo;

public class Address {
    private String address;

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    @Override
    public String toString() {
        return "Address{" +
                "address='" + address + '\'' +
                '}';
    }
}

```



2、真实测试对象

```java
package com.liu.pojo;

import java.util.List;
import java.util.Map;
import java.util.Properties;

public class Student {
    private String name;
    private Address address;
    private String [] book;
    private List<String> hobby;
    private Map<String,String> card;
    private String wife;
    private Properties info;

```

3、beans.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="student" class="com.liu.pojo.Student">
        <property name="name" value="liuhao"/>

    </bean>
</beans>
```

4、测试类

```java
import com.liu.pojo.Student;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class MyTest {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        Student student = (Student) context.getBean("student");
        System.out.println(student.getName());

    }
}

```

5、完善注入信息

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="address" class="com.liu.pojo.Address">
        <property name="address" value="西安"/>
    </bean>
    <bean id="student" class="com.liu.pojo.Student">
<!--        第一种，普通值注入,value-->
        <property name="name" value="liuhao"/>
<!--        第二种，bean注入 ref-->
        <property name="address" ref="address"/>
<!--        数组注入 -->
        <property name="book">
            <array>
                <value>红楼梦</value>
                <value>西游记</value>
                <value>水浒传</value>
                <value>三国演义</value>
            </array>
        </property>
<!--        list-->
        <property name="hobby">
            <list>
                <value>听歌</value>
                <value>敲代码</value>
                <value>看电影</value>
            </list>
        </property>
<!--        Map-->
        <property name="card">
            <map>
                <entry key="身份证" value="123456123456123"/>
                <entry key="银行卡" value="111111111111"/>
            </map>
        </property>
<!--        Set-->
        <property name="games">
           <set>
               <value>LOL</value>
               <value>COC</value>
               <value>BOB</value>
           </set>
        </property>
<!--        null-->
        <property name="wife">
            <null/>
        </property>
<!--        properties-->
        <property name="info">
            <props>
                <prop key="学号">20190525</prop>
                <prop key="性别">男</prop>
                <prop key="姓名">小明</prop>
            </props>
        </property>
    </bean>
</beans>
```



## 7.3、拓展方式注入

我们可以使用P命名空间和C命名空间进行注入

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:c="http://www.springframework.org/schema/c"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">


    <!--p命名空间注入，可以直接注入属性的值，相当于property-->
    <bean id="user" class="com.liu.pojo.User" p:name="刘浩"/>
    <!--c命名空间注入，通过构造器注入:construct-args-->
    <bean id="user2" class="com.liu.pojo.User" c:name="刘浩浩"/>

</beans>
```

测试：

```java
    public void Test(){
        ApplicationContext context = new ClassPathXmlApplicationContext("userBeans.xml");
        User user = (User) context.getBean("user2");
        System.out.println(user.toString());
    }
```

## 7.4 Bean的作用域

![7FC202FB-46AC-4F11-9E15-4C9BE7942082](/Users/apple/Library/Containers/com.tencent.qq/Data/Library/Application Support/QQ/Users/781936292/QQ/Temp.db/7FC202FB-46AC-4F11-9E15-4C9BE7942082.png)

1、单例模式(Spring默认机制)

```xml
<bean id="user" class="com.liu.pojo.User" p:name="刘浩" scope="singleton"/>
```

```java
    public void Test(){
        ApplicationContext context = new ClassPathXmlApplicationContext("userBeans.xml");
        User user = (User) context.getBean("user");
        User user1 = (User) context.getBean("user");
        System.out.println(user==user1);
    }
输出：true
```

2、原型模式：每次从容器中get的时候，都会产生一个新对象

```xml
  <bean id="user2" class="com.liu.pojo.User" c:name="刘浩浩" scope="prototype"/>
```

```java
    public void Test(){
        ApplicationContext context = new ClassPathXmlApplicationContext("userBeans.xml");
        User user = (User) context.getBean("user2");
        User user1 = (User) context.getBean("user2");
        System.out.println(user==user1);
    }
输出：false
```

3、其余的request、session、application这些个只能在web开发中使用到

# 8.Bean的自动装配

自动装配是Spring满足bean依赖的一种方式

Spring会在上下文中自动寻找，并自动给bean装配属性



在Spring中有三种装配方式

1、在xml中显示的配置

2、在java中显示配置

3、隐式的自动装配bean【重要】

## 8.1、测试

1、环境搭建

​	一个人有两个宠物

## 8.2、ByName自动装配

```xml
<!--  byName:会自动在容器上下文中查找，和自己对象set方法后面的值对应的bean id -->
    <bean id="people" class="com.liu.pojo.People" autowire="byName">
        <property name="name" value="刘浩"/>
    </bean>

```

## 8.3ByType自动装配

```xml
<!--  byType:会自动在容器上下文中查找class相同额bean,但是只能有一个bean-->
    <bean id="people" class="com.liu.pojo.People" autowire="byType">
        <property name="name" value="刘浩"/>
    </bean>
```

小结：

byname的时候，需要保证所有的bean id唯一，并且这个bean需要和注入的属性的set方法的值唯一

bytype的时候，需要保证所有bean的class唯一，并且这个bean需要和自动注入的属性的类型一致

## 8.4、使用注解进行自动装配

jdk1.5支持的注解，Spring2.5就支持注解了

使用注解须知：

1、导入约束：context约束

2、配置注解的支持：<context:annotation-config/>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>
    <bean id="cat" class="com.liu.pojo.Cat"/>
    <bean id="dog" class="com.liu.pojo.Dog"/>
    <bean id="people" class="com.liu.pojo.People"/>
</beans>

```

**@Autowired**

直接在属性上使用即可，也可以在set方式上使用

使用Autowired我们可以不用编写set方法了，前提是你这个自动装配的属性在IOC（Spring）容器中存在，且符合名字byName

```java
package com.liu.pojo;

import org.springframework.beans.factory.annotation.Autowired;

public class People {
    @Autowired
    private Cat cat;
    @Autowired
    private Dog dog;
    private String name;

    public Cat getCat() {
        return cat;
    }

    public void setCat(Cat cat) {
        this.cat = cat;
    }

    public Dog getDog() {
        return dog;
    }

    public void setDog(Dog dog) {
        this.dog = dog;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "People{" +
                "cat=" + cat +
                ", dog=" + dog +
                ", name='" + name + '\'' +
                '}';
    }
}

```

科普：

```xml
@Nullable  字段标记了这个注解，说明这个字段可以为null;
```

@**Resource**

```java
public class People {
    @Resource(name = "cat2")
    private Cat cat;
    @Resource
    private Dog dog;
}
```

小结：

@Resource和@Autowired的区别

都是用来自动装配，都可以放在属性字段上

@Autowired通过bytype的方式实现

@Resource默认通过byname的方式实现，如果找不到名字，则通过bytype实现，如果两个都不找不到的情况下，就报错

执行顺序不同:autowired先类型后名字；resource先名字后类型

# 9、使用注解开发

在spring4之后，要使用注解开发，必须保证aop的包导入了

使用注解需要导入context约束，增加注解的支持!

1、bean

```xml
    <!--    指定要扫描的包，这个包下面的注解就会生效-->
    <context:component-scan base-package="com.liu.pojo"/>
    <context:annotation-config/>
```



2、属性如何注入

```java
//等价于<bean id="user" class="com.liu.pojo.User"/>
//@Component组件
@Component
public class User {
    //相当于<property name="name" value="刘浩"/>
    @Value("刘浩")
    public String name;
}

```

3、衍生的注解

​	@Component 有几个衍生注解，我们在web开发中，会按照mvc三层架构分层

​		dao 【@Repository】

​		service【@Service】

​		controller【@Controller】

​			这四个注解功能都是一样的，都是代表将某个类注册到Spring中，装配

4、自动装配置

```
- @Autowired：自动装配通过类型，名字
- @Nullable：字段标记了这个注解，说明这个字段可以为null
- @Resource: 自动装配通过名字，类型
```

5、作用域

```java
@Component
@Scope("singleton")
public class User {
    //相当于<property name="name" value="刘浩"/>
    @Value("刘浩")
    public String name;
}

```



6、小结

xml与注解：

​	xml更加万能， 适用于任何场合！维护简单方便

​	注解 不是自己类使用不了，维护相对复杂

xml与注解最佳实践：

​	xml管理bean	

​	注解负责属性的注入

# 10、使用Java的方式配置Spring

我们现在要完全不使用Spring的xml配置了，全权交给Java来做

JavaConfig是Spring的一个子项目，在Spring4之后，他成为一个核心功能

**实体类**

```java
package com.liu.pojo;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Controller;

//这里这个注解的意思，就是说明这个类被Spring接管了，注册到了容器中
@Controller
public class User {
    private String name;

    public String getName() {
        return name;
    }
    @Value("刘浩")//注入值
    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                '}';
    }
}

```

**配置文件**

```java
package com.liu.config;

import com.liu.pojo.User;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;

@Configuration //这个也会Spring容器托管，注册到容器中，因为他本来就是一个@Component
//@Configuration代表这是一个配置类，这和我们之前的beans.xml一样的
@ComponentScan("com.liu.pojo")
@Import(Config2.class)
public class Config {
    //注册一个bean，就相当于我们之前写的bean标签
    //这个方法的名字，就相当于bean标签的id属性
    //这个方法的返回值，就相当于bean标签中的class属性
    @Bean
    public User getUser(){
        return new User();
    }
}


```



**测试类**

```java
import com.liu.config.Config;
import com.liu.pojo.User;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class MyTest {
    @Test
    public void Test(){
        //如果完全使用了配置类方式去做，我们就只能通过AnnotationConfig去获取上下文
        ApplicationContext context = new AnnotationConfigApplicationContext(Config.class);
        User getUser = (User) context.getBean("getUser");
        System.out.println(getUser.toString());
    }

}

```

# 11、代理模式

为什么要学代理模式？因为这就是Spring AOP的底层【SpringAOP和Spring MVC】

代理模式分类：

静态代理

动态代理

## 11.1、静态代理

角色分析：

​	抽象角色：一般会使用接口或者抽象类来解决

​	真实角色：被代理的角色

​	代理角色：代理真实角色，代理真实角色后，我们一般会做一些附属操作

​	客户：访问代理对象的人

​	

代理步骤：

 	1、接口

```java
package com.liu.proxy;

//租房
public interface Rent {
    public void rent();
}

```



​	 2、真实角色

```java
package com.liu.proxy;

//房东
public class Host implements Rent{


    public void rent() {
        System.out.println("房东要出租房子");
    }
}

```



​	 3、代理角色

```java
package com.liu.proxy;

public class Proxy implements Rent{
    private Host host;

    public Proxy() {
    }

    public Proxy(Host host) {
        this.host = host;
    }

    public void rent() {
        host.rent();
    }
    //看房
    public void seeHouse(){
        System.out.println("中介带你看房子");
    }
    //签合同
    public void heTong(){
        System.out.println("签合同");
    }
    //收中介费
    public void fare(){
        System.out.println("收中介费");
    }
}

```



​     4：客户端访问代理角色

```java
package com.liu.proxy;

public class Client  {
    public static void main(String[] args) {
        //房东要租房子
        Host host = new Host();
        //代理，中介要帮房东租房子，但是代理角色一般会有一些附属操作
        Proxy proxy = new Proxy(host);
        //你不用面对房东，直接找中介租房子即可
        proxy.rent();
    }
}

```



代理模式的好处：

​	可以使真实角色的操作更加纯粹，不用去关注一些公共的业务

​	公共业务就交给代理角色，实现了业务的分工

​	公共业务发生拓展的时候，方便集中管理

缺点：

​	一个真实角色就会产生一个代理角色；代码量会翻倍，开发效率变低

## 11.2、动态代理

​	动态代理和静态代理角色一样

​	动态代理的代理类是动态生成的，不是我们直接写好的

​	动态代理分为两大类：基于接口的动态代理，基于类的动态代理

​		基于接口：JDK的动态代理

​		基于类：cglib

​		java字节码实现：javasist

需要了解两个类：Proxy，InvocationHandler：调用处理程序

动态代理的好处：

​	一个动态代理类代理的是一个接口，一般就是对应的一类业务

​	一个动态代理类可以代理多个类，只要是实现了同一个接口即可

# 12、AOP

## 12.1、 什么是AOP

AOP意为：面向切面编程，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术，AOP是OOP的延续，是软件开发中的一个热点，也是Spring框架中的一个重要内容，是函数式编程的一种衍生泛型。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发效率

## 12.2、AOP在Spring中的作用

**提供声明式事务，允许用户自定义切面**

横切关注点：跨越应用程序多个模块的方法或功能。即，与我们业务逻辑无关的，但是我们需要关注的部分，就是横切关注点，如日志，安全，缓存，事务等

切面(ASPECT)：横切关注点背模块化的特殊对象，即，他是一个类

通知(ADVICE)：切面必须要完成的工作，即，他是类中的一个方法

目标(TARGET)：被通知对象

代理(PROXY)：向目标对象应用通知之后创建的对象

切入点(POINTCUT)：切面通知执行的“地点”的定义

连接点(JOINTPOINT)：与切入点匹配的执行点



SpringAOP中，通过ADVICE定义横切逻辑，Spring中支持5中类型的Advice

![5A7C9A1B-0A9F-4E83-9F05-6F3F58A5DD7E](/Users/apple/Library/Containers/com.tencent.qq/Data/Library/Application Support/QQ/Users/781936292/QQ/Temp.db/5A7C9A1B-0A9F-4E83-9F05-6F3F58A5DD7E.png)

即AOP在不改变原有代码的情况下，去增加新的功能

## 11.3使用Spring实现Aop

【重点】使用AOP植入，需要导入一个依赖包！

```xml
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.6</version>
        </dependency>

```

方式一：使用Spring的API接口

```xml
    <!--    注册bean-->
    <bean id="log" class="com.liu.log.log"/>
    <bean id="AfterLog" class="com.liu.log.AfterLog"/>
    <bean id="userService" class="com.liu.service.UserServiceImpl"/>
        方式一：使用原生Spring API接口
        配置aop:需要导入aop的约束
        <aop:config>
    <!--        切入点：expression:表达式，execution(要执行的位置)-->
            <aop:pointcut id="pointcut" expression="execution(* com.liu.service.UserServiceImpl.*(..))"/>
    <!--        执行环绕增加-->
            <aop:advisor advice-ref="log" pointcut-ref="pointcut"/>
            <aop:advisor advice-ref="AfterLog" pointcut-ref="pointcut"/>
        </aop:config>
```



方式二、自定义来实现AOP【主要是切面定义】

```xml
<!--    方式二：自定义类-->
    <bean id="diy" class="com.liu.diy.DIyPointCut"/>
    <aop:config>
<!--        自定义切面 ref 要引用的类-->
        <aop:aspect ref="diy">
<!--            切入点-->
            <aop:pointcut id="point" expression="execution(* com.liu.service.UserServiceImpl.*(..))"/>
<!--             通知-->
            <aop:before method="before" pointcut-ref="point"/>
            <aop:after method="after" pointcut-ref="point"/>
        </aop:aspect>
    </aop:config>
```

方式三、使用注解实现

```java
package com.liu.diy;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.Signature;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

//方式三：使用注解方式实现aop
@Aspect //标注这个类是一个切面
public class Annotation {
    @Before("execution(* com.liu.service.UserServiceImpl.*(..))")
    public void before(){
        System.out.println("=======方法执行前=======");
    }
    @After("execution(* com.liu.service.UserServiceImpl.*(..))")
    public void after(){
        System.out.println("=======方法执行后=======");
    }
    @Around("execution(* com.liu.service.UserServiceImpl.*(..))")
    public void around(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("环绕前");
        Signature signature = joinPoint.getSignature();//获得签名
        System.out.println("signature"+signature);
        //执行方法
        Object proceed = joinPoint.proceed();
        System.out.println("环绕后");
    }
}

```

```xml
    <!--    方式三：使用注解-->
    <bean id="annotation" class="com.liu.diy.Annotation"/>
    <!--    开启注解支持-->
    <aop:aspectj-autoproxy/>
```



aop的思想：在不影响原来业务类的情况下，进行动态的增强

# 13、整合Mybatis

步骤：

​	1、导入相关jar包

​		junit

​		mybatis

​		mysql

​		spring相关的

​		aop织入

​		mybatis-spring

​	2、编写配置文件

​	3、测试

## 13.1、回忆mybatis

1、编写实体类

2、编写核心配置文件

3、编写接口

4、编写Mapper.xml

5、测试

## 13.2、Mybatis-spring

1、编写数据源配置

2、sqlSessionFactory

3、sqlSessionTemplate

4、需要给接口加实现类

5、将自己写的实现类，注入到Spring中

6、测试使用即可

