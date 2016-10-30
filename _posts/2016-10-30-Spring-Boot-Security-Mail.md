---
layout: post
title:  "Spring-Boot-Security-Mail"
categories: spring
tag: spring
---

* content
{:toc}


## Security

### 导入jar包

``` xml
    <!--security-->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
```
### 配置

> spring-security可以使用xml和java两种方式配置，在学习Spring时使用的是XML配置方式，[here](https://gerry-yu.github.io/2016/10/22/Spring-security/)和[here](https://gerry-yu.github.io/2016/10/23/Spring-Security-Remember-me/)是原来使用XML时的配置。其实都两种方式都非常相似，这里只使用inMemoryAuthentication的方式，没有使用数据库，上次学习spring时在[这里](https://gerry-yu.github.io/2016/10/26/spring-security-UserDetailService/)自己实现了UserDetailService。  

URL的映射如下，这里使用的是`thymeleaf`作为模板，而没有使用JSP和常用的Velocity和FreeMarker，Velocity在spring4.3之后不再支持，thymeleaf是官方推荐的模板。

> 其中前4个映射都使用的是模板，并且在templates目录下，而要返回在public目录下的publictest.html，需要使用forward或者redirect，才能返回。

``` java
@RequestMapping("/")
public String home() {
    return "index";
}
@RequestMapping("/hello")
public String hello() {
    return "hello";
}
@RequestMapping("/login")
public String login() {
    return "login";
}
@RequestMapping("/test")
public String test() {
    return "test";
}
@RequestMapping("/ptt")
public String redirect() {
    return "forward:publictest.html";
}
```

``` java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter{
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/").permitAll()
                .anyRequest().authenticated()
                .and()
            .formLogin()
                .loginPage("/login")
                .permitAll()
                .and()
            .logout()
                .permitAll();
    }

    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
        auth
            .inMemoryAuthentication().withUser("user").password("password").roles("USER").and()
            .withUser("test").password("test").roles("USER");
    }
}
```

## Email

### jar包

``` xml
    <!--mail-->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-mail</artifactId>
    </dependency>

    <!--thymeleaf-->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
```

### application.properties

> 可以在application.properties目录配置Email的相关信息，下面是最简单的配置。

``` properties
spring.mail.host=邮箱的host
spring.mail.username=要发送的邮箱名
spring.mail.password=你的密码
```

### EmailServiceImpl

> 这里邮箱使用的模板也是themeleaf，上次学习spring时发送邮件使用的是FreeMarker([这里](https://gerry-yu.github.io/2016/10/26/Spring-Mail/))。在Spring-Boot，mail和themeleaf的使用非常简单，Spring-Boot已经配置好了JavaMailSender和TemplateEngine，这里直接使用就行了。

``` java
@Service
public class MailServiceImpl implements MailService{

    @Autowired
    private JavaMailSender mailSender;

    @Autowired
    private TemplateEngine templateEngine;

    @Override
    public void sendMail(User user) {
        MimeMessage message = null;
        try {
            message = mailSender.createMimeMessage();
            MimeMessageHelper helper = new MimeMessageHelper(message, true, "UTF-8");
            helper.setFrom("邮箱名");
            helper.setTo("邮箱名");
            helper.setSubject("Test");
            helper.setText(getHtmlText(user), true);
        } catch (Exception e) {
            e.printStackTrace();
        }
        mailSender.send(message);
    }
    private String getHtmlText(User user) {
        Context model = new Context();
        model.setVariable("user",user);
        String htmlText = templateEngine.process("mail",model);
        return htmlText;
    }
}
```

这里使用的themeleaf的模板如下，其实各种模板写法都差不多，有一些区别，感觉themeleaf的写法比较繁琐，没有FreeMarker随意，并且格式有严格要求，比如meta标签后面必须要有/，br后面必须有/。

``` html
<!DOCTYPE html>
<html lang="zh-CN"
      xmlns="http://www.w3.org/1999/xhtml"
      xmlns:th="http://www.thymeleaf.org"
      xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity3">
    <head>
        <meta charset="UTF-8"/>
    </head>
    <body>
        <h1>Hello</h1>
        <p th:text="${user.name}"> 您好，</p>
        <p>您的ID是<span th:text="${user.id}"/></p>
        <br/>
        <img src="http://*/images/1.jpg"/>
    </body>
</html>
```

## 源代码

[GitHub](https://github.com/Gerry-Yu/spring-boot-security-mail)


