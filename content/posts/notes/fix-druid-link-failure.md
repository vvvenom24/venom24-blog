---
title: "Druid The last packet successfully received from the server was xxx milliseconds ago"
date: 2023-07-06T10:32:36+08:00
lastmod: 2023-07-06T10:32:36+08:00
draft: false
series: [Notes]
tags: [Druid]
summary: "处理 Druid 连接超时问题，以及 SpringBoot 引用 Druid 和 Druid 的常用配置"
---
## Fix CommunicationsException

异常信息：

```java
org.springframework.dao.RecoverableDataAccessException:
### Error querying database.  Cause: com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure

The last packet successfully received from the server was 1,243,528 milliseconds ago.  The last packet sent successfully to the server was 0 milliseconds ago.
### The error may exist in URL [jar:file:/opt/...]
### The error may involve defaultParameterMap
### The error occurred while setting parameters
### SQL: ...
### Cause: com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure

The last packet successfully received from the server was 1,243,528 milliseconds ago.  The last packet sent successfully to the server was 0 milliseconds ago.
; SQL []; Communications link failure

The last packet successfully received from the server was 1,243,528 milliseconds ago.  The last packet sent successfully to the server was 0 milliseconds ago.; nested exception is com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure

The last packet successfully received from the server was 1,243,528 milliseconds ago.  The last packet sent successfully to the server was 0 milliseconds ago.
```

原因：

数据库连接池中的连接在服务端已超时，导致客户端直接从连接池中取出的连接是已失效的，所以产生如上异常

处理：

设置连接池连接的检测时间以及连接的检测 SQL

Druid 的配置如下：

```yml
spring:
  datasource:
    testWhileIdle: true
    timeBetweenEvictionRunMillis: 60000
    validationQuery: SELECT 1 FROM DUAL
```

或

```yml
spring:
  datasource:
    druid:
      testWhileIdle: true
      timeBetweenEvictionRunMillis: 60000
      validationQuery: SELECT 1 FROM DUAL
```

***注意：*** 这里的两种不同配置取决于你引入 Druid 的方式，下面会具体讲到。

## SpringBoot 中引入 Druid

Druid 有两种包

针对 SpringBoot 做了集成

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.2.6</version>
</dependency>
```
Druid 核心包

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.2.6</version>
</dependency>
```

两种包主要的区别在于配置上

使用 `druid-spring-boot-starter 包` 引入

```yml
spring:
  datasource:
    druid:
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://localhost:3306/springboot_test?characterEncoding=utf8&serverTimezone=UTC
      username: root
      password: root
```

使用 `druid 包` 引入

```yml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/springboot_test?characterEncoding=utf8&serverTimezone=UTC
    username: root
    password: root
    type: com.alibaba.druid.pool.DruidDataSource
```

***注意：*** 使用 `druid 包` 的话需要将配置手动引入

```java
@Configuration
public class DruidDataSourceConfig {

    /**
     * 添加 DruidDataSource 组件到容器中，并绑定属性
     */
    @Bean
    @ConfigurationProperties(prefix = "spring.datasource")
    @ConditionalOnProperty(name = "spring.datasource.type", havingValue = "com.alibaba.druid.pool.DruidDataSource")
    public DataSource druid(){
        return  new DruidDataSource();
    }
}
```

使用 `druid-spring-boot-starter 包` 的话，druid的相关配置可以使用 druid 的专属配置直接配置 yml 或者 properties 即可

## Druid 相关配置

这里只展示 `druid-spring-boot-starter 包` 包的配置，使用 `druid 包` 的话将配置中的 `.druid` 去掉即可：

```properties
# 初始化时建立物理连接的个数，初始化发生在显示调用 init 方法，或者第一次 getConnection 时
spring.datasource.druid.initialSize = 0
# 最大连接池数量
spring.datasource.druid.maxActive = 55
# 最小连接池数量
spring.datasource.druid.minIdle = 1
# 获取连接时最大等待时间，单位毫秒
# 配置了 maxWait 之后，缺省启用公平锁，并发效率会有所下降（可以通过配置 useUnfairLock=true 使用非公平锁）
spring.datasource.druid.maxWait = -1
# 是否缓存 preparedStatement，即 PsCache
# PSCache 对支持游标的数据库性能提升巨大，比如说 oracle，而 mysql 则建议关闭
spring.datasource.druid.poolPreparedStatements = false
# 要启用 PSCache，必须配置大于0
# 当大于 0 时，poolPreparedStatements 自动触发修改为 true
# 在 Druid 中，不会存在 Oracle 下 PSCache 占用内存过多的问题，可以把这个数值配置大一点，比如 100
spring.datasource.druid.maxOpenPreparedStatements = 10
# 用来检测连接是否有效的 sql，要求是一个查询语句
# 如果 validationQuery 为null，testOnBorrow、testOnReturn 、testWhileIdle 都不会起作用
spring.datasource.druid.validationQuery = SELECT 1 FROM DUAL
# 申请连接时执行 validationQuery 检测连接是否有效，做了这个配置会降低性能
spring.datasource.druid.testOnBorrow = false
# 归还连接时执行 validationQuery 检测连接是否有效，做了这个配置会降低性能
spring.datasource.druid.testOnReturn = false
# 申请连接的时候检测，如果空闲时间大于 timeBetweenEvictionRunMills，执行 validationQuery 检测连接是否有效
# 建议配置为 true，不影响性能，并且保证安全性
spring.datasource.druid.testWhileIdle = true
# 间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
# Destory 线程会检测连接的间隔时间
# testWhileIdle 的判断依据
spring.datasource.druid.timeBetweenEvictionRunMillis = 60000
# 一个连接在池中最小生存的时间，单位是毫秒
spring.datasource.druid.minEvictableIdleTimeMillis = 300000
# 物理连接初始化的时候执行 sql
spring.datasource.druid.connectionInitSqls =
# 当数据库抛出一些不可恢复的异常时，抛弃连接
spring.datasource.druid.exceptionSorter =
# 通过别名的方式配置扩展插件，属性类型是字符串
# 常用的插件有：监控统计用的 filter（stat：监控统计，log：4:日志记录，wall：防御sql注入）
spring.datasource.druid.filters =
# 类型是 List<com.alibaba.druid,filter.Filter>，如果同时配置 filter 和 proxyFilters，是组合关系（并非）
spring.datasource.druid.proxyFilters =
```