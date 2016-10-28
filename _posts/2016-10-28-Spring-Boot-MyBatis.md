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

> MyBatis可以有两种方式配置，一种是全使用注解，一种是使用xml文件配置。这两种配置参考[这里](http://www.mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/index.html)。这里使用注解的方式，所有注解参考[这里](http://www.mybatis.org/mybatis-3/zh/java-api.html)。  
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

## 源代码

其他都和spring差不多，代码[GitHub](https://github.com/Gerry-Yu/spring-boot-mybatis)