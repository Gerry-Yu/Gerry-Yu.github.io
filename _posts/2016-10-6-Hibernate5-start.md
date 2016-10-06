---
layout: post
title:  "Hibernate-Start"
categories: SSH
tag: SSH
---

* content
{:toc}

## Hibernate 文件配置
Hibernate配置文件一般由以下两种，一般采用第二种方式
+ hibernate.properties
+ hibernate.cfg.xml

> 使用hibernate.cfg.xml文件配置时，可以使用注解和.hbm.xml文件两种方式实现表的映射。在这里，采用`注解`方式。使用Intellij idea hibernate.cfg.xml文件应放在resource目录下。   

使用C3P0数据库连接池的配置文件如下：

``` xml
<?xml version='1.0' encoding='UTF-8'?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">

<hibernate-configuration>
    <session-factory>
        <!-- 配置数据库的连接属性 -->
        <property name="connection.driver_class">com.mysql.jdbc.Driver</property>
        <property name="connection.url">jdbc:mysql://localhost:3306/hibernate?useUnicode=true&amp;characterEncoding=gb2312</property>
        <property name="connection.username">root</property>
        <property name="connection.password"></property>

        <property name="hibernate.c3p0.max_size">20</property>
        <property name="hibernate.c3p0.min_size">5</property>
        <property name="hibernate.c3p0.timeout">5000</property>
        <property name="hibernate.c3p0.max_statements">100</property>
        <property name="hibernate.c3p0.idle_test_period">3000</property>
        <!-- 当连接池耗尽并接到获得连接的请求，则新增加连接的数量 -->
        <property name="hibernate.c3p0.acquire_increment">2</property>
        <!-- 是否验证，检查连接 -->
        <property name="hibernate.c3p0.validate">true</property>

        <property name="hibernate.dialect">org.hibernate.dialect.MySQL5Dialect</property>
        <property name="hbm2ddl.auto">update</property>
        <property name="show_sql">true</property>
        <property name="hibernate.format_sql">true</property>

        <!-- 配置持久化映射文件 -->
        <mapping class="com.demo.entity.News"/>
    </session-factory>
</hibernate-configuration>
```

## 持久化注解

@Entity 声明一个Hibernate持久化类  
@Table 指定映射表  
@Id 标识属性，通常映射到表的主键列  
@GeneratedValue 主键生成的策略，省略则主键值需要自己输入。strategy有4个属性值：  

+ GenerationType.AUTO：Hibernate自动选择最适合底层数据库的主键生成策略  
+ GenerationType.IDENTITY 主键值自动增长（MySQL, SQL Server）  
+ GenerationType.SEQUENCE 基于Sequence的主键生成策略，与@SequencetGenerator一起使用（Oracle）  
+ GenerationType.TABLE 使用辅助表生成主键，与@TableGenerator一起使用  


``` java
@Entity
@Table(name="newsInfo")
public class News {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    private String title;
    private String content;
    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
    public String getTitle() {
        return title;
    }
    public void setTitle(String title) {
        this.title = title;
    }
    public String getContent() {
        return content;
    }
    public void setContent(String content) {
        this.content = content;
    }
}
```

## Junit测试
> PO操作在Session管理下。Session由SessionFactory产生，SessionFactory是数据库编译后的内存镜像。SessionFactory由Configuration对象生成，Configuration加载Hibernate配置文件。

``` java
    @Test
    public void InsertNews() {
        Configuration configuration = new Configuration().configure();
        SessionFactory sessionFactory = configuration.buildSessionFactory();
        Session session = sessionFactory.openSession();
        Transaction transaction = session.beginTransaction();

        News news = new News();

        news.setTitle("testTitle");
        news.setContent("TestContent");

        System.out.println(news);

        session.save(news);
        transaction.commit();
        session.close();
        sessionFactory.close();
    }
````












