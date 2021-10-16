# mybatis笔记


![avatar](/mybatis.jpg)

[代码](https://github.com/xslorry/lorry_javaNotebook.github.io)



# 第一个mybatis程序

## 搭建环境

### 搭建数据库

```sql
CREATE DATABASE `mybatis`;
USE `mybatis`;
CREATE TABLE `user`(
    `id` INT(20) NOT NULL PRIMARY KEY,
    `name` VARCHAR(30) DEFAULT NULL,
    `pwd` VARCHAR(30) DEFAULT NULL
)ENGINE=INNODB DEFAULT CHARSET=utf8;
INSERT INTO `user`(`id`,`name`,`pwd`) VALUES
(1,'张三','123456'),
(2,'李四','123456'),
(3,'王五','123456'),
(4,'赵六','123456'),
(5,'肖七','123456')
```

### 新建项目

1. 新建一个普通的maven'项目（狂神教程是先建一个大的maven项目然后在里面建小项目）

   有一个问题：建立了maven06之后删除再建立显示maven06存在，在父目录里面删除后还有

2. 删除src目录

   

3. 导入maven依赖

```xml
    <!--daoruyilai-->
    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.25</version>
        </dependency>

        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.7</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>

    </dependencies>
```

### 使用mybatis的基础配置

#### 编写mybatis的核心配置文件

在resource文件夹下建立mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8&amp;serverTimezone=Asia/Shanghai"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
<!--每一个mapper.xml都需要在Mybatis核心配置文件中注册！-->
    <mappers>
        <mapper resource="UserMapper.xml"></mapper>
    </mappers>
</configuration>
```

因为使用的是MySQL8.0，注意时区问题

#### 编写mybatis工具类

```java
public class MybatisUtils {
    private static SqlSessionFactory sqlSessionFactory;
    //这里不加static会报空指针异常
    static{
        try {
            // 使用mybatis获得sqlSessionFactory对象
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    /*
    * 既然有了 SqlSessionFactory，顾名思义，我们可以从中获得 SqlSession 的实例。
    * SqlSession 提供了在数据库执行 SQL 命令所需的所有方法。
    * 你可以通过 SqlSession 实例来直接执行已映射的 SQL 语句。例如：
    * */
    public static SqlSession getSqlSession(){
//        SqlSession sqlSession = sqlSessionFactory.openSession();
//        return sqlSession;
        return sqlSessionFactory.openSession();

    }
}
```

参考官方文档，编写工具类的原因是能够更简便的开发

## 实现CRUD

在lorry下新建maaper文件下存放接口，接口命名为UserMapper

```java
public interface UserMapper {
    //查询全部用户
    List<User> getUserList();
    //根据ID查询用户
    User getUserById(int id);
    //map
    User getUserById2(Map<String,Object>map);
    //insert一个用户
    int addUser(User user);
    //万能的map
    int addUser2(Map<String,Object>map);
    //修改用户
    int updateUser(User user);
    //删除用户
    int deleteUser(int id);
    List<User> getUserLike(String value);
}

```

在lorry下新建pojo文件夹放实体类并生成有参无参构造get set方法和toString

```java
public class User {
    private int user_id;
    private String user_name;
    private String user_pwd;
}
```

在resource下新建UserMapper.xml文件

UserMapper.xml需要在mybatis-config.xml下注册

```xml
<!--每一个mapper.xml都需要在Mybatis核心配置文件中注册！-->
    <mappers>
        <mapper resource="UserMapper.xml"></mapper>
    </mappers>
```

配置可以写resource也可以写class，写class直接对应着类，用注释的时候常用

每一个namespace对应一个Mapper接口

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--namesapce绑定一个个对应的Dao/mapper接口-->
<mapper namespace="com.lorry.mapper.UserMapper">
    <select id="getUserList" resultType="com.lorry.pojo.User">
        select * from mybatis.user
    </select>

    <select id="getUserById" parameterType="int" resultType="com.lorry.pojo.User">
        select * from mybatis.user where user_id = #{id}
    </select>

    <insert id="addUser" parameterType="com.lorry.pojo.User">
        insert into mybatis.user(user_id, user_name, user_pwd) value (#{user_id}, #{user_name}, #{user_pwd});
    </insert>

    <update id="updateUser" parameterType="com.lorry.pojo.User">
        update mybatis.user
        set user_name=#{user_name}, user_pwd=#{user_pwd}
        where user_id=#{user_id};
    </update>

    <delete id="deleteUser" parameterType="int">
        delete
        from mybatis.user
        where user_id=#{user_id};
    </delete>
    <!--使用map，传递map中的key, 实体类或数据库里面的表字段或者参数过多，考虑使用map-->
    <insert id="addUser2" parameterType="map">
        insert into mybatis.user(user_id, user_name, user_pwd) value (#{id}, #{name}, #{pwd});
    </insert>

    <select id="getUserById2" parameterType="map" resultType="com.lorry.pojo.User">
        select * from mybatis.user where user_id = #{id} and user_name = #{name}
    </select>
    <select id="getUserLike" resultType="com.lorry.pojo.User">
            select * from mybatis.user where user_name like concat('%',#{name},'%');
    </select>
</mapper>
```

增删改需要提交事务，可以在工具类里面修改默认提交

### 测试

这里主要用到了mybatis的工具类

```java
@Test
public void test(){

    /*
        * 从SqlSession执行sql
        * 先拿到sql
        * UserMapper是UserDao的实现类
        *面向接口编程所以我们们拿到接口“UserDao.class”就可以
        * 返回后就可以执行接口的方法了
        * */
    //第一步获得SqlSession对象
    //这里使用了try-with-resources不必写sqlSession.close();
    try(SqlSession sqlSession = MybatisUtils.getSqlSession()){
        //方式1：执行SQL
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        List<User> userList = mapper.getUserList();
        //方式2：不推荐使用
        //        List<User> userList = sqlSession.selectList("com.lorry.dao.UserDao.getUserList");
        for (User user : userList) {
            //输出的是地址原因是没写toString
            System.out.println(user);
        }
    }
}
```

## Map传递参数

```xml
<!--使用map，传递map中的key, 实体类或数据库里面的表字段或者参数过多，考虑使用map-->
<insert id="addUser2" parameterType="map">
    insert into mybatis.user(user_id, user_name, user_pwd) value (#{id}, #{name}, #{pwd});
</insert>

<select id="getUserById2" parameterType="map" resultType="com.lorry.pojo.User">
    select * from mybatis.user where user_id = #{id} and user_name = #{name}
</select>
```

```java
@Test
public void getUserById2(){
    try(SqlSession sqlSession = MybatisUtils.getSqlSession()){
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        Map<String, Object> map = new HashMap<String,Object>();
        map.put("id",2);
        map.put("name","梅世杰");
        User user = mapper.getUserById2(map);
        System.out.println(user);
    }
}

```

**使用map优点：**如果一个表中的字段非常多而需要的字段只有少数时，new一个实体对象显然很麻烦，需要对所有的字段都进行一次赋值。使用map传值可以更方便的实现需要进行的操作。如：修改用户密码时，只需要传入id和密码即可，而无需new一个用户对象。

## mybatis的模糊查询

### 方式一：

手动添加“%” 通配符

* xml配置：

```xml
<!--模糊查询--><select id="fuzzyQuery" resultType="com.bin.pojo.Book">    select * from mybatis.book where bookName like #{info};</select>
```

```java
@Test
public void fuzzyQuery(){    
    SqlSession sqlSession = MybatisUtils.getSqlSession();    
    BookMapper mapper = sqlSession.getMapper(BookMapper.class);    
    List<Book> books = mapper.fuzzyQuery("%萨%");    
    for (Book book : books) 
    {        
        System.out.println(book);    
    }    
    sqlSession.close();
}
```

**说明：**需要手动添加”%”通配符，显然这种方式很麻烦，并且如果忘记添加通配符的话就会变成普通的查询语句，匹配全部字符查询。

- 缺点：
  - 麻烦
  - 易出错

### 方式二

在xml配置文件中添加”%”通配符，拼接字符串形式

```xml
<select id="fuzzyQuery" resultType="com.bin.pojo.Book">    
    select * from mybatis.book where bookName like '%${info}%';
</select>
```

**说明：**在mapper.xml配置文件中添加”%”通配符，但是需要用单引号将其包裹住，但是用单引号裹住之后#{}就无法被识别，要改成${}这种拼接字符串的形式。虽然通过方式二优化了方式一的缺点，但同时也造成了SQL安全性的问题，也就是用户可以进行SQL注入。

- 缺点：
  - 不安全，可进行SQL注入

### 方式三

在xml配置文件中添加”%”通配符，借助mysql函数

```xml
<select id="fuzzyQuery" resultType="com.bin.pojo.Book">    
    select * from mybatis.book where bookName like concat('%',#{info},'%');</select>
```

**说明：**解决了SQL注入且能在配置文件中写”%”通配符的问题，完美实现了模糊查询

- 优点：
  - 安全
  - 方便

### 方式四

与方式三一样使用mysql函数，但使的用是${}形式，不过需要用单引号包裹住

```xml
<select id="fuzzyQuery" resultType="com.bin.pojo.Book">    
    select * from mybatis.book where bookName like concat('%','${info}','%');</select>
```

#### 总结：

- \#{}是预编译处理，mybatis在处理#{}时，会将其替换成”?”，再调用PreparedStatement的set方法来赋值。
- ${}是拼接字符串，将接收到的参数的内容不加任何修饰的拼接在SQL语句中，会引发SQL注入问题。

# 配置解析

## 核心配置

- MyBatis 的配置文件包含了会深深影响 MyBatis 行为的设置和属性信息

```xml
configuration（配置）
properties（属性）
settings（设置）
typeAliases（类型别名）
typeHandlers（类型处理器）
objectFactory（对象工厂）
plugins（插件）
environments（环境配置）
environment（环境变量）
transactionManager（事务管理器）
dataSource（数据源）
databaseIdProvider（数据库厂商标识）
mappers（映射器）
```

### 环境配置

MyBatis可以配置成适应多种环境

**尽管可以配置多个环境，但每个 SqlSessionFactory 实例只能选择一种环境。**

- Mybatis默认的事务管理器是JDBC
- 连接池：POOLED

```XML
<transactionManager type="JDBC"/>
<dataSource type="POOLED">
```

### 属性(properties)

可以通过properties属性来实现引用配置文件

这些属性可以在外部进行配置，并可以进行动态替换。可以在典型的 Java 属性文件中配置这些属性，也可以在 properties 元素的子元素中设置。【db.properties 】

编写一个db.properties

```xml
driver=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatis?useSSL=true&useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai
username=root
password=123456
```

```xml
<!--引入外部配置文件-->
    <properties resource="db.properties"/>
```

- 可以直接引入外部文件
- 可以在其中增一些属性配置
- 如果两个文件有相同的name字段，优先使用外部配置文件的

### 类型别名(typeAliases)

#### 类取别名

- 类型别名是java类型设置一个短的名字
- 存在的意义仅在于用来减少类完全限定名的冗余

```xml
<!--可以给实体类取别名-->
<typeAliases>   
    <typeAlias type="com.bin.pojo.User" alias="User"/>    
    <typeAlias type="com.bin.pojo.Book" alias="Book"/>
</typeAliases>
```

#### 指定包

也可以指定一个包名，mybatis在包名下面搜索需要的javaBean

没有注解时，它的默认别名为这个类的类名，首字母小写；

```xml
<typeAliases>
    <package name="com.lorry.pojo"/>
</typeAliases>
```

若有注解，则别名为其注解值。

```java
@Alias("author")
public class Author {
    ...
}
```

- 在实体类较少的时候，可以使用给类取别名
- 在实体类十分多时，可以使用指定包

# 生命周期和作用域

![img](/mybatis笔记.assets/kuangstudyd0e12f71-934d-4aff-ac34-f37036d8a50e.png)

生命周期和作用域是至关重要的，因为错误的使用会导致非常严重的并发问题

**SqlSessionFactoryBuilder：**

- 一旦创建SqlSessionFactory就不需要它了
- 局部变量

**SqlSessionFactory：**

- 可以看成数据库连接池
- 一旦被创建就应该在应用运行期间一直存在，**没有任何理由丢弃或者重新创建另一个实例**
- SqlSessionFactory的最佳作用域是应用作用域
- 简单的是使用单例模式或者静态单例模式

**SqlSession：**

- 相当于连接到连接池的一个请求
- SqlSession的实例不是线程安全的，所以不能被共享
- 它的最佳作用域是请求或方法作用域
- 用完之后需关闭，否则会浪费资源

SqlSessionFactory可以创建多个SqlSession，每个sqlSession都可以创建多个Mapper

一个Mapper代表着一个具体的业务

![img](/mybatis笔记.assets/kuangstudye32c7f81-214f-4c63-b370-d8249cdeff5c.png)

# 解决属性名和字段名不一致的问题

数据库中的字段

![img](/mybatis笔记.assets/image-20210725152739364.png)

实体类的属性

```java
private int user_id;
private String user_name;
private String user_password;
```

解决办法：

1. 起别名

   ```sql
   select user_id,user_name,user_pwd as user_password from mybatis.user where user_id = #{id}
   ```

2. ***resultmap***

结果集映射

```xml
<!--结果集映射-->
    <resultMap id="UserMap" type="user">
        <!--column数据库中的字段，property实体类中的属性-->
<!--        <result column="user_id" property="user_id"/>-->
<!--        <result column="user_name" property="user_name"/>-->
        <result column="user_pwd" property="user_password"/>
    </resultMap>
    <select id="getUserById" parameterType="int" resultMap="UserMap">
           select * from mybatis.user where user_id = #{id}
    </select>
```

- `resultMap` 元素是 MyBatis 中最重要最强大的元素
- ResultMap 的设计思想是
  - 对简单的语句做到零配置
  - 对于复杂一点的语句，只需要描述语句之间的关系就行了
- 一样的字段可以不做映射

# 日志

## 日志工程

日志是查找错误的好助手

- SLF4J
- ==LOG4J==⭐
- LOG4J2
- JDK_LOGGING
- COMMONS_LOGGING
  - ==STDOUT_LOGGING==⭐ 标准日志输出
- NO_LOGGING

在mybatis中具体使用哪一个日志实现，在mybatis核心配置文件中设置

```xml
<settings>
    <!--标准的日志工厂实现-->
    <setting name="logImpl" value="STDOUT_LOGGING"/>
    <!--        <setting name="logImpl" value="Log4J"/>-->
</settings>
```

## LOG4J

- LOG4J是Apache的一个开源项目，通过使用Log4j，我们可以控制日志信息输送的目的地是控制台、文件、GUI组件
- 可以控制每一条日志的输出格式
- 通过定义每一条日志信息的级别，能够更加细致地控制日志的生成过程
- 通过一个配置文件来灵活地进行配置，而不需要修改应用的代码



1. 导入LOG4J的包

```xml
<!-- https://mvnrepository.com/artifact/log4j/log4j -->
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

2. log4j.properties配置

```xml
#将等级为DEBUG的日志信息输出到console和file这两个目的地，console和file的定义在下面的代码
log4j.rootLogger=DEBUG,console,file
#控制台输出的相关设置
log4j.appender.console = org.apache.log4j.ConsoleAppender
log4j.appender.console.Target = System.out
log4j.appender.console.Threshold=DEBUG
log4j.appender.console.layout = org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=[%c]-%m%n
#文件输出的相关设置
log4j.appender.file = org.apache.log4j.RollingFileAppender
log4j.appender.file.File=./log/lorry.log
log4j.appender.file.MaxFileSize=10mb
log4j.appender.file.Threshold=DEBUG
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=[%p][%d{yy-MM-dd}][%c]%m%n
#日志输出级别
log4j.logger.org.mybatis=DEBUG
log4j.logger.java.sql=DEBUG
log4j.logger.java.sql.Statement=DEBUG
log4j.logger.java.sql.ResultSet=DEBUG
log4j.logger.java.sql.PreparedStatement=DEBUG
```

3. 配置log4j为日志实现

```xml
<settings>
    <!--标准的日志工厂实现-->
    <!--        <setting name="logImpl" value="STDOUT_LOGGING"/>-->
    <setting name="logImpl" value="Log4J"/>
</settings>
```

### 简单实用

1. 在使用Log4j的类中，导入包import org.apache.log4j.Logger;
2. 由于多个类会使用，设置为静态

![img](/mybatis笔记.assets/image-20210725163324898.png)

```java
logger.info("info:进入了testLog4j");
logger.debug("debug:进入了testLog4j");
logger.error("error:进入了testLog4j");
```

# 使用mybatis实现分页

## 使用Limit分页（代码在mybatis_04）

1. 接口

```java
    //分页
    List<User> getUserByLimit(Map<String,Integer> map);
```

2. Mapper.xml

```xml
<!--//分页-->
    <select id="getUserByLimit" parameterType="map" resultType="User">
        select * from mybatis.user limit #{startIndex},#{endIndex}
    </select>
```

3. 测试

```JAVA
@Test
public void getUserByLimit(){
    try(SqlSession sqlSession = MybatisUtils.getSqlSession()){
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        Map<String, Integer> map = new HashMap<String,Integer>();
        map.put("startIndex",1);
        map.put("endIndex",4);
        List<User> userList = mapper.getUserByLimit(map);
        for (User user : userList) {
            logger.info("分页查询用户：" + user);
        }
    }
}
```

## RewBounds分页

1. 接口

```java
// 分页2
List<User> gteUserByRowBounds();
```

2. UserMapper.xml

```java
<!--//分页2-->
    <select id="getUserByRowBounds" parameterType="map" resultMap="UserMap">
    select * from mybatis.user
    </select>
```

3. 测试

```java
@Test
public void getUserByRowBounds(){
    try(SqlSession sqlSession = MybatisUtils.getSqlSession()){
        RowBounds rowBounds = new RowBounds(1, 4);
        //通过java代码层面实现分页
        List<User> userList = sqlSession.selectList("com.lorry.mapper.UserMapper.getUserByRowBounds",null,rowBounds);
        for (User user : userList) {
            System.out.println(user);
        }
    }
}
```

## 分页插件

# 使用注解开发

### 面向接口编程

之前学到的都是面向对象编程，也学习过接口，但在真正开发中，很多时候我们会选择面向接口编程；
**根本原因 ： 解耦、可扩展、提高复用，分层开发中，上层不用管具体的实现，大家都遵守共同的标准，使得开发变得更容易，规范性更好**。
在一个面向对象的系统中，系统的各种功能是由许许多多的不同对象协作完成的，在这种情况下，各个对象内部是如何实现的，对系统设计人员来讲是不那么重要的；
而各个对象之间的协作关系则成为系统设计的关键。小到不同类之间的通信，大到各模块之间的交互，在系统设计之初都是要着重考虑的，这也是系统设计的主要工作内容。面向接口编程就是按照这种思想来编程。

**关于接口的理解**

- 接口从更深层次的理解，应是定义（规范，约束）与实现（名实分离的原则）的分离。
- 接口的本身反映了系统设计人员对系统的抽象理解。
- 接口应有两类：
- 第一类是对一个个体的抽象，它可对应为一个抽象体(abstract class)；
- 第二类是对一个个体某一方面的抽象，即形成一个抽象面（interface）；
- 一个体有可能有多个抽象面。抽象体与抽象面是有区别的。

**三个面向区别**

- 面向对象是指，我们考虑问题时，以对象为单位，考虑它的属性及方法 .
- 面向过程是指，我们考虑问题时，以一个具体的流程（事务过程）为单位，考虑它的实现 .
- 接口设计与非接口设计是针对复用技术而言的，与面向对象（过程）不是一个问题.更多的体现就是对系统整体的架构

## 使用注解开发

1. 注解在接口上实现

```java
public interface TeacherMapper {
    @Select("select * from teacher where id = #{tid}")
    Teacher getTeacher(@Param("tid") int id);
}
```

2. 在mybatis-config.xml中绑定接口

```xml
<mappers>
    <mapper class="com.lorry.mapper.UserMapper"/>
</mappers>
```

3. 测试

```java
@Test
public void getUsers(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    List<User> users = mapper.getUsers();
    for (User user : users) {
        logger.info("使用注解查询：" + user);
    }
}
```

本质：反射机制实现

底层：动态代理

## CRUD

在工具类创建的时候可以实现自动提交

```java
publicstaticSqlSession  getSqlSession(){
    
    return sqlSessionFactory.openSession(true);
}
```

1. 编写接口

```java
public interface UserMapper02 {
    @Select("select * from user")
    List<User> getUsers();
    @Select("select * from user where id = #{id}")
    List<User> getUserById(@Param("id") int id);
    @Insert("insert into user(id,name,pwd) values(#{id},#{name},#{pwd})")
    int addUser(User user);
    @Update("update user set name=#{name},pwd=#{pwd} where id=#{id}")
    int updateUser(User user);
    @Delete("delete from user where id = #{id}")
    int deleteUser(@Param("id") int id);
}
```

2. 将接口注册绑定到mybatis-config文件中

   ```xml
   <mapper class="com.lorry.mapper.UserMapper02"/>
   ```

   关于[@Param](https://github.com/Param)()注解

   - 基本类型的参数或String类型需要使用
   - 引用类型不需要
   - 只有一个基本类型可以忽略不写，但建议写上
   - SQL中引用的是[@Param](https://github.com/Param)()注解中设定的属性名

   \#{}和${}的区别：

   - \#{} 预编译，可避免SQL注入
   - ${} 拼接字符串，可以被进行SQL注入

# Lombok

1. 在IDEA中安装Lombok插件

![ima](/mybatis笔记.assets/image-20210725195905638.png)



2. 在项目中导入Lombok的jar包

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.20</version>
    <scope>provided</scope>
</dependency>
```

3. 常用的注释

- [@Data](https://github.com/Data) 无参构造、get、set、toString、equals、hashcode
- [@AllArgsConstructor](https://github.com/AllArgsConstructor) 有参构造
- [@NoArgsConstructor](https://github.com/NoArgsConstructor) 无参构造

# 多对一处理

## 编写SQL，创建数据库表

```sql
CREATE TABLE `teacher` (
  `id` INT(10) NOT NULL,
  `name` VARCHAR(30) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8
INSERT INTO teacher(`id`, `name`) VALUES (1, '秦老师'); 
CREATE TABLE `student` (
  `id` INT(10) NOT NULL,
  `name` VARCHAR(30) DEFAULT NULL,
  `tid` INT(10) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `fktid` (`tid`),
  CONSTRAINT `fktid` FOREIGN KEY (`tid`) REFERENCES `teacher` (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8 
INSERT INTO `student` (`id`, `name`, `tid`) VALUES (1, '小明', 1); 
INSERT INTO `student` (`id`, `name`, `tid`) VALUES (2, '小红', 1); 
INSERT INTO `student` (`id`, `name`, `tid`) VALUES (3, '小张', 1); 
INSERT INTO `student` (`id`, `name`, `tid`) VALUES (4, '小李', 1); 
INSERT INTO `student` (`id`, `name`, `tid`) VALUES (5, '小王', 1);
```

1. 导入lombok
2. 创建javaBean(即实体类)
3. 建立Mapper接口
4. 建立Mapper.xml文件
5. 在核心配置文件中绑定mapper接口或配置文件
6. 测试查询

## 按照查询嵌套处理

```xml
    <select id="getStudent" resultMap="StudentTeacher">
        select * from Student
    </select>
    <resultMap id="StudentTeacher" type="Student">
        <result property="id" column="id"/>
        <result property="name" column="name"/>
        <!--多对一处理，1。查询出所有的学生信息 2。根据查询出来的学生的tid，寻找对应的老师-->
        <!--复杂的属性，我们需要单独处理-->
        <!--对象就用 association-->
        <!--集合使用 collection-->
        <!--复杂的属性，我们需要单独处理，我们用association来操作对象，并加了javaType和select 用select来找Teacher-->
        <!--用这个column tid去查老师-->
        <association property="teacher" column="tid" javaType="teacher" select="getTeacher"/>
    </resultMap>
    <select id="getTeacher" resultType="Teacher">
        select * from teacher where id=#{tid}
    </select>
<--属性中的引用对象单独处理，对象：association，集合：collection-->		
```

## 按照结果嵌套处理

```xml
<select id="getStudentList" resultMap="StudentsTeacher2">
    select s.id as sid,s.name as sname,s.tid as tid, t.name as tname
    from mybatis.student s
    join mybatis.teacher t
    on s.tid = t.id;
</select>
<resultMap id="StudentsTeacher2" type="student">
    <result property="id" column="sid"/>
    <result property="name" column="sname"/>
    <association property="teacher" javaType="teacher">
        <result property="id" column="tid"/>
        <result property="name" column="tname"/>
    </association>
</resultMap>
```

多对一查询方式：

- 子查询
- 联表查询

# 一对多处理

1. 创建javaBean

```java
public class Student {
    private int id;
    private String name;
    private int tid;
}
public class Teacher {
    private int id;
    private String name;
    private List<Student> students;
}
```

## 按照查询嵌套处理(子查询)

```xml
    <select id="getTeacher2" resultMap="TeacherStudent2">
        select * from  teacher where id=#{tid}
    </select>
    <resultMap id="TeacherStudent2" type="Teacher">
        <!--一样的省略-->
        <collection property="students"  javaType="ArrayList" ofType="Student" column="id" select="getStudentByTeacherId"/>
    </resultMap>

    <select id="getStudentByTeacherId" resultType="Student">
        <!--这里#{里面的可以随便写，第一个SQL的不行}-->
        select * from student where tid=#{kkkk}
    </select>
```

1. 关联（对象）：association 【多对一】
2. 集合：collection 【一对多】
3. javaType 用来指定实体类中的属性类型
4. ofType 用来指定映射到List或集合中的pojo类型，泛型中约束的类型

**注意点：**

- 保证SQL可读性，尽量保证通俗易懂
- 注意一对多和多对一中属性名和字段的问题
- 若错误不好排查，可以使用日志，建议使用Log4j

**面试高频：**

- MySQL引擎
- innoDB底层原理
- 索引
- 索引优化

# 动态SQL

**动态SQL是根据不同条件来生成不同的SQL语句**

## 编写SQL，创建数据库表

```sql
CREATE TABLE `blog`(
`id` VARCHAR(50) NOT NULL COMMENT '博客id',
`title` VARCHAR(100) NOT NULL COMMENT '博客标题',
`author` VARCHAR(30) NOT NULL COMMENT '博客作者',
`create_time` DATETIME NOT NULL COMMENT '创建时间',
`views` INT(30) NOT NULL COMMENT '浏览量'
)ENGINE=INNODB DEFAULT CHARSET=utf8;
```

## 测试环境搭建

1. 导包
2. 创建javaBean

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Blog {
    private String id;
    private String title;
    private String author;
    private Date createTime;
    private int views;
}
```

3. 建立Mapper接口

```java
public interface BlogMapper {
    //插入数据
    int addBook(Blog blog);

    //查询博客
    List<Blog> queryBlogIF(Map map);

    //
    List<Blog> queryBlogChoose(Map map);

    int updateBlog(Map map);

    //查询第1-2-3号记录的博客
    List<Blog> queryBlogForeach(Map map);
}

```

4. 建立Mapper.xml文件夹
5. 在mybatis-config文件中绑定mapper接口或配置文件
6. 测试查询

## IF

```xml
<select id="queryBlogIF" parameterType="map" resultType="blog">
    select * from mybatis.blog
    <where>
        <if test="title != null">
            title = #{title}
        </if>
        <if test="author != null">
            and author = #{author}
        </if>
    </where>
</select>
```

常用的部分可以作为SQL片段 在需要使用的地方使用include标签引用

```xml
<sql id="if-title-author">
    <if test="title != null">
        title = #{title}
    </if>
    <if test="author != null">
        and author = #{author}
    </if>
</sql>
```

```xml
    <select id="queryBlogIF" parameterType="map" resultType="blog">
<!--        select * from blog-->
<!--        <where>-->
<!--            <if test="title != null">-->
<!--                title = #{title}-->
<!--            </if>-->
<!--            <if test="author != null">-->
<!--                and author = #{author}-->
<!--            </if>-->
<!--        </where>-->
        select * from blog
        <where>
            <include refid="if-title-author"></include>
        </where>

    </select>
```

## choose

```xml
    <select id="queryBlogChoose" parameterType="map" resultType="blog">

        select * from blog
        <where>
            <choose>
                <when test="title != null">
                    title = #{title}
                </when>
                <when test="author != null">
                    and author = #{author}
                </when>
                <otherwise>
                    and views = #{views}
                </otherwise>
            </choose>
        </where>

    </select>

```

## set

```xml
<update id="updateBlog" parameterType="map">
    update blog
    <set>
        <if test="title != null">
            title = #{title},
        </if>
        <if test="author != null">
            author = #{author}
        </if>
    </set>
    where id = #{id}
</update>
<!--我们现在传递一个万能的map，map里面放一个集合
select * from blog where 1=1 and (id=1 or id=2 or id=3)
-->
```

## Foreach

```xml
<select id="queryBlogForeach" parameterType="map" resultType="blog">
    select * from mybatis.blog
    <where>
        <!--
        id in
        <foreach collection="ids" item="id" open="(" separator="," close=")">
            #{id}
        </foreach>
-->
        <foreach collection="ids" item="id"  separator="or">
            id = #{id}
        </foreach>
    </where>
</select>
```

两种foreach SQL语句的区别

select * from mybatis.blog WHERE id in ( ? , ? , ? )

select * from mybatis.blog WHERE id = ? or id = ? or id = ?

## trim

```xml
<update id="updateBlogTrim" parameterType="map">
    update mybatis.blog
    <trim prefix="set" suffixOverrides=",">
        <if test="title != null">
            title = #{title},
        </if>
        <if test="author != null">
            author = #{author},
        </if>
        <if test="views != null">
            views = #{views}
        </if>
    </trim>
    where id = #{id};
</update>
<select id="queryBlogTrim" parameterType="map" resultType="blog">
    select * from mybatis.blog
    <trim prefix="WHERE" prefixOverrides="AND |OR ">
        <if test="title != null">
            title = #{title}
        </if>
        <if test="author != null">
            and author = #{author}
        </if>
    </trim>
</select>
```

**总结：**

where元素在子元素存在（有值）的情况下才插入 “WHERE” 子句。而且，若子句的开头为 “AND” 或 “OR”，where 元素也会将它们去除。

注意事项：

- 最好基于单表来定义sql片段
- 不要存在where标签

# 缓存

**查询：**连接数据库，耗资源！一次查询的结果，给他暂存在一个可以直接取到的地方！—>内存：缓存我们再次查询相同数据的时候，直接走缓存，就不用走数据库了

1. 什么是缓存 [ Cache ]？
   - 存在内存中的临时数据。
   - 将用户经常查询的数据放在缓存（内存）中，用户去查询数据就不用从磁盘上(关系型数据库数据文件)查询，从缓存中查询，从而提高查询效率，解决了高并发系统的性能问题。
2. 为什么使用缓存？
   - 减少和数据库的交互次数，减少系统开销，提高系统效率。
3. 什么样的数据能使用缓存？
   - 经常查询并且不经常改变的数据。【可以使用缓存】

## mybatis缓存

- MyBatis包含一个非常强大的查询缓存特性，它可以非常方便地定制和配置缓存。缓存可以极大的提升查询效率。

- MyBatis系统中默认定义了两级缓存：

  一级缓存和二级缓存

  - 默认情况下，只有一级缓存开启。（SqlSession级别的缓存，也称为本地缓存）
  - 二级缓存需要手动开启和配置，他是基于namespace级别的缓存。
  - 为了提高扩展性，MyBatis定义了缓存接口Cache。我们可以通过实现Cache接口来自定义二级缓存

## 一级缓存

- 一级缓存也叫本地缓存： SqlSession
  - 与数据库同一次会话期间查询到的数据会放在本地缓存中。
  - 以后如果需要获取相同的数据，直接从缓存中拿，没必须再去查询数据库；

测试步骤：

1. 开启日志！
2. 测试在一个Sesion中查询两次相同记录
3. 查看日志输出

缓存失效的情况：

1. 查询不同的东西

```java
@Test
public void queryUserById(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    User user = mapper.queryUserById(1);
    logger.info("一级缓存：" + user);
    User user1 = mapper.queryUserById(2);
    logger.info("一级缓存：" + user1);
    logger.info("一级缓存：" + (user == user1));
    sqlSession.close();
}
```

![img](/mybatis笔记.assets/kuangstudyd485aa2d-ce15-4227-9902-251ebc718db3.png)

2. 增删改操作，可能会改变原来的数据，所以必定会刷新缓存！

```java
@Test
public void queryUserById(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    User user = mapper.queryUserById(1);
    logger.info("一级缓存：" + user);
    mapper.updateUser(new User(2,"张三","123456"));
    User user1 = mapper.queryUserById(1);
    logger.info("一级缓存：" + user1);
    logger.info("一级缓存：" + (user == user1));
    sqlSession.close();
}
```

![img](/mybatis笔记.assets/kuangstudyf53fb35b-30dd-47c8-9c53-b613cd05d222.png)

3. 查询不同的Mapper.xml
4. 手动清理缓存！

```java
@Test
public void queryUserById(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    User user = mapper.queryUserById(1);
    logger.info("一级缓存：" + user);
    // 清除缓存
    sqlSession.clearCache();
    User user1 = mapper.queryUserById(1);
    logger.info("一级缓存：" + user1);
    logger.info("一级缓存：" + (user == user1));
    sqlSession.close();
}
```

![img](/mybatis笔记.assets/kuangstudy60d048f1-349a-475e-9baf-b00a4858e68b.png)

## 二级缓存

- 二级缓存也叫全局缓存，一级缓存作用域太低了，所以诞生了二级缓存
- 基于namespace级别的缓存，一个名称空间，对应一个二级缓存；
- 工作机制
  - 一个会话查询一条数据，这个数据就会被放在当前会话的一级缓存中；
  - 如果当前会话关闭了，这个会话对应的一级缓存就没了；但是我们想要的是，会话关闭了，一级缓存中的数据被保存到二级缓存中；
  - 新的会话查询信息，就可以从二级缓存中获取内容；
  - 不同的mapper查出的数据会放在自己对应的缓存（map）中；

步骤：

1. 开启全局缓存

```xml
<!--显示的开启全局缓存-->
<setting name="cacheEnabled" value="true"/>
```

2. 在要使用二级缓存的Mapper中开启

```xml
<!--在当前mapper.xml中使用二级缓存-->
<cache/>
```

3. 自定义参数

```xml
<!--在当前Mapper.xml中使用二级缓存-->
<cache
       eviction="FIFO"
       flushInterval="60000"
       size="512"
       readOnly="true"/>
```

4. 测试

```java
@Test
public void queryUserById(){
SqlSession sqlSession1 = MybatisUtils.getSqlSession();
SqlSession sqlSession2 = MybatisUtils.getSqlSession();
UserMapper mapper1 = sqlSession1.getMapper(UserMapper.class);
UserMapper mapper2 = sqlSession2.getMapper(UserMapper.class);
User user1 = mapper1.queryUserById(1);
logger.info("一级缓存：" + user1);
sqlSession1.close();
User user2 = mapper2.queryUserById(1);
logger.info("一级缓存：" + user1);
logger.info("一级缓存：" + (user1 == user2));
sqlSession2.close();
}
```

![img](/mybatis笔记.assets/kuangstudy58c20070-fffe-4753-aace-3fdd84e7e05a.png)

**问题：**我们需要将实体类序列化！否则就会报错！

```java
Caused by: java.io.NotSerializableException: com.bin.pojo.User
```

小结：

- 只要开启了二级缓存，在同一个Mapper下就有效
- 所有的数据都会先放在一级缓存中；
- 只有当会话提交，或者关闭的时候，才会提交到二级缓冲中！

## 缓存原理

![img](/mybatis笔记.assets/kuangstudy7f3f8e72-a720-489f-a8b9-b7258aa72cdc-1627263018627.png)

## 自定义缓存-ehcache

```xml
Ehcache是一种广泛使用的开源Java分布式缓存。主要面向通用缓存`
```

先要导包！

```xml
<!-- https://mvnrepository.com/artifact/org.mybatis.caches/mybatis-ehcache -->
<dependency>
    <groupId>org.mybatis.caches</groupId>
    <artifactId>mybatis-ehcache</artifactId>
    <version>1.1.0</version>
</dependency>
```

在mapper中指定使用我们的ehcache缓存实现！

```xml
<!--在当前Mapper.xml中使用二级缓存-->
<cache type="org.mybatis.caches.ehcache.EhcacheCache"/>
```

# **Mybatis详细的执行流程！**⭐


