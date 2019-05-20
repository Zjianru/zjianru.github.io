---
layout:     post
title:      Spring Security填坑记
subtitle:   Spring Security - There is no PasswordEncoder mapped for the id “null”
date:       2019-05-14
author:     Alessio
header-img: img/PostBack_11.jpg
catalog: true
tags:
    - Spring Security
---

## 错误描述

最近想使用 SpringBoot 和 SpringSecurity 完成一个简单的入门 Demo，实现除主页面之外使用 SpringSecurity 验证策略

但是发生了如下错误
```bash
$ curl localhost:8080/hello -u user:password

{
	"timestamp":"2019-05-14T15:03:49.322+0000",
	"status":500,
	"error":"Internal Server Error",
	"message":"There is no PasswordEncoder mapped for the id \"null\"",
	"path":"/hello"
}
```
IDEA 控制台报错如下：
```
[dispatcherServlet] in context with path [] threw exception

java.lang.IllegalArgumentException: There is no PasswordEncoder mapped for the id "null"
```

## 解救方式

在查阅资料之后得知

在 Spring Security 5.0之前，默认的 `PasswordEncoder` 是 `NoOpPasswordEncoder`。它需要纯文本密码。

但是在 Spring Security 5.0 中，默认值为 `DelegatingPasswordEncoder`，它需要密码存储格式。

详情见![官方文档](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#pe-dpe)

## 解救代码

进而我们有两种解决方式

### 解救方式一
我们更改代码，在密码之前指定存储格式  `{noop}`
```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAuthentication().withUser("admin").password("{noop}000000").roles("ADMIN");
}
```

### 解救方式二

使用 `UserDetailsServic` 的 `User.withDefaultPasswordEncoder()` 方法，代码如下

```java
@Configuration
public class SpringSecurityConfig extends WebSecurityConfigurerAdapter {

    @Bean
    public UserDetailsService userDetailsService() {

        User.UserBuilder users = User.withDefaultPasswordEncoder();
        InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
        manager.createUser(users.username("user").password("password").roles("USER").build());
        manager.createUser(users.username("admin").password("password").roles("USER", "ADMIN").build());
        return manager;

    }

}
```