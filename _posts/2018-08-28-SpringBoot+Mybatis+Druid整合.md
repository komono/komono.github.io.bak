---
layout:     post
title:      SpringBoot+Mybatis+Druid整合
subtitle:   SpringBoot+Mybatis+Druid整合
date:       2018-08-28
author:     ptpan
header-img: img/post-bg-pexels-photo-695299.jpeg
catalog: true
tags:
    - Java
    - Spring Boot
    - Demo
---

>
> **SpringBoot+Mybatis+Druid整合**



### 1.新建SpringBoot项目

[可以直接到SpringBoot官网快速获取](https://start.spring.io/)

![20180828-springboot-qkstart](http://ptpan.top/img/post/2018/20180828-springboot-qkstart.PNG)



### 2.pom.xml引入相关依赖



```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.ps</groupId>
	<artifactId>druid-demo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>druid-demo</name>
	<description>Demo project for Spring Boot</description>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.4.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
		<mysql-connector-java.version>8.0.12</mysql-connector-java.version>
		<mybatis.version>3.3.0</mybatis.version>
		<druid.version>1.0.10</druid.version>
		<ibatis-core.version>3.0</ibatis-core.version>
	</properties>

	<dependencies>
		<!-- database start -->
		<!-- Druid 监控 -->
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>druid-spring-boot-starter</artifactId>
			<version>1.1.10</version>
		</dependency>

		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
		</dependency>
		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis</artifactId>
			<version>${mybatis.version}</version>
		</dependency>

		<!--添加监控之后避免冲突注释掉 <dependency> <groupId>com.alibaba</groupId> <artifactId>druid</artifactId> 
			<version>${druid.version}</version> </dependency> -->

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-jdbc</artifactId>
		</dependency>

		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis</artifactId>
			<version>${mybatis.version}</version>
		</dependency>

		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis-spring</artifactId>
			<version>1.2.2</version>
		</dependency>
		<!-- database end -->

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.mybatis.generator</groupId>
				<artifactId>mybatis-generator-maven-plugin</artifactId>
				<version>1.3.2</version>
				<configuration>
					<!-- 指定配置文件的位置 -->
					<configurationFile>${basedir}/src/main/resources/generatorConfig.xml</configurationFile>
					<!-- 输出详细日志 -->
					<verbose>true</verbose>
					<!-- 覆盖已有文件 -->
					<overwrite>true</overwrite>
				</configuration>
				<dependencies>
					<dependency>
						<groupId>com.freetmp</groupId>
						<artifactId>dolphin-mybatis-generator</artifactId>
						<version>1.1.0</version>
					</dependency>
					<dependency>
						<groupId>mysql</groupId>
						<artifactId>mysql-connector-java</artifactId>
						<version>${mysql-connector-java.version}</version>
					</dependency>
				</dependencies>
			</plugin>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>

```

### 3.新数据表+配置generatorConfig.xml

**建表**

```sql
CREATE TABLE IF NOT EXISTS user_info (
	user_id BIGINT ( 20 ) NOT NULL AUTO_INCREMENT,
	user_name CHAR ( 15 ) NOT NULL,
	user_password CHAR ( 15 ) NOT NULL,
	user_email VARCHAR ( 20 ) NOT NULL UNIQUE,
	user_birthday DATETIME,
	user_age INT,
	user_salary DECIMAL ( 10, 2 ),
	PRIMARY KEY ( user_id ) 
	) ENGINE = INNODB DEFAULT charset = utf8mb4;
```

**配置generatorConfig.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE generatorConfiguration PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN" "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd" >
<generatorConfiguration>
	<!-- 参考文档：中文 http://generator.sturgeon.mopaas.com/afterRunning.html，原文http://www.mybatis.org/generator/ -->
	<!-- 示例table：create table mybatis_user(user_name CHAR(15) not null,user_password 
		CHAR(15) not null,user_email VARCHAR(20) not null unique,user_birthday DATETIME,user_age 
		INT,user_salary DECIMAL(10,2),primary key(user_name))engine=innodb default 
		charset=utf8; -->
	<!-- 依赖包直接定义在mybatis-generator-maven-plugin的dependencies中 -->
	<context id="default" targetRuntime="MyBatis3">
		<property name="javaFileEncoding" value="UTF-8" />

		<!-- 替换Example为Criteria -->
		<plugin
			type="org.mybatis.generator.plugins.RenameExampleClassPlugin">
			<property name="searchString" value="Example$" />
			<property name="replaceString" value="Criteria" />
		</plugin>
		<!-- 表格对应Java bean类型实现java.io.Serializable接口，添加默认的serialVersionUID -->
		<plugin type="org.mybatis.generator.plugins.SerializablePlugin">
			<property name="suppressJavaInterface " value="true" />
		</plugin>
		<!-- 表格对应Java bean类型增加toString方法 -->
		<plugin type="org.mybatis.generator.plugins.ToStringPlugin" />
		<!-- 增加分页支持 -->
		<plugin
			type="com.freetmp.mbg.plugin.page.MySqlPaginationPlugin" />
		<!-- GeneratedCriteria中addCriterion(String condition, Object value, String 
			property)不会throw new RuntimeException -->

		<!-- 生成注释不带时间戳，否则即使表结构没变，每次生成文件都有变更 -->
		<commentGenerator>
			<property name="suppressDate" value="true" />
		</commentGenerator>
		<!-- jdbc连接定义 -->
		<jdbcConnection driverClass="com.mysql.jdbc.Driver"
			connectionURL="jdbc:mysql://127.0.0.1:3306/druid_test?useUnicode=true&amp;characterEncoding=UTF-8"
			userId="root" password="123456" />
		<!-- 强制数据库小数类型为java.math.BigDecimal -->
		<javaTypeResolver>
			<property name="forceBigDecimals" value="true" />
		</javaTypeResolver>
		<!-- 表格对应Java bean类型生成，结果为$TABLE_NAME$.java和$TABLE_NAME$Criteria.java -->
		<javaModelGenerator
			targetPackage="com.ps.druiddemo.dao.dto"
			targetProject="./src/main/java/">
			<property name="constructorBased" value="false" />
			<!-- 指定Java bean公共父类 <property name="rootClass" value="" /> -->
			<property name="trimStrings" value="true" />
		</javaModelGenerator>
		<!-- sql语句生成，结果为$TABLE_NAME$Mapper.xml -->
		<sqlMapGenerator
			targetPackage="com.ps.druiddemo.dao.sqlmap"
			targetProject="./src/main/java/" />
		<!-- 应用客户端生成，结果为$TABLE_NAME$Mapper.java -->
		<javaClientGenerator
			targetPackage="com.ps.druiddemo.dao.mapper"
			targetProject="./src/main/java/" type="XMLMAPPER">
			<!-- 指定客户端公共接口 <property name="rootInterface" value="" /> -->
		</javaClientGenerator>

		<!-- 以下定义需要生成的表 -->
		<table schema="" tableName="user_info" />

	</context>
</generatorConfiguration>
```

### 4.生成mybatisMapper

**Maven下执行命令：**

```bash
mybatis-generator:generate -X
```

**生成效果：**

![20180828-mybatis-generator](http://ptpan.top/img/post/2018/20180828-mybatis-generator.png)

### 5.配置数据源信息

 **application.properties中添加配置：** 

```properties
##database start
druid.driverClassName=com.mysql.jdbc.Driver 
druid.url=jdbc:mysql://127.0.0.1:3306/druid_test?useUnicode=true&characterEncoding=UTF-8
druid.username=root
#druid.password=123456
druid.initialSize=10
druid.minIdle=6
druid.maxActive=50
druid.maxWait=60000
druid.timeBetweenEvictionRunsMillis=60000
druid.minEvictableIdleTimeMillis=300000
druid.validationQuery=SELECT 'x'
druid.testWhileIdle=true
druid.testOnBorrow=false
druid.testOnReturn=false
druid.poolPreparedStatements=false
druid.maxPoolPreparedStatementPerConnectionSize=20
#druid.filters=wall,stat
druid.filters=wall,stat,config
druid.publicKey=MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAJIozuO9Uc1GwR1kwMI+E1eKzehr56NGiZnzamxSXnqEyy1pcECuWTfRFf3hKdyr0YuoUuj8yXRRSYn8JZFwI/UCAwEAAQ==
druid.password=XPKL0bkj/9ECEtf4Jl/1k//FCWyM0q8iY+V34uqyhTc/nA67KZ4BVC1yq/hSnF9Ihyxq4+IyanH7yPYEe/STCQ==
##database end
```



### 6.添加Druid+Mybatis配置

1. **新建datasource.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:jdbc="http://www.springframework.org/schema/jdbc"
	xmlns:mybatis-spring="http://mybatis.org/schema/mybatis-spring"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.3.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd
		http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc-4.3.xsd
		http://mybatis.org/schema/mybatis-spring http://mybatis.org/schema/mybatis-spring-1.2.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.3.xsd">

	<!-- 数据源 -->
	<bean id="dataSource"
		class="com.alibaba.druid.pool.DruidDataSource" init-method="init"
		destroy-method="close">
		<!-- 数据库基本信息配置 -->
		<property name="url" value="${druid.url}" />
		<property name="username" value="${druid.username}" />
		<property name="password" value="${druid.password}" />
		<property name="connectionProperties" value="config.decrypt=true;config.decrypt.key=${druid.publicKey}" />
		<property name="driverClassName"
			value="${druid.driverClassName}" />

		<!-- 初始化连接数量 -->
		<property name="initialSize" value="${druid.initialSize}" />
		<!-- 最小空闲连接数 -->
		<property name="minIdle" value="${druid.minIdle}" />
		<!-- 最大并发连接数 -->
		<property name="maxActive" value="${druid.maxActive}" />
		<!-- 配置获取连接等待超时的时间 -->
		<property name="maxWait" value="${druid.maxWait}" />

		<!-- 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒 -->
		<property name="timeBetweenEvictionRunsMillis"
			value="${druid.timeBetweenEvictionRunsMillis}" />

		<!-- 配置一个连接在池中最小生存的时间，单位是毫秒 -->
		<property name="minEvictableIdleTimeMillis"
			value="${druid.minEvictableIdleTimeMillis}" />
		<property name="validationQuery"
			value="${druid.validationQuery}" />
		<property name="testWhileIdle" value="${druid.testWhileIdle}" />
		<property name="testOnBorrow" value="${druid.testOnBorrow}" />
		<property name="testOnReturn" value="${druid.testOnReturn}" />

		<!-- 打开PSCache，并且指定每个连接上PSCache的大小 如果用Oracle，则把poolPreparedStatements配置为true，mysql可以配置为false。 -->
		<property name="poolPreparedStatements"
			value="${druid.poolPreparedStatements}" />
		<property name="maxPoolPreparedStatementPerConnectionSize"
			value="${druid.maxPoolPreparedStatementPerConnectionSize}" />

		<!-- 配置监控统计拦截的filters -->
		<property name="filters" value="${druid.filters}" />
	</bean>

<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource" />
    </bean>
    <bean id="transactionTemplate" class="org.springframework.transaction.support.TransactionTemplate">
        <property name="transactionManager" ref="transactionManager" />
        <property name="timeout" value="30"/> <!-- in seconds -->
        <property name="propagationBehaviorName" value="PROPAGATION_REQUIRED"/>
        <property name="isolationLevelName" value="ISOLATION_DEFAULT"/>
    </bean>
    <tx:annotation-driven transaction-manager="transactionManager" />

    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="configLocation" value="classpath:/mybatis.xml"></property>
        <property name="dataSource" ref="dataSource" />
        <property name="mapperLocations">
            <array>
                <value><![CDATA[classpath*:com/ps/druiddemo/dao/sqlmap/**/*Mapper.xml]]></value>
            </array>
        </property>
    </bean>
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.ps.druiddemo.dao.mapper" />
        <property name="nameGenerator">
            <bean class="org.springframework.beans.factory.support.DefaultBeanNameGenerator"></bean>
        </property>
    </bean>
</beans>

```

2. **新建mybatis.xml配置**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//ibatis.apache.org//DTD Config 3.0//EN"
        "http://ibatis.apache.org/dtd/ibatis-3-config.dtd">
<configuration>
    <settings>
        <setting name="cacheEnabled" value="false"/>
        <setting name="lazyLoadingEnabled" value="false"/>
        <setting name="aggressiveLazyLoading" value="true"/>
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>
</configuration> 
```

### 7.添加spring配置

 **新建druid-test.spring.xml导入数据源配置：**

```xml
<import resource="classpath:datasource.xml"/>
<context:component-scan base-package="com.ps.druiddemo" />
```

**com.ps.druiddemo.config包下新增两个类否则配置不会被加载**

Config.java

```java
package com.ps.druiddemo.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.ImportResource;

@Configuration
@ImportResource(locations={"classpath:druid-test.spring.xml"})
public class Config {
}
```

DataSourceConfiguration.java

```java
package com.ps.druiddemo.config;

import javax.sql.DataSource;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import com.alibaba.druid.spring.boot.autoconfigure.DruidDataSourceBuilder;

@Configuration
@EnableTransactionManagement // 启注解事务管理，等同于xml配置方式的 <tx:annotation-driven />
public class DataSourceConfiguration {
    @Bean
    public DataSource dataSource(org.springframework.core.env.Environment environment) {
        return DruidDataSourceBuilder
                .create()
                .build(environment, "spring.datasource.druid.");
    }
}

```

### 8.编写Controller接口

**UserInfoController.java**

```java
package com.ps.druiddemo.controller;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import com.ps.druiddemo.dao.dto.UserInfo;
import com.ps.druiddemo.dao.dto.UserInfoCriteria;
import com.ps.druiddemo.dao.mapper.UserInfoMapper;

@Controller
@RequestMapping("/user")
public class UserInfoController {
    @Autowired
    UserInfoMapper userInfoMapper;

    @ResponseBody
    @RequestMapping("/getAll")
    public List<UserInfo> getAllUser() {
        return userInfoMapper.selectByExample(new UserInfoCriteria());
    }
}
```

### 9.启动项目测试接口

![20180828-druid-demo-start](http://ptpan.top/img/post/2018/20180828-druid-demo-start.png)

![20180826-druid-demo-test](E:\Img\md-img\20180826-druid-demo-test.png)

![20180826-console](http://ptpan.top/img/post/2018/20180826-console.png)



### 10.数据库密码加密

**使用druid-1.1.10.jar加密密码**

```
C:\Software\apache-maven-3.5.3\repository\com\alibaba\druid\1.1.10>java -cp druid-1.1.10.jar com.alibaba.druid.filter.config.ConfigTools 123456
privateKey:MIIBUgIBADANBgkqhkiG9w0BAQEFAASCATwwggE4AgEAAkEAkijO471RzUbBHWTAwj4TV4rN6Gvno0aJmfNqbFJeeoTLLWlwQK5ZN9EV/eEp3KvRi6hS6PzJdFFJifwlkXAj9QIDAQABAkAP59j797JbQIPrivdfLBo2wKg/zt5aama3FkJSn3QgqQBPM9Vc967c718WuqPo1+dqB2qhWCU5/icjkTEG8NRRAiEA4NlQCVMRokij+dRwGVH2bCTmQzCdK0ybYnEszntOt7MCIQCmaKAzpFmGq3h827QqHgbLtAuMv+IuhKDBj65LvTBhtwIffLgNrR5mqZ2hVvJ/O4w7I8FT9/D/PQVBK1mbgOzkvQIgCT9jN7t4Zi19QqMK/hQxGHzm72lybldcf6U2cGsRFz0CIESmGij7EIsrCl/3uGrI3xTRSxjc6e2hXqWMXcQuy351
publicKey:MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAJIozuO9Uc1GwR1kwMI+E1eKzehr56NGiZnzamxSXnqEyy1pcECuWTfRFf3hKdyr0YuoUuj8yXRRSYn8JZFwI/UCAwEAAQ==
password:XPKL0bkj/9ECEtf4Jl/1k//FCWyM0q8iY+V34uqyhTc/nA67KZ4BVC1yq/hSnF9Ihyxq4+IyanH7yPYEe/STCQ==
```

**datasource.xml配置修改**

```
<property name="password" value="${druid.password}" />
		<property name="connectionProperties" value="config.decrypt=true;config.decrypt.key=${druid.publicKey}" />
```

**application.properties配置修改**

```properties
druid.filters=wall,stat,config
druid.publicKey=MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAJIozuO9Uc1GwR1kwMI+E1eKzehr56NGiZnzamxSXnqEyy1pcECuWTfRFf3hKdyr0YuoUuj8yXRRSYn8JZFwI/UCAwEAAQ==
druid.password=XPKL0bkj/9ECEtf4Jl/1k//FCWyM0q8iY+V34uqyhTc/nA67KZ4BVC1yq/hSnF9Ihyxq4+IyanH7yPYEe/STCQ==

```

### 11.Druid监控配置

1. **application.properties添加配置**

```properties
##监控
#是否启用StatFilter默认值true
spring.datasource.druid.filter.stat.log-slow-sql= true
spring.datasource.druid.filter.stat.slow-sql-millis=1000
spring.datasource.druid.filter.stat.merge-sql=true
spring.datasource.druid.filter.stat.db-type=mysql
spring.datasource.druid.filter.stat.enabled=true


#spring.datasource.druid.filters=slf4j
#配置slf4j
spring.datasource.druid.filter.slf4j.enabled=true
spring.datasource.druid.filter.slf4j.connection-log-enabled=true
spring.datasource.druid.filter.slf4j.connection-close-after-log-enabled=true
spring.datasource.druid.filter.slf4j.connection-commit-after-log-enabled=true
spring.datasource.druid.filter.slf4j.connection-connect-after-log-enabled=true
spring.datasource.druid.filter.slf4j.connection-connect-before-log-enabled=true
spring.datasource.druid.filter.slf4j.connection-log-error-enabled=true
spring.datasource.druid.filter.slf4j.data-source-log-enabled=true
spring.datasource.druid.filter.slf4j.result-set-log-enabled=true
spring.datasource.druid.filter.slf4j.statement-log-enabled=true

#配置web-stat-filter
spring.datasource.druid.web-stat-filter.enabled=true
spring.datasource.druid.web-stat-filter.url-pattern=/*
spring.datasource.druid.web-stat-filter.exclusions=*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*
spring.datasource.druid.stat-view-servlet.allow=127.0.0.1
#spring.datasource.druid.stat-view-servlet.allow=
#你可以配置principalSessionName，使得druid能够知道当前的cookie的用户是谁
spring.datasource.druid.web-stat-filter.principal-cookie-name=admin
#你可以配置principalSessionName，使得druid能够知道当前的session的用户是谁
spring.datasource.druid.web-stat-filter.principal-session-name=admin
#置profileEnable能够监控单个url调用的sql列表。
spring.datasource.druid.web-stat-filter.profile-enable=true
#session统计功能
spring.datasource.druid.web-stat-filter.session-stat-enable=true
#最大session数
spring.datasource.druid.web-stat-filter.session-stat-max-count=100000

#配置StatViewServlet
spring.datasource.druid.stat-view-servlet.enabled=true
spring.datasource.druid.stat-view-servlet.login-username=admin
spring.datasource.druid.stat-view-servlet.login-password=admin
spring.datasource.druid.stat-view-servlet.url-pattern=/druid/*
spring.datasource.druid.stat-view-servlet.reset-enable=true

#配置wall filter
spring.datasource.druid.filter.wall.enabled=true
spring.datasource.druid.filter.wall.db-type=mysql
spring.datasource.druid.filter.wall.config.alter-table-allow=false
spring.datasource.druid.filter.wall.config.truncate-allow=false
spring.datasource.druid.filter.wall.config.drop-table-allow=false
#是否允许非以上基本语句的其他语句，缺省关闭，通过这个选项就能够屏蔽DDL
spring.datasource.druid.filter.wall.config.none-base-statement-allow=false
#检查UPDATE语句是否无where条件，这是有风险的，但不是SQL注入类型的风险
spring.datasource.druid.filter.wall.config.update-where-none-check=true
#SELECT ... INTO OUTFILE 是否允许，这个是mysql注入攻击的常见手段，缺省是禁止的
spring.datasource.druid.filter.wall.config.select-into-outfile-allow=false
#是否允许调用Connection.getMetadata方法，这个方法调用会暴露数据库的表信息
spring.datasource.druid.filter.wall.config.metadata-allow=true
#对被认为是攻击的SQL进行LOG.error输出
spring.datasource.druid.filter.wall.log-violation=true
#对被认为是攻击的SQL抛出SQLExcepton
spring.datasource.druid.filter.wall.throw-exception=true

#配置spring关联
#设置使用Cglib进行代理，因为部分需要代理的不是接口不适用于JDK动态代理，会报错
spring.aop.proxy-target-class=true
#配置Druid监控Spring包方法的调用
spring.datasource.druid.aop-patterns=packages
```

2. **重启应用**

3. **浏览器访问[http://localhost:9090/druid/login.html](http://localhost:9090/druid/login.html)**

   ![20180828-druid-monitor](http://ptpan.top/img/post/2018/20180828-druid-monitor.png)



### 12.[项目源码](https://github.com/komono/druid-demo)

