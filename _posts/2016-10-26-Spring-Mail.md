---
layout: post
title:  "Spring-Mail"
categories: spring
tag: spring
---

* content
{:toc}


## 使用SimpleMailMessage 

### spring-mail.xml

> Spring使用JavaMail,需要添加javax.mail jar包，当然也需要spring-context-support支持

``` xml
    <bean id="mailSender" class="org.springframework.mail.javamail.JavaMailSenderImpl">
        <property name="host" value="smtp.yeah.net" />
        <property name="username" value="发送的邮箱名"/>
        <property name="password" value="登录密码" />
    </bean>

    <bean id="templateMessage" class="org.springframework.mail.SimpleMailMessage">
        <property name="from" value="发送的邮箱名"/>
        <property name="subject" value="主题"/>
    </bean>
```

### Java代码
#### 接口

``` java
public interface UserMailManager {
    void sendUserMail(User user);
}
```

#### 接口实现

``` java
@Component
public class SimpleUserMailManager implements UserMailManager {

    @Resource
    private MailSender mailSender;
    @Resource
    private SimpleMailMessage templateMessage;

    public void sendUserMail(User user) {
        SimpleMailMessage msg = new SimpleMailMessage(this.templateMessage);
        msg.setTo(user.getEmialAddress());
        msg.setText(
                "Dear " + user.getUsername() + ", thank you"
        );
        try {
            this.mailSender.send(msg);
        } catch (MailException e) {
            System.out.println(e.getMessage());
        }
    }
}
```

> 注意这里全部使用自动扫描注解，需要在spring的配置文件中添加以下配置。

``` xml
<context:component-scan base-package="要扫描的包名" />
```

## 使用FreeMarker模板

> 邮件模板有Velocity和FreeMarker，但是Velocity从spring 4.3开始不再支持，所以这里使用FreeMarker。需要导入freemarker.jar

``` xml
<!--freemarker-->
<dependency>
  <groupId>org.freemarker</groupId>
  <artifactId>freemarker</artifactId>
  <version>2.3.23</version>
</dependency>
```

### spring-mail.xml

> 配置JavaMailSender和FreeMarkerConfigurer。FreeMarker的模板文件在/WEN-INF/template/目录下。

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="javaMailSender" class="org.springframework.mail.javamail.JavaMailSenderImpl">
        <property name="host" value="smtp.yeah.net" />
        <property name="username" value="邮箱名"/>
        <property name="password" value="密码" />
    </bean>

    <bean id="freeMarkerConfigurer" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
        <property name="templateLoaderPath" value="/WEB-INF/template/" />
        <property name="freemarkerSettings">
            <props>
                <prop key="template_update_delay">0</prop>
                <prop key="default_encoding">UTF-8</prop>
            </props>
        </property>
    </bean>
</beans>
```

### Java(Service的实现类)

> 这里发送邮件添加了三个附件。获取当前的绝对路径(statics在WEB-INF下)，使用相对目录要出错，不解。

``` java
@Service
public class MailServiceImpl implements MailService {
    @Resource
    private JavaMailSender javaMailSender;
    @Resource
    private FreeMarkerConfigurer freeMarkerConfigurer;

    public void sendUserMail(User user) {
        MimeMessage mimeMessage = null;
        try {
            mimeMessage = javaMailSender.createMimeMessage();
            MimeMessageHelper mimeMessageHelper = new MimeMessageHelper(mimeMessage, true, "UTF-8");
            mimeMessageHelper.setTo(user.getEmailAddress());
            mimeMessageHelper.setSubject("主题");
            mimeMessageHelper.setFrom("发送邮件的邮箱名");
            mimeMessageHelper.setText(getMailText(user), true);


            String filePath = this.getClass().getClassLoader().getResource("../statics/").getPath();
            File file1 = new File(filePath+"test.docx");
            File file2 = new File(filePath+"1.jpg");

            mimeMessageHelper.addAttachment(MimeUtility.encodeWord(file1.getName()), file1);
            mimeMessageHelper.addAttachment("test.docx", file1);
            mimeMessageHelper.addAttachment("1.jpg", file2);

        } catch (Exception e) {
            e.printStackTrace();
        }
        javaMailSender.send(mimeMessage);
    }
    private String getMailText(User user) throws Exception {
        Template template = freeMarkerConfigurer.getConfiguration().getTemplate("hello.ftl");
        Map map = new HashMap();
        map.put("user", user);
        String htmlText = FreeMarkerTemplateUtils.processTemplateIntoString(template, map);
        return htmlText;
    }
}

```

### FreeMarker模板 hello.ftl

> 这里图片链接是直接来自服务器，从本地嵌入图片到邮件里需要另外设置

``` html
<html>
<head></head>
<body>
<h3>Hi ${user.username}, welcome!</h3>
    Your email address is ${user.emailAddress}.<br><br>
    <a href="http://**"><img src="http://**/images/1.jpg"></a><br>
    <br>
    这是中文
</body>
</html>
```
## 源代码

刚学习，写的特别渣，和spring-mail一起写了第一个用户注册、登录验证和权限管理。
[GitHub](https://github.com/Gerry-Yu/spring-security-mail)

> [这是](https://gerry-yu.github.io/2016/10/26/spring-security-UserDetailService/)spring-security重写UserDetailService的  
> [这是](https://gerry-yu.github.io/2016/10/23/Spring-Security-Remember-me/)spring-security的remember-me的  
> [这是](https://gerry-yu.github.io/2016/10/22/Spring-security/)spring-security刚入门的


