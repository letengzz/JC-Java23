# Forest 使用方式

Forest 是一个以接口 + 注解形式定义请求为主的HTTP框架，发送请求前先需要定义一个 interface 接口类，再组合使用 Forest 的注解（如：`@Get`）定义绑定的HTTP请求参数，随后再实例化接口类对象，最终调用该接口绑定HTTP参数注解的方法实现请求的发送。

这样做有很多优点，比如在 Java 代码和 HTTP 协议之间实现解耦，同时方便众多请求接口的管理。但缺点也很明显：步骤较多，如果想要直接快速的请求一个简单的URL地址就显得过重了。

不过，Forest 自 `1.5.3` 版本起，就提供了编程式接口，不用定义接口类也能发送请求，使用也很方便。

****

Forest 提供两种开发使用模式：**声明式**和**编程式**

- 声明式需要定义Java接口，以及请求对应的接口方法，这样会有一个非常清晰的网络接口结构，十分有利于API管理

  声明式的模式在不同项目环境中也有所不同，需要根据项目需要，选择不同的方式进行。

- 编程式接口短小精悍，快速便捷，十分有利于开发那些数量少、或大量参数（如URL）不固定的API

## 声明式接口

### 构建接口

在 Forest 依赖加入好之后，就可以构建 HTTP 请求的接口了。

在 Forest 中，所有的 HTTP 请求信息都要绑定到某一个接口的方法上，不需要编写具体的代码去发送请求。请求发送方通过调用事先定义好 HTTP 请求信息的接口方法，自动去执行 HTTP 发送请求的过程，其具体发送请求信息就是该方法对应绑定的 HTTP 请求信息。

### 简单请求

创建一个`interface`，并用`@Request`注解修饰接口方法。

```java
public interface MyClient {

    @Request("http://localhost:8080/hello")
    String simpleRequest();

}
```

通过`@Request`注解，将上面的`MyClient`接口中的`simpleRequest()`方法绑定了一个 HTTP 请求， 其 URL 为`http://localhost:8080/hello` ，并默认使用`GET`方式，且将请求响应的数据以`String`的方式返回给调用者。

### 稍复杂点的请求

```java
public interface MyClient {

    @Request(
            url = "http://localhost:8080/hello/user",
            headers = "Accept: text/plain"
    )
    String sendRequest(@Query("uname") String username);
}
```

上面的`sendRequest`方法绑定的 HTTP 请求，定义了 URL 信息，以及把`Accept:text/plain`加到了请求头中， 方法的参数`String username`绑定了注解`@Query("uname")`，它的作用是将调用者传入入参 username 时，自动将`username`的值加入到 HTTP 的请求参数`uname`中。

如果调用方代码如下所示：

- Springboot / Spring
- 原生 Java

```java
@Resource
MyClient myClient;

myClient.sendRequest("foo");
```

这段调用所实际产生的 HTTP 请求如下：

```
GET http://localhost:8080/hello/user?uname=foo
HEADER:
    Accept: text/plain
```

### 请求方法

### 请求地址

### URL 参数

### 请求头

### 请求体

### 后端框架

### 接口注解

### 接收数据

### 数据转换

### 成功/失败条件

### 重试机制

### 重定向

### Gzip解压

### 日志管理

### 回调函数

### 异步请求

### HTTPS

### 使用Cookie

### 使用代理

### 上传下载

### 异常处理

## 编程式接口

### 发送请求

以字符串形式接受响应数据

```java
// Get请求
// 并以 String 类型接受数据
String str = Forest.get("/").executeAsString();
```

以自定义类型形式接受响应数据

```java
// Post请求
// 并以自定义的 MyResult 类型接受
MyResult myResult = Forest.post("/").execute(MyResult.class);
```

以带复杂泛型参数的类型形式接受响应数据

```java
// 通过 TypeReference 引用类传递泛型参数
// 就可以将响应数据以带复杂泛型参数的类型接受了
Result<List<User>> userList = Forest.post("/")
        .execute(new TypeReference<Result<List<User>>>() {});
```

异步请求

```java
// 异步 Post 请求
// 通过 onSuccess 回调函数处理请求成功后的结果
// 而 onError 回调函数则在请求失败后被触发
Forest.post("/")
        .async()
        .onSuccess(((data, req, res) -> {
            // data 为响应成功后返回的反序列化过的数据
            // req 为Forest请求对象，即 ForestRequest 类实例
            // res 为Forest响应对象，即 ForestResponse 类实例
        }))
        .onError(((ex, req, res) -> {
            // ex 为请求过程可能抛出的异常对象
            // req 为Forest请求对象，即 ForestRequest 类实例
            // res 为Forest响应对象，即 ForestResponse 类实例
        }))
        .execute();
```

定义请求的各种参数

```java
// 定义各种参数
// 并以 Map 类型接受
Map<String, Object> map = Forest.post("/")
      .backend("okhttp3")        // 设置后端为 okhttp3
      .host("127.0.0.1")         // 设置地址的host为 127.0.0.1
      .port(8080)                // 设置地址的端口为 8080
      .contentTypeJson()         // 设置 Content-Type 头为 application/json
      .addBody("a", 1)           // 添加 Body 项(键值对)： a, 1
      .addBody("b", 2)           // 添加 Body 项(键值对)： b, 2
      .maxRetryCount(3)          // 设置请求最大重试次数为 3
      // 设置 onSuccess 回调函数
      .onSuccess((data, req, res) -> { log.info("success!"); })
      // 设置 onError 回调函数
      .onError((ex, req, res) -> { log.info("error!"); })
      // 设置请求成功判断条件回调函数
      .successWhen((req, res) -> res.noException() && res.statusOk())
      // 执行并返回Map数据类型对象
      .executeAsMap();
```

Forest 的快速接口（如：`Forest.get(String url)`、`Forest.post(String url)`等方法）本质上是返回了一个 Forest 请求对象（即 `ForestRequest` 类对象），Forest 的绝大部分操作都是围绕请求对象所作的工作。