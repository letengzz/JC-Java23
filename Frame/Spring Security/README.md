# Spring Security

## 认证授权基本概念

### Authentication 认证

认证是为了保护系统的隐私数据与资源，用户的身份合法方可访问该系统的资源。

**认证**：**用户认证就是判断一个用户的身份是否合法的过程。**

常见的用户身份认证方式：

- 用户名密码登录
- 二维码登录
- 手机短信登录
- 指纹认证
- 人脸识别
- 等等...

### Session 会话

用户认证通过后，为了避免用户的每次操作都进行认证可将用户的信息保存在会话中。会话就是系统为了保持当前用户的登录状态所提供的机制，**常见的有基于session方式、基于token方式**等。

#### 基于session的认证方式

交互流程：用户认证成功后，在服务端生成用户相关的数据保存在session(当前会话)中，发给客户端的sesssion_id 存放到 cookie 中，这样用户客户端请求时带上 session_id 就可以验证服务器端是否存在 session 数据，以此完成用户的合法校验，当用户退出系统或session过期销毁时,客户端的session_id也就无效了。

#### 基于token的认证方式

交互流程：用户认证成功后，服务端生成一个token发给客户端，客户端可以放到 cookie 或 localStorage等存储中，每次请求时带上 token，服务端收到token通过验证后即可确认用户身份。可以使用Redis 存储用户信息（分布式中**共享session**）。

基于session的认证方式由Servlet规范定制，服务端要存储session信息需要占用内存资源，客户端需要支持cookie；基于token的方式则一般不需要服务端存储token，并且不限制客户端的存储方式。**如今移动互联网时代更多类型的客户端需要接入系统，系统多是采用前后端分离的架构进行实现，所以基于token的方式更适合**。

### Authorization 授权

因为不同的用户可以访问的资源是不一样的，需要控制被访问的资源。

授权： **授权是用户认证通过后，根据用户的权限来控制用户访问资源的过程。**

**拥有资源的访问权限则正常访问，没有权限则拒绝访问。**

### RBAC(Role-Based Access Control) 基于角色的访问控制

![img](./assets/clip_image002.jpg)

把权限打包给角色(角色拥有一组权限)，分配给用户(用户拥有多个角色)。

最少包括五张表 （用户表、角色表、用户角色表、权限表、角色权限表）

## Spring Security概述

Spring Security是一个能够为基于Spring的**声明式（注解）的安全访问控制解决方案的安全框架**。它提供了一组可以在Spring应用上下文中配置的Bean，充分利用了Spring IoC，DI(控制反转Inversion of Control ,DI:Dependency Injection 依赖注入)和AOP(面向切面编程)功能，提供了一套 Web 应用安全性的完整解决方案。

一般来说，Web 应用的安全性包括**用户认证（Authentication）和用户授权（Authorization）**两个部分，这两点也是 SpringSecurity 重要核心功能。

- 用户认证指的是：验证某个用户是否为系统中的合法主体，也就是说用户能否访问该系统。用户认证一般要求用户提供用户名和密码，系统通过校验用户名和密码来完成认证过程。

  **通俗点说就是系统认为用户是否能登录**

- 用户授权指的是验证某个用户是否有权限执行某个操作。在一个系统中，不同用户所具有的权限是不同的。比如对一个文件来说，有的用户只能进行读取，而有的用户可以进行修改。一般来说，系统会为不同的用户分配不同的角色，而每个角色则对应一系列的权限。

  **通俗点讲就是系统判断用户是否有权限去做某些事情。**

Spring Security 核心组件有 Authentication(认证/身份验证)、AuthenticationProvider(认证
提供者)、AuthenticationManager(认证管理者)

官方网站：https://spring.io/projects/spring-security  

### Spring Security 历史

Spring Security 开始于 2003 年年底，spring 的 acegi 安全系统。 起因是 Spring开发者邮件列表中的一个问题，有人提问是否考虑提供一个基于 spring 的安全实现。

Spring Security 以"The Acegi Secutity System for Spring" 的名字始于 2013 年晚些时候。一个问题提交到 Spring 开发者的邮件列表，询问是否已经有考虑一个基于 Spring 的安全性社区实现。那时候 Spring 的社区相对较小（相对现在）。实际上 Spring 自己在2013 年只是一个存在于 ScourseForge 的项目，这个问题的回答是一个值得研究的领域，虽然目前时间的缺乏组织了我们对它的探索。

