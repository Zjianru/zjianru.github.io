---
layout:     post
title:      SpringEmail 完成邮件发送
subtitle:   
date:       2020-02-26
author:     Alessio
header-img: img/PostBack_04.jpg
catalog: true
tags:
    - java
    - SpringBoot
---
## 起因

最近在写的 demo 里面涉及到使用 java 完成邮件发送、手机验证码发送功能。

因为发送手机验证码等其他交互功能交由第三方（如阿里）处理，且文档齐全，故不进行赘述。

在这里只记录完成邮件发送的配置

## 邮件客户端配置

在邮件客户端上开启 SMTP 

这步为必需

## 项目配置

### 导入 maven 依赖

我在项目中使用 SpringBoot 来进行管理开发，引入 email-stater
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```
### 配置参数
我使用 application.yml 完成项目参数配置  语法如下

```yml
mail:
    host: smtp.sina.com
    port: 465
    username:
    password:
    protocol: smtps
    properties:
        mail.smtp.ssl.enable: true
```

### javaMailSender 代码
```java
@Component
public class MailClient {
	private static final Logger log = LoggerFactory.getLogger(MailClient.class);

	private final JavaMailSender mailSender;

	public MailClient(JavaMailSender mailSender) {
		this.mailSender = mailSender;
	}

	@Value("$(spring.mail.username)")
	private String from;

	public void sendMail(String to, String subject, String content) {
		try {
			MimeMessage message = mailSender.createMimeMessage();
			MimeMessageHelper helper = new MimeMessageHelper(message);
			helper.setFrom(from);
			helper.setText(to);
			helper.setSubject(subject);
			helper.setText(content,true);
			mailSender.send(helper.getMimeMessage());
		} catch (MessagingException e) {
			log.error("send mail fail" + e.getMessage());
		}
	}
}
```
### 测试类

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@ContextConfiguration(classes = CommunityApplication.class)
public class MailTest {
	@Autowired
	private MailClient client;

	@Test
	public void testTextMail() {
		client.sendMail("xx@xx.com", "testTextMail", "testTextMail");
	}
}
```

### 发送 HTML 模板类型

#### HTML 模板

位置 --> `/mail/demo`

使用 thymeleaf 模板引擎

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Mail Test</title>
</head>
<body>
    <p> welcome <span style="color: azure" th:text="${username}"></span>!</p>
</body>
</html>
```

#### 测试类
```java
@RunWith(SpringRunner.class)
@SpringBootTest
@ContextConfiguration(classes = CommunityApplication.class)
public class MailTest {
	@Autowired
	private MailClient client;
	@Autowired
	private TemplateEngine templateEngine;

	@Test
	public void testTextMail() {
		client.sendMail("xx@xx.com", "testTextMail", "testTextMail");
	}
	@Test
	public void testHtmlMail() {
		Context context = new Context();
		context.setVariable("username","sunday");
		String content = templateEngine.process("/mail/demo",context);
		client.sendMail("xx@xx.com", "HTML", content);
	}
```