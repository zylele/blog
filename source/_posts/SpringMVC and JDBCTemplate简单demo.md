---
title: SpringMVC and JDBCTemplate简单demo
date: 2015-02-07 17:15:45
tags:
 - JDBCTemplate
 - SpringMVC
 - 连接池
---

在开发过程中与数据库交互的时候，经常使用JDBC与数据库连接并对之进行操作，产生了大量的重复代码。

<!-- more -->

使用Spring的jdbcTemplate进一步简化JDBC操作，做个简单demo笔记。

该demo使用dpcp连接池，首先先引入相关jar包。commons-dbcp-1.2.1.jar, commons-pool-1.2.jar, commons-collections-3.1.jar

在src目录下创建数据源的相关配置信息：jdbc.properties

```java
driverClassName=com.mysql.jdbc.driver
url=jdbc:mysql://localhost:3306/znyalor?useUnicode\=true&characterEncoding\=UTF-8
username=root
password=root
initialSize=1
maxActive=500
maxIdle=2
minIdle=1
```

创建spring的配置文件，将jdbc.properties与JdbcTemplate粘合起来的配置文件：beans.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-2.5.xsd
       http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd
       http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd">

    <!-- 引入上面提到的数据源的相关配置文件 -->
    <context:property-placeholder location="classpath:jdbc.properties"/>

    <!-- 配置连接池 -->
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">

         <property name="driverClassName" value="${driverClassName}"/>
         <property name="url" value="${url}"/>
         <property name="username" value="${username}"/>
         <property name="password" value="${password}"/>

     <!-- 连接池启动时的初始值 -->
     <property name="initialSize" value="${initialSize}"/>

     <!-- 连接池的最大值 -->
     <property name="maxActive" value="${maxActive}"/>

     <!-- 最大空闲值.当经过一个高峰时间后，连接池可以慢慢将已经用不到的连接慢慢释放一部分，一直减少到maxIdle为止 -->
     <property name="maxIdle" value="${maxIdle}"/>

     <!-- 最小空闲值.当空闲的连接数少于阀值时，连接池就会预申请去一些连接，以免洪峰来时来不及申请 -->
     <property name="minIdle" value="${minIdle}"/>

    </bean>

  <!-- 注入Bean -->
  <bean id="personService" class="cn.comp.service.impl.PersonServiceImpl">
    <property name="dataSource" ref="dataSource"/>
  </bean>

</beans>
```

创建数据库实体类：Person.java

```java
public class Person {
    private Integer id;
    private String name;

    public Person(){}

    public Person(String name) {
    this.name = name;
    }
    public Integer getId() {
    return id;
    }
    public void setId(Integer id) {
    this.id = id;
    }
    public String getName() {
    return name;
    }
    public void setName(String name) {
    this.name = name;
    }
}
```java

创建Person的操作接口：PersonService.java

```java
public interface PersonService {

    public List<Person> getPersons();

}
```java

创建对接口的实现类：PersonServiceImpl.java

```java
public class PersonServiceImpl implements PersonService {
    private JdbcTemplate jdbcTemplate;

    public void setDataSource(DataSource dataSource) {
    this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    @SuppressWarnings("unchecked")
    public List<Person> getPersons() {
      //执行查询的sql语句，并且对结果集进行处理，返回一个List对象，所有的数据库其他操作由jdbcTemplate和连接池进行维护
    return (List<Person>)jdbcTemplate.query("select * from person", new PersonRowMapper());
    }

}
```java

创建在查询对象时，记录的映射回调类：PersonRowMapper.java

```java
//实现RowMapper接口，对ResultSet结果集进行处理，不需要rs.next()

public class PersonRowMapper implements RowMapper {

    public Object mapRow(ResultSet rs, int index) throws SQLException {
    Person person = new Person(rs.getString("name"));
    person.setId(rs.getInt("id"));
    return person;
    }
}
```

创建一个查询Persons的Action：PersonsAction.java

```java
@Controller 
public class PersonsAction {

	//Spring自动装配Bean
	@Autowired
	PersonService personService

	//@ResponseBody注解会将对象封装成json格式返回
	@RequestMapping(value = "getPersons")
	public @ResponseBody List<Person> getPersons() {
		return personService.getPersons();
	}
}
```

还可以增加对事务的控制。

事务的操作首先要通过配置文件，取得spring的支持，再在java程序中显示的使用@Transactional注解来使用事务操作。