考虑到这一点，一个简单的安全实现建成但是并没有发布。几周后，Spring 社区的其他成员询问了安全性，这次这个代码被发送给他们。其他几个请求也跟随而来。到 2014 年一月大约有 20 万人使用了这个代码。这些创业者的人提出一个 SourceForge 项目加入是为了，这是在 2004 三月正式成立。

在早些时候，这个项目没有任何自己的验证模块，身份验证过程依赖于容器管理的安全性和 Acegi 安全性。而不是专注于授权。开始的时候这很适合，但是越来越多的用户请求额外的容器支持。容器特定的认证领域接口的基本限制变得清晰。还有一个相关的问题增加新的容器的路径，这是最终用户的困惑和错误配置的常见问题。

Acegi 安全特定的认证服务介绍。大约一年后，Acegi 安全正式成为了 Spring 框架的子项目。1.0.0 最终版本是出版于 2006 -在超过两年半的大量生产的软件项目和数以百计的改进和积极利用社区的贡献。

Acegi 安全 2007 年底正式成为了 Spring 组合项目，更名为"Spring Security"。

### 同款产品对比

#### Spring Security

Spring Security是Spring 技术栈的组成部分。

通过提供完整可扩展的认证和授权支持保护你的应用程序。

**SpringSecurity 特点：**

- 和 Spring 无缝整合。


- 全面的权限控制。


- 专门为 Web 开发而设计。

  - 旧版本不能脱离 Web 环境使用。
  - 新版本对整个框架进行了分层抽取，分成了核心模块和 Web 模块。单独引入核心模块就可以脱离 Web 环境。

- 重量级。


#### Shiro

Apache 旗下的轻量级权限控制框架。

**特点：**

- 轻量级。Shiro 主张的理念是把复杂的事情变简单。针对对性能有更高要求


的互联网应用有更好表现。

- 通用性。
  - 好处：不局限于 Web 环境，可以脱离 Web 环境使用。
  - 缺陷：在 Web 环境下一些特定的需求需要手动编写代码定制。

Spring Security 是 Spring 家族中的一个安全管理框架，实际上，在 Spring Boot 出现之前，Spring Security 就已经发展了多年了，但是使用的并不多，安全管理这个领域，一直是 Shiro 的天下。

相对于 Shiro，在 SSM 中整合 Spring Security 都是比较麻烦的操作，所以，Spring Security 虽然功能比 Shiro 强大，但是使用反而没有 Shiro 多(Shiro 虽然功能没有Spring Security 多，但是对于大部分项目而言，Shiro 也够用了)。

自从有了 Spring Boot 之后，Spring Boot 对于 Spring Security 提供了自动化配置方案，可以使用更少的配置来使用 Spring Security。

因此，一般来说，常见的安全管理技术栈的组合是这样的：

- SSM + Shiro


- Spring Boot/Spring Cloud + Spring Security

**以上只是一个推荐的组合而已，如果单纯从技术上来说，无论怎么组合，都是可以运行的**

## Spring Security实现权限

要对Web资源进行保护，最好的办法莫过于Filter
要想对方法调用进行保护，最好的办法莫过于AOP。

Spring Security进行认证和鉴权的时候,就是利用的一系列的Filter来进行拦截的：

