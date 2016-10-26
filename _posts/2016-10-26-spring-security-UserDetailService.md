---
layout: post
title:  "Spring-Security-UserDetailService"
categories: spring
tag: spring
---

* content
{:toc}


## MyUserDetailService

> Spring-security默认查询数据库使用JdbcDaoImpl，但是建立的数据库表必须确定，如果自己想怎么建表就怎么建表，至少自己要实现UserDetailsService。实现UserDetailsService就是要自己实现以下函数，根据username返回UserDetail，与输入的username和password对比。
``` java
public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException
```  

> 下面是我自己实现的UserDetailsService，使用MyBatis来访问数据库。层次很分明：首先查看数据库是否有username这个用户，没有直接抛出异常。再查看这个用户的权限，没有权限也直接抛出异常。最后将完整的UserDetail返回。这里使用默认实现UserDetail的org.springframework.security.core.userdetails.User，当然也可以自己实现UserDetail。


``` java
@Service
public class MyUserDetailsService implements UserDetailsService {

    private final MessageSourceAccessor messages = SpringSecurityMessageSource.getAccessor();

    @Resource
    private UserDao userDao;

    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        Map userMap = userDao.loadUsersByUsername(username);

        if(userMap == null || userMap.size() == 0) {
            throw new UsernameNotFoundException(this.messages.getMessage("JdbcDaoImpl.notFound", new Object[]{username}, "Username {0} not found"));
        } else {

            List<String> authoritiesList = userDao.loadUserAuthorities((Integer) userMap.get("userId"));
            if (authoritiesList.size() == 0) {
                throw new UsernameNotFoundException(this.messages.getMessage("JdbcDaoImpl.noAuthority", new Object[]{username}, "User {0} has no GrantedAuthority"));
            }else {
                HashSet<GrantedAuthority> authenticationHashSet = new HashSet<GrantedAuthority>();
                for (String index:authoritiesList) {
                    authenticationHashSet.add(new SimpleGrantedAuthority(index));
                }
                User user = new User((String) userMap.get("username"), (String) userMap.get("password"),authenticationHashSet);
                System.out.println(user.toString());
                return new  User((String) userMap.get("username"), (String) userMap.get("password"),(Boolean) userMap.get("enable"),true, true, true,authenticationHashSet);
            }

        }

    }
}
```

## spring-security.xml

> 这里使用BCrypt对密码加密。

``` xml
<authentication-manager alias="authenticationManager">
    <authentication-provider user-service-ref="myUserDetailsService">
        <password-encoder ref="bcryptEncoder"/>
    </authentication-provider>
</authentication-manager>
<beans:bean name="bcryptEncoder" class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder"/>
```

## UserDao

``` java
Map loadUsersByUsername(String username); //根据username返回user信息
List<String> loadUserAuthorities(int userId); //根据userId返回权限
```

## Mapper

``` xml
    <select id="loadUsersByUsername" resultType="map">
        select userId,username,password,enable from users where username = #{username};
    </select>
    <select id="loadUserAuthorities" resultType="String" >
        select authority from authorities where userId = #{userId};
    </select>
```

## 源代码

刚学习，写的特别渣，和spring-mail一起写了第一个用户注册、登录验证和权限管理。  

[GitHub](https://github.com/Gerry-Yu/spring-security-mail)  

[这是](https://gerry-yu.github.io/2016/10/26/Spring-Mail/)spring-mail的