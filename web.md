# 1、MVC三层架构

什么是MVC:model view contrller![2F1FF6F5-42C5-41C0-9661-6FB6EA4F7608](/Users/apple/Desktop/2F1FF6F5-42C5-41C0-9661-6FB6EA4F7608.png)

Model：

业务处理：业务逻辑(Service)

数据持久层：CRUD(Dao)

View:

展示数据

提供链接发起Servlet请求(a form img...)

Controller(Servlet)

接收用户的请求(req:请求参数、Session信心)

交给业务层处理对应的代码

控制视图的跳转

```java
登录--->接收用户的登录请求--->处理用户的请求(获取用户登录的参数、username、pwd)--->交给业务层处理登录业务(判断用户密码是否正确、事务)--->Dao层查询用户名和密码是否正确--->数据库
```

# 2、Filter

Filter：过滤器，用来过滤网站的数据；

● 处理中文乱码

● 登录验证

Filter开发步骤

1、导包

2、建过滤器

​	1、导包不要错javax.servlet

​	2、实现Filter接口，重写对应的方法即可

```java
package com.liu.filter;

import javax.servlet.*;
import java.io.IOException;

public class CharacterEncodingFilter implements Filter {
    //初始化
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("CharacterEncodingFilter初始化");
    }
    //Chain：链
//     过滤中的所有代码，在过滤特定请求的时候都会执行
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        request.setCharacterEncoding("utf-8");
        response.setCharacterEncoding("utf-8");
        response.setContentType("text/html;charset=utf-8");
        System.out.println("CharacterEncodingFilter执行前");
        chain.doFilter(request,response);//让我们的程序继续走，如果不写就没法执行。
        System.out.println("CharacterEncodingFilter执行后");
    }
//web服务器关闭的时候，过滤器会销毁
    public void destroy() {

    }
}

```

​	3、在web.xml中配置filter过滤器

```xml
    <filter>
        <filter-name>CharacterEncodingFilter</filter-name>
        <filter-class>com.liu.filter.CharacterEncodingFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>CharacterEncodingFilter</filter-name>
        <!--   只要是/servlet的所有请求，都要经过过滤器     -->
        <url-pattern>/Servlet/*</url-pattern>
    </filter-mapping>
```



# 3、JDBC

需要jar包的支持

●java.sql

●javax.sql

●mysql-conneter-java...连接驱动

### 实验环境搭建

1、新建数据库

2、导入mysql依赖

```xml
<dependencies>
    <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.23</version>
    </dependency>

```

3、建立JDBC

```java
package com.liu.test;

import java.sql.*;

public class Testjdbc {
    public static void main(String[] args) throws ClassNotFoundException, SQLException {

        //配置信息
        //useUnicode=true&characterEncoding=utf-8解决中文乱码
        String url = "jdbc:mysql://localhost:3306/jdbc?useUnicode=true&characterEncoding=utf-8";
        String username = "root";
        String password = "liuhao123";

        //1、加载驱动
        Class.forName("com.mysql.cj.jdbc.Driver");
        //2、连接数据库、代表数据库
        Connection connection = DriverManager.getConnection(url, username, password);
        //3、向数据库发送sql的对象statement：CRUD
        Statement statement = connection.createStatement();
        //4、编写sql
        String sql = "select * from people";
        //5、执行sql,返回一个resultset结果集
        ResultSet resultSet = statement.executeQuery(sql);

        while(resultSet.next()){
            System.out.println("id"+resultSet.getObject("id"));
            System.out.println("name"+resultSet.getObject("name"));
            System.out.println("age"+resultSet.getObject("age"));
            System.out.println("address"+resultSet.getObject("address"));
        }
        //6、关闭连接，释放资源，先开的后关
        resultSet.close();
        statement.close();
        connection.close();

    }
}

```

JDBC固定六部曲

1、加载驱动

2、连接数据库

3、向数据库发送sql的对象statement：CRUD

4、编写sql

5、执行sql

6、关闭连接，释放资源，先开的后关

### 事务

要么都成功，要么都失败

ACID原则： 保证数据的安全

```java
1、开启事务
2、事务提交 commit()
3、事务回滚 rollback()
4、关闭事务
  

```

## 项目搭建

1、搭建一个maven web项目

2、配置Tomcat

3、测试项目是否能够跑起来

4、导入项目中会用到的jar包

jsp、servlet、mysql驱动、jstl、standard