![img](https://img-blog.csdnimg.cn/20201231155747261.png)

如图所示，一个请求想要访问到API就会从左到右经过蓝线框里的过滤器，其中**绿色部分是负责认证的过滤器，蓝色部分是负责异常处理，橙色部分则是负责授权**。进过一系列拦截最终访问到我们的API。

只需要重点关注两个过滤器即可：`UsernamePasswordAuthenticationFilter`负责登录认证，`FilterSecurityInterceptor`负责权限授权。

说明：**Spring Security的核心逻辑全在这一套过滤器中，过滤器里会调用各种组件完成功能，掌握了这些过滤器和组件你就掌握了Spring Security**！这个框架的使用方式就是对这些过滤器和组件进行扩展。

### 登录认证

1. 使用脚手架创建SpringBoot项目，并添加依赖：

   ![image-20230421133239740](./assets/image-20230421133239740.png)

   spring-boot-starter-security依赖：

   ```xml
   <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-security</artifactId>
   </dependency>
   ```

   引入spring-boot-starter-security依赖后，项目中除登录退出外所有资源都会被保护起来

2. 创建三个controller：

   > StudentController.java

   ```java
   @RestController
   @RequestMapping("/student")
   public class StudentController {
       @GetMapping("/query")
       public String queryInfo(){
           return "I am a student,My name is Eric!";
       }
   }
   ```

   > TeacherController.java

   ```java
   @RestController
   @RequestMapping("/teacher")
   public class TeacherController {
       @GetMapping("/query")
       public String queryInfo(){
           return "I am a teacher,My name is Thomas!";
       }
   }
   ```

   > AdminController.java

   ```java
   @RestController
   @RequestMapping("/admin")
   public class AdminController {
       @GetMapping("/query")
       public String queryInfo(){
           return "I am a administrator, My name is Obama!";
       }
   }
   ```

3. 启动程序，并访问 http://localhost:8080/student/query 会跳转到登录页面：

   ![image-20230421134318049](./assets/image-20230421134318049.png)

   运行结果说明：spring Security默认拦截了所有请求，但登录退出不拦截。

#### 登录与退出系统

**认证（登录）用户可以访问所有资源，不经过认证用户任何资源也访问不了。**

**登录系统**：使用默认用户user登录系统，密码是随机生成的UUID字符串，可以在控制台（console）上找到：

![image-20230421134535357](./assets/image-20230421134535357.png)

登录后的用户均可以正常访问：http://localhost:8080/student/query、http://localhost:8080/teacher/query、http://localhost:8080/admin/query  

![image-20230421135008411](./assets/image-20230421135008411.png)

**退出系统**：访问 http://localhost:8080/logout 单击Log Out 按钮，成功退出：

![image-20230421135246200](./assets/image-20230421135246200.png)

![image-20230421135326049](./assets/image-20230421135326049.png)

#### 使用配置文件配置用户名和密码

所有资源均已保护，但是用户只用一个，密码是随机的，只能在开发环境使用。可以通过配置文件配置用户和密码，解决了使用随机生成密码的问题。

添加Spring Security 配置信息：

> application.yaml

```yaml
spring:
  security:
    user:
      name: admin
      password: 123456
```

重新启动并访问 http://localhost:8080/student/query 。使用配置文件中的用户名和密码可以正常访问。配置文件中配置用户后，默认的user用户就没有了。

**登录用户原理**：可以看到默认用户名为user，密码为使用UUID随机生成的字符串：

![image-20230421140556272](./assets/image-20230421140556272.png)

#### 基于内存的多用户管理

Spring Security配置文件中默认配置用户是单一的用户，大部分系统都有多个用户，需要配置多个用户。

![img](./assets/clip_image002-1682259983037-1.jpg)

1. 添加配置类MySecurityUserConfig：

   ```java
   @Configuration
   public class MySecurityUserConfig {
   
       @Bean
       public UserDetailsService userDetailService() {
           //使用org.springframework.security.core.userdetails.User类来定义用户
   
           //定义两个用户
           UserDetails user1 = User.builder().username("eric").password("123456").roles("student").build();
           UserDetails user2 = User.builder().username("thomas").password("123456").roles("teacher").build();
           //创建两个用户
           InMemoryUserDetailsManager userDetailsManager = new InMemoryUserDetailsManager();
           userDetailsManager.createUser(user1);
           userDetailsManager.createUser(user2);
           return userDetailsManager;
       }
   }
   ```

2. 启动程序，登录页面输入用户名(thomas)和密码(123456)，然后单击登录后，控制台报错：

   ![image-20230421142456866](./assets/image-20230421142456866.png)

   报错原因：**因为spring Sercurity强制要使用密码加密，当然也可以不加密，但是官方要求是不管你是否加密，都必须配置一个密码编码（加密）器**

3. 添加密码加密器 bean 但是不对密码加密，在MySecurityUserConfig类中加入bean：

   ```java
   /*
   * 从 Spring5 开始，强制要求密码要加密
   * 如果非不想加密，可以使用一个过期的 PasswordEncoder 的实例 NoOpPasswordEncoder，
   * 但是不建议这么做，毕竟不安全。
   * @return
   */
   @Bean
   public PasswordEncoder passwordEncoder(){
   	//不对密码进行加密，使用明文
       return NoOpPasswordEncoder.getInstance();
   }
   ```

   重启程序再次使用thomas/123456登录测试，可以登录正常访问了。

   使用admin/123456登录，登录不成功，说明：**只要添加了安全配置类，那么在yml里面的配置就失效了**

#### 密码处理

##### 加密方案

密码加密一般使用散列函数，又称散列算法，哈希函数，这些函数都是单向函数(从明文到密文，反之不行)。常用的散列算法有MD5和SHA

Spring Security提供多种密码加密方案，基本上都实现了PasswordEncoder接口，官方推荐使用BCryptPasswordEncoder

**使用 BCryptPasswordEncoder**

特点：相同的字符串加密之后的结果都不一样，但是比较的时候是一样的，因为加了盐（salt）了。

创建测试类PasswordEncoderTest：

```java
import static org.junit.jupiter.api.Assertions.assertTrue;

@Slf4j
public class PasswordEncoderTest {
    /**
     * 开发代码时不允许使用main方法测试，而是使用单元测试来测试
     * 代码中一般不允许使用System.out.println 直接输出，而是使用日志输出
     * 单元测试尽量使用断言，而不是使用System.out.println输出
     */
    @Test
    @DisplayName("测试加密类BCryptPasswordEncoder")
    void testPassword() {
        BCryptPasswordEncoder bCryptPasswordEncoder = new BCryptPasswordEncoder();
        //加密 (明文到密文)
        String encode1 = bCryptPasswordEncoder.encode("123456");
        log.info("encode1:{}",encode1);
        String encode2 = bCryptPasswordEncoder.encode("123456");
        log.info("encode2:"+encode2);
        String encode3 = bCryptPasswordEncoder.encode("123456");
        log.info("encode3:"+encode3);
        //匹配方法，判断明文经过加密后是否和密文一样 参数1：明文 参数2：密文
        boolean result1 = bCryptPasswordEncoder.matches("123456", encode1);
        boolean result2 = bCryptPasswordEncoder.matches("123456", encode1);
        boolean result3 = bCryptPasswordEncoder.matches("123456", encode1);
//        log.info(result1+":"+result2+":"+result3);
        //单元测试推荐使用断言判断 assertTrue期望值：ture就是成功√ false就是错误×
        assertTrue(result1);
        assertTrue(result2);
        assertTrue(result3);
    }
}
```

**修改加密器bean**

修改MySecurityUserConfig类中的加密器bean：

```java
@Bean
public PasswordEncoder passwordEncoder(){
    //使用加密算法对密码进行加密
	return new BCryptPasswordEncoder();
}
```

对系统中的用户密码进行加密：

```java
@Configuration
public class MySecurityUserConfig {

    @Bean
    public UserDetailsService userDetailService() {
        //使用org.springframework.security.core.userdetails.User类来定义用户

        //定义两个用户
        UserDetails user1 = User.builder().username("eric").password(passwordEncoder().encode("123456")).roles("student").build();
        UserDetails user2 = User.builder().username("thomas").password(passwordEncoder().encode("123456")).roles("teacher").build();
        //创建两个用户
        InMemoryUserDetailsManager userDetailsManager = new InMemoryUserDetailsManager();
        userDetailsManager.createUser(user1);
        userDetailsManager.createUser(user2);
        return userDetailsManager;
    }
    /**
     * 从 Spring5 开始，强制要求密码要加密
     * 如果非不想加密，可以使用一个过期的 PasswordEncoder 的实例 NoOpPasswordEncoder，
     * 但是不建议这么做，毕竟不安全。
     * @return
     */
//    @Bean
//    public PasswordEncoder passwordEncoder(){
//        //不对密码进行加密，使用明文
//        return NoOpPasswordEncoder.getInstance();
//    }
    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }
}
```

启动程序，访问 http://localhost:8080/student/query ，使用thomas/123456登录测试

#### 查看当前登录用户信息及配置用户权限

```java
@RestController
public class CurrentLoginUserInfoController {
    /**
     * 从当前请求对象中获取
     * @param principal
     * @return
     */
    @GetMapping("/getLoginUserInfo")
    public Principal getLoginUserInfo(Principal principal){
        return principal;
    }

    /**
     * 从当前请求对象中获取
     * 注意Authentication接口继承自 Principal
     */
    @GetMapping("/getLoginUserInfo1")
    public Authentication getLoginUserInfo1(Authentication authentication){
        return authentication;
    }

    /**
     * 从SecurityContextHolder获取
     */
    @GetMapping("/getLoginUserInfo2")
    public Authentication getLoginUserInfo(){
        //通过安全上下文持有器获取安全上下文，再获取认证信息
        Authentication authentication  = SecurityContextHolder.getContext().getAuthentication();
        return authentication;
    }
}
```

访问 http://localhost:8080/getLoginUserInfo 、http://localhost:8080/getLoginUserInfo1 、http://localhost:8080/getLoginUserInfo2

运行结果：

> {
> 	"authorities":[{
> 		"authority":"ROLE_teacher"
> 	}],
> 	"details":{
> 		"remoteAddress":"0:0:0:0:0:0:0:1",
> 		"sessionId":"49940FDC14D58E1D6436B34407551613"
> 	},
> 	"authenticated":true,
> 	"principal":{
> 		"password":null,
> 		"username":"thomas",
> 		"authorities":				
> 			[{"authority":"ROLE_teacher"}],
> 		"accountNonExpired":true,
> 		"accountNonLocked":true,
> 		"credentialsNonExpired":true,
> 		"enabled":true
> 	},
> 	"credentials":null,"name":"thomas"
> }

**说明**：

- Principal 定义认证的而用户，如果用户使用用户名和密码方式登录，principal通常就是一个UserDetails

- Credentials：登录凭证，一般就是指密码。当用户登录成功之后，登录凭证会被自动擦除，以方式泄露。

- authorities：用户被授予的权限信息。 

#### 配置用户权限

配置用户权限的方式：

- 配置roles
- 配置authorities

**注意**：

1. 如果给一个用户同时配置roles和authorities，哪个写在后面哪个起作用
2. 配置roles时，权限名会加上`ROLE_`

添加WebSecurity：

> MySecurityUserConfig.java

```java
@Bean
public UserDetailsService userDetailService() {
    //使用org.springframework.security.core.userdetails.User类来定义用户

    //定义两个用户
    UserDetails user1 = User.builder().username("eric").password(passwordEncoder().encode("123456")).roles("student").build();
    UserDetails user2 = User.builder().username("thomas").password(passwordEncoder().encode("123456")).roles("teacher").build();
//        注意 1 哪个写在后面哪个起作用 2 角色变成权限后会加一个ROLE_前缀，比如ROLE_teacher
    UserDetails user3 = User.builder().username("hjc").password(passwordEncoder().encode("123456")).roles("teacher").authorities("teacher:add","teacher:update").build();
    //创建两个用户
    InMemoryUserDetailsManager userDetailsManager = new InMemoryUserDetailsManager();
    userDetailsManager.createUser(user1);
    userDetailsManager.createUser(user2);
    userDetailsManager.createUser(user3);
    return userDetailsManager;
}
```

重启程序使用hjc登录，然后查看用户认证信息 http://localhost:8080/getLoginUserInfo：

![image-20230422110211642](./assets/image-20230422110211642.png)

### 角色授权

#### 对URL进行授权

虽然实现了认证功能，但是受保护的资源是默认的，默认所有认证(登录)用户均可以访问所有资源，不能根据实际情况进行角色管理，要实现授权功能，需重写WebSecurityConfigureAdapter 中的一个configure方法。

新建WebSecurityConfig类，重写configure(HttpSecurity http)方法：

```java
@Configuration
@Slf4j
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                //角色student或者teacher都可以访问/student/** 这样的url
                .mvcMatchers("/student/*").hasAnyRole("student", "teacher")
                // 角色teacher 可以访问teacher/**
                .mvcMatchers("/teacher/**").hasRole("teacher")
                //权限admin:query 可以访问/admin**
//                .mvcMatchers("/admin/**").hasAuthority("admin:query")
                //角色teacher 或者权限admin:query 觉可以访问admin/**
                .mvcMatchers("/admin/**").access("hasRole('teacher') or hasAuthority('admin:query')")
                //任何请求均需要认证
                .anyRequest().authenticated();
        //使用表单登录
        http.formLogin();
    }
}
```

登录不同权限的用户，访问：http://localhost:8080/teacher/query 、http://localhost:8080/student/query 、http://localhost:8080/admin/query 当权限不足时，则无法访问。

#### 方法级别的权限控制

新建WebSecurityConfig类：

- EnableGlobalMethodSecurity 注解的属性`prePostEnabled = true` 解锁`@PreAuthorize` 和`@PostAuthorize`注解，`@PreAuthorize` 在方法执行前进行验证，`@PostAuthorize` 在方法执行后进行验证

- EnableGlobalMethodSecurity的`securedEnabled = true` 解锁`@Secured`注解，`@Secured`和`@PreAuthorize`用法基本一样 `@Secured`对应的角色必须要有ROLE_前缀

> WebSecurityConfig.java

```java
@Configuration
@Slf4j
@EnableGlobalMethodSecurity(prePostEnabled = true) // 启用全局方法安全注解
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {

        //任何访问均需要认证
        http.authorizeRequests().anyRequest().authenticated();
        http.formLogin().permitAll();

    }
}
```

新建安全配置类：

> MySecurityUserConfig.java

```java
@Configuration
public class MySecurityUserConfig {

    @Bean
    public UserDetailsService userDetailService() {
        //使用org.springframework.security.core.userdetails.User类来定义用户

        //定义两个用户
        UserDetails user1 = User.builder().username("eric").password(passwordEncoder().encode("123456")).roles("student").build();
        UserDetails user2 = User.builder().username("thomas").password(passwordEncoder().encode("123456")).roles("teacher").build();
//        注意 1 哪个写在后面哪个起作用 2 角色变成权限后会加一个ROLE_前缀，比如ROLE_teacher
        UserDetails user3 = User.builder().username("admin").password(passwordEncoder().encode("123456")).roles("teacher").authorities("teacher:add","teacher:update").build();
        //创建两个用户
        InMemoryUserDetailsManager userDetailsManager = new InMemoryUserDetailsManager();
        userDetailsManager.createUser(user1);
        userDetailsManager.createUser(user2);
        userDetailsManager.createUser(user3);
        return userDetailsManager;
    }
    /**
     * 从 Spring5 开始，强制要求密码要加密
     * 如果非不想加密，可以使用一个过期的 PasswordEncoder 的实例 NoOpPasswordEncoder，
     * 但是不建议这么做，毕竟不安全。
     * @return
     */
    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }
}
```

新建service及实现：

> TeacherService.java

```java
public interface TeacherService {
    String add();
    String update();
    String delete();
    String query();
}
```

> TeacherServiceImpl.java

```java
@Service
@Slf4j
public class TeacherServiceImpl implements TeacherService {
    @Override
    public String add() {
        log.info("添加教师成功");
        return "添加教师成功";
    }

    @Override
    public String update() {
        log.info("修改教师成功");
        return "修改教师成功";
    }

    @Override
    public String delete() {
        log.info("删除教师成功");
        return "删除教师成功";
    }

    @Override
    public String query() {
        log.info("查询教师成功");
        return "查询教师成功";
    }
}
```

修改TeacherController：

> TeacherController.java

```java
@Service
@Slf4j
public class TeacherServiceImpl implements TeacherService {
    @Override
    @PreAuthorize("hasAnyAuthority('teacher:add') OR hasRole('teacher')")
    public String add() {
        log.info("添加教师成功");
        return "添加教师成功";
    }

    @Override
    @PreAuthorize("hasAuthority('teacher:update')")
    public String update() {
        log.info("修改教师成功");
        return "修改教师成功";
    }

    @Override
    @PreAuthorize("hasAuthority('teacher:delete')")
    public String delete() {
        log.info("删除教师成功");
        return "删除教师成功";
    }

    @Override
    @PreAuthorize("hasRole('teacher')")
    public String query() {
        log.info("查询教师成功");
        return "查询教师成功";
    }
}
```

启动程序并访问   http://localhost:8080/teacher/add  http://localhost:8080/teacher/update  http://localhost:8080/teacher/delete  http://localhost:8080/teacher/query  ：thomas可以访问添加和查询，别的不能访问，amdin可以访问添加和更新，别的不能访问。

## SpringSecurity 返回json

前后端分离成为企业应用开发中的主流，前后端分离通过json进行交互，登录成功和失败后不用页面跳转，而是一段json提示。

创建统一响应类HttpResult：

> HttpResult.java

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class HttpResult {
    private Integer code;
    private String msg;
    private Object data;

    public HttpResult(Integer code, String msg) {
        this.code = code;
        this.msg = msg;
    }
}
```

创建登录成功的处理器：

> MyAutheticationSuccessHandle.java

```java
/**
 * 登录成功处理器
 * @author hjc
 */
@Component
public class MyAuthenticationSuccessHandler implements AuthenticationSuccessHandler {

    @Resource
    private ObjectMapper objectMapper;

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
        response.setCharacterEncoding("UTF-8");
        response.setContentType("application/json;charset=utf-8");
        HttpResult httpResult = new HttpResult(200, "登录成功",authentication);
        String str = objectMapper.writeValueAsString(httpResult);
        response.getWriter().write(str);
        response.getWriter().flush();

    }
}
```

创建登录失败的处理器：

> MyAuthenticationFailureHandler.java

```java
/**
 * 登录失败的处理器
 * @author hjc
 */
