---
title: Mybatis
date: 2024-03-06
update: 2024-03-15

tags:
categories:
  - ORM框架
keywords:
description: 主要关于Mybatis框架的使用和工作流程
top_img:
cover: https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_09_21_14_50.png
---

# MyBatis3.5.16

[官方文档](https://mybatis.org/mybatis-3/zh_CN/configuration.html#databaseIdProvider)	|	[github](https://github.com/mybatis)	|	[github mybatis-spring-boot-starter(mybatis-spring >3 版本才适配 SpringBoot3)](https://github.com/mybatis/spring/releases/tag/mybatis-spring-3.0.2) 



## SpringBoot整合Mybatis

导入Maven依赖

```pom
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>3.0.3</version>
</dependency>
```

context配置文件

```yml
spring:
  datasource:
    url:   # 要设置时区
    username: root
    password: 123
    driver-class-name: com.mysql.cj.jdbc.Driver  # Spring Boot 2.1.x默认使用了MySQL 8.0的驱动，所以这里要使用新的驱动jar包
```

在 SpringBoot 项目的入口类 SpringBootApplication 上添加注解@MapperScan

```java
@MapperScan(basePackages = {"com.chinasofti.usermanager.mapper"})
```



## 解决IntelliJ中Mapper警告无法自动注入

原因：MyBatis 的 Mapper 对象是在运行时动态生成并注入 IOC 容器的，IDE 编译时无法检测

解决：

0不解决也能运行

1(推荐)

```java
@Service
@RequiredArgsConstructor(onConstructor = @__(@Autowired))
public class TestService {
    private final UserMapper userMapper;
    ...
}

// Lombok生成的代码：
@Service
public class TestService {
    private final UserMapper userMapper;
    @Autowired
    public TestService(final UserMapper userMapper) {
        this.userMapper = userMapper;
    }
    ...
}
```

2 `@Resource` 替换 `@Autowired`
3 `@Autowired(required = false)`

4 在Mapper接口上加上 `@Repository` 注解(以下不推荐)
5 关 IntelliJ 警告
6 IntelliJ的mybatis plugin

[Intellij IDEA中Mybatis Mapper自动注入警告的6种解决方案(已吸收)](https://www.imooc.com/article/287865)



## Mybatis的工作流程

SqlSessionFactory读取配置，创建SqlSession读取Mapper来执行SQL

![1608443422128](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_09_21_14_50.png)

### 构建SqlSessionFactory单例 

核心SqlSessionFactory，全局唯一，用于1读取配置文件，2初始化mybatis框架，3创建sqlSession(完成CURD)

```java
package com.imooc.mybatis.utils;
// MyBatisUtils工具类,创建全局唯一的SqlSessionFactory对象
public class MyBatisUtils {
    private static SqlSessionFactory sqlSessionFactory = null;    
    static {  //利用静态块在初始化类时实例化sqlSessionFactory
        Reader reader = null;
        try {
            // 从XML的Mybatis配置文件中构建SqlSessionFactory
            reader = Resources.getResourceAsReader("mybatis-config.xml");  // 工具类Resources，用于从classpath或InputStream加载资源文件
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);            
            // 如果是Java的mybatis配置文件
            // SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(configuration);
        } catch (IOException e) {
            e.printStackTrace();            
            throw new ExceptionInInitializerError(e);  //初始化错误时,通过抛出异常ExceptionInInitializerError通知调用者
    }	}

    // 获取一个新的SqlSession对象
    public static SqlSession openSession(){
        //默认SqlSession会自动提交事务数据(commit)，设置false代表关闭自动提交，改为手动提交事务数据
        return sqlSessionFactory.openSession(false);
    }

    // 释放一个有效的SqlSession对象
    public static void closeSession(SqlSession session){
        if(session != null){
            session.close();
}	}	}
```







#### 配置文件

不用深入了解Mybatis自己的配置方式，反正都通过SpringBoot配置

```bash
resources
|-mappers
|	|-goods.xml
|	\_goods_detail.xml
\_mybatis.config.xml  # 配置文件
```



##### XML

>mybatis-config.xml内标签
>configuration（配置）
>	typeAliases（类型别名）
>	properties（属性）
>	settings（设置）	
>	typeHandlers（类型处理器）
>	objectFactory（对象工厂）
>	plugins（插件）
>	environments（环境配置）
>		environment（环境变量）
>			transactionManager（事务管理器）
>			dataSource（数据源）
>	databaseIdProvider（数据库厂商标识）
>	mappers（映射器）

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>   
    <typeAliases>  <!-- 用来减少类完全限定名的冗余 -->
        <typeAlias alias="Blog" type="domain.blog.Blog"/>
        <package name="domain.blog"/>  <!-- 包内类，首字母小写的非限定类名作为别名 -->
    </typeAliases>
    
    <properties resource="org/mybatis/example/config.properties">
      <!-- 启用默认值特性 -->
      <property name="org.apache.ibatis.parsing.PropertyParser.enable-default-value" value="true"/> 
      <property name="username" value="dev_user"/>
      <property name="password" value="F2Fa3!33TYyg"/>
    </properties>
    <dataSource type="POOLED">
      <property name="driver" value="${driver}"/>
      <property name="url" value="${url}"/>
      <property name="username" value="${username:ut_user}"/>  <!-- 如果属性username没有被配置，则username属性的值将为ut_user -->
      <property name="password" value="${password}"/>
    </dataSource>

    <settings>
        <!-- goods_id ==> goodsId 驼峰命名转换 -->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
    
    <typeHandlers>  <!-- typeHandlers：用于决定将值转换成的Java类型 -->
		<typeHandler handler="org.mybatis.example.ExampleTypeHandler"/>
    </typeHandlers>
    
    <objectFactory type="org.mybatis.example.ExampleObjectFactory">  <!-- ObjectFactory：用于目标类的实例化 -->
		<property name="someProperty" value="100"/>
    </objectFactory>

    <plugins>
        <plugin interceptor="org.mybatis.example.ExamplePlugin">
            <property name="someProperty" value="100"/>
        </plugin>
    </plugins>

    
	<environments default="dev">  <!--环境默认为dev 指向默认的数据库-->        
        <environment id="dev">
            <transactionManager type="JDBC"></transactionManager>  <!-- Spring项目不需要配置该项，使用自带的管理器 -->
            <!-- JDBC：采用JDBC方式对数据库进行commit/rollback,，它依赖于从数据源得到的连接来管理事务作用域 -->  
            
            <dataSource type="POOLED">  <!-- POOLED(sqlSession.close()将connection回收到连接池而不是直接close())、UNPOOLED -->
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    
        <!--prd生产环境-->
        <environment id="prd">
            <transactionManager type="JDBC"></transactionManager>
            <dataSource type="POOLED">  
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
    
    <!-- 数据厂商标识：MyBatis会加载不带databaseId属性和带有匹配当前数据库databaseId属性的所有语句，以至于可以根据不同的数据库厂商执行不同的语句 -->
    <databaseIdProvider type="DB_VENDOR" />
    
    <mappers>    	
        <mapper resource="mappers/goods.xml"/>
        <mapper resource="mappers/goods_detail.xml"/>        
        <mapper url="file:///var/mappers/PostMapper.xml"/>
        <mapper class="org.mybatis.builder.PostMapper"/>
        <package name="org.mybatis.builder"/>
    </mappers>
</configuration>
```



##### Java

```java
// 创建环境，需要id和事务管理器以及数据源
TransactionFactory transactionFactory = new JdbcTransactionFactory();
DataSource dataSource = BlogDataSourceFactory.getBlogDataSource();
Environment environment = new Environment("development", transactionFactory, dataSource);
Configuration configuration = new Configuration(environment);

// configuration添加映射类(mapper class)，注解方式映射
configuration.addMapper(BlogMapper.class);
// MyBatis 会自动查找并加载和BlogMapper同名的配置文件（先选择BlogMapper.xml后BlogMapper.class）
```

```java
@Alias("author")
public class Author {
    ...
}
```

```java
// 自定义TypeHandler
@MappedJdbcTypes(JdbcType.VARCHAR)
public class ExampleTypeHandler extends BaseTypeHandler<String> {

    public void setNonNullParameter(PreparedStatement ps, int i, String parameter, JdbcType jdbcType) throws SQLException {
    	ps.setString(i, parameter);
    }

    public String getNullableResult(ResultSet rs, String columnName) throws SQLException {
    	return rs.getString(columnName);
    }

    public String getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
    	return rs.getString(columnIndex);
    }
    
    public String getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
    	return cs.getString(columnIndex);
}	}

