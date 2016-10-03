---
layout: post
title:  "struts2-Start"
categories: SSH
tag: SSH
---

* content
{:toc}



## 创建工程

#### 方式一 idea Struts2

+ 创建Struts2
![图1]({{ '/styles/images/20161003-1.jpg' | prepend: site.baseurl  }})
+ Add Marven
![图2]({{ '/styles/images/20161003-2.jpg' | prepend: site.baseurl  }})


### 方式二 Marven Webapp
+ ![图3]({{ '/styles/images/20161003-3.jpg' | prepend: site.baseurl  }})

### 比较
+ 目录结构稍有不同，但不影响
+ 一需要将lib下jar包拷贝到WEB-INF/lib目录下（两个地方都要）
+ 二需要在pom.xml添加依赖包（比较喜欢这种），但是struts.xml需要在resource目录下！！！

## Demo（用户登录）
#### web.xml

``` xml
<filter>
    <filter-name>struts2</filter-name>
    <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>struts2</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

#### struts.xml

``` xml
<package name="default" namespace="/" extends="struts-default"> //package
    <action name="loginAction" class="com.demo.actions.LoginAction"> //action
        <result name="success">WEB-INF/success.jsp</result> //处理结果和物理视图之间对应关系
        <result name="error">WEB-INF/error.jsp</result>
    </action>
</package>
```

#### index.jsp

``` html
<form action="loginAction" method="post">
    <input type="text" name="username" >
    <input type="password" name="password" >
    <input type="submit" value="Submit">
</form>
```

#### LoginAction

``` java
public class LoginAction extends ActionSupport{
    private String username;
    private String password;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    @Override
    public String execute() throws Exception {
        if (getUsername().equals("test") && getPassword().equals("test")) {
            return SUCCESS;
        } else {
            return ERROR;
        }
    }
}
```

#### Demo-GitHub
[Gerry-Yu/strurs2Demo](https://github.com/Gerry-Yu/struts2Demo)