@Component
public class MyAuthenticationFailureHandler implements AuthenticationFailureHandler {

    @Resource
    private ObjectMapper objectMapper;

    /**
     * @param request 当前的请求对象
     * @param response 当前的响应对象
     * @param exception 失败的原因的异常
     * @throws IOException
     * @throws ServletException
     */
    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
        System.out.println("登录失败");
        //设置响应编码
        response.setCharacterEncoding("UTF-8");
        response.setContentType("application/json;charset=utf-8");
        //返回JSON出去
        HttpResult result = new HttpResult(-1, "登录失败");
        if(exception instanceof BadCredentialsException){
            result.setData("密码不正确");
        }else if(exception instanceof DisabledException){
            result.setData("账号被禁用");
        }else if(exception instanceof UsernameNotFoundException){
            result.setData("用户名不存在");
        }else if(exception instanceof CredentialsExpiredException){
            result.setData("密码已过期");
        }else if(exception instanceof AccountExpiredException){
            result.setData("账号已过期");
        }else if(exception instanceof LockedException){
            result.setData("账号被锁定");
        }else{
            result.setData("未知异常");
        }
        //把result 转成 JSON
        String json = objectMapper.writeValueAsString(result);
        //响应出去
        PrintWriter out = response.getWriter();
        out.write(json);
        out.flush();

    }
}
```

创建无权限的处理器：

> MyAccessDeniedHandler.java

```java
/**
 * 无权限的处理器
 * @author hjc
 */
