# 异步任务

1. 启动类上加上`@EnableAsync`
2. 方法上加 `@Async`

---
# 邮件任务
1. 导入依赖
    ```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-mail</artifactId>
        </dependency>
    ```
2. 配置配置文件
    ```yml
    spring:
      mail:
        username: lul789@qq.com
        password: xxxxx
        host: smtp.qq.com
        #    开启加密验证
        properties:
          mail:
            smtp:
              ssl:
                enable: true
    ```
3. JavaMailSenderImpl.send()方法
    ```java
    //        复杂邮件
            MimeMessage mimeMessage = mailSender.createMimeMessage();
            MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, true, "utf-8");
            helper.setSubject("springboot 邮件任务");
            helper.setText("<p style='color:red'>狂神说springboot邮件任务</p>", true);
    //        添加附件
            File attachment = new File("src/main/resources/application.yml");
            helper.addAttachment("application.yml", attachment);
            helper.setFrom("1411205284@qq.com");
            helper.setTo("2854670929@qq.com");
            mailSender.send(mimeMessage);
    ```
---
# 定时任务
1. 启动类上加上`@EnableScheduling`
2. 方法上加 `@Scheduled` 使用 cron 表达式