## 说明

## 目录

## mybatis简介

### 传统jdbc工具和hibernate

![1567324061167](images/1567324061167.png)

### mybatis

![1567324342328](images/1567324342328.png)

### 为何要使用mybatis

> - MyBatis是一个半自动化的持久化层框架
> - JDBC
>   - SQL夹在Java代码块里，耦合度高导致硬编码内伤
>   - 维护不易且实际开发需求中sql是有变化，频繁修改的情况多见
> - Hibernate和JPA
>   - 长难复杂SQL，对于Hibernate而言处理也不容易
>   - 内部自动生产的SQL，不容易做特殊优化
>   - 基于全映射的全自动框架，大量字段的POJO进行部分映射时比较困难。导致数据库性能下降
> - 对开发人员而言，核心sql还是需要自己优化
> - **sql和java编码分开，功能边界清晰，一个专注业务、一个专注数据**

## 下载

> mybatis已经迁移到github,下载地址
>
> [mybatis3](https://github.com/mybatis/mybatis-3/)

## HelloWorld

### 创建一个测试表employee

```sql
create table employee
(
    id        int          null,
    last_name varchar(255) null,
    gender    char         null,
    email     varchar(255) null
);
```

![1567328768703](images/1567328768703.png)

### 创建一个maven工程

```xml
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>

    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.4.1</version>
    </dependency>

    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.17</version>
    </dependency>

    <dependency>
      <groupId>log4j</groupId>
      <artifactId>log4j</artifactId>
      <version>1.2.17</version>
    </dependency>

  </dependencies>
```

![1567328921726](images/1567328921726.png)

### 创建对应的javabean

```java
package com.sugar.bean;

public class Employee {
    private int id;
    private String lastName;
    private String gender;
    private String email;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
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
        return "Employee{" +
                "id=" + id +
                ", lastName='" + lastName + '\'' +
                ", gender='" + gender + '\'' +
                ", email='" + email + '\'' +
                '}';
    }
}
```

### 创建mybatis主配置文件mybatis-config.xml

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
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis_test?useSSL=false&amp;serverTimezone=UTC"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
    <!-- 将我们写好的sql映射文件（EmployeeMapper.xml）一定要注册到全局配置文件（mybatis-config.xml）中 -->
    <mappers>
        <mapper resource="conf/employeeMapper.xml"/>
    </mappers>
</configuration>
```

### 创建sql映射文件employeeMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.sugar.dao.EmployeeMapper">
    <!--
    namespace:名称空间;指定为接口的全类名
    id：唯一标识
    resultType：返回值类型
    #{id}：从传递过来的参数中取出id值，可以防止sql注入

    public Employee getEmpById(Integer id);
     -->
    <select id="getEmpById" resultType="com.sugar.bean.Employee">
		select id,last_name lastName,email,gender from employee where id = #{id}
	</select>
</mapper>
```

### 添加log4j配置文件log4j.xml

> 添加后可以在控制台看到mybatis发送的sql语句

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">

<log4j:configuration xmlns:log4j="http://jakarta.apache.org/log4j/">

    <appender name="STDOUT" class="org.apache.log4j.ConsoleAppender">
        <param name="Encoding" value="UTF-8"/>
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%-5p %d{MM-dd HH:mm:ss,SSS} %m  (%F:%L) \n"/>
        </layout>
    </appender>
    <logger name="java.sql">
        <level value="debug"/>
    </logger>
    <logger name="org.apache.ibatis">
        <level value="info"/>
    </logger>
    <root>
        <level value="debug"/>
        <appender-ref ref="STDOUT"/>
    </root>
</log4j:configuration>
```

### 创建测试类

```java
public class App {
    public static void main(String[] args) throws IOException {
        String resource = "conf/mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        try (SqlSession session = sqlSessionFactory.openSession()) {
            Employee employee = session.selectOne("com.sugar.dao.EmployeeMapper.getEmpById", 1);
            System.out.println(employee);
        }
    }
}
```

### 总结

> 1. 根据xml配置文件（全局配置文件）创建一个SqlSessionFactory对象 有数据源一些运行环境信息
> 2. sql映射文件；配置了每一个sql，以及sql的封装规则等。
> 3. 将sql映射文件注册在全局配置文件中
> 4. 写代码
>    - 根据全局配置文件得到SqlSessionFactory；
>    - 使用sqlSession工厂，获取到sqlSession对象使用他来执行增删改查
>    - 一个sqlSession就是代表和数据库的一次会话，用完关闭
>    - 使用sql的唯一标志来告诉MyBatis执行哪个sql。sql都是保存在sql映射文件中的

## HelloWorld-接口式编程

> 1. 配置文件和接口动态绑定，通过namespace属性值指定，值为接口全类名
> 2. sql和方法进行绑定，id值为方法名即可绑定
> 3. sqlSession.getMapper方法获取对应接口的代理对象

### 创建EmployeeMapper接口

```java
public interface EmployeeMapper {