@Component
public class MyAccessDeniedHandler implements AccessDeniedHandler {

    //声明一个把对象转成JSON的对象
    @Resource
    private ObjectMapper objectMapper;

    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException, ServletException {
        //设置响应编码
        response.setCharacterEncoding("UTF-8");
        response.setContentType("application/json;charset=utf-8");
        //返回JSON
        HttpResult result = new HttpResult(-1, "您没有权限访问");
        //把result转换成JSON
        String json = objectMapper.writeValueAsString(result);
        //响应
        PrintWriter out = response.getWriter();
        out.write(json);
        out.flush();
    }
}
```

创建登出(退出)的处理器：

> MyLogoutSuccessHandler.java

```java
/**
 * 退出成功处理器
 * @author hjc
 */
@Component
public class MyLogoutSuccessHandler implements LogoutSuccessHandler {
    //声明一个把对象转成JSON的对象
    @Resource
    private ObjectMapper objectMapper;
    /**
     *
     * @param request
     * @param response
     * @param authentication 当前退出的用户对象
     * @throws IOException
     * @throws ServletException
     */
    @Override
    public void onLogoutSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
        response.setCharacterEncoding("UTF-8");
        response.setContentType("application/json;charset=utf-8");
        //返回JSON
        HttpResult result = new HttpResult(200, "退出成功");
        //把result转换成JSON
        String json = objectMapper.writeValueAsString(result);
        //响应
        PrintWriter out = response.getWriter();
        out.write(json);
        out.flush();
    }
}
```

创建用户配置类：

> MySecurityUserConfig.java

```java
@Configuration
public class MySecurityUserConfig {