public class GenericTypeHandler<E extends MyObject> extends BaseTypeHandler<E> {
	private Class<E> type;

    public GenericTypeHandler(Class<E> type) {
        if (type == null) throw new IllegalArgumentException("Type argument cannot be null");
        this.type = type;
	}
    ...
}
```

```java
// 自定义ObjectFactory
public class ExampleObjectFactory extends DefaultObjectFactory {
    public Object create(Class type) {
    	return super.create(type);
    }
    public Object create(Class type, List<Class> constructorArgTypes, List<Object> constructorArgs) {
    	return super.create(type, constructorArgTypes, constructorArgs);
    }
    public void setProperties(Properties properties) {
    	super.setProperties(properties);
    }
    public <T> boolean isCollection(Class<T> type) {
    	return Collection.class.isAssignableFrom(type);
}	}
```

```java
// 实现Interceptor接口的插件，可以在SQL语句执行过程中的某一点进行拦截
@Intercepts({@Signature(  // 拦截在Executor实例中所有的"update"方法调用， 这里的Executor是负责执行底层映射语句的内部对象
    type= Executor.class,
    method = "update",
    args = {MappedStatement.class,Object.class})})
public class ExamplePlugin implements Interceptor {
    private Properties properties = new Properties();
    
    public Object intercept(Invocation invocation) throws Throwable {
    	// implement pre processing if need
        Object returnObject = invocation.proceed();
        // implement post processing if need
        return returnObject;
    }
    
    public void setProperties(Properties properties) {
    	this.properties = properties;
}	}
```



### SqlSession 类持久化操作

从SqlSessionFactory中获取SqlSession，从SqlSession中获取Mapper接口实例，Mapper实例提供了在数据库执行SQL命令所需的所有方法

```java
try (SqlSession session = sqlSessionFactory.openSession()) {
      BlogMapper mapper = session.getMapper(BlogMapper.class); // 可以是class类，也可以是xml
      Blog blog = mapper.selectBlog(101);
}
```



### 注解定义SQL语句(重要)

一些高级SQL(嵌套联合SQL等)仍然需要使用XML定义

示例：

```java
public interface BlogMapper {
  @Select("SELECT * FROM blog WHERE id = #{id}")
  Blog selectBlog(int id);
}
```

| 注解                                                         | 元注解Target                   | 释义                                                         | 成员变量                                                     |
| ------------------------------------------------------------ | ------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| @MapperScan(basePackages = {"com.chinasofti.usermanager.mapper"}) |                                |                                                              |                                                              |
|                                                              |                                |                                                              |                                                              |
| @Mapper                                                      | TYPE、METHOD、FIELD、PARAMETER | 标记一个接口为 MyBatis 的 Mapper 接口                        |                                                              |
| @Select("SELECT * FROM USER WHERE NAME = #{name} LIMIT 1")   | METHOD                         | 定义select语句                                               | SQL语句(value)                                               |
| @Insert("INSERT INTO USER(NAME, AGE) VALUES(#{name}, #{age})") | METHOD                         | 定义insert语句                                               | SQL语句(value)                                               |
| @Update("UPDATE user SET age=#{age} WHERE name=#{name}")     | METHOD                         | 定义update语句                                               | SQL语句(value)                                               |
| @Delete("DELETE FROM user WHERE id =#{id}")                  | METHOD                         | 定义delete语句                                               | SQL语句(value)                                               |
| @Param                                                       |                                | 定义方法参数与 SQL 语句中的参数映射，用于传参                |                                                              |
| @Results                                                     |                                | 定义结果映射关系<br />@Results({@Result(property = "id", column = "user_id"), @Result(property = "name", column = "user_name")}) |                                                              |
| @Result                                                      |                                | 定义单个字段与Java对象属性的映射关系                         |                                                              |
| @Options                                                     | METHOD                         | 定义默认行为，常配合@Insert使用<br />[等价XML标签](./Mybatis.md#selectKey标签与useGeneratedKeys属性的区别) | useGeneratedKeys使用JDBC3.0支持的自动生成主键特性<br />keyProperty包含键值的属性名 |



### XML定义SQL语句(映射文件)

只要会理解和编写基本的 MyBatis XML 配置。当你遇到具体需求时，可以再查阅官方文档或参考资料，深入了解更多标签和属性的细节。

```bash
resources
|-mappers
|	|-goods.xml
|	\_goods_detail.xml
\_mybatis.config.xml  # 配置文件
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
```



#### XML标签

| XML标签    | 释义                                                         | 属性                                                         |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **mapper** | 最外层的标签                                                 | namespace="org.mybatis.example.XXMapper"可以直接映射到映射器类XXMapper.class |
|            |                                                              |                                                              |
| **cache**  | 对给定命名空间的二级缓存缓存配置                             |                                                              |
| cache-ref  | 对其他命名空间缓存配置的引用                                 |                                                              |
|            |                                                              |                                                              |
| **select** | 映射查询语句                                                 | parameterType表示#{xx}参数类型(不写也没事，很多时候 MyBatis 可以自己推断出来)<br />resultType结果集类型，可以是类完全限定命、hashmap<br />resultMap指定结果映射 |
| resultMap  | 是最复杂也是最强大的元素，用来描述如何从数据库结果集中来加载对象。 |                                                              |
|            |                                                              |                                                              |
| where      |                                                              |                                                              |
| if         |                                                              |                                                              |
| foreach    | 用于多行操作                                                 |                                                              |
|            |                                                              |                                                              |
| **insert** | 映射插入语句                                                 |                                                              |
| selectKey  | 回填自增主键<br />支持“自增主键”类型的数据库可以使用useGeneratedKeys="true"属性 |                                                              |
|            |                                                              |                                                              |
| **update** | 映射更新语句                                                 |                                                              |
|            |                                                              |                                                              |
| **delete** | 映射删除语句                                                 |                                                              |
|            |                                                              |                                                              |
| **sql**    | 定义可被其他SQL语句引用的可重用SQL语句块。                   |                                                              |
| include    | 引用用SQL语句块                                              |                                                              |
| property   | 配合占位符使用                                               | name形参名字                                                 |



| 标签介绍                            | 释义                                                         | 属性 |
| ----------------------------------- | ------------------------------------------------------------ | ---- |
| resultMap                           | id、type。可以将查询结果映射为复杂类型的Data Transfer Object对象<br />ResultMap可以将嵌套的结果集映射到一个合适的**对象树**中，作为使用额外 select 语句的替代方案，将多表连接操作的结果映射成一个单一的 ResultSet。<br />这样的 ResultSet 将会将包含重复或部分数据重复的结果集。为了将结果集正确地映射到嵌套的对象树中，MyBatis 允许你 **“串联”结果映射**，以便解决嵌套结果集的问题。想了解更多内容，请参考下面的关联元素。 |      |
|                                     |                                                              |      |
| constructor                         | 用于在实例化类时，注入结果到构造方法中                       |      |
| idArg                               | column、javaType。将被注入到构造方法的一个普通结果，会标识作为 ID 的结果以提高整体性能。 |      |
| arg                                 | column、javaType。将被注入到构造方法的一个普通结果           |      |
|                                     |                                                              |      |
| **结果映射最基本的内容**            | id 和 result 元素都将一个字段的值映射到一个简单数据类型（String, int, double, Date 等） |      |
| id                                  | property、column。会标识对象属性，在比较对象实例和进行缓存以及嵌套结果映射（也就是连接映射）的时候能提高整体的性能<br />你应该总是指定一个或多个可以唯一标识结果的属性。 虽然即使不指定这个属性，MyBatis 仍然可以工作，但是会产生严重的性能问题。 |      |
| result                              | property、column。                                           |      |
|                                     |                                                              |      |
| association                         | property、javaType。一个**复杂类型的串联**；许多结果将包装成这种类型 |      |
| 嵌套ResultMap（嵌套id和result标签） | 关联本身可以是一个 resultMap 元素，或者从别处引用一个        |      |
|                                     |                                                              |      |
| collection                          | 一个复杂类型的集合，**并联**                                 |      |
| 嵌套ResultMap（嵌套id和result标签） | 关联本身可以是一个 resultMap 元素，或者从别处引用一个        |      |
|                                     |                                                              |      |
| discriminator                       | **辨别器**，使用结果值来决定使用哪个 resultMap，作用类似switch语句 |      |
| case                                | 基于某些值的结果映射                                         |      |
| 嵌套ResultMap（嵌套id和result标签） | case 本身可以是一个 resultMap 元素，因此可以具有相同的结构和元素，或者从别处引用一个 |      |



#### 属性

| 属性介绍    | 释义                                                         |
| ----------- | ------------------------------------------------------------ |
| property    | 映射到列结果的属性。把JavaBean的属性和property配对，如果不存在再配对字段名，配对成功那么它将会被使用。 可以使用点式分隔形式(“address.street.number”)进行复杂属性映射。 |
| column      | 数据库中的字段名。一般情况下，这和JDBC中传递给 resultSet.getString(columnName) 方法的参数一样。 |
|             |                                                              |
| javaType    | 一个 Java 类的完全限定名或类型别名。 如果你映射到一个 JavaBean，MyBatis 通常可以推断类型。然而，如果你映射到的是 HashMap，那么你应该明确地指定 javaType 来保证行为与期望的相一致。 |
|             |                                                              |
| jdbcType    | JDBC 类型，所支持的 JDBC 类型参见这个表格之后的“支持的 JDBC 类型”。 只需要在可能执行插入、更新和删除的且允许空值的列上指定 JDBC 类型。这是 JDBC 的要求而非 MyBatis 的要求。如果你直接面向 JDBC 编程，你需要对可能存在空值的列指定这个类型。 |
| typeHandler | 我们在前面讨论过默认的类型处理器。使用这个属性，你可以覆盖默认的类型处理器。 这个属性值是一个类型处理器实现类的完全限定名，或者是类型别名。 |



| 属性      | 释义                                                         |
| --------- | ------------------------------------------------------------ |
| column    | 数据库中的列名，或者是列的别名。一般情况下，这和传递给 resultSet.getString(columnName) 方法的参数一样。 注意：在使用复合主键的时候，你可以使用 column="{prop1=col1,prop2=col2}" 这样的语法来指定多个传递给嵌套 Select 查询语句的列名。这会使得 prop1 和 prop2 作为参数对象，被设置为对应嵌套 Select 语句的参数。 |
| select    | 用于加载复杂类型属性的映射语句的 ID，它会从 column 属性指定的列中检索数据，作为参数传递给目标 select 语句。 具体请参考下面的例子。注意：在使用复合主键的时候，你可以使用 column="{prop1=col1,prop2=col2}" 这样的语法来指定多个传递给嵌套 Select 查询语句的列名。这会使得 prop1 和 prop2 作为参数对象，被设置为对应嵌套 Select 语句的参数。 |
| fetchType | lazy、eager。指定属性后，将在映射中忽略全局配置参数 lazyLoadingEnabled，使用属性的值。<br />Mybatis联合查询的延迟加载机制：在主查询时不立即加载所有关联对象，而是在访问关联对象时触发关联查询 |

| 属性          | 释义                                                         |
| ------------- | ------------------------------------------------------------ |
| resultMap     | 结果映射的 ID，可以将此关联的嵌套结果集映射到一个合适的对象树中。 它可以作为使用额外 select 语句的替代方案。它可以将多表连接操作的结果映射成一个单一的 ResultSet。这样的 ResultSet 有部分数据是重复的。 为了将结果集正确地映射到嵌套的对象树中, MyBatis 允许你“串联”结果映射，以便解决嵌套结果集的问题。使用嵌套结果映射的一个例子在表格以后。 |
| columnPrefix  | 当连接多个表时，你可能会不得不使用列别名来避免在 ResultSet 中产生重复的列名。指定 columnPrefix 列名前缀允许你将带有这些前缀的列映射到一个外部的结果映射中。 详细说明请参考后面的例子。 |
| notNullColumn | 默认情况下，在至少一个被映射到属性的列不为空时，子对象才会被创建。 你可以在这个属性上指定非空的列来改变默认行为，指定后，Mybatis 将只在这些列非空时才创建一个子对象。可以使用逗号分隔来指定多个列。默认值：未设置（unset）。 |
| autoMapping   | 如果设置这个属性，MyBatis 将会为本结果映射开启或者关闭自动映射。 这个属性会覆盖全局的属性 autoMappingBehavior。注意，本属性对外部的结果映射无效，所以不能搭配 select 或 resultMap 元素使用。默认值：未设置（unset）。 |

| 属性          | 释义                                                         |
| ------------- | ------------------------------------------------------------ |
| column        | 当使用多个结果集时，该属性指定结果集中用于与 foreignColumn 匹配的列（多个列名以逗号隔开），以识别关系中的父类型与子类型 |
| foreignColumn | 指定外键对应的列名，指定的列将与父类型中 column 的给出的列进行匹配。 |
| resultSet     | 指定用于加载复杂类型的结果集名字。                           |
|               |                                                              |
|               |                                                              |
|               |                                                              |

| 属性        | 值                                                           | 释义                                                         |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ofType      |                                                              | 用来将 JavaBean（或字段）属性的类型和集合存储的类型区分开来。 |
| autoMapping | NONE - 禁用自动映射。仅对手动映射的属性进行映射。<br/>PARTIAL - 对除在内部定义了嵌套结果映射（也就是连接的属性）以外的属性进行映射<br/>FULL - 自动映射所有属性。 | MyBatis 会获取结果中返回的列名并在 Java 类中查找相同名字的属性（忽略大小写） |
|             |                                                              |                                                              |



#### mapper标签

```xml
<mapper namespace="org.mybatis.example.XXMapper">
    ...
