---
layout: post
title:  "Spring整合Struts2"
categories: SSH
tag: SSH
---

* content
{:toc}

## 启动Spring容器

``` xml
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:conf/spring-config.xml</param-value> <!--配置文件在resource/conf下-->
  </context-param>
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>

  <filter>
    <filter-name>struts2</filter-name>
    <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>struts2</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
```

## 两种整合策略

> 两种方式都需要struts2的Spring插件struts2-spring-plugin-xxx.jar，两种方式Action都由Spring插件创建
> Spring容器负责管理Action，利用依赖注入为控制器注入业务逻辑


#### 方式一


Example  

+ LoginAction部分代码

``` java
    private UserService userService;
    public void setUserService(UserService userService) {
        this.userService = userService;
    }
```

以上代码在LoginAction中，UserService用于验证用户输入

+ struts.xml

``` xml
       <action name="login" class="loginAction">
            <result name="success">WEB-INF/success.jsp</result>
            <result name="error">WEB-INF/error.jsp</result>
        </action>
```

> `class="loginAction"` 并不是实际的Action处理类，而是下面将要配置的bean id

+ spring-config.xml

``` xml
    <bean id="userService" class="com.demo.service.impl.UserServiceImpl"/>
    <bean id="loginAction" class="com.demo.action.LoginAction" scope="prototype" p:userService-ref="userService"/>
    <!--xmlns:p="http://www.springframework.org/schema/p"-->
```

> 所以这种方式Struts中的Action只有一个代号，真正的Action在Spring容器中，`userSerive`为LoginAction中setting变量名  
`缺点`：需要在struts.xml文件中配置`伪Action`，还需要在spring配置文件中配置scope和ref，代码冗余。scope="prototype"为每个请求都对应一个Action。

#### 方式二

> 自动装配方式，Action自动从Spring容器中获得业务逻辑组件，这种方式也是默认的，自动装配方式为"byName"

+ strurs.xml

``` xml
   <!-- <constant name="struts.objectFactory.spring.autoWire" value="name"/>-->

    <package name="default" extends="struts-default">
        <action name="login" class="com.demo.action.LoginAction">
            <result name="success">WEB-INF/success.jsp</result>
            <result name="error">WEB-INF/error.jsp</result>
        </action>
    </package>
```

> 修改自动装配方式可以修改<constant name="struts.objectFactory.spring.autoWire" value="name"/>的value值（name,type,auto,constructor）不设置默认为byName。自动装配方式Action的class为实际完整目录

+ spring-config.xml

``` xml
    <bean id="userService" class="com.demo.service.impl.UserServiceImpl"/>
```

> 这里`不需要`配置Action，`userSerive`为LoginAction中setting变量名