5、创建项目包结构

6、编写实体类

ORM映射：表-类映射

7、编写基础公共类

数据库配置文件

```properties
driver=com.mysql.cj.jdbc.Driver
url = jdbc:mysql//localhost:3306?useUnicode=true&characterEncoding=UTF-8
username=root;
password=123456;
```

编写数据库的公共类

```java
package com.liu.dao;


import java.io.IOException;
import java.io.InputStream;
import java.sql.*;
import java.util.Properties;

//操作数据库的公共类
public class BaseDao {
    private static String driver;
    private static String url;
    private static String username;
    private static String password;

    //静态代码块，类加载的时候就初始化了
    static {
        Properties properties = new Properties();
        InputStream is = BaseDao.class.getClassLoader().getResourceAsStream("db.properties");
        try {
            properties.load(is);
        } catch (IOException e) {
            e.printStackTrace();
        }
        driver = properties.getProperty("driver");
        url = properties.getProperty("url");
        username = properties.getProperty("username");
        password = properties.getProperty("password");
    }

   public static Connection getConnection() throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.cj.jdbc.Driver");
       Connection connection = DriverManager.getConnection(url, username, password);
       return connection;
   }
    //编写查询公共类
    public static ResultSet execute(Connection connection,String sql,Object[] param,ResultSet resultSet,PreparedStatement preparedStatement) throws SQLException {
        preparedStatement = connection.prepareStatement(sql);
        for (int i = 0; i < param.length; i++) {
            preparedStatement.setObject(i+1,param[i]);
        }
       resultSet = preparedStatement.executeQuery();
        return resultSet;
    }
    //编写增删改查公共方法
    public static int execute(Connection connection,String sql,Object[] param,PreparedStatement preparedStatement) throws SQLException {
        preparedStatement = connection.prepareStatement(sql);
        for (int i = 0; i < param.length; i++) {
            preparedStatement.setObject(i+1,param[i]);
        }
        int updateRows = preparedStatement.executeUpdate();
        return updateRows;
    }
    //释放资源
    public static boolean closeResource(Connection connection,PreparedStatement preparedStatement,ResultSet resultSet) throws SQLException {

        boolean flag = true;
        if (resultSet!=null){
            resultSet.close();
        }else {
            flag = false;
        }
        if (preparedStatement!=null){
            preparedStatement.close();
        }else {
            flag = false;
        }
        if (connection!=null){
            connection.close();
        }else {
            flag = false;
        }
        return flag;


    }
}

```



编写字符编码过滤器

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>com.liu.filter.CharacterEncodingFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>

```

8.导入静态资源

### 登录功能实现

编写前端页面

在XML设置登录页面

```xml
 <welcome-file-list>
        <welcome-file>login.jsp</welcome-file>
    </welcome-file-list>
```

编写Dao层登录用户的接口

```java
public interface userDao {

    //得到登录的用户
    public User getLoginUser(Connection connection, String userCode) throws SQLException;

}

```

编写dao接口的实现类

```java
package com.liu.dao.user;

import com.liu.dao.BaseDao;
import com.liu.pojo.User;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class userDaoImpl implements userDao {

    @Override
    public User getLoginUser(Connection connection, String userCode) throws SQLException {

        PreparedStatement pstm = null;
        ResultSet resultSet = null;
        User user = null;


        if (connection!=null){
            String sql = "select * from smbms user where userCode=?";
            Object[] param = {userCode};
            resultSet = BaseDao.execute(connection, pstm, resultSet, sql, param);
            if (resultSet.next()){
                user = new User();
                user.setId(resultSet.getInt("id"));
                user.setUserCode(resultSet.getString("userCode"));
                user.setUserName(resultSet.getString("userName"));
                user.setUserPassword(resultSet.getString("userPassword"));
                user.setGender(resultSet.getInt("gender"));
                user.setBirthday(resultSet.getDate("birthday"));
                user.setPhone(resultSet.getString("phone"));
                user.setAddress(resultSet.getString("address"));
                user.setUserRole(resultSet.getInt("userRole"));
                user.setCreatedBy(resultSet.getInt("createdBy"));
                user.setCreationDate(resultSet.getDate("creationDate"));
                user.setModifyBy(resultSet.getInt("modifyBy"));
                user.setModifyDate(resultSet.getDate("modifyDate"));
            }
        }
        BaseDao.closeResource(connection,pstm,resultSet);
        return user;



    }
}

```