</mapper>
```



#### select标签

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="org.mybatis.example.XXMapper">
    <cache eviction="LRU" flushInterval="600000" size="512" readOnly="true"/>
    
    <!-- 结果类型：List<Map> -->
    <select id="selectPersonById" parameterType="int" resultType="hashmap">
  		SELECT * FROM PERSON WHERE ID = #{id}
	</select>  
    
    <!-- 结果类型：List<Goods> -->
    <select id="selectAll" resultType="com.zjbti.mybatis.entity.Goods" useCache="false">
        select * from t_goods order by goods_id desc limit 10
    </select>
    
    <!-- 结果类型：List<Goods> -->
    <select id="selectByPriceRange" parameterType="java.util.Map" resultType="com.zjbti.mybatis.entity.Goods">
        select * from t_goods
        where
          current_price between #{min} and #{max}
        order by current_price
        limit 0,#{limt}
    </select>
    
    <!-- 结果类型：List<Map> -->
    <select id="selectGoodMap" resultType="java.util.LinkedHashMap">  <!-- 因为HashMap是乱序的，所以查询出来不是select的顺序，可以使用LinkedHashMap使其有序 -->
        select g.*,c.category_name,'1' as test from t_goods g,t_category c
        where g.category_id=c.category_id
	</select>
    
    <select id="selectPage" resultType="com.zjbti.mybatis.entity.Goods">
        select * from t_goods where current_price &lt; 1000
    </select>
</mapper>
```

```java
@Test public void testSelectByPriceRange() throws Exception {
    SqlSession session=null;
    try {
        session = MyBatisUtils.openSession();
        Map param = new HashMap();
        param.put("min",100);
        param.put("max", 500);
        param.put("limt", 10);
        XXMapper mapper = session.getMapper("XXMapper.class");
        List<Goods> list = mapper.selectByPriceRange(param);
        // List<Goods> list = session.selectList("org.mybatis.example.XXMapper.selectByPriceRange", param);  // 通过命名空间和mapper Select id找到对应的dao对象，如果mapper Select id是全局唯一的也可以不用命名空间
        for (Goods g : list) {
            System.out.println(g.getTitle()+":"+g.getCurrentPrice());
        }
    } catch (Exception e) {
        e.printStackTrace();
        throw e;
    }finally {
        MyBatisUtils.closeSession(session);
}	}
```



##### 关联对象

关联对象的1+N查询问题：比如查询文章列表和文章作者信息，触发1(文章列表)+N(作者信息)次查询
解决办法：使用嵌套查询、延迟加载



###### 嵌套Select

通过执行另外一个 SQL 映射语句来加载期望的复杂类型

