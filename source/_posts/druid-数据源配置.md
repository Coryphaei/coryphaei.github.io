---
title: druid-数据源配置
date: 2016-03-17 10:12:46
tags: [Java]
categories: coding
---

### 写在前面
druid数据连接池是阿里巴巴旗下出的一款数据库连接池jar包，我个人觉得还是蛮好用的，这里附上[maven repository的地址](http://mvnrepository.com/artifact/com.alibaba/druid),下载jar包也好，还是maven配置也好，这里都有。如何向参考具体的项目，可以去我[github](https://github.com/xiaowei1118)上看看[java_server](https://github.com/xiaowei1118/java_server)这个项目，这个项目就是基于druid数据源配置的SpringMVC+spring+mybatis的。

### 以下是数据源连接配置

#### jndi方式数据源配置

在spring里面设置数据库连接方式为jndi配置:
```xml
<bean id="dataSource" class="org.springframework.jndi.JndiObjectFactoryBean">
  <property name="jndiName">
     <value>java:comp/env/jdbc/demoDB</value>
  </property>
</bean>  

<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
  <property name="dataSource">
      <ref bean="dataSource" />
  </property>
</bean>
```
在tomcat的config目录下的context文件里面做如下配置（lib目录下要放置druid的jar文件，和数据库驱动jar包，我这里用到的是postgresql）。

我个人觉得最好是用jndi配置，将数据源配置在tomcat的配置文件中，因为我们的代码是要编译运行的，生产环境的数据库配置和开发环境肯定不一样，基于jndi配置的话，就免去了，在生产环境下还要修改数据库配置了。

```xml
<Resource    
    name="jdbc/demoDB"
    factory="com.alibaba.druid.pool.DruidDataSourceFactory"  
    auth="Container"  
    type="javax.sql.DataSource"
    maxActive="10"    
    maxWait="10000"
    removeAbandoned="true"
    logAbandoned="true"
    removeAbandonedTimeout="1800"
    TimeBetweenEvictionRunsMillis="3000"
    poolPreparedStatements="true"
    maxPoolPreparedStatementPerConnectionSize="20"
    filters="stat,mergeStat"
    validationQuery="select 1"
    testWhileIdle="true"
    url="jdbc:postgresql://*.*.*.*:5432/flowlog?characterEncoding=UTF8"
    username="postgres"
    password="" />
```

#### 普通数据源配置
spirng配置如下，项目中需要上面的两个jar包，加载properties文件
```xml
 <bean name="dataSource" class="com.alibaba.druid.pool.DruidDataSource"
  init-method="init" destroy-method="close">
  <property name="url" value="${jdbc_url}" />
  <property name="username" value="${jdbc_username}" />
  <property name="password" value="${jdbc_password}" />

  <!-- 初始化连接大小 -->
  <property name="initialSize" value="0" />
  <!-- 连接池最大使用连接数量 -->
  <property name="maxActive" value="20" />
  <!-- 连接池最大空闲 -->
  <property name="maxIdle" value="20" />
  <!-- 连接池最小空闲 -->
  <property name="minIdle" value="0" />
  <!-- 获取连接最大等待时间 -->
  <property name="maxWait" value="60000" />

  <property name="poolPreparedStatements" value="true" />
  <property name="maxPoolPreparedStatementPerConnectionSize"
   value="33" />

  <property name="validationQuery" value="${validationQuery}" />
  <property name="testOnBorrow" value="false" />
  <property name="testOnReturn" value="false" />
  <property name="testWhileIdle" value="true" />

  <!-- 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒 -->
  <property name="timeBetweenEvictionRunsMillis" value="60000" />
  <!-- 配置一个连接在池中最小生存的时间，单位是毫秒 -->
  <property name="minEvictableIdleTimeMillis" value="25200000" />

  <!-- 打开removeAbandoned功能 -->
  <property name="removeAbandoned" value="true" />
  <!-- 1800秒，也就是30分钟 -->
  <property name="removeAbandonedTimeout" value="1800" />
  <!-- 关闭abanded连接时输出错误日志 -->
  <property name="logAbandoned" value="true" />

  <!-- 监控数据库 -->
  <property name="filters" value="stat" />
  <!-- <property name="filters" value="mergeStat" /> -->
 </bean>
```
properties文件如下
```
hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
driverClassName=org.postgresql.Driver
validationQuery=SELECT 1
jdbc_url=jdbc:postgresql://.......:5432/flowlog?characterEncoding=UTF8
jdbc_username=postgres
jdbc_password=
```