    @Bean
    public UserDetailsService userDetailService() {
        //使用org.springframework.security.core.userdetails.User类来定义用户

        //定义两个用户
        UserDetails user1 = User.builder().username("eric").password(passwordEncoder().encode("123456")).roles("student").build();
        UserDetails user2 = User.builder().username("thomas").password(passwordEncoder().encode("123456")).roles("teacher").build();
//        注意 1 哪个写在后面哪个起作用 2 角色变成权限后会加一个ROLE_前缀，比如ROLE_teacher
        UserDetails user3 = User.builder().username("admin").password(passwordEncoder().encode("123456")).roles("admin").build();
        //创建用户
        InMemoryUserDetailsManager userDetailsManager = new InMemoryUserDetailsManager();
        userDetailsManager.createUser(user1);
        userDetailsManager.createUser(user2);
        userDetailsManager.createUser(user3);
        return userDetailsManager;
    }
    /**
     * 从 Spring5 开始，强制要求密码要加密
     * 如果非不想加密，可以使用一个过期的 PasswordEncoder 的实例 NoOpPasswordEncoder，
     * 但是不建议这么做，毕竟不安全。
     * @return
     */
    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }
}
```

创建安全配置类：

> WebSecurityConfig.java

```java
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    //注入登陆成功的处理器
    @Resource
    private MyAuthenticationSuccessHandler myAuthenticationSuccessHandler;

    //注入登录失败的处理器
    @Resource
    private MyAuthenticationFailureHandler myAuthenticationFailureHandler;

    //注入没有权限的处理器
    @Resource
    private MyAccessDeniedHandler myAccessDeniedHandler;

    //注入退出成功的处理器
    @Resource
    private MyLogoutSuccessHandler myLogoutSuccessHandler;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.exceptionHandling().accessDeniedHandler(myAccessDeniedHandler);
        http.formLogin().successHandler(myAuthenticationSuccessHandler).failureHandler(myAuthenticationFailureHandler).permitAll();
        http.logout().logoutSuccessHandler(myLogoutSuccessHandler);
        http.authorizeRequests()
                .mvcMatchers("/teacher/**").hasRole("teacher")
                .mvcMatchers("/student/*").hasRole("student")
                .mvcMatchers("admin").hasRole("admin")
                .anyRequest().authenticated();

    }
}
```

启动程序，使用admin 访问 http://localhost:8080/teacher/query 查看无权限访问效果，及访问 http://localhost:8080/admin/query 查看登录成功、登录失败、退出等效果。

## 使用UserDetailsService获取用户认证信息

用户实体类需要实现UserDetails接口，并实现该接口中的7个方法， UserDetails 接口的7个方法如下图：

![img](./assets/clip_image002-1682260683694-3.jpg)

新建SecurityUser 类，该类实现接口UserDetails接口：

```java
public class SecurityUser implements UserDetails {
    //获取当前用户对象所具有的角色信息
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        //配置用户权限信息
        //GrantedAuthority g1=new SimpleGrantedAuthority("student:query");
        //使用lambda表达式
        GrantedAuthority g1 =()->"admin:query";
        List<GrantedAuthority> grantedAuthorityList = new ArrayList<>();
        grantedAuthorityList.add(g1);
        return grantedAuthorityList;
    }

    //获取当前用户对象的密码
    @Override
    public String getPassword() {
        //明文为 123456
        return new BCryptPasswordEncoder().encode("123456");
    }

    //获取当前用户对象的用户名
    @Override
    public String getUsername() {
        return "Obama";
    }

    //当前账户是否未过期
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    //当前账户是否未锁定
    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    //当前账户密码是否过期
    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    //当前账户是否可用
    @Override
    public boolean isEnabled() {
        return true;
    }
}
```

新建UserServiceImpl 实现UserDetailService：

```java
@Service
public class UserServiceImpl implements UserDetailsService {
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        SecurityUser securityUser = new SecurityUser();
        if(username==null || !username.equals(securityUser.getUsername())){
            throw new UsernameNotFoundException("该用户不存在");
        }
        return securityUser;
    }
}
```

新建WebSecurityConfig类，配置密码编码器：

```

```

启动项目，使用thomas/123456 登录系统，发现可以访问student/query，不可以访问teacher/query

再次访问：http://localhost:8080/getLoginUserInfo 查看用户信息

## 基于数据库的认证



## 基于数据库的方法授权



- [认证授权基本概念](authentication_authorization.md)
- [Spring Security概述](concept.md)
- [登录认证]()
- [角色授权]()
- []()

