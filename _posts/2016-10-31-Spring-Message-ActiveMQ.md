---
layout: post
title:  "Spring-Boot-Security-Mail"
categories: spring
tag: spring
---

* content
{:toc}


> 首先应在操作系统安装ActiveMQ

## jar依赖

``` xml
<dependency>
  <groupId>org.apache.activemq</groupId>
  <artifactId>activemq-all</artifactId>
  <version>5.14.1</version>
</dependency>
```

## 消息发送

``` java
@Service
public class SendMessageServiceImpl implements SendMessageService {
    @Autowired
    private JmsTemplate jmsTemplate;
    public void sendMessage(final String message) {
        String destination = jmsTemplate.getDefaultDestination().toString();
        if (destination != null) {
            System.out.println("destination is "+destination);
        } else {
            System.out.println("destination is null");
        }
        jmsTemplate.send(new MessageCreator() {
            public Message createMessage(Session session) throws JMSException {
                return session.createTextMessage(message);
            }
        });
    }
}
```

## 消息接收（手动）

> 这种方式就是不使用QueueListenerContainer，自己手动触发消息接收

### java代码

``` java
@Service
public class ReceiveMessageImpl implements ReceiveMessageService{
    @Autowired
    private JmsTemplate jmsTemplate;

    public String receiveMessage() {
        TextMessage textMessage = (TextMessage) jmsTemplate.receive();
        try {
            return textMessage.getText();
        } catch (JMSException e) {
            throw JmsUtils.convertJmsAccessException(e);
        }
    }
}
```

### 配置文件

需要在web.xml文件中添加配置文件位置

#### web.xml

``` xml
<!-- 读取spring配置文件 -->
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>
      classpath:conf/spring-config.xml
      classpath:conf/ActiveMQ.xml
    </param-value>
  </context-param>
```
#### ActiveMQ.xml

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="connectionFactory" class="org.apache.activemq.spring.ActiveMQConnectionFactory"
          p:brokerURL="tcp://localhost:61616"
          p:userName="admin"
          p:password="admin"/>


    <bean id="queueDestination" class="org.apache.activemq.command.ActiveMQQueue">
        <constructor-arg value="message.queue" />
    </bean>

<!--    <bean id="topic" class="org.apache.activemq.command.ActiveMQTopic">
        <constructor-arg value="message.topic" />
    </bean>-->

    <bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
        <property name="connectionFactory" ref="connectionFactory"/>
        <property name="defaultDestination" ref="queueDestination"/>
        <property name="pubSubDomain" value="false"/>
    </bean>

    <!--使用QueueListenerContainer-->
<!--    <bean id="queueListenerContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
        <property name="connectionFactory" ref="connectionFactory"/>
        <property name="destination" ref="queueDestination"  />
        <property name="messageListener" ref="queueMessageListener"/>
    </bean>-->
</beans>
```

#### Controller

``` java
@Controller
public class MessageController {
    @Autowired
    private ReceiveMessageService receiveMessageService;
    @Autowired
    private SendMessageService sendMessageService;

    @RequestMapping("/producer")
    public String  producer(String text) {
        System.out.println(text);
        sendMessageService.sendMessage(text);
        return "redirect:index.html";
    }
    @RequestMapping("/consumer")
    public String consumer() {
        String message = receiveMessageService.receiveMessage();
        System.out.println("Receive message is: " + message);
        return "redirect:index.html";
    }
}
```

#### index.html

``` html
<form action="/producer" method="post">
    <input type="text" name="text">
    <input type="submit" value="提交">
</form>
<a href="/consumer">consumer</a>
```

#### 结果

输入并点击提交，发送消息，消息输入为2,3，345346。三次点击consumer，消息输出。这种方式为手动接收消息的方式。

```
2
destination is queue://message.queue
3
destination is queue://message.queue
345346
destination is queue://message.queue

Receive message is: 2
Receive message is: 3
Receive message is: 345346
```

## 消息接收（自动）

### java代码

> 实现onMessage，将自动处理发送来的消息

``` java
@Service
public class QueueMessageListener implements MessageListener {
    public void onMessage(Message message) {
        TextMessage textMessage = (TextMessage) message;
        try {
            System.out.println("From listener,message is "+textMessage.getText());
        } catch (JMSException e) {
            e.printStackTrace();
        }
    }
}
```

### ActiveMQ.xml

``` xml
<!--使用QueueListenerContainer-->
<bean id="queueListenerContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
    <property name="connectionFactory" ref="connectionFactory"/>
    <property name="destination" ref="queueDestination"  />
    <property name="messageListener" ref="queueMessageListener"/>
</bean>
```

### 结果

只要一发送消息，消息便立即被自动取出并处理。

``` 
124
destination is queue://message.queue
From listener,message is 124
234
destination is queue://message.queue
From listener,message is 234
345
destination is queue://message.queue
From listener,message is 345
```

## 源代码

[GitHub](https://github.com/Gerry-Yu/spring-message-ActiveMQ)