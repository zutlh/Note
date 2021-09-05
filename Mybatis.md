# Mybatis

### 1.2持久化

数据持久化

​	 ● 持久化就是将程序的数据在持久状态和瞬时状态转化的过程

​	 ● 内存：断电即失

​	 ● 数据库，io文件持久化

​	 ● 生活：冷藏

##### 为什么需要持久化

 	●有些对象，不能让他丢失

​	 ● 内存太贵了

### 1.3 持久层

Dao层 Service层 Controller层

●完成持久化工作的代码块

●层界限十分明显

### 1.4为什么需要Mybatis

●方便

●传统的jdbc代码太复杂了

●帮助程序员将数据存入到数据库中

## 2、第一个Mybatis程序

思路：搭建环境-->导入Mybatis-->编写代码-->测试！

#### 2.1搭建环境

搭建数据库

```sql
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(30) DEFAULT NULL,
  `pwd` varchar(30) NOT NULL DEFAULT ''123456'',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8
```

新建项目

1、新建一个maven项目

2、删除src目录

3、导入maven依赖

### 2.2创建一个模块

编写mybatis核心配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!--核心配置文件-->
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/Mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
                <property name="username" value="root"/>
                <property name="password" value="liuhao123"/>
            </dataSource>
        </environment>
    </environments>

</configuration>
```



编写工具类

```java
package com.liu.utils;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;

//sqlSessionFactory --->sqlSession
public class MybatisUtils {
    private static SqlSessionFactory sqlSessionFactory;
    static {
        try {
            //使用Mybatis的第一步，获取sqlSessionFactory对象
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    //既然有了SqlSessionFactory,顾明思义，我们就可以从中获得SqlSession的实例了
    //SqlSession完全包含了面向数据对库执行SQL命令所需的所有方法
    public static SqlSession getSqlSession(){
        SqlSession sqlSession = sqlSessionFactory.openSession();
        return sqlSession;
    }
}

```

### 2.3 编写代码

实体类

```java
package com.liu.pojo;

public class User {
    private int id;
    private String name;
    private String password;

    public User() {
    }

    public User(int id, String name, String password) {
        this.id = id;
        this.name = name;
        this.password = password;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}

```



Dao接口

```java
public interface UserDao {
    List<User> getUserList();
}

```

接口实现类由原来的UserDaoImpl转换成一个Mapper配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--namespace=绑定一个对应的Dao/Mapper接口-->
<mapper namespace="com.liu.dao.UserDaoInterface">
<!--id对应方法名字-->
    <select id="getUserList" resultType="com.liu.pojo.User">
        select * from Mybatis.user
  </select>
</mapper>
```



### 2.4测试

注意点：

org.apache.ibatis.binding.BindingException: Type interface com.liu.dao.UserDao is not known to the MapperRegistry.

junit测试

```java
package com.liu.dao;

import com.liu.pojo.User;
import com.liu.utils.MybatisUtils;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;

import java.util.List;

public class UserDaoTest {

    @Test
    public void test(){
        //第一步，获得SQLSession对象
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        //执行SQL
        UserDao mapper = sqlSession.getMapper(UserDao.class);
        List<User> userList = mapper.getUserList();
        for (User user : userList) {
            System.out.println(user);
        }//关闭sqlSession
        sqlSession.close();
    }


}

```

## 3、CRUD

### 1、namespace

namespace中的包名要和Dao/Mapper接口的包名一致

### 2、select

选择、查询语句

● id就是对应的namespace中的方法名

● resultType：Sql语句的返回值！

● param