```xml
<resultMap id="blogResult" type="Blog">
  <association property="author" column="author_id" javaType="Author" select="selectAuthor"/>
</resultMap>
<select id="selectBlog" resultMap="blogResult">
  SELECT * FROM BLOG WHERE ID = #{id}
</select>

<select id="selectAuthor" resultType="Author">
  SELECT * FROM AUTHOR WHERE ID = #{id}
</select>
```



###### 嵌套ResultMap

使用嵌套的结果映射来处理连接结果的重复子集

```xml
<resultMap id="authorResult" type="Author">  <!-- 可重用 -->
  <id property="id" column="author_id"/>
  <result property="username" column="author_username"/>
  <result property="password" column="author_password"/>
  <result property="email" column="author_email"/>
  <result property="bio" column="author_bio"/>
</resultMap>

<resultMap id="blogResult" type="Blog">  <!-- 嵌套结果映射 -->
  <id property="id" column="blog_id" />
  <result property="title" column="blog_title"/>
  <association property="author" column="blog_author_id" javaType="Author" resultMap="authorResult"/>
</resultMap>

<!-- 等价 -->
<resultMap id="blogResult" type="Blog">
  <id property="id" column="blog_id" />
  <result property="title" column="blog_title"/>
  <association property="author" javaType="Author">
    <id property="id" column="author_id"/>
    <result property="username" column="author_username"/>
    <result property="password" column="author_password"/>
    <result property="email" column="author_email"/>
    <result property="bio" column="author_bio"/>
  </association>
</resultMap>

<select id="selectBlog" resultMap="blogResult">
  select
    B.id            as blog_id,
    B.title         as blog_title,
    B.author_id     as blog_author_id,
    A.id            as author_id,
    A.username      as author_username,
    A.password      as author_password,
    A.email         as author_email,
    A.bio           as author_bio
  from Blog B left outer join Author A on B.author_id = A.id
  where B.id = #{id}
</select>
```

```xml
<resultMap id="blogResult" type="Blog">
  <id property="id" column="blog_id" />
  <result property="title" column="blog_title"/>
  <association property="author"
    resultMap="authorResult" />
  <association property="coAuthor"  <!-- 共同作者 -->
    resultMap="authorResult"
    columnPrefix="co_" />  <!-- 由于结果中的列名(co_author_id)与resultMap中的列名(author_id)不同。你需要指定 columnPrefix 以便重复使用该结果映射来映射 co_author 的结果。 -->
</resultMap>

<select id="selectBlog" resultMap="blogResult">
  select
    B.id            as blog_id,
    B.title         as blog_title,
    A.id            as author_id,
    A.username      as author_username,
    A.password      as author_password,
    A.email         as author_email,
    A.bio           as author_bio,
    CA.id           as co_author_id,
    CA.username     as co_author_username,
    CA.password     as co_author_password,
    CA.email        as co_author_email,
    CA.bio          as co_author_bio
  from Blog B
  left outer join Author A on B.author_id = A.id
  left outer join Author CA on B.co_author_id = CA.id
  where B.id = #{id}
</select>
```



##### ResultMap标签

ResultMap结果映射，可以将查询结果映射为复杂类型的java对象DTO

```bash
dto  # Data Transfer Object是对实体类扩展的对象，不能对实体类进行修改，因为实体类必须对应数据库
\_GoodsDTO
entity  # 实体类
|-Category
|-Goods
\_GoodsDetail
```

```java
@Getter @Setter
public class GoodsDTO {
    private Goods goods=new Goods();
    private Category category;
    private String test;
}
```

```xml
<!-- 返回多对象 映射为DTO -->
<resultMap id="rmGoods" type="com.zjbti.mybatis.dto.GoodsDTO">
    
    <!-- id表示匹配主键字段，必写 -->
    <id property="goods.goodsId" column="goods_id"></id>
    <!-- 表示为GoodsDTO的属性goods对象的goodsId属性，赋值查询goods_id的结果 -->
    <result property="goods.title" column="title"></result>
    <result property="goods.originalCost" column="original_cost"></result>
    <result property="goods.currentPrice" column="current_price"></result>
    <result property="goods.discount" column="discount"></result>
    <result property="goods.isFreeDelivery" column="is_free_delivery"></result>

    <result property="goods.categoryId" column="category_id"></result>
    <result property="category.categoryId" column="category_id"></result>
    <result property="category.categoryName" column="category_name"></result>

    <result property="category.parentId" column="parent_id"></result>
    <result property="category.categoryLevel" column="category_level"></result>

    <result property="category.categoryOrder" column="category_order"></result>

    <result property="category.categoryName" column="category_name"></result>
    <result property="test" column="test"/>
</resultMap>
<!-- 结果类型：List<GoodsDTO> -->
<select id="selectGoodsDTO" resultMap="rmGoods">
    select g.*,c.*,'1' as test from t_goods g,t_category c
    where g.category_id=c.category_id
</select>

<!-- 一对多，type是一 -->
<resultMap id="rmGoods1" type="com.zjbti.mybatis.entity.Goods">
    <id column="goods_id" property="goodsId"/>
    <collection property="goodsDetails" select="goodsDetail.selectByGoodsId" column="goods_id"/>
</resultMap>
<!-- 表示从表第0个开始，只取1个-->
<select id="selectOneToMany" resultMap="rmGoods1">
    select * from t_goods limit 0,1
</select>
```
```java
@Test public void testSelectGoodsDTO() throws Exception {
    SqlSession session=null;
    try {
        session = MyBatisUtils.openSession();
        XXMapper mapper = session.getMapper(XXMapper.class);
        List<GoodsDTO> list = mapper.selectGoodsDTO();
        // List<GoodsDTO> list = session.selectList("org.mybatis.example.XXMapper.selectGoodsDTO");
        for (GoodsDTO g:list) {
            System.out.println(g.getGoods().getTitle());
        }
    } catch (Exception e) {
        e.printStackTrace();
        throw e;
    }finally {
        MyBatisUtils.closeSession(session);
}	}
```


###### constructor标签



###### collection标签、association标签

```xml
<collection property="posts" ofType="domain.blog.Post">
  <id property="id" column="post_id"/>
  <result property="subject" column="post_subject"/>
  <result property="body" column="post_body"/>
</collection>
```

```xml
<resultMap id="blogResult" type="Blog">
  <collection property="posts" javaType="ArrayList" column="id" ofType="Post" select="selectPostsForBlog"/>
</resultMap>

<select id="selectBlog" resultMap="blogResult">
  SELECT * FROM BLOG WHERE ID = #{id}
</select>

<select id="selectPostsForBlog" resultType="Post">
  SELECT * FROM POST WHERE BLOG_ID = #{id}
</select>
```

```xml
<resultMap id="blogResult" type="Blog">
  <id property="id" column="blog_id" />
  <result property="title" column="blog_title"/>
  <collection property="posts" ofType="Post">
    <id property="id" column="post_id"/>
    <result property="subject" column="post_subject"/>
    <result property="body" column="post_body"/>
  </collection>
</resultMap>
<resultMap id="blogResult" type="Blog">
  <id property="id" column="blog_id" />
  <result property="title" column="blog_title"/>
  <collection property="posts" ofType="Post" resultMap="blogPostResult" columnPrefix="post_"/>
</resultMap>

<resultMap id="blogPostResult" type="Post">
  <id property="id" column="id"/>
  <result property="subject" column="subject"/>
  <result property="body" column="body"/>
</resultMap>

<select id="selectBlog" resultMap="blogResult">
  select
  B.id as blog_id,
  B.title as blog_title,
  B.author_id as blog_author_id,
  P.id as post_id,
  P.subject as post_subject,
  P.body as post_body,
  from Blog B
  left outer join Post P on B.id = P.blog_id
  where B.id = #{id}
</select>
```



###### resultSet属性

```xml
<resultMap id="blogResult" type="Blog">
  <id property="id" column="id" />
  <result property="title" column="title"/>
  <collection property="posts" ofType="Post" resultSet="posts" column="id" foreignColumn="blog_id">
    <id property="id" column="id"/>
    <result property="subject" column="subject"/>
    <result property="body" column="body"/>
  </collection>
</resultMap>

<select id="selectBlog" resultSets="blogs,posts" resultMap="blogResult">
  {call getBlogsAndPosts(#{id,jdbcType=INTEGER,mode=IN})}
</select>
```



###### autoMapping属性

```xml
<resultMap id="userResultMap" type="User" autoMapping="false">
  <result property="password" column="hashed_password"/>
</resultMap>
```



###### discriminator鉴别器标签