    Employee getEmpById(Integer id);
}
```

### 修改employeeMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.sugar.dao.EmployeeMapper">
    <!--
    namespace:名称空间;指定为接口的全类名
    id：唯一标识
    resultType：返回值类型
    #{id}：从传递过来的参数中取出id值

    public Employee getEmpById(Integer id);
     -->
    <select id="getEmpById" resultType="com.sugar.bean.Employee">
		select id,last_name lastName,email,gender from employee where id = #{id}
	</select>
</mapper>
```

### 测试

```java
    @Test
    public void testMybatisInteface() throws IOException {
        String resource = "conf/mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        EmployeeMapper employeeMapper = sqlSession.getMapper(EmployeeMapper.class);
        Employee employee = employeeMapper.getEmpById(1);
        System.out.println(employee);
        sqlSession.close();
    }
```

## SqlSession

> - SqlSession代表和数据库的一次会话，用完必须关闭
> - SqlSession和connection一样是非线程安全的，不能把它作为成员变量，每次使用都需要获取一个新的
> - sqlSession.getMapper(EmployeeMapper.class)，mybatis会根据配置文件为接口生成一个代理对象

## mybatis全局配置文件

![1567333727049](images/1567333727049.png)

### properties属性引入外部配置文件

> - 作用：可以引入外部properties配置文件，如数据库配置
>
> - properties有2个属性：resource、url
>
>   - resource：用于引入类路径下资源
>   - url：用于引入网络资源或磁盘文件系统下的资源
>
> ```
> <properties resource="" url=""></properties>
> ```
> - 如果属性在不只一个地方进行了配置，那么MyBatis 将按照下面的顺序来加载
>
>   - 在properties 元素体内指定的属性首先被读取,即子标签property设置的值
>
>     ```xml
>         <properties resource="conf/dbconfig.properties">
>             <property name="" value=""/>
>         </properties>
>     ```
>
>   - 然后根据properties 元素中的resource 属性读取类路径下属性文件或根据url 属性指定的路径读取属性文件，并覆盖已读取的同名属性。
>
>   - 最后读取作为方法参数传递的属性，并覆盖已读取的同名属性

### settings设置

> - 这是MyBatis 中极为重要的调整设置，它们会改变MyBatis 的运行时行
>
>   ![1567334780824](images/1567334780824.png)
>
> - 如上设置可以将数据库中使用下划线分割的字段和javabean的驼峰命名字段进行映射

### typeAliases别名处理器

> - 类型别名是为Java 类型设置一个短的名字，可以方便我们引用某个类,别名默认为类名
>
> - **注意：别名不区分大小写，别名冲突就会报错**
>
>   ```xml
>       <typeAliases>
>           <typeAlias type="com.sugar.bean.Employee" alias="employee"/>
>       </typeAliases>
>   ```
>
> - **typeAlias**：子标签可以为某个类指定别名
>
> - **package**：子标签可以为某个包下的所有类批量起别名
>
> - **@Alias注解**：也可以起别名
>
> - 推荐使用全类名，因为可以直接定位到目标类
>
> - mybatis默认起的一些别名
>
>   ![1567336276465](images/1567336276465.png)

### typeHandlers类型处理器

> - 无论是MyBatis 在预处理语句（PreparedStatement）中设置一个参数时，还是从结果集中取出一个值时，都会用类型处理器将获取的值以合适的方式转换成Java 类型
>
>   ![1567338067077](images/1567338067077.png)
>
> - 日期类型的处理
>
>   - 日期和时间的处理，JDK1.8以前一直是个头疼的问题。我们通常使用JSR310规范领导者Stephen Colebourne创建的Joda-Time来操作。1.8已经实现全部的JSR310规范了。
>
>   - 日期时间处理上，我们可以使用MyBatis基于JSR310（Date and Time API）编写的各种日期时间类型处理器
>
>   - MyBatis3.4以前的版本需要我们手动注册这些处理器，以后的版本都是自动注册的
>
>    ![1567338149822](images/1567338149822.png)
>
> - 自定义类型处理器
>
>   - 我们可以重写类型处理器或创建自己的类型处理器来处理不支持的或非标准的类型
>
>   - 步骤：
>
>     > 1. 实现org.apache.ibatis.type.TypeHandler接口或者继承org.apache.ibatis.type.BaseTypeHandler
>     > 2. 指定其映射某个JDBC类型（可选操作）
>     > 3. 在mybatis全局配置文件中注册

### plugins插件

> - 插件是MyBatis提供的一个非常强大的机制，我们可以通过插件来修改MyBatis的一些核心行为。插件通过动态代理机制，可以介入四大对象的任何一个方法的执行。后面会有专门的章节我们来介绍mybatis运行原理以及插件
> - Executor(update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)
> - ParameterHandler(getParameterObject, setParameters)
> - ResultSetHandler(handleResultSets, handleOutputParameters)
> - StatementHandler(prepare, parameterize, batch, update, query)

