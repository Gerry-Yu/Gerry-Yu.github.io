---
layout: post
title:  "Spring-Boot-MyBatis"
categories: spring
tag: spring
---

* content
{:toc}


# Spring-Boot整合MyBatis

## Spring-Boot工程

### 方法一

spring-boot工程可以直接在http://start.spring.io/构建需要的spring-boot工程，这里选择Web和MyBatis。使用这种比较方便。

### 方法二

自己构建，就需要pom.xml

``` xml
<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.4.1.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.mybatis.spring.boot</groupId>
			<artifactId>mybatis-spring-boot-starter</artifactId>
			<version>1.1.1</version>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>

		<!-- mysql驱动包 -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.31</version>
		</dependency>

	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
```

## 配置数据库连接池

> 在resources/application.properties文件添加以下信息，这里可以参考[这里](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-configure-datasource)，还可以配置spring.datasource.tomcat.*, spring.datasource.hikari.*等数据库连接池

``` properties
# database
spring.datasource.url=jdbc:mysql://localhost:3306/boot?useUnicode=true&amp;characterEncoding=utf8
spring.datasource.username=root
spring.datasource.password=
spring.datasource.driver-class-name=com.mysql.jdbc.Driver

# dbcp2 pool
spring.datasource.dbcp2.initial-size=2
spring.datasource.dbcp2.max-idle=8
spring.datasource.dbcp2.max-total=10
```  

## MyBatis使用

> MyBatis可以有两种方式配置，一种是全使用注解，一种是使用xml文件配置，当然这两种方式可以结合起来啦。这两种配置参考[这里](http://www.mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/index.html)。这里使用注解的方式，所有注解参考[这里](http://www.mybatis.org/mybatis-3/zh/java-api.html)。  

### 纯注解方式

代码参考以下：这里使用@Results注解代替ResultMap

``` java
@Mapper
public interface UserDao {
    @Results(id = "userResult", value = {
            @Result(property = "id", column = "userId", id = true),
            @Result(property = "name",column = "username")
    })

    @Select("select * from user where userId = #{id} ")
    User getUserById(int id);


    @Results(id = "userResult")
    @ResultMap("userResult")
    @Select("select * from user")
    List<User> getAllUsers();
}
```

### 注解和XML结合

> 如果使用xml配置方式，需要在application.properties文件中添加xml文件配置目录

``` properties
#MYBATIS
mybatis.type-aliases-package=com.demo.model
mybatis.mapper-locations=classpath*:/mapper/*Mapper.xml
```

然后在xml文件中配置所需，这就和以前配置一样了

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.demo.dao.UserDao">
    <resultMap id="userMapXML" type="User">
        <id property="id" column="userId" />
        <result property="name" column="username"/>
    </resultMap>

<!--    <select id="getAllUsers" resultMap="userMapXML">
        select * from user;
    </select>-->
</mapper>
```

## 源代码

其他都和spring差不多，代码[GitHub](https://github.com/Gerry-Yu/spring-boot-mybatis)