```xml
<discriminator javaType="int" column="draft">
  <case value="1" resultType="DraftPost"/>
</discriminator>

<resultMap id="vehicleResult" type="Vehicle">
  <id property="id" column="id" />
  <result property="vin" column="vin"/>
  <result property="year" column="year"/>
  <result property="make" column="make"/>
  <result property="model" column="model"/>
  <result property="color" column="color"/>
  <discriminator javaType="int" column="vehicle_type">  <!-- 如果不能匹配任何一个 case，MyBatis 就只会使用鉴别器块外定义的结果映射 -->
    <case value="1" resultMap="carResult"/>
    <case value="2" resultMap="truckResult"/>
    <case value="3" resultMap="vanResult"/>
    <case value="4" resultMap="suvResult"/>
  </discriminator>
</resultMap>
```



##### 复杂ResultMap例子

constructor标签、association标签、collection标签

```xml
<!-- mybatis-config.xml 中 -->
<typeAlias type="com.someapp.model.User" alias="User"/>  <!-- 可以为类的完全限定名取别名 -->
```

```xml
<!-- XXMapper.xml 中 -->
<!-- 普通方式 -->
<select id="selectUsers" resultType="User">
  select id, userName, hashedPassword
  from some_table
  where id = #{id}
</select>

<!-- as方式 -->
<select id="selectUsers" resultType="User">
  select
    user_id             as "id",
    user_name           as "userName",
    hashed_password     as "hashedPassword"
  from some_table
  where id = #{id}
</select>

<!-- 普通ResultMap方式 -->
<resultMap id="userResultMap" type="User">
  <id property="id" column="user_id" />
  <result property="username" column="user_name"/>
  <result property="password" column="hashed_password"/>
</resultMap>
<select id="selectUsers" resultMap="userResultMap">
  select user_id, user_name, hashed_password
  from some_table
  where id = #{id}
</select>

<!-- 非常复杂的结果映射 -->
<resultMap id="detailedBlogResultMap" type="Blog">
  <constructor>
    <idArg column="blog_id" javaType="int"/>
  </constructor>
  <result property="title" column="blog_title"/>
  <association property="author" javaType="Author">
    <id property="id" column="author_id"/>
    <result property="username" column="author_username"/>
    <result property="password" column="author_password"/>
    <result property="email" column="author_email"/>
    <result property="bio" column="author_bio"/>
    <result property="favouriteSection" column="author_favourite_section"/>
  </association>
  <collection property="posts" ofType="Post">
    <id property="id" column="post_id"/>
    <result property="subject" column="post_subject"/>
    <association property="author" javaType="Author"/>
    <collection property="comments" ofType="Comment">
      <id property="id" column="comment_id"/>
    </collection>
    <collection property="tags" ofType="Tag" >
      <id property="id" column="tag_id"/>
    </collection>
    <discriminator javaType="int" column="draft">
      <case value="1" resultType="DraftPost"/>
    </discriminator>
  </collection>
</resultMap>
```



#### insert标签

```xml
<insert id="insert" parameterType="com.zjbti.mybatis.entity.Goods" flushCache="true">
    INSERT into t_goods(title, sub_title, original_cost, current_price, discount, is_free_delivery)
    values (#{title},#{subTitle},#{originalCost},#{currentPrice},#{discount},#{isFreeDelivery})
    
    <!-- 数据库主键自增的，回填到DAO对象 -->
    <!--<selectKey resultType="Integer" keyProperty="goodsId" order="AFTER">-->
      <!--select last_insert_id()-->
    <!--</selectKey>-->
</insert>
```
```java
@Test public void testInsert() throws Exception {
    SqlSession session=null;
    try {
        session = MyBatisUtils.openSession();
        Goods goods=new Goods();
        goods.setTitle("测试商品");
        goods.setSubTitle("测试子标题");
        goods.setOriginalCost(200f);
        goods.setCurrentPrice(100f);
        goods.setDiscount(0.5f);
        goods.setIsFreeDelivery(1);
        goods.setCategoryId(43);
        XXMapper mapper = session.getMapper(XXMapper.class);
        int num = mapper.insert2(goods);
        // int num = session.insert("goods.insert2", goods);
        session.commit();
        System.out.println(goods.getGoodsId());
    } catch (Exception e) {
        if(session!=null)
            session.rollback();
        e.printStackTrace();
        throw e;
    }finally {
        MyBatisUtils.closeSession(session);
}	}
```

##### selectKey标签与useGeneratedKeys属性的区别

```xml
<insert id="insert2"
        parameterType="com.zjbti.mybatis.entity.Goods"
        
        useGeneratedKeys="true"
        keyProperty="goodsId"
        keyColumn="goods_id">
    INSERT into t_goods(title, sub_title, original_cost, current_price, discount, is_free_delivery)
    values (#{title},#{subTitle},#{originalCost},#{currentPrice},#{discount},#{isFreeDelivery})
</insert>
```
区别：
selectKey标签需要明确编写获取最新主键的SQL语句，适用于所有的关系型数据库
useGeneratedKeys属性会自动根据驱动生成对应SQL语句，只支持“自增主键”类型的数据库，Oracle没自增主键

```xml
<!-- Oracle数据库selectKey回填自增主键写法 -->
<insert id="insert2"
        parameterType="com.zjbti.mybatis.entity.Goods">
	SQL语句
    <selectKey resultType="Integer" order="BEFORE" keyProperty="goodsId">
    	SELECT seq_goods.nextval as id from dual
    </selectKey>
</insert>
```



#### update标签

```xml
<update id="update" parameterType="com.zjbti.mybatis.entity.Goods">
    update t_goods
    set
      title=#{title},
      sub_title=#{subTitle},
      original_cost=#{originalCost},
      current_price=#{currentPrice},
      discount=#{discount},
      is_free_delivery=#{isFreeDelivery},
      category_id=#{categoryId}
    where
      goods_id=#{goodsId}
</update>
```
```java
@Test public void testUpdate() throws Exception {
    SqlSession session=null;
    try {
        session = MyBatisUtils.openSession();
        Goods goods=session.selectOne("goods.selectById",739);
        goods.setTitle("更新测试商品");
        XXMapper mapper = session.getMapper(XXMapper.class);
        int num = mapper.update(goods);
        // int num = session.update("goods.update", goods);
        session.commit();
    } catch (Exception e) {
        if(session!=null)
            session.rollback();
        e.printStackTrace();
        throw e;
    }finally {
        MyBatisUtils.closeSession(session);    
}	}
```



#### delete标签

```xml
<delete id="delete" parameterType="Integer">
    delete from t_goods where goods_id=#{value }
</delete>
```
```java
@Test public void testDelete() throws Exception {
    SqlSession session=null;
    try {
        session = MyBatisUtils.openSession();
        XXMapper mapper = session.getMapper(XXMapper.class);
        int num = mapper.delete(740);
        // int num = session.delete("goods.delete", 740);
        session.commit();
    } catch (Exception e) {
        if(session!=null)
            session.rollback();
        e.printStackTrace();
        throw e;
    }finally {
        MyBatisUtils.closeSession(session);
}	}
```



#### sql标签

用于定义可重用的SQL代码段，支持占位符配合property标签使用

```xml
<sql id="userColumns"> ${alias}.id,${alias}.username,${alias}.password </sql>

<sql id="sometable">
  ${prefix}Table
</sql>

<sql id="someinclude">
  from
    <include refid="${include_target}"/>
</sql>
```



#### include标签和property标签

include标签：引用已经被定义好的可重用SQL代码段

property标签：配合占位符使用

```xml
<select id="selectUsers" resultType="map">
  select
    <include refid="userColumns">
        <property name="alias" value="t1"/>
    </include>,
    <include refid="userColumns">
        <property name="alias" value="t2"/>
    </include>
  from some_table t1
    cross join some_table t2
</select>
```

```xml
<select id="select" resultType="map">
  select
    field1, field2, field3
  <include refid="someinclude">    
    <property name="include_target" value="sometable"/>
    <property name="prefix" value="Some"/>
  </include>
</select>
```



##### 占位符${}和#{}的区别

${}：原文传值，用于传送SQL语句的一部分
参数可以作为SQL语句的一部分，在一些高级功能可能要用到，

#{}：预编译传值，用于传递参数值。
使用预编译传值可以预防SQL注入攻击，会对参数值转义，不会作为SQL语句的一部分