### environments环境

> - MyBatis可以配置多种环境，比如开发、测试和生产环境需要有不同的配置。
> - 每种环境使用一个environment标签进行配置并指定唯一标识符
> - 可以通过environments标签中的default属性指定一个环境的标识符来快速的切换环境

![1567339673786](images/1567339673786.png)

#### transactionManager

> type：JDBC | MANAGED | 自定义
>
> - JDBC：使用了JDBC 的提交和回滚设置，依赖于从数据源得到的连接来管理事务范围。JdbcTransactionFactory
> - MANAGED：不提交或回滚一个连接、让容器来管理事务的整个生命周期（比如J2EE 应用服务器的上下文）。ManagedTransactionFactory
> - 自定义：实现TransactionFactory接口，type=全类名/别名

#### dataSource

> type：UNPOOLED | POOLED | JNDI | 自定义
>
> - UNPOOLED：不使用连接池，UnpooledDataSourceFactory
> - POOLED：使用连接池，PooledDataSourceFactory
> - JNDI：在EJB 或应用服务器这类容器中查找指定的数据源
> - 自定义：实现DataSourceFactory接口，定义数据源的获取方式
>
> **实际开发中我们使用Spring管理数据源，并进行事务控制的配置来覆盖上述配置**

### databaseIdProvider环境

> - 可以和environments配合使用，做到动态切换数据库
>
> - MyBatis 可以根据不同的数据库厂商执行不同的语句，为不同的厂商起别名如下，也可以不起，则别名为name值
>
>   ![1567436854779](images/1567436854779.png)
>
> - Type：DB_VENDOR
>
>   > - 使用MyBatis提供的VendorDatabaseIdProvider解析数据库厂商标识（根据不同的驱动判断）。也可以实现DatabaseIdProvider接口来自定义
>   > - 会通过DatabaseMetaData#getDatabaseProductName()返回的字符串进行设置。由于通常情况下这个字符串都非常长而且相同产品的不同版本会返回不同的值，所以最好通过设置属性别名来使其变短
>
> - Property-name：数据库厂商标识
>
> - Property-value：为标识起一个别名，方便SQL语句使用databaseId属性引用，在sql映射文件中可以使用databaseId进行引用，如：
>
>   ![1567437020462](images/1567437020462.png)
>
> - MyBatis匹配规则如下：
>
>   > - 如果没有配置databaseIdProvider标签，那么databaseId=null
>   > - 如果配置了databaseIdProvider标签，使用标签配置的name去匹配数据库信息，匹配上设置databaseId=配置指定的值，否则依旧为null
>   > - 如果databaseId不为null，他只会找到配置databaseId的sql语句
>   > - MyBatis 会加载不带databaseId属性和带有匹配当前数据库databaseId 属性的所有语句。如果同时找到带有databaseId 和不带databaseId 的相同语句，则后者会被舍弃，mybatis会使用更精确的
>
>   

### mapper映射

#### 逐个注册SQL映射文件

> - mapper逐个注册SQL映射文件
>
>   > - resource:用于引用类路径下资源
>   >
>   > - url:网络资源或磁盘文件资源
>   >
>   > - class：引用或注册接口，值为**接口全类名**
>   >
>   >   - **若有sql映射文件**，接口和映射文件必须放在同一目录下，并且除后缀名称必须相同，否则mybatis不知道接口和哪个配置文件进行映射
>   >   - **若没有sql映射文件**，可以使用基于注解的方式指定sql
>   >
>   >   ```java
>   >   public interface EmployeeMapper2 {
>   >       @Select("select * from employee where id = #{id}")
>   >       Employee getEmpById(Integer id);
>   >   }
>   >   ```
>
>   ![1567515308900](images/1567515308900.png)

#### 批量注册

> - 这种方式要求SQL映射文件名必须和接口名相同并且在同一目录下,或者使用基于注解的方式
> - 为了将sql配置文件和java代码分开，可以建立2个根目录，并在2个目录下创建同样的包
>
> ```xml
>     <mappers>
>         <package name="com.sugar.dao"/>
>     </mappers>
> ```

## MyBatis-映射文件

### 作用

> 映射文件指导着MyBatis如何进行数据库增删改查，有着非常重要的意义

### 常用标签

> - cache –命名空间的二级缓存配置
> - cache-ref –其他命名空间缓存配置的引用。
> - resultMap–自定义结果集映射
> - parameterMap –已废弃！老式风格的参数映射
> - sql –抽取可重用语句块。
> - insert –映射插入语句
> - update –映射更新语句
> - delete –映射删除语句
> - select –映射查询语句