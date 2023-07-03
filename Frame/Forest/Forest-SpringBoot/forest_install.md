# Forest 安装及使用

## 安装

对于 Springboot 项目来说， 只需要添加 forest-spring-boot-starter 依赖即可

```xml
<dependency>
    <groupId>com.dtflys.forest</groupId>
    <artifactId>forest-spring-boot-starter</artifactId>
    <version>1.5.32</version>
</dependency>
```

**SpringBoot3依赖**：

```xml
<dependency>
    <groupId>com.dtflys.forest</groupId>
    <artifactId>forest-spring-boot3-starter</artifactId>
    <version>1.5.32</version>
</dependency>
```

**JSON框架依赖**：Springboot 已自带 Jackson 框架依赖，如需要 Fastjson，请添加依赖(Fastjson依赖：版本 >= 1.2.48)：

```xml
<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>fastjson</artifactId>
  <version>1.2.73</version>
</dependency>
```

 **XML框架依赖**：

```xml
  <dependency>
      <groupId>com.dtflys.forest</groupId>
      <artifactId>forest-jaxb</artifactId>
      <version>1.5.32</version>
  </dependency>
```

**Protobuf框架依赖**：如果项目需要使用 Protobuf, 就需要引入 Google 的 Protobuf 包依赖，若是已经引入了 `forest-spring-boot-starter` 启动包，就不用再引入该依赖了，因为在 Forest 的启动包中已经传递依赖此包(版本 >= 3.14.0)

```xml
<dependency>
  <groupId>com.google.protobuf</groupId>
  <artifactId>protobuf-java</artifactId>
  <version>3.18.1</version>
</dependency>
```

## 使用

在 Forest 中，所有的 HTTP 请求信息都要绑定到某一个接口的方法上，不需要编写具体的代码去发送请求。请求发送方通过调用事先定义好 HTTP 请求信息的接口方法，自动去执行 HTTP 发送请求的过程，其具体发送请求信息就是该方法对应绑定的 HTTP 请求信息。

**操作步骤**：

1. **扫描接口**：在`Springboot`的配置类或者启动类上加上`@ForestScan`注解，并在`basePackages`属性里填上远程接口的所在的包名。

   Forest 会扫描`@ForestScan`注解中`basePackages`属性指定的包下面所有的接口，然后会将符合条件的接口进行动态代理并注入到 Spring 的上下文中。

   **注意**：1.5.1以后版本可以跳过此步，不需要 `@ForestScan` 注解来指定扫描的包范围

   ```java
   @SpringBootApplication
   @ForestScan("com.example.forest")
   public class ForestApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(ForestApplication.class, args);
       }
   }
   ```

2. **创建一个接口**，并创建一个接口方法名为`helloForest`，用`@Get`注解修饰：

   通过`@Get`注解，将`HelloForest`接口中的`helloForest()`方法绑定了一个 HTTP 请求， 其 URL 为`http://localhost:8080/hello` ，并默认使用`GET`方式，且将请求响应的数据以`String`的方式返回给调用者。

   ```java
   public interface HelloForest {
   
       @Get("http://localhost:8080/hello")
       String helloForest();
   }
   ```

3. **发送请求**：从 Spring 上下文注入接口实例，调用接口：

   ```java
   @Service
   public class HelloService {
       @Resource
       private HelloForest helloForest;
   
       public String helloService(){
           // 调用自定义的 Forest 接口方法
           // 等价于发送 HTTP 请求，请求地址和参数即为 helloForest 方法上注解所标识的内容
           String result = helloForest.helloForest();
           // result 即为 HTTP 请求响应后返回的字符串类型数据
           return result;
       }
   }
   ```

4. **编写controller**：

   ```java
   @RestController
   public class HelloController {
   
       @Resource
       private HelloService helloService;
   
       @GetMapping("/hello")
       public String hello() {
           return "hello";
       }
   
       @GetMapping("/hello2")
       public String hello2() {
           return helloService.helloService();
       }
   }
   ```

5. **查看结果及控制台**：

   ![image-20230703141326526](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/202307031413576.png)

   ![image-20230703141020713](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/202307031410788.png)