```xml
<!-- 使用${}原文传值时，参数一定不能是用户外键输入的，而应该是程序控制的 -->
<select id="selectByTitle" parameterType="java.util.Map" resultType="com.zjbti.mybatis.entity.Goods">
    select * from t_goods where title=#{title}
    ${order}
</select>
```

```java
@Test public void testSelectByTitle() throws Exception {
    SqlSession session=null;
    try {
        session = MyBatisUtils.openSession();
        Map param=new HashMap();
        param.put("title","斯利安 孕妈专用 洗发水 氨基酸表面活性剂 舒缓头皮 滋养发根 让你的秀发会喝水 品质孕妈");
        param.put("order"," order by goods_id asc");
        XXMapper mapper = seession.getMapper(XXMapper.class);
        List<Goods> list = mapper.selectByTitle(param);
        // List<Goods> list = session.selectList("goods.selectByTitle",param);
        for (Goods g : list) {
            System.out.println(g.getGoodsId()+g.getTitle()+":"+g.getCurrentPrice());
        }
    } catch (Exception e) {
        if(session!=null)
            session.rollback();
        e.printStackTrace();
        throw e;
    }finally {
        MyBatisUtils.closeSession(session);
}	}
```



#### where标签和if标签

```xml
<select id="dynamicSQL" parameterType="java.util.Map" resultType="com.zjbti.mybatis.entity.Goods">
    select * from  t_goods
    <where>
      <if test="categoryId!=null">
         and category_id=#{categoryId}
      </if>
      <if test="currentPrice!=null">
          and current_price &lt; #{currentPrice}
      </if>
    </where>
</select>
```



#### foreach标签

```xml
<!-- 如果你的数据库还支持多行插入, 你也可以传入一个数组或集合，并返回自动生成的主键 -->
<insert id="insertAuthor"
        useGeneratedKeys="true"
    	keyProperty="id">
  insert into Author (username, password, email, bio) values
  <foreach item="item" collection="list" separator=",">
    (#{item.username}, #{item.password}, #{item.email}, #{item.bio})
  </foreach>
</insert>

<insert id="batchInsert" parameterType="java.util.List">
    insert into t_goods(title, sub_title, original_cost, current_price, discount, is_free_delivery,category_id)
    values    
    <foreach collection="list" item="item" index="index" separator=",">
        (#{item.title},#{item.subTitle},#{item.originalCost},#{item.currentPrice},#{item.discount},#{item.isFreeDelivery},#{item.categoryId})
    </foreach>
</insert>

<delete id="batchDelete" parameterType="java.util.List">
    delete from t_goods where goods_id in
    <foreach collection="list" item="item" index="index" open="(" close=")" separator=",">
        #{item}
    </foreach>
</delete>
```



### Mybatis的缓存机制

默认情况下：只启用了一级缓存，二级缓存关闭
一级缓存：本地会话缓存，是SqlSession级别的缓存，在同一个SqlSession内部有效
二级缓存：全局缓存，是Mapper接口级别的缓存，可以被多个SqlSession共享

| 注解               | 释义           |      |
| ------------------ | -------------- | ---- |
| @CacheNamespace    |                |      |
| @CacheNamespaceRef | 指定缓存作用域 |      |



```xml
<!-- cache标签：对给定命名空间中的select语句结果启用二级缓存缓存，insert、update 和 delete 语句会刷新缓存 -->
<cache eviction="FIFO" flushInterval="60000" size="512" readOnly="true"/>

<!-- 自定义缓存 -->
<cache type="com.domain.something.MyCustomCache">  <!-- MyCustomCache需要实现 org.apache.ibatis.cache.Cache接口，且提供一个接受String参数作为id的构造方法 -->
  <property name="cacheFile" value="/tmp/my-custom-cache.tmp"/>
</cache>

<!-- Mapper中的每条SQL语句可以通过使用两个简单属性来自定义与缓存交互的方式 -->
<select ... flushCache="false" useCache="true"/>

<!-- cache-ref标签：引用另一个命名空间中的缓存配置和实例 -->
<cache-ref namespace="com.someone.application.data.SomeMapper"/>
```

| 属性          | 值                                                           | 释义                               |
| ------------- | ------------------------------------------------------------ | ---------------------------------- |
| eviction驱逐  | LRU(默认) – 最近最少使用：移除最长时间不被使用的对象<br />FIFO – 先进先出：按对象进入缓存的顺序来移除它们<br />SOFT – 软引用：基于垃圾回收器状态和软引用规则移除对象<br />WEAK – 弱引用：更积极地基于垃圾收集器状态和弱引用规则移除对象 |                                    |
| flushInterval | 默认没有刷新间隔，缓存仅会在调用insert、update和delete语句后刷新 | 刷新间隔(毫秒)                     |
| size          | 1024(默认)                                                   | 可以存储的结果对象或列表的引用数目 |
| readOnly      | true表示返回只读缓存，每次从缓存中取出的是缓存对象本身，这些对象不能被修改，效率高<br />false(默认)表示返回可读写缓存(副本)，每次从缓存中取出的是缓存对象的副本，可以安全地被调用者修改，而不干扰其他调用者或线程所做的潜在修改 |                                    |



### 日志机制

日志机制：按顺序查找(SLF4J、ACL、Log4j2、Log4j、JDK logging)



# MBG

MyBatis Generator

