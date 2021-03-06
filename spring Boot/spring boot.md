# spring boot
`environment.acceptsProfiles(Profiles.of("dev"));` 判断是否为 dev 环境
- # 原理初探
---
自动装配：
pom.xml
- 核心依赖在父工程当中
    ```
            <parent>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-parent</artifactId>
                <version>2.4.0</version>
                <relativePath/>
            </parent>
    ```
- [**启动器**](https://docs.spring.io/spring-boot/docs/2.4.0/reference/html/using-spring-boot.html#using-boot-starter)(相当于启动场景)
    ```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
    ```
    ---
    web 启动器
    ```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    ```
    > springboot 会将所有的功能场景， 变成一个个启动器
    > 如果要使用什么功能， 就只需要找对应的启动器就可以了 `starter`

- **主程序**
    ![自动装配及原理](_v_attachments/20201113173423482_1811248475/自动装配及原理.svg)
    
    - 结论
    springboot所有自动装配都是在启动的时候扫描并加载 `spring.factories`所有的配置类都在这里面， 但是不一定生效， 要判断条件是否成立， 只要导入了对应的start, 就有对应的其启动器了， 有了启动器， 我们自动装配就会生效， 然后就配置成功 详细见 [自动装配及原理.km](_v_attachments/20201113173423482_1811248475/自动装配及原理.km)
    1. spring-boot-autoconfigure.jar 所有的自动配置都在这个包下
2.  容器中会存在非常多的xxxautoconfig
    
- # spring boot 配置 `application`
    -  yml 类是json(推荐)
    - 通过`@ConfigurationProperties(prefix = "xxx")` 可以直接赋值给一个对象
    ---
    - js303 校验
        > 类似代码提示 校验 `@Validated` 数据校验
        ```
            @Null
            @NotNull
            ...
        ```
    - application 配置文件 优先级从上往
        1. 项目根目录下/config
        2. 项目根目录下/
        3. classpath:/config
        4. classpath:/
        ---
        - `spring.profiles.active=xxx` (properties 多环境配置) 
          
            1. application-xxx
        - yaml 多环境配置
            ```
            server:
              port: 8081
            spring:
              profiles:
                active: dev
            ---
            server:
              port: 8082
            spring:
              profiles: dev
            ---
            server:
              port: 8082
            spring:
              profiles: test
            ```
    ---
    - ## 配置`application.yaml`
        通过看源码配置 yaml文件
        1. spring-boot-autoconfigure-2.4.0.jar!/META-INF/spring.factories
            所有的配置类都在这个文件里面
        2. 打开对应的配置类可以看到
            ```
            @Configuration(proxyBeanMethods = false)
            @EnableConfigurationProperties(ServerProperties.class)
            @ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)
            @ConditionalOnClass(CharacterEncodingFilter.class)
            @ConditionalOnProperty(prefix = "server.servlet.encoding", value = "enabled", matchIfMissing = true)
            public class HttpEncodingAutoConfiguration
            private final Encoding properties; // 对应的配置属性
            ```
        3. 配置 server.servlet.encoding 下的各个属性
        ---
        所有的配置类都有自己的子配置类 比如说这玩意
        ```
        spring: //配置类
          mvc: //  子配置类
            format: // 子配置类
              date-time: yyyy-MM-dd HH:mm:ss //最后是属性
        ```
        ---
        ## 原理
            1. spring boot 启动会加载大量的自动配置类
            2. 看我们需要的功能有没有在 spring boot 默认写好的自动装配类当中 spring-boot-autoconfigure-2.4.0.jar!/META-INF/spring.factories 这个文件下
        3. 在看这个自动配置类配了些啥，会从一个叫properties类中获取一些属性 可以自行更改一些默认选项
        
            ---
            
                ```
                xxxAutoConfigurartion : 自动装配类
                xxxProperties : 装配文件中的相关属性
            ```
        
            ---
            设置 `debug = true` 可以打印配置报告
            ```
---
- # spring boot web 
    静态资源
    - webjars 'localhost:8080/webjars/'
    - public static /** resources 'localhost:8080/'
    - 优先级 resources > static (默认) > public 
    ---
    spring boot web 
    自定义图标  spring.mvc.favicon.enabled=false 2.2 以后 这个被禁用
    只需要在静态资源里面个放个 favicon.ico 覆盖即可 这也是官方推荐的
    
    ---
    
    - ## 模板引擎 ([thymeleaf](https://www.thymeleaf.org/doc/articles/standarddialect5minutes.html))
        >   templates 下的 html 文件 需要导入对应的依赖
        - 传递参数使用()
        ---
        - ${...} ：变量表达式。
        - *{...} ：选择表达式。
        - #{...} ：消息（i18n）表达式。
        - @{...} ：链接（URL）表达式。
        - ~{...} ：片段表达式。
        - 提取公共页面
            1. `th:fragment="topbar"` //提取
            2. `th:replace="~{commons/commons::topbar}"`//使用
    - ## 扩展spring mvc 需要 加上 configuration 注解 然后 实现 `WebMvcConfigurer` 接口
        ```
        @Configuration
        public class MyMvcConfig implements WebMvcConfigurer
        ```
    
        - 国际化 
            需要实现 `LocaleResolver` 接口
            1. 配置i18n文件
            2. 自定义 LocaleResolver 组件
            3. 配置到自定义扩展类中
        - 拦截器
            和mvc 一样 不同的是需要到配置类里面重写 `addInterceptors` 方法 
- # data
    ```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
    ```
    
    yml 中配置 spring : datasource
    
    - ## druid :
        1. 自定义配置类 `@Configuration`
        2. 然后加上druidDataSource bean
        ---
        -  [配置_StatViewServlet配置](https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE_StatViewServlet%E9%85%8D%E7%BD%AE)
            1. 配置 `ServletRegistrationBean`
            2. new 一个 `StatViewServlet` 添加到 bean 中 随便 添加 映射路径 
            3. 设置 管理员 与 密码 
            4. 初始化 `setInitParameters`
            5. 返回该 bean
        - [配置 WebStatFilter 用于采集web-jdbc关联监控的数据](https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE_%E9%85%8D%E7%BD%AEWebStatFilter)
            1. 配置 FilterRegistrationBean
            2. 在 该bean 中 添加 WebStatFilter
            3. 设置 规则 
            4. 初始化 `setInitParameters`
            5. 返回 该 bean
---
- ## SpringSecurity (安全框架)
    - `WebSecurityConfigureAdabter` //自定义 security 策略
    - `AuthenticationManagerBuilder` // 自定义认证
    - `@EnbleWebSecurity` //开启 websecurity 模式 @Enablexxx 开启某个功能
    ---
    1. 自定义类 加入 该注解 `@EnbleWebSecurity`
    2. 定义过滤策略 **更多操作记得看源码**
        ```
            protected void configure(HttpSecurity http) //这是链式编程 具体可以看源码 是怎么使用的
            http.authorizeRequests()定义策略
            .antMatchers()设置路径 
            .permitAll()表示全部都能访问
            .hasRole()设置权限
            
            http.**formLogin**()表示 没有权限跳转到该页面 可以设置登陆页面
            
            http.**logout**().logoutSuccessUrl("/"); 开启注销功能 可以设置登出页面

            http.rememberMe()//开启记住我
            
        ```
    3. 认证
        ```java
        //    认证
            @Override
            protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        //        从内存中读取
                auth.inMemoryAuthentication().passwordEncoder(new BCryptPasswordEncoder())加密规则
                        .withUser("l") 用户
                        .password(new BCryptPasswordEncoder().encode("123456")) 设置密码
                        .roles("vip2", "vip3") 设置权限
                        .and()//设置下一个用户

                auth.userDetailsService//自定义
            }        
        ```
        ---
    - 与 `thymeleaf` 整合
        1. 导入依赖
            ```
            <dependency>
                <groupId>org.thymeleaf.extras</groupId>
                <artifactId>thymeleaf-extras-springsecurity5</artifactId>
            </dependency>
            ```
        2. 加入 xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity4" 约束
        3. `sec:authorize` 判断显示还是不显示
           
            1. `isAuthenticated()`判断是否登陆
            2. `principal` 可以查看当前登陆的角色
---



- ## [Shiro (安全框架)](./shiro.md)
    1. 导入依赖
    2. 配置文件
    3. helloworld
- ## [swagger](./swagger.md) 

- ## [任务](./task.md)
- ## [Dubbo](./Dubbo.md)

- ## [JPA](./SpringJPA.md)