[github](https://github.com/mybatis/generator)	|	[csdn MBG详解](https://blog.csdn.net/isea533/article/details/42102297)



# mybatis-mapper2.2.1

[github-通用mapper4(停止更新 不支持SpringBoot3)](https://github.com/abel533/Mapper)	|	[通用mapper4文档](https://github.com/abel533/Mapper/wiki)

[github-mybatis-mapper](https://github.com/mybatis-mapper/mapper)	|	[官网](https://mapper.mybatis.io/)

```pom
<!-- mybatis-mapper  -->
<dependency>
    <groupId>io.mybatis</groupId>
    <artifactId>mybatis-mapper</artifactId>
    <version>2.2.1</version>
</dependency>
```



## 通用mapper4配置

```xml
<!--/resources/generator-->
<!DOCTYPE generatorConfiguration PUBLIC
        "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <properties resource="generator/config.properties"/>

    <context id="Mysql" targetRuntime="MyBatis3Simple" defaultModelType="flat">
        <property name="beginningDelimiter" value="`"/>
        <property name="endingDelimiter" value="`"/>

        <!--专用代码生成器mapper插件-->
        <plugin type="tk.mybatis.mapper.generator.MapperPlugin">
            <!--生成的 Mapper 接口都会自动继承上该接口-->
            <property name="mappers" value="tk.mybatis.mapper.common.Mapper"/>

            <!--caseSensitive 是否区分大小写，默认值 false。如果数据库区分大小写，这里就需要配置为 true，这样当表名为 USER 时，会生成 @Table(name = "USER") 注解，否则使用小写 user 时会找不到表。-->
            <property name="caseSensitive" value="true"/>

            <!--forceAnnotation 是否强制生成注解，默认 false，如果设置为 true，不管数据库名和字段名是否一致，都会生成注解（包含 @Table 和 @Column）。-->
            <!--<property name="forceAnnotation" value="true"/>-->

            <!--beginningDelimiter 和 endingDelimiter 开始和结束分隔符，对于有关键字的情况下适用。-->
            <!--<property name="beginningDelimiter" value="`"/>-->
            <!--<property name="endingDelimiter" value="`"/>-->

            <!--useMapperCommentGenerator 是否使用通用 Mapper 提供的注释工具，默认 true 使用，这样在生成代码时会包含字段的注释（目前只有 mysql 和 oracle 支持），设置 false 后会用默认的，或者你可以配置自己的注释插件。-->

            <!--generateColumnConsts 在生成的 model中，增加字段名的常量，便于使用 Example 拼接查询条件的时候使用。-->

            <!--lombok 增加 model 代码生成时，可以直接生成 lombok 的 @Getter@Setter@ToString@Accessors(chain = true) 四类注解， 使用者在插件配置项中增加 <property name="lombok" value="Getter,Setter,ToString,Accessors"/> 即可生成对应包含注解的 model 类。-->
            <property name="lombok" value="Getter,Setter,ToString"/>
        </plugin>

        <jdbcConnection driverClass="${jdbc.driverClass}"
                        connectionURL="${jdbc.url}"
                        userId="${jdbc.user}"
                        password="${jdbc.password}">
        </jdbcConnection>

        <!--生成实体-->
        <javaModelGenerator targetPackage="cn.cxc.usercenter.domain.entity.${moduleName}"
                            targetProject="src/main/java"/>

        <!--生成mapper.xml-->
        <sqlMapGenerator targetPackage="cn.cxc.usercenter.dao.${moduleName}"
                         targetProject="src/main/resources"/>

        <!--生成mapper接口-->
        <javaClientGenerator targetPackage="cn.cxc.usercenter.dao.${moduleName}"
                             targetProject="src/main/java"
                             type="XMLMAPPER"/>

        <!--指定表-->
        <table tableName="${tableName}">
            <generatedKey column="id" sqlStatement="JDBC"/>
        </table>
    </context>
</generatorConfiguration>
```

```bash
mybatis-generator:generate
```



# Mybatis-Plus3.5.3.2

国人开发的，星标比通用Mapper多

[官网](https://baomidou.com/pages/24112f/#%E7%89%B9%E6%80%A7)	|	[github](https://github.com/baomidou/mybatis-plus)

```pom
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.3.2</version>
</dependency>
```



# 未整理的笔记

## MyBatis 的 Java API

### SqlSession

​	使用 MyBatis 的主要 Java 接口就是 SqlSession。你可以通过这个接口来执行命令，获取映射器和管理事务。
​	SqlSession 是由 SqlSessionFactory 实例创建的。SqlSessionFactory 对象包含创建 SqlSession 实例的所有方法。而 SqlSessionFactory 本身是由 SqlSessionFactoryBuilder 创建的，它可以从 XML、注解或手动配置 Java 代码来创建 SqlSessionFactory。

当 Mybatis 与一些依赖注入框架（如 Spring 或者 Guice）同时使用时，SqlSession 将被依赖注入框架所创建



### SqlSessionFactoryBuilder

SqlSessionFactory build(InputStream inputStream)
SqlSessionFactory build(InputStream inputStream, String environment)
SqlSessionFactory build(InputStream inputStream, Properties properties)
SqlSessionFactory build(InputStream inputStream, String env, Properties props)
SqlSessionFactory build(Configuration config)
注意优先级

有 org.apache.ibatis.io.Resources 的工具类，会帮助你从类路径下、文件系统或一个 web URL 中加载资源文件
	URL getResourceURL(String resource)
	URL getResourceURL(ClassLoader loader, String resource)
	InputStream getResourceAsStream(String resource)
	InputStream getResourceAsStream(ClassLoader loader, String resource)
	Properties getResourceAsProperties(String resource)
	Properties getResourceAsProperties(ClassLoader loader, String resource)
	Reader getResourceAsReader(String resource)
	Reader getResourceAsReader(ClassLoader loader, String resource)
	File getResourceAsFile(String resource)
	File getResourceAsFile(ClassLoader loader, String resource)
	InputStream getUrlAsStream(String urlString)
	Reader getUrlAsReader(String urlString)
	Properties getUrlAsProperties(String urlString)
	Class classForName(String className)



### SqlSessionFactory

	SqlSession openSession()
	SqlSession openSession(boolean autoCommit)
	SqlSession openSession(Connection connection)
	SqlSession openSession(TransactionIsolationLevel level)
	SqlSession openSession(ExecutorType execType, TransactionIsolationLevel level)
	SqlSession openSession(ExecutorType execType)
	SqlSession openSession(ExecutorType execType, boolean autoCommit)
	SqlSession openSession(ExecutorType execType, Connection connection)
	Configuration getConfiguration();

默认的 openSession()方法没有参数，它会创建有如下特性的 SqlSession：
	会开启一个事务（也就是不自动提交）。
	将从由当前环境配置的 DataSource 实例中获取 Connection 对象。
	事务隔离级别将会使用驱动或数据源的默认设置。
	预处理语句不会被复用，也不会批量处理更新。

autoCommit	
	自动提交功能
Connection	
	使用自己的 Connection 实例
TransactionIsolationLevel	
	Java 枚举包装器，若不使用它，将使用 JDBC 所支持五个隔离级（NONE、READ_UNCOMMITTED、READ_COMMITTED、REPEATABLE_READ 和 SERIALIZABLE），并按它们预期的方式来工作。
ExecutorType
	ExecutorType.SIMPLE：这个执行器类型不做特殊的事情。它为每个语句的执行创建一个新的预处理语句。
	ExecutorType.REUSE：这个执行器类型会复用预处理语句。
	ExecutorType.BATCH：这个执行器会批量执行所有更新语句，如果 SELECT 在它们中间执行，必要时请把它们区分开来以保证行为的易读性。

 getConfiguration()。这 个方法会返回一个 Configuration 实例，在运行时你可以使用它来自检 MyBatis 的配置。

### SqlSession

执行语句、提交或回滚事务和获取映射器实例的方法

SqlSession执行语句方法
	这些方法被用来执行定义在 SQL 映射的 XML 文件中的 SELECT、INSERT、UPDATE 和 DELETE 语句。它们都会自行解释，每一句都使用语句的 ID 属性和参数对象，参数可以是原生类型（自动装箱或包装类）、JavaBean、POJO 或 Map。

```java
<T> T selectOne(String statement, Object parameter)		返回多个会抛出异常，用于查看可行方案是否存在
<E> List<E> selectList(String statement, Object parameter)		返回多个
<T> Cursor<T> selectCursor(String statement, Object parameter)	Cursor游标
<K,V> Map<K,V> selectMap(String statement, Object parameter, String mapKey)	将多结果集转为 Map 类型值
int insert(String statement, Object parameter)
int update(String statement, Object parameter)
int delete(String statement, Object parameter)
```



void select (String statement, Object parameter, RowBounds rowBounds, ResultHandler<T> handler)
允许你限制返回行数的范围，或者提供自定义结果控制逻辑

int offset = 100;
int limit = 25;
RowBounds rowBounds = new RowBounds(offset, limit);
RowBounds 参数会告诉 MyBatis 略过指定数量的记录，还有限制返回结果的数量。RowBounds 类有一个构造方法来接收 offset 和 limit，另外，它们是不可二次赋值的。

package org.apache.ibatis.session;
public interface ResultHandler<T> {
  void handleResult(ResultContext<? extends T> context);
}
ResultContext 参数允许你访问结果对象本身、被创建的对象数目、以及返回值为 Boolean 的 stop 方法，你可以使用此 stop 方法来停止 MyBatis 加载更多的结果。
从被 ResultHandler 调用的方法返回的数据不会被缓存。
当使用结果映射集（resultMap）时，MyBatis 大多数情况下需要数行结果来构造外键对象。如果你正在使用 ResultHandler，你可以给出外键（association）或者集合（collection）尚未赋值的对象。


SqlSession批量立即跟新方法
	List<BatchResult> flushStatements()
	可以刷新（执行）存储在 JDBC 驱动类中的批量更新语句。当你将 ExecutorType.BATCH 作为 ExecutorType 使用时可以采用此方法

SqlSession事务控制方法
	控制事务作用域有四个方法。当然，如果你已经设置了自动提交或你正在使用外部事务管理器，这就没有任何效果了。然而，如果你正在使用 JDBC 事务管理器，由Connection 实例来控制，那么这四个方法就会派上用场：
	void commit()
	void commit(boolean force)
	void rollback()
	void rollback(boolean force)

SqlSession本地缓存
	Mybatis 使用到了两种缓存：本地缓存（local cache）和二级缓存（second level cache）。
	每当一个新 session 被创建，MyBatis 就会创建一个与之相关联的本地缓存。任何在 session 执行过的查询语句本身都会被保存在本地缓存中，那么，相同的查询语句和相同的参数所产生的更改就不会二度影响数据库了。本地缓存会被增删改、提交事务、关闭事务以及关闭 session 所清空。
	默认情况下，本地缓存数据可在整个 session 的周期内使用，这一缓存需要被用来解决循环引用错误和加快重复嵌套查询的速度，所以它可以不被禁用掉，但是你可以设置 localCacheScope=STATEMENT 表示缓存仅在语句执行时有效。
	注意，如果 localCacheScope 被设置为 SESSION，那么 MyBatis 所返回的引用将传递给保存在本地缓存里的相同对象。对返回的对象（例如 list）做出任何更新将会影响本地缓存的内容，进而影响存活在 session 生命周期中的缓存所返回的值。因此，不要对 MyBatis 所返回的对象作出更改，以防后患。



	void clearCache()	清空本地缓存的方法

确保SqlSession被关闭
	void close()
	工作模式最好try-catch-finally

使用映射器
	<T> T getMapper(Class<T> type)
	一个更通用的方式来执行映射语句是使用映射器类。一个映射器类就是一个仅需声明与 SqlSession 方法相匹配的方法的接口类。
下面的示例展示了一些方法签名以及它们是如何映射到 SqlSession 上的。

```java
public interface AuthorMapper {
    // (Author) selectOne("selectAuthor",5);
    Author selectAuthor(int id);
    // (List<Author>) selectList(“selectAuthors”)
    List<Author> selectAuthors();
    // (Map<Integer,Author>) selectMap("selectAuthors", "id")
    @MapKey("id")
    Map<Integer, Author> selectAuthors();
    // insert("insertAuthor", author)
    int insertAuthor(Author author);
    // updateAuthor("updateAuthor", author)
    int updateAuthor(Author author);
    // delete("deleteAuthor",5)
    int deleteAuthor(int id);
}
```

​	每个映射器方法签名应该匹配相关联的 SqlSession 方法，而字符串参数 ID 无需匹配。相反，方法名必须匹配映射语句的 ID
​	映射器接口不需要去实现任何接口或继承自任何类。只要方法可以被唯一标识对应的映射语句
​	唯一的限制就是你不能在两个继承关系的接口中拥有相同的方法签名
​	方法签名就是方法名+形参列表



# 类加载器

ClassLoader类加载器
	作用：将 Class 加载到 JVM 中、审查每个类由谁加载（父优先的等级加载机制）、将 Class 字节码重新解析成 JVM 统一要求的对象格式

类被加载到JVM后到卸载出内存的生命周期
	Loading加载、Verification验证、Preparation准备、Resolution解析、Initialization初始化、使用Using和卸载UnLoading

	Resolution和Using阶段顺序不一定，为支持Java语言运行时绑定

Initialization	五种触发类初始化的情况	这五种情况被称为主动引用会触发初始化，被动引用则不会
	1、创建类的实例
	2、对类进行反射调用，如果类没有初始化，先进行初始化
	3、初始化类，如果其父类没有初始化，先初始化其父类
	4、虚拟机启动时，先初始化用户指定的main主类
	5、使用jdk动态语言支持时，java.lang.invoke.MethodHandle实例最后的解析结果REF_getstatic,REF_putstatic,REF_invokeStatic的方法句柄没有进行初始化，则需要先出触发其初始化。

类的实例化和初始化的区别
类的实例化是指创建一个类的实例(对象)的过程；
类的初始化是指为类中各个类成员(被static修饰的成员变量)赋初始值的过程，是类生命周期中的一个阶段。

被动引用：
1.子类调用父类的静态变量，子类不会被初始化。只有父类被初始化。。对于静态字段，只有直接定义这个字段的类才会被初始化.
2.通过数组定义来引用类，不会触发类的初始化
	因为定义不会触发初始化，要你自己初始化才会初始化，定义只是开了内存空间
	编译器在编译阶段对函数体外的变量只进行内存分配，如果要赋值只能是在对变量进行初始化的时候。main()函数是程序执行的入口，只有在main函数中才能对全局变量进行赋值操作，当然也包括可能会被主函数调用的其它函数体中。
3.访问类的常量，不会初始化类

ClassLoader 的等级加载机制
	BootStrap ClassLoader启动类加载器，是Java类加载层次中最顶层的类加载器，负责加载JDK中的核心类库，如：rt.jar、resources.jar、charsets.jar等，这个ClassLoader完全是JVM自己控制的，需要加载哪个类，怎么加载都是由JVM自己控制，别人也访问不到这个类，所以BootStrap ClassLoader不遵循委托机制(后面再阐述什么是委托机制)，没有子加载器。
	EtxClassLoader扩展类加载器，负责加载Java的扩展类库，Java 虚拟机的实现会提供一个扩展库目录，该类加载器在此目录里面查找并加载 Java 类。默认加载JAVA_HOME/jre/lib/ext/目下的所有jar。
	AppClassLoader系统类加载器，负责加载应用程序classpath目录下的所有jar和class文件。一般来说，Java 应用的类都是由它来完成加载的。可以通过 ClassLoader.getSystemClassLoader()来获取它。我们可以通过System.getProperty(“java.class.path”) 来查看 classpath。

EtxClassLoader是BootStrap ClassLoader的子类，AppClassLoader是EtxClassLoader的子类，开发人员写的类加载器是是加载此类Java 类的类加载器AppClassLoader的子类。类加载器通过这种方式组织起来，形成树状结构。树的根节点就是引导类加载器。

defineClass(String name, byte[] b, int off, int len)
	将 字节流解析成 JVM 能够识别的 Class 对象，有了这个方法意味着我们不仅仅可以通过 class 文件实例化对象，还可以通过其他方式实例化对象，如果我们通过网络接收到一个类的字节码，拿到这个字节码流直接创建类的 Class 对象形式实例化对象。如果直接调用这个方法生成类的 Class 对象，这个类的 Class 对象还没有 resolve ，这个 resolve 将会在这个对象真正实例化时才进行。

//查找指定名称的类
findClass(String name) 

//加载指定名称的类
loadClass(String name) 

//链接指定的类
resolveClass(Class<?>) 

ClassLoader加载类的原理
ClassLoader使用的是双亲委托模型来搜索加载类的
双亲委托模型
	ClassLoader使用的是双亲委托机制来搜索加载类的，每个ClassLoader实例都有一个父类加载器的引用（不是继承的关系，是一个组合的关系），虚拟机内置的类加载器（Bootstrap ClassLoader）本身没有父类加载器，但可以用作其它ClassLoader实例的的父类加载器。当一个ClassLoader实例需要加载某个类时，它会试图亲自搜索某个类之前，先把这个任务委托给它的父类加载器，这个过程是由上至下依次检查的，首先由最顶层的类加载器Bootstrap ClassLoader试图加载，如果没加载到，则把任务转交给Extension ClassLoader试图加载，如果也没加载到，则转交给App ClassLoader 进行加载，如果它也没有加载得到的话，则返回给委托的发起者，由它到指定的文件系统或网络等URL中加载该类。如果它们都没有加载到这个类时，则抛出ClassNotFoundException异常。否则将这个找到的类生成一个类的定义，并将它加载到内存当中，最后返回这个类在内存中的Class实例对象。

双亲委托模型好处
	因为这样可以避免重复加载，当父亲已经加载了该类的时候，就没有必要 ClassLoader再加载一次。考虑到安全因素，我们试想一下，如果不使用这种委托模式，那我们就可以随时使用自定义的String来动态替代java核心api中定义的类型，这样会存在非常大的安全隐患，而双亲委托的方式，就可以避免这种情况，因为String已经在启动时就被引导类加载器（Bootstrcp ClassLoader）加载，所以用户自定义的ClassLoader永远也无法加载一个自己写的String，除非你改变JDK中ClassLoader搜索类的默认算法。

对于任意一个类，都需要由加载它的类加载器和这个类本身一同确立其在Java虚拟机中的唯一性

双亲委派
	先检查是否已经被加载过，若没有加载则调用父加载器的loadClass() 方法，若父加载器为空，则默认使用启动类加载器作为父加载器。如果父加载器失败，再调用自己的findClass 方法进行加载，就是先自己的父类加载器加载，最后BootStrap ClassLoader

configuration 类包含你可能需要了解 SqlSessionFactory 实例的所有内容。
手动配置configuration来创建sqlsessionfactory
	DataSource dataSource = BaseDataTest.createBlogDataSource();
	TransactionFactory transactionFactory = new JdbcTransactionFactory();
	Environment environment = new Environment("development", transactionFactory, dataSource);
	Configuration configuration = new Configuration(environment);
	configuration.setLazyLoadingEnabled(true);
	configuration.setEnhancementEnabled(true);
	configuration.getTypeAliasRegistry().registerAlias(Blog.class);
	configuration.getTypeAliasRegistry().registerAlias(Post.class);
	configuration.getTypeAliasRegistry().registerAlias(Author.class);
	configuration.addMapper(BoundBlogMapper.class);
	configuration.addMapper(BoundAuthorMapper.class);
	SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
	SqlSessionFactory factory = builder.build(configuration);

