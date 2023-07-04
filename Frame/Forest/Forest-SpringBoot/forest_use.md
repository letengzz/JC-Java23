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

**例**：

创建一个`interface`，并用`@Request`注解修饰接口方法。

```java
public interface MyClient {

    @Request("http://localhost:8080/hello")
    String simpleRequest();

}
```

通过`@Request`注解，将上面的`MyClient`接口中的`simpleRequest()`方法绑定了一个 HTTP 请求， 其 URL 为`http://localhost:8080/hello` ，并默认使用`GET`方式，且将请求响应的数据以`String`的方式返回给调用者。

```java
public interface MyClient {

    @Request(
            url = "http://localhost:8080/user",
            headers = "Accept: text/plain"
    )
    String sendRequest(@Query("uname") String username);
}
```

`sendRequest`方法绑定的 HTTP 请求，定义了 URL 信息，以及把`Accept:text/plain`加到了请求头中， 方法的参数`String username`绑定了注解`@Query("uname")`，它的作用是将调用者传入入参 username 时，自动将`username`的值加入到 HTTP 的请求参数`uname`中。

如果调用方代码如下所示：

```java
@Resource
MyClient myClient;

myClient.sendRequest("foo");
```

这段调用所实际产生的 HTTP 请求：

![image-20230704085849274](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/202307040858414.png)

### 请求方法

Forest 使用不同的**请求注解**来标识某个接口方法来进行发送不同类型的请求，其支持的HTTP方法:

| HTTP 请求方法 | 请求注解                      | 描述               |
| ------------- | ----------------------------- | ------------------ |
| **GET**       | `@Get`、`@GetRequest`         | 获取资源           |
| **POST**      | `@Post`、`@PostRequest`       | 传输实体文本       |
| **PUT**       | `@Put`、`@PutRequest`         | 上传资源           |
| **HEAD**      | `@HeadRequest`                | 获取报文首部       |
| **DELETE**    | `@Delete`、`@DeleteRequest`   | 删除资源           |
| **OPTIONS**   | `@Options`、`@OptionsRequest` | 询问支持的方法     |
| **TRACE**     | `@Trace`、`@TraceRequest`     | 追踪路径           |
| **PATCH**     | `@Patch`、`@PatchRequest`     | 更新资源的某一部分 |
| **不定方法**  | `@Request`                    | 可动态传入HTTP方法 |

#### GET 请求

使用`@Get`注解或`@GetRequest`注解

```java
@Get("http://localhost:8080/hello")
String simpleGet1();

@GetRequest("http://localhost:8080/hello")
String simpleGet2();
```

#### POST 请求

使用`@Post`注解或`@PostRequest`注解

```java
@Post("http://localhost:8080/hello")
String simplePost1();

@PostRequest("http://localhost:8080/hello")
String simplePost2();
```

#### PUT 请求

使用`@Put`注解或`@PutRequest`注解

```java
@Put("http://localhost:8080/hello")
String simplePut1();

@PutRequest("http://localhost:8080/hello")
String simplePut2();
```

#### HEAD 请求

使用`@HeadRequest`注解

为了避免于`@Header`注解产生歧义和混淆，Forest 没有提供`@Head`注解

```java
@HeadRequest("http://localhost:8080/hello")
String simpleHead();
```

#### DELETE 请求

使用`@Delete`注解或`@DeleteRequest`注解

```java
@Delete("http://localhost:8080/hello")
String simpleDelete1();

@DeleteRequest("http://localhost:8080/hello")
String simpleDelete2();
```

#### OPTIONS 请求

使用`@Options`注解或`@OptionsRequest`注解

```java
@Options("http://localhost:8080/hello")
String simpleOptions1();

@OptionsRequest("http://localhost:8080/hello")
String simpleOptions2();
```

#### TRACE 请求

使用`@Trace`注解或`@TraceRequest`注解

```java
@Trace("http://localhost:8080/hello")
String simpleTrace1();

@TraceRequest("http://localhost:8080/hello")
String simpleTrace2();
```

#### PATCH 请求

使用`@Patch`注解或`@PatchRequest`注解

```java
@Patch("http://localhost:8080/hello")
String simplePatch1();

@PatchRequest("http://localhost:8080/hello")
String simplePatch2();
```

#### 动态 HTTP 请求方法

若不想在接口定义的时候直接定死为某个具体的 HTTP 请求方法，而是想从全局变量或方法参数中动态传入

可以使用 `@Request` 请求注解

```java
/**
 * 通过在 @Request 注解的 type 属性中定义字符串模板
 * 在字符串模板中引用方法的参数
 */
@Request(
    url = "http://localhost:8080/hello",
    type = "{type}"
)
String simpleRequest(@Var("type") String type);
```

在调用改方法时通过参数传入 HTTP 请求方法类型（字符串类型，大小写不敏感）

```java
// POST 请求
String result1 = simpleRequest("post");
// DELETE 请求
String result2 = simpleRequest("DELETE");
```

### 请求地址

HTTP请求可以没有请求头、请求体，但一定会有请求地址，即`URL`，以及很多请求的参数都是直接绑定在`URL`的`Query`部分上

#### 设置URL

基本`URL`设置方法：只要在`url`属性中填入完整的请求地址即可。

除此之外，也可以通过 `@Var` 注解修饰的参数从外部动态传入`URL`:

```java
/**
 * 整个完整的URL都通过参数传入
 * {0}代表引用第一个参数
 */
@Get("{0}")
String send1(String myURL);

/**
 * 整个完整的URL都通过 @Var 注解修饰的参数动态传入
 */
@Get("{myURL}")
String send2(@Var("myURL") String myURL);

/**
 * 通过参数转入的值作为URL的一部分
 */
@Get("http://{myURL}/abc")
String send3(@Var("myURL") String myURL);

/**
 * 参数转入的值可以作为URL的任意一部分
 */
@Get("http://localhost:8080/test/{myURL}?a=1&b=2")
String send4(@Var("myURL") String myURL);
```

#### URL结构

一个标准的URL一般会包含以下几个部分：

- **协议**: 如URL的协议部分为`http` ，就代表请求使用的是HTTP协议
- **域名**: 如URL的域名部分为`baidu.com`，就代表请求域名地址为`baidu.com`，IP地址和主机名也可以当作域名一样使用
- **端口号**: 跟在域名后面的是端口，域名和端口之间使用`:`作为分隔符。端口不是一个URL必须的部分，如果省略端口部分，HTTP协议请求将采用默认端口`80`，HTTPS则用默认端口`443`
- **路径地址**: 路径地址是从域名后的第一个`/`开始，一直到`?`为止的部分，如果没有`?`，则是从域名后的最后一个`/`开始到`#`为止的部分，如果没有`？`和`#`，那么从域名后的第一个`/`开始到结束，都是路径地址部分
- **查询参数**: 从`？`开始到`#`为止之间的部分为查询参数部分，又称搜索部分、查询部分
- **锚**: 从`#`开始到最后，都是锚部分

![](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/202307040917768.png)

#### 根地址

一个URL地址包含协议、站点地址(域名/IP地址/主机名)、端口号、路径地址、查询参数等几个部分。

而协议、站点地址(域名/IP地址/主机名)、端口号这三部分称为**根地址**。

但如果代码中大量URL接口都来自同一个站点，那就会存在两大重复的域名或IP地址，如果它们都以字符串形式散落在各个Forest请求接口方法的URL属性中，那就会变得难以维护。

##### @Address 注解

Forest 从`1.5.3`版本开始提供了 `@Address` 注解，帮助您将URL的地址部分提取出来，方便管理

```java
// 通过 @Address 注解绑定根地址
// host 绑定到第一个参数， port 绑定到第二个参数
@Post("/data")
@Address(host = "{0}", port = "{1}")
ForestRequest<String> sendHostPort(String host, int port);

// 若调用 sendHostPort("192.168.0.2", 8080);
// 则最终产生URL:
// http://192.168.0.2:8080/data
```

`@Address` 注解可以绑定到接口类上，根地址绑定返回就扩展到整个接口下的所有方法

```java
// 整个接口下的所有方法请求都默认绑定该根地址
@Address(host = "127.0.0.1", port = "8080")
public interface MyClient {

    // 绑定接口上的默认根地址
    // 最终URL: http://127.0.0.1:8080/data1
    @Post("/data1")
    ForestRequest<String> sendData1();

    // 绑定接口上的默认根地址
    // 最终URL: http://127.0.0.1:8080/data2
    @Post("/data2")
    ForestRequest<String> sendData2();

    // 使用方法上的根地址
    // 最终URL: http://192.168.0.1:7000/data3
    @Post("/data3")
    @Address(host = "192.168.0.1", port = "7000")
    ForestRequest<String> sendData3();
}
```

##### 动态根地址

每次发送请求可以动态地从3个IP地址中随机选取一个作为根地址

Forest 提供了**地址来源**接口，即`AddressSource`接口来实现该功能：

```java
// 实现 AddressSource 接口
public class MyAddressSource implements AddressSource {

    @Override
    public ForestAddress getAddress(ForestRequest request) {
        // 定义 3 个 IP 地址
        String[] ipArray = new String[] {
                "192.168.0.1",
                "192.168.0.2",
                "192.168.0.3",
        };
        // 随机选出其中一个
        Random random = new Random();
        int i = random.nextInt(3);
        String ip = ipArray[i];
        // 返回 Forest 地址对象
        return new ForestAddress(ip, 80);
    }
}
```

绑定自定义的`AddressSource`接口实现类：

```java
// 也是通过 @Address 注解来绑定动态地址来源
// 每次调用该方法，都可能是不同的根地址
@Post("/data")
@Address(source = MyAddressSource.class)
ForestRequest<String> sendData();
```

若连续调用多次`sendData()`，则每次请求的URL根地址都可能会不同

```java
myClient.sendData(); // 第一次调用, URL: http://192.168.0.2/data
myClient.sendData(); // 第二次调用, URL: http://192.168.0.2/data
myClient.sendData(); // 第三次调用, URL: http://192.168.0.1/data
myClient.sendData(); // 第四次调用, URL: http://192.168.0.3/data
myClient.sendData(); // 第五次调用, URL: http://192.168.0.1/data
```

### URL 参数

URL参数，也称为 URL 查询字符串，即跟在 URL 地址中`?`后面的那串字符串，可以用`=`表示一对键值对，多个键值对用`&`隔开，其可以作为 HTTP 请求的参数

通过这些参数可以告诉服务端要做哪些事情，以及这些事相关的数据（简单数据，数据大小受到 URL 长度标准的限制）

#### 字符串模板传参

HTTP的`URL`不光有协议名、域名、端口号等等基本信息，更为重要的是它能携带各种参数，称为`Query`参数，它通常包含参数名和参数值两部分。

Forest给`URL`的`Query`部分传参也有多种方式，其中最简洁直白的就数字符串拼接了。

```java
/**
 * 直接在url字符串的问号后面部分直接写上 参数名=参数值 的形式
 * 等号后面的参数值部分可以用 {参数序号} 这种字符串模板的形式替代
 * 在发送请求时会动态拼接成一个完整的URL
 * 使用这种形式不需要为参数定义额外的注解
 * 
 * 注：参数序号是从 0 开始记的方法参数的序号
 * 0 代表第一个参数，1 代表第二个参数，以此类推
 */
@Get("http://localhost:8080/abc?a={0}&b={1}&id=0")
String send1(String a, String b);

/**
 * 直接在url字符串的问号后面部分直接写上 参数名=参数值 的形式
 * 等号后面的参数值部分可以用 {变量名} 这种字符串模板的形式替代
 * 在发送请求时会动态拼接成一个完整的URL
 * 使用这种方式需要通过 @Var 注解或全局配置声明变量
 */
@Get("http://localhost:8080/abc?a={a}&b={b}&id=0")
String send2(@Var("a") String a, @Var("b") String b);


/**
 * 如果一个一个变量包含多个Query参数，比如: "a=1&b=2&c=3"
 * 为变量 parameters 的字符串值
 * 就用 ${变量名} 这种字符串模板格式
 * 使用这种方式需要通过 @Var 注解或全局配置声明变量
 */
@Get("http://localhost:8080/abc?${parameters}")
String send3(@Var("parameters") String parameters);
```


调用`{参数序号}`字符串模板的方法

```java
// 会对第二个参数 B&c=C 进行URL Encode
myClient.send1("A", "B&c=C");

// 产生的URL为
// http://localhost:8080/abc?a=A&b=B%26c%3DC&id=0
```

调用`{变量名}`字符串模板的方法

```java
// 会对第二个参数 B&c=C 进行URL Encode
myClient.send2("A", "B&c=C");

// 产生的URL为
// http://localhost:8080/abc?a=A&b=B%26c%3DC&id=0
```

调用`${变量名}`字符串模板的方法

```java
// 会用参数输入的字符串替换URL中的 ${parameters} 部分
myClient.send3("a=A&b=B&c=C");

// 产生的URL为
// http://localhost:8080/abc?a=A&b=B&c=C
```

#### @Query 注解

可以不用所有`Query`参数直接写在`url`属性的字符串，可以使用`@Query` 注解解决。

**说明**：`@Query` 注解修饰的参数一定会出现在 URL 中。

```java
/**
 * 使用 @Query 注解，可以直接将该注解修饰的参数动态绑定到请求url中
 * 注解的 value 值即代表它在url的Query部分的参数名
 */
@Get("http://localhost:8080/abc?id=0")
String send(@Query("a") String a, @Query("b") String b);
```

当传的`URL`参数太多了，可以使用Map或对象来解决：

```java
/**
 * 使用 @Query 注解，可以修饰 Map 类型的参数
 * 很自然的，Map 的 Key 将作为 URL 的参数名， Value 将作为 URL 的参数值
 * 这时候 @Query 注解不定义名称
 */
@Get("http://localhost:8080/abc?id=0")
String send1(@Query Map<String, Object> map);


/**
 * @Query 注解也可以修饰自定义类型的对象参数
 * 依据对象类的 Getter 和 Setter 的规则取出属性
 * 其属性名为 URL 参数名，属性值为 URL 参数值
 * 这时候 @Query 注解不定义名称
 */
@Get("http://localhost:8080/abc?id=0")
String send2(@Query UserInfo user);
```

**注意**：

1. 需要单个单个定义 `参数名=参数值` 的时候，`@Query`注解的value值一定要有，比如 `@Query("name") String name`
2. 需要绑定对象的时候，`@Query`注解的value值一定要空着，比如 `@Query User user` 或 `@Query Map map`

#### 数组参数

有些时候，需要通过URL参数传递一个数组或者一个列表

**列表类型参数**

```java
/*
 * 接受列表参数为URL查询参数
 */
@Get("http://localhost:8080/abc")
String send1(@Query("id") List idList);
```

若调用 **send1(Arrays.asList(1, 2, 3, 4))**

则产生的最终URL为

```text
http://localhost:8080/abc?id=1&id=2&id=3&id=4
```

**数组类型参数**

```java
/*
 * 接受数组参数为URL查询参数
 */
@Get("http://localhost:8080/abc")
String send2(@Query("id") int[] idList);
```

若调用 **send2(new int[] {1, 2, 3, 4})**

则产生的最终URL为

```text
http://localhost:8080/abc?id=1&id=2&id=3&id=4
```

#### 带 [] 的数组参数

有些场景用带方括号(`[]`)的参数名来表示数组类型的 Query 参数

```java
/*
 * 在 @Query 注解的参数名后跟上 [] 即可
 */
@Get("http://localhost:8080/abc")
String send(@Query("id[]") int[] idList);
```

若调用 **send2(new int[] {1, 2, 3, 4})**

则产生的最终URL为

```text
http://localhost:8080/abc?id[]=1&id[]=2&id[]=3&id[]=4
```

#### 带下标的数组参数

在字符串模板中引用内置变量`_index`

```java
/*
 * 内置变量 _index 代表数组的下标
 */
@Get("http://localhost:8080/abc")
String send(@Query("id[${_index}]") int[] idList);
```

若调用 **send2(new int[] {1, 2, 3, 4})**

则产生的最终URL为

```text
http://localhost:8080/abc?id[0]=1&id[1]=2&id[2]=3&id[3]=4
```

#### JSON参数

如果不想以URL参数的标准格式传递列表或者数组，JSON字符串也是一种选择

这时，可以使用`@JSONQuery`注解

```java
@Get("http://localhost:8080/abc")
String send(@JSONQuery("id") List idList);
```

若调用 **send(Arrays.asList(1, 2, 3, 4))**

则产生的最终URL为

```text
http://localhost:8080/abc?id=[1, 2, 3, 4]
```

#### 延迟 Query 参数

延迟 Query 参数 （也称作 Lambda Query 参数），在您需要不马上求值的情况使用。

有很多情况，Query的参数值不能马上得出，而是在请求发送前的那一刻（所有请求参数都到位时）才能得出，典型的例子就是加签验证的场景（在请求体参数中添加一个参数token，而token的值是对整个URL Query(除token之外所有参数)做加密的结果）

```java
// 使用 Lazy 作为 Query 的参数类型
@Post("/data")
String sendData(@Query("a") String a, @Query("b") String b, @Query("token") Lazy<String> token);
```

在调用的时候，传入 Lambda

```java
myClient.sendData("Foo", "Bar", req -> {
    // req 为请求对象
    // 返回值将作为参数值添加到 URL Query 中
    return "";
});
```

在 Lambda 代码块中可以调用`req.queryString()` 来生成整个请求的URL Query字符串

但会自动排除 Lambda 所对应的 Query 参数（即 token 参数），而不用担心会造成死循环

```java
// Lambda 的参数为 ForestRequest 请求对象
// 调用 req.queryString() 可以生成整个请求的URL Query字符串
myClient.sendData("Foo", "Bar", req -> Base64.encode(req.queryString().getBytes()));
```

### 请求头

## headers属性

其中`headers`属性接受的是一个字符串数组，在接受多个请求头信息时以以下形式填入请求头：

```java
{
    "请求头名称1: 请求头值1",
    "请求头名称2: 请求头值2",
    "请求头名称3: 请求头值3",
    ...
 }
```

1
2
3
4
5
6

其中组数每一项都是一个字符串，每个字符串代表一个请求头。请求头的名称和值用`:`分割。

具体代码请看如下示例：

```java
public interface MyClient {

    @Request(
            url = "http://localhost:8080/hello/user",
            headers = {
                "Accept-Charset: utf-8",
                "Content-Type: text/plain"
            }
    )
    String multipleHeaders();
}
```

1
2
3
4
5
6
7
8
9
10
11

该接口调用后所实际产生的 HTTP 请求如下：

```
GET http://localhost:8080/hello/user
HEADER:
    Accept-Charset: utf-8
    Content-Type: text/plain
```

如果要每次请求传入不同的请求头内容，可以在`headers`属性的请求头定义中加入`数据绑定`。

```java
public interface MyClient {

    @Request(
            url = "http://localhost:8080/hello/user",
            headers = {
                "Accept-Charset: ${encoding}",
                "Content-Type: text/plain"
            }
    )
    String bindingHeader(@Var("encoding") String encoding);
}
```

1
2
3
4
5
6
7
8
9
10
11

如果调用方代码如下所示：

```java
myClient.bindingHeader("gbk");
```

1

这段调用所实际产生的 HTTP 请求如下：

```
GET http://localhost:8080/hello/user
HEADER:
    User-Agent: forest/1.5.32
    Accept-Charset: gbk
    Content-Type: text/plain
```

## [#](https://forest.dtflyx.com/pages/1.5.32/http_header/#user-agent-请求头)User-Agent 请求头

其中, `User-Agent`是特殊的通用请求头，所有的 Forest 请求都会默认自动加上`User-Agent: forest/{version}`这样的请求头。

该请求头无法删除，但可以用其他的值来覆盖

```java
@Request(
        url = "http://localhost:8080/hello/user",
        headers = {
            "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6)",    
            "Accept-Charset: ${encoding}",
            "Content-Type: text/plain"
        }
)
String bindingHeader(@Var("encoding") String encoding);
```

1
2
3
4
5
6
7
8
9

若被覆盖，则`User-Agent`请求头的值就会自定义的`User-Agent`内容

## [#](https://forest.dtflyx.com/pages/1.5.32/http_header/#header-注解)@Header 注解

想必大家都已经了解通过 `headers` 属性设置请求头的方法了。不过这种方式虽然直观，但如要没通过参数传入到请求头中就显得比较啰嗦了。

所以Forest还提供了 `@Header` 注解来帮助您把方法的参数直接绑定到请求体中。

```java
/**
 * 使用 @Header 注解将参数绑定到请求头上
 * @Header 注解的 value 指为请求头的名称，参数值为请求头的值
 * @Header("Accept") String accept将字符串类型参数绑定到请求头 Accept 上
 * @Header("accessToken") String accessToken将字符串类型参数绑定到请求头 accessToken 上
 */
@Post("http://localhost:8080/hello/user?username=foo")
void postUser(@Header("Accept") String accept, @Header("accessToken") String accessToken);
```

1
2
3
4
5
6
7
8
9
10

如果有很多很多的请求头要通过参数传入，我需要定义很多很多参数吗？当然不用！

```java
/**
 * 使用 @Header 注解可以修饰 Map 类型的参数
 * Map 的 Key 指为请求头的名称，Value 为请求头的值
 * 通过此方式，可以将 Map 中所有的键值对批量地绑定到请求头中
 */
@Post("http://localhost:8080/hello/user?username=foo")
void headHelloUser(@Header Map<String, Object> headerMap);


/**
 * 使用 @Header 注解可以修饰自定义类型的对象参数
 * 依据对象类的 Getter 和 Setter 的规则取出属性
 * 其属性名为 URL 请求头的名称，属性值为请求头的值
 * 以此方式，将一个对象中的所有属性批量地绑定到请求头中
 */
@Post("http://localhost:8080/hello/user?username=foo")
void headHelloUser(@Header MyHeaderInfo headersInfo);
```

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18

注意

- (1) 需要单个单个定义请求头的时候，**@Header**注解的**value**值一定要有，比如 **@Header("Content-Type") String contentType**
- (2) 需要绑定对象的时候，**@Header**注解的value值一定要空着，比如 **@Header MyHeaders headers** 或 **@Header Map headerMap**

## [#](https://forest.dtflyx.com/pages/1.5.32/http_header/#延迟请求头参数)延迟请求头参数

延迟请求头参数 （也称作 Lambda 请求头参数），在您需要不马上求值的情况使用。

有很多情况，请求头的参数值不能马上得出，而是在请求发送前的那一刻（所有请求参数都到位时）才能得出，典型的例子就是加签验证的场景

```java
// 使用 Lazy 作为 Query 的参数类型
@Post("/data")
String sendData(@Body data, @Header("token") Lazy<String> token);
```

1
2
3

在调用的时候，传入 Lambda

```java
myClient.sendData("Foo", "Bar", req -> {
    // req 为请求对象
    // 返回值将作为参数值添加到请求头中
    return "";
});
```

1
2
3
4
5

在 Lambda 代码块中可以调用`req.body().encodeToString()` 将整个请求体的数据序列化为字符串

```java
// Lambda 的参数为 ForestRequest 请求对象
// 调用 req.body().encodeString() 可以立即将整个请求体的数据序列化为字符串
// encrypt 是自定义的加密方法
myClient.sendData("Foo", "Bar", req -> encrypt(req.body().encodeToString()));
```

### 请求体



### 后端框架

Forest分为前端和后端两部分，而后端是由`okhttp3`和`httpclient`这样的后端HTTP框架构成的

这是因为没有一种框架都是完美的，都有各自的优缺点以及差异，比如`okhttp3`接入相对简单、同步异步容易切换，但很难支持带请求体的GET请求这种非标准的畸形请求（但往往业务中需要这种类型请求）; 而`httpclient(5.0版本以下)`性能较好，支持各种标准和非标准请求，但接入较为麻烦，还需要依赖很多的jar包，且不支持`HTTP/2`协议

Forest在某种程度上可以被理解为是一个屏蔽层，尽可能地屏蔽不同后端底层HTTP框架之间的差异，比如有统一的接口调用方式、统一的配置等等，但还是有些底层的特性差异是Forest无能为力的，比如让`okhttp3`的GET请求携带Body这件事就难以做到

对于这样的问题，Forest给出的解决方案就是组合使用不同的后端框架，换句话说就是在需要的地方使用相应的后端框架

## 全局后端框架

Forest有一个全局唯一的后端，并以此为HTTP请求的默认后端，默认情况下为`okhttp3`

- Yaml
- Properties
- Spring
- Java

```yaml
# 设置全局后端框架
# 目前版本有两种选择：okhttp3 和 httpclient
# 不填的默认请求为 okhttp3

forest:
    backend: httpclient # 设置全局后端为 httpclient
```

1
2
3
4
5
6

```yaml
# 设置全局后端框架
# 目前版本有两种选择：okhttp3 和 httpclient
# 不填的默认请求为 okhttp3

forest:
    backend: okhttp3 # 设置全局后端为 okhttp3
```



如果要通过代码方式设置后端框架，建议将后端对象作为静态常量常驻于内存，而不是经常重复实例化同样的后端对象

```java
public class MyBackend {
    // httpclient 后端对象
    public final static HttpclientBackend HTTPCLIENT = new HttpclientBackend();
    // okhttp3 后端对象
    public final static OkHttp3Backend OKHTTP3 = new OkHttp3Backend();
}
```



```java
// 获取Forest全局配置对象
ForestConfiguration configuration = Forest.config();
// 设置全局后端框架为 httpclient
configuration.setBackend(MyBackend.HTTPCLIENT);
```



```java
// 获取Forest全局配置对象
ForestConfiguration configuration = Forest.config();
// 设置全局后端框架为 okhttp3
configuration.setBackend(MyBackend.OKHTTP3);
```



## 接口/请求后端框架

全局后端框架可以配置和切换，甚至可以进行动态切换，但很多时候需要同时使用不同的后端框架，比如两个不同的接口分别使用不同的后端

自`1.5.5`版本后，Forest支持了接口/请求级别的后端框架配置，方便HTTP请求灵活设置后端

### 后端快捷注解

Forest提供了 `@HttpClient` 注解和 `OkHttp3` 注解，分别用于绑定请求的后端为 `httpclient` 和 `okhttp3`



```java
// 绑定请求的后端为 httpclient
@HttpClient
@Post("/data1")
String send1(@Body MyUser user);

// 绑定请求的后端为 okhttp3
@OkHttp3
@Post("/data2")
String send2(@Body MyUser user);
```



如果此类注解也绑定到**接口**上，那么该接口下的所有方法的请求默认为该接口注解指定的后端框架



```java
// 设置该请求接口的后端框架默认为 httpclient
@HttpClient
public interface BackendClient2 {

    // 未设置请求的后端，则默认为接口指定的后端框架，即 httpclient
    @Post("/data1")
    String send1(@Body MyUser user);

    // 绑定某一方法请求的后端为 okhttp3
    // 会覆盖掉接口上绑定的后端
    @OkHttp3
    @Post("/data2")
    String send2(@Body MyUser user);
}
```



### @Backend 注解

还有一种更为灵活和通用的注解 `@Backend`, 可以通过传入的字符串参数来确定具体要绑定的后端框架

```java
// 设置该请求接口的后端框架默认为 httpclient
@Backend("httpclient")
public interface BackendClient2 {

    // 未设置请求的后端，则默认为接口指定的后端框架，即 httpclient
    @Post("/data1")
    String send1(@Body MyUser user);

    // 绑定请求的后端为 okhttp3
    @Backend("okhttp3")
    @Post("/data2")
    String send2(@Body MyUser user);
}
```

该注解的参数也支持字符串模板，即可以通过全局变量和参数来动态传入

```java
@Backend("{0}")
@Post("/data")
String send(String backend, @Body MyUser user);
```

## 自定义后端 Client 对象

Forest 在默认情况下会自动生成后端 Client 对象实例，并进行缓存。但如果您想进行更细致的操作时也可以自行生成和配置后端 Client 对象。

### OkHttpClient

自定义 OkHttp3 框架的 OkHttpClient 对象，只需实现`OkHttpClientProvider`接口

```java
public class MyOkHttpClientProvider implements OkHttpClientProvider {

    @Override
    public OkHttpClient getClient(ForestRequest request, LifeCycleHandler lifeCycleHandler) {
        OkHttpClient okHttpClient = new OkHttpClient.Builder()
                .connectTimeout(700, TimeUnit.SECONDS)
                .readTimeout(700, TimeUnit.SECONDS)
                .writeTimeout(700, TimeUnit.SECONDS)
                .callTimeout(700, TimeUnit.SECONDS)
                .followSslRedirects(false)
                .retryOnConnectionFailure(false)
                .followRedirects(false)
                .build();
        return okHttpClient;
    }
}
```

绑定该自定义的 OkHttpClient 对象提供者



 





```java
@Get("/")
@OkHttp3(client = MyOkHttpClientProvider.class)
ForestResponse<String> sendData();
```



### HttpClient

同理，自定义 Apache Httpclient 框架的 HttpClient 对象，也只需实现`HttpClientProvider`接口

```java
public class MyHttpClientProvider implements HttpClientProvider {

    @Override
    public HttpClient getClient(ForestRequest request, LifeCycleHandler lifeCycleHandler) {
        RequestConfig config = RequestConfig.custom()
                .setConnectTimeout(700 * 1000)
                .setSocketTimeout(700 * 1000)
                .setConnectionRequestTimeout(700 * 1000)
                .setRedirectsEnabled(false)
                .build();
        CloseableHttpClient httpClient = HttpClients.custom()
                .setDefaultRequestConfig(config)
                .build();
        return httpClient;
    }
}
```

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16

绑定该自定义的 HttpClient 对象提供者



 





```java
@Get("/")
@HttpClient(client = MyHttpClientProvider.class)
ForestResponse<String> sendData();
```

## 后端 Client 缓存

为提高 Forest 请求的执行性能，默认情况下，每个请求所对应的后端客户端对象都会被**缓存**

请求前，会先去缓存中寻找所需的后端 Client 对象实例，如若没有，则新创建一个并放入该请求所对应的缓存中

接口的缓存开关设定如下:





 





```java
// 关闭后端 Client 缓存
@Get("/")
@BackendClient(cache = false)
ForestRequest<String> sendData();
```

1
2
3
4

### 接口注解

### 接收数据

### 数据转换

### 成功/失败条件

### 重试机制

### 重定向

### Gzip解压

### 日志管理

### 回调函数

在Forest中的回调函数使用单方法的接口定义，这样可以使您在 `Java 8` 或 `Kotlin` 语言中方便使用 `Lambda` 表达式。

## [#](https://forest.dtflyx.com/pages/1.5.32/callback/#成功-失败回调函数)成功/失败回调函数

在接口方法加入`OnSuccess<T>`类型或`OnError`类型的参数

```java
@Request(
        url = "http://localhost:8080/hello/user",
        headers = {"Accept:text/plain"},
        data = "username=${username}"
)
String send(@Var("username") String username, OnSuccess<String> onSuccess, OnError onError);
```

1
2
3
4
5
6

如这两个回调函数的类名所示的含义一样，`OnSuccess<T>`在请求成功调用响应时会被调用，而`OnError`在失败或出现错误的时候被调用。

其中`OnSuccess<T>`的泛型参数`T`定义为请求响应返回结果的数据类型。

```java
myClient.send("foo", (String resText, ForestRequest request, ForestResponse response) -> {
        // 成功响应回调
        System.out.println(resText);    
    },
    (ForestRuntimeException ex, ForestRequest request, ForestResponse response) -> {
        // 异常回调
        System.out.println(ex.getMessage());
    });
```

1
2
3
4
5
6
7
8

提示

- 在异步请求中只能通过OnSuccess<T>回调函数接或Future返回值接受数据。
- 而在同步请求中，OnSuccess<T>回调函数和任何类型的返回值都能接受到请求响应的数据。
- OnError回调函数可以用于异常处理，一般在同步请求中使用try-catch也能达到同样的效果。

## [#](https://forest.dtflyx.com/pages/1.5.32/callback/#下载进度回调函数)下载进度回调函数

在接口方法加入`OnProgress`类型的参数

```java
/**
 * OnProgress 回调函数可以用于下载文件类请求方法的参数中
 */
@Get("/xxx-img.jpg")
byte[] downloadFile(OnProgress onProgress);
```

1
2
3
4
5

如果请求成功访问到URL指定的文件资源，且开始进行下载，则会反复调用参数中传入的 `OnProgress` 回调函数

```java
// 每传输一定的字节数，便会调用一次 OnProgress 回调函数
byte[] bytes = downloadClient.downloadFile(progress -> {
    System.out.println("------------------------------------------");
    System.out.println("total bytes: " + progress.getTotalBytes()); // 文件总字节数
    System.out.println("current bytes: " + progress.getCurrentBytes()); // 当前已传输字节数
    System.out.println("progress: " + Math.round(progress.getRate() * 100) + "%"); // 传输百分百
    if (progress.isDone()) {
        // 若已传输完毕
        System.out.println("--------   Download Completed!   --------");
        atomicProgress.set(progress);
    }
});
```

1
2
3
4
5
6
7
8
9
10
11
12

## [#](https://forest.dtflyx.com/pages/1.5.32/callback/#重定向回调函数)重定向回调函数

当前请求响应接受到的是 `302`、`304` 等状态码时，Forest 会触发自动重定向，即会立刻发送一个新的请求

若要拦截或修改新的跳转请求可以使用 `OnRedirection` 回调函数

```java
/**
 * OnRedirection 回调函数可以用自动重定向请求方法的参数中
 */
@Post("/")
String testRedirect(OnRedirection onRedirection);
```

1
2
3
4
5

当触发重定向时，在发送新的跳转请求前，会调用参数中传入的 `OnRedirection` 回调函数

```java
String result = redirectClient.testNotAutoRedirect(((redirectReq, prevReq, prevRes) -> {
    // prevReq 为跳转前的请求对象
    // prevRes 为跳转前接受到的响应对象
    // redirectReq 为新的即将跳转的请求对象
}));
```

1
2
3
4
5

### 异步请求

## 设置异步/同步

在Forest使用异步请求，可以通过设置`@Request`注解的`async`属性为`true`实现，不设置或设置为`false`即为同步请求

```java
/**
 * async 属性为 true 即为异步请求，为 false 则为同步请求
 * 不设置该属性时，默认为 false
 */
@Get(
        url = "http://localhost:8080/hello/user?username=${0}",
        async = true
)
String asyncGet(String username);
```

1
2
3
4
5
6
7
8
9

## [#](https://forest.dtflyx.com/pages/1.5.32/async/#使用回调函数)使用回调函数

异步请求的方法无法直接通过返回值接受服务端响应的结果，因为在网络还在没完成连接和响应的过程中，方法已经返回了

此时需要成功/失败回调函数来响应网络请求的结果

```java
@Get(
        url = "http://localhost:8080/hello/user?username=${0}",
        async = true,
        headers = {"Accept:text/plain"}
)
void asyncGet(String username， OnSuccess<String> onSuccess,OnError onError);
```

1
2
3
4
5
6

一般情况下，异步请求都通过`OnSuccess<T>` 回调函数来接受响应返回的数据，而不是通过接口方法的返回值，所以这里的返回值类型一般会定义为`void`。

文档导航

关于回调函数的使用请参见 《[回调函数](https://forest.dtflyx.com/pages/1.5.32/callback/)》

```java
// 异步执行
myClient.asyncGet("foo",(result,req,res)->{
        // 请求成功，处理响应结果
        System.out.println(result);
        },(ex,req,res)->{
        // 请求失败，处理失败信息
        System.out.println(ex.getMessage());
        });
```

1
2
3
4
5
6
7
8

## [#](https://forest.dtflyx.com/pages/1.5.32/async/#使用-future-接受异步数据)使用 Future 接受异步数据

此外，若有些小伙伴不习惯这种函数式的编程方式，也可以用`Future<T>`类型定义方法返回值的方式来接受响应数据。

```java
@Request(
        url = "http://localhost:8080/hello/user?username=foo",
        async = true,
        headers = {"Accept:text/plain"}
)
Future<String> asyncFuture();
```

1
2
3
4
5
6

这里`Future<T>`类就是`JDK`自带的`java.util.concurrent.Future`类, 其泛型参数`T`代表您想接受的响应数据的类型。

关于如何使用`Future`类，这里不再赘述。

```java
// 异步执行
Future<String> future=myClient.asyncFuture();

// 做一些其它事情

// 等待数据
        String result=future.get();
```

1
2
3
4
5
6
7

## [#](https://forest.dtflyx.com/pages/1.5.32/async/#开启-kotlin-协程)开启 Kotlin 协程

自`1.5.27`版本开始 Forest 支持 Kotlin 语言的协程特性，可以在发送异步请求的时候使用 Kotlin 协程代替 JVM 的线程池

在使用 Kotlin 协程之前，需要先确保可以使用 Kotlin 语言，并有以下依赖

- Maven
- Gradle

```xml
<dependency>
    <groupId>org.jetbrains.kotlin</groupId>
    <artifactId>kotlin-stdlib</artifactId>
    <version>1.6.20</version>
</dependency>

<dependency>
    <groupId>org.jetbrains.kotlinx</groupId>
    <artifactId>kotlinx-coroutines-core</artifactId>
    <version>1.6.2</version>
</dependency>
```

1
2
3
4
5
6
7
8
9
10
11

Forest 默认情况下，请求的异步模式为`platform`，即 JVM 平台自带的线程池

如果要启用 Kotlin 线程，需要将异步模式设置为`kotlin_coroutine`

- Yaml
- Properties
- 原生 Java

```yaml
forest:
  async-mode: kotlin_coroutine
```

### HTTPS

为保证网络访问安全，现在大多数企业都会选择使用SSL验证来提高网站的安全性。

所以Forest自然也加入了对HTTPS的处理，现在支持单向认证和双向认证的HTTPS请求。

## [#](https://forest.dtflyx.com/pages/1.5.32/https/#单向认证)单向认证

如果访问的目标站点的SSL证书由信任的Root CA发布的，那么您无需做任何事情便可以自动信任

```java
public interface Gitee {
    @Request(url = "https://gitee.com")
    String index();
}
```

1
2
3
4
5

Forest的单向验证的默认协议为`TLS`，如果一些站点的API不支持该协议，您可以在全局配置中将`ssl-protocol`属性修改为其它协议，如：`SSL`, `TLS`, `TLSv1.1`, `TLSv1.2`,`TLSv1.3`, `SSLv3`等等。

```yaml
forest:
  ...
  ssl-protocol: TLS
```

1
2
3

全局配置可以配置一个全局统一的SSL协议，但现实情况是有很多不同服务（尤其是第三方）的API会使用不同的SSL协议，这种情况需要针对不同的接口设置不同的SSL协议。

```java
/**
 * 在某个请求接口上通过 sslProtocol 属性设置单向SSL协议
 */
@Get(
    url = "https://localhost:5555/hello/user",
    sslProtocol = "TLS"
)
ForestResponse<String> truestSSLGet();
```

1
2
3
4
5
6
7
8

在一个个方法上设置太麻烦，也可以在 `@BaseRequest` 注解中设置一整个接口类的SSL协议

```java
@BaseRequest(sslProtocol = "TLS")
public interface SSLClient {

    @Get("https://localhost:5555/hello/user")
    String testSend();

}
```

1
2
3
4
5
6
7

## [#](https://forest.dtflyx.com/pages/1.5.32/https/#简单双向认证)简单双向认证

若是需要在Forest中进行双向验证的HTTPS请求，也很简单。

在全局配置中添加`keystore`配置：

- SpringBoot
- Spring
- Java

```yaml
forest:
 ...
 ssl-key-stores:
   - id: keystore1           # id为该keystore的名称，必填
     file: test.keystore     # 公钥文件地址
     keystore-pass: 123456   # keystore秘钥
     cert-pass: 123456       # cert秘钥
     protocols: TLS          # SSL协议
```

1
2
3
4
5
6
7
8

接着，在`@Request`中引入该`keystore`的`id`即可

```java
@Get(url = "/user_info", keyStore = "keystore1")
ForestResponse<String> getUserInfo();
```

1
2

另外，您也可以在全局配置中配多个`keystore`：

```yaml
forest:
  ...
  ssl-key-stores:
    - id: keystore1          # 第一个keystore
      file: test1.keystore    
      keystore-pass: 123456  
      cert-pass: 123456      
      protocols: SSL       

    - id: keystore2          # 第二个keystore
      file: test2.keystore    
      keystore-pass: abcdef  
      cert-pass: abcdef      
      protocols: TLS       
      ...
```

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15

随后在某个具体`@Request`中配置其中任意一个`keystore`的`id`都可以

## [#](https://forest.dtflyx.com/pages/1.5.32/https/#更复杂的ssl验证)更复杂的SSL验证

对于一些更复杂的SSL验证，光靠简单的配置无法完全应付，但在Forest中也可以通过自定义`SSLSocketFactory`和`HostnameVerifier`的方式提供更灵活的解决方案

### [#](https://forest.dtflyx.com/pages/1.5.32/https/#自定义sslsocketfactory)自定义SSLSocketFactory

Forest提供了`SSLSocketFactoryBuilder`接口，通过实现该接口可以自定义`SSLSocketFactory`

```java
/**
 * 自定义 SSLSocketFactory 构造器
 */
public class MySSLSocketFactoryBuilder implements SSLSocketFactoryBuilder {

    /**
     * 获取SSL Socket Factory
     */
    @Override
    public SSLSocketFactory getSSLSocketFactory(ForestRequest request, String protocol) throws Exception {
        SSLContext sslContext = SSLContext.getInstance("TLS");
        sslContext.init(null,
                new TrustManager[] { new TrustAllManager() },
                new SecureRandom());
        return sslContext.getSocketFactory();
    }
}
```

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17

然后通过`@SSLSocketFactoryBuilder`注解引入自定义的`SSLSocketFactoryBuilder`接口实现类

```java
@Get("/user_info")
@SSLSocketFactoryBuilder(MySSLSocketFactoryBuilder.class)
ForestRequest<String> getUserInfo();
```

1
2
3

### [#](https://forest.dtflyx.com/pages/1.5.32/https/#自定义hostnameverifier)自定义HostnameVerifier

在Forest中，也可以指定自定义的`HostnameVerifier`接口实现类

```java
/**
 * 自定义主机名验证器
 */
public class MyHostnameVerifier implements HostnameVerifier {
    @Override
    public boolean verify(String s, SSLSession sslSession) {
        System.out.println("do MyHostnameVerifier");
        if ("localhost".equals(s)) {
            return false;
        }
        return true;
    }
}
```

1
2
3
4
5
6
7
8
9
10
11
12
13

然后通过`@SSLHostnameVerifier`注解引入自定义的主机名验证器类

```java
@Get("/user_info")
@SSLHostnameVerifier(MyHostnameVerifier.class)
ForestRequest<String> getUserInfo();
```



### 使用Cookie

Cookie是由服务器端生成，发送给客户端（一般为浏览器），并以key-value形式处理和保存在客户端的一组数据。在下次请求同一域名网站时，会将该Cookie数据再次发送到服务端。

Forest从`1.5.0-RC1`版本开始支持Cookie，可以通过回调函数和拦截器两种方式来处理Cookie。

## [#](https://forest.dtflyx.com/pages/1.5.32/cookie/#回调函数方式)回调函数方式

在请求接口的参数列表中加入`OnSaveCookie` 和 `OnLoadCookie` 回调函数

`OnSaveCookie`： 在请求响应成功后，需要保存Cookie时调用该回调函数

`OnLoadCookie`: 在发送请求前，需要加载Cookie时调用该回调函数

```java
/**
 * 登入接口(需要接受Cookie)
 */
@Post("http://localhost:8080/login?username=foo")
ForestResponse testLogin(@Body UserLoginDTO userLogin, OnSaveCookie onSaveCookie);

/**
 * 登入后测试接口(需要传入Cookie)
 */
@Post("http://localhost:8080/test")
ForestResponse testAfterLogin(OnLoadCookie onLoadCookie);
```

1
2
3
4
5
6
7
8
9
10
11
12
13

Forest**不会**自动处理或持久化服务端传来的Cookie数据，需要自己在回调函数中接到Cookie后自行处理。

```java
AtomicReference<ForestCookie> cookieAtomic = new AtomicReference<>(null);

// 调用登入接口
testClient.testLogin(userLogin, (request, cookies) -> {
    // 将服务端传来的Cookie放入cookieAtomic
    cookieAtomic.set(cookies.allCookies().get(0));
});

// 获取Cookie
ForestCookie cookie = cookieAtomic.get();

// 调用登入后的测试接口
ForestResponse response = testClient.testAfterLogin((request, cookies) -> {
    // 将之前调用登入接口获得的Cookie传入请求发送到服务端
    cookies.addCookie(cookie);
});
```

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18

## [#](https://forest.dtflyx.com/pages/1.5.32/cookie/#拦截器方式)拦截器方式

拦截器方式原理上和回调函数方式一样，只不过`OnSaveCookie` 和 `OnLoadCookie` 回调函数接口变成了拦截器中的 `onSaveCookie(ForestRequest, ForestCookies)` 方法和 `onLoadCookie(ForestRequest, ForestCookies)` 方法。

```java
/**
 * 处理Cookie的拦截器
 */
public class CookieInterceptor implements Interceptor {
    
    // Cookie在本地存储的缓存
    private Map<String, List<ForestCookie>> cookieCache = new ConcurrentHashMap<>();

    /**
     * 在请求响应成功后，需要保存Cookie时调用该方法
     *
     * @param request Forest请求对象
     * @param cookies Cookie集合，通过响应返回的Cookie都从该集合获取
     */
    @Override
    public void onSaveCookie(ForestRequest request, ForestCookies cookies) {
        // 获取请求URI的主机名
        String host = request.getURI().getHost();
        // 将从服务端获得的Cookie列表放入缓存，主机名作为Key
        cookieCache.put(host, cookies.allCookies());
    }

    /**
     * 在发送请求前，需要加载Cookie时调用该方法
     *
     * @param request Forest请求对象
     * @param cookies Cookie集合, 需要通过请求发送的Cookie都添加到该集合
     */
    @Override
    public void onLoadCookie(ForestRequest request, ForestCookies cookies) {
        // 获取请求URI的主机名
        String host = request.getURI().getHost();
        // 从缓存中获取之前获得的Cookie列表，主机名作为Key
        List<ForestCookie> cookieList = cookieCache.get(host);
        // 将缓存中的Cookie列表添加到请求Cookie列表中，准备发送到服务端
        // 默认情况下，只有符合条件 (和请求同域名、同URL路径、未过期) 的 Cookie 才能被添加到请求中
        cookies.addAllCookies(cookieList);
    }

    @Override
    public void onError(ForestRuntimeException ex, ForestRequest request, ForestResponse response) {
        // ... ...
    }

    @Override
    public void onSuccess(Object data, ForestRequest request, ForestResponse response) {
        // ... ...
    }
}
```

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49

## [#](https://forest.dtflyx.com/pages/1.5.32/cookie/#严格匹配模式-v1-5-25)严格匹配模式 (v1.5.25)

默认情况下，只有符合条件 (和请求同域名、同URL路径、未过期) 的 Cookie 才能被添加到请求中

这是因为 Forest 的 Cookie 集合默认为严格匹配模式，如果想添加符合匹配要求的 Cookie，只需修改严格匹配为`false`即可

```java
@Override
public void onLoadCookie(ForestRequest request, ForestCookies cookies) {
    cookies.strict(false) // 设置为非严格匹配模式
        .addCookie(new ForestCookie("attr1", "foo")) // 不设域名，默认情况下也能添加
        .addCookie(new ForestCookie("attr2", "bar")) // 不设域名，默认情况下也能添加
        // 不设域名，只有在非严格匹配模式下可以添加到请求中
        .addCookie(new ForestCookie("attr3", "foobar").setDomain("xxx.com"));
}
```

1
2
3
4
5
6
7
8

### 使用代理

众所周知，我们平时访问HTTP，就是直接输入URL外加Query参数或Body参数就开始直接发送请求，随后等待服务端响应便可。请求通过内网或公网一般都可以正常工作（如果客户端和服务端设备正常联通网络的话），就如下图所示：

![proxy](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/202307041419510.svg+xml)

### [#](https://forest.dtflyx.com/pages/1.5.32/proxy/#正向代理)正向代理

但也有很多服务由于网络限制等诸多原因，无法直接访问到。这时候就需要先连接代理服务器，然后再由代理服务器转发请求到您原本要访问的原始服务器。 这种方式，便称为正向代理，过程如下图所示：

![proxy](https://forest.dtflyx.com/img/proxy.svg)

当然，正向代理除了能访问原本访问不到的资源这一功能外，还有其它用处：

- 访问原来无法访问的资源，如: Google、油管、企业私有网络服务等
- 做缓存，加速访问速度
- 对客户端访问授权和认证
- 记录用户访问记录（上网行为管理）

除了有正向代理外，自然也有反向代理，但这个概念不在本文档讨论范围，有兴趣可以自行搜索查询相关资料。

### [#](https://forest.dtflyx.com/pages/1.5.32/proxy/#设置正向代理)设置正向代理

Forest从`1.5.0-RC1`版本开始支持HTTP网络代理，而作为一个HTTP客户端框架，自然提供的是对正向代理的支持。

通过`@HTTPProxy`注解便可以非常简单地为某一个请求方法设置代理，该注解有两个属性：

- host: 代理服务器主机地址
- port: 代理服务器端口号

```java
/**
 * 使用 @HTTPProxy 注解设置代理服务器
 * host属性为代理服务器主机地址
 * port属性为代理服务器端口号
 */
@Post(
    url = "http://localhost:8080/hello",
    data = "username=foo&password=123456"
)
@HTTPProxy(host = "127.0.0.1", port = "10801")
String simplePostWithProxy(@Header("Accept") String accept);
```

1
2
3
4
5
6
7
8
9
10
11

`@HTTPProxy`注解也可以设置在接口类上，批量给接口中所有方法设置相同的代理服务器

```java
/**
 * 为 PostClient 接口中所有的请发方法设置代理服务器
 */
@BaseRequest(baseURL = "http://localhost:8080")
@HTTPProxy(host = "127.0.0.1", port = "10801")
public interface PostClient {
    
    ... ...

}
```

1
2
3
4
5
6
7
8
9
10
11

### [#](https://forest.dtflyx.com/pages/1.5.32/proxy/#设置用户名密码)设置用户名密码

如果您访问的代理服务器需要进行身份校验，则在 `@HTTProxy` 注解中设置用户名和密码

```java
/**
 * 在 @HTTPProxy 注解中有 username 和 password 属性
 * 分别用于设置代理服务的用户名和密码进行身份验证
 */
@Post(
    url = "http://localhost:8080/hello",
    data = "username=foo&password=123456"
)
@HTTPProxy(
        host = "127.0.0.1",
        port = "10801",
        username = "foo",
        password = "bar"
)
String simplePostWithProxy(@Header("Accept") String accept);
```

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16

### 上传下载

Forest从 `1.4.0` 版本开始支持多种形式的文件上传和文件下载功能

### [#](https://forest.dtflyx.com/pages/1.5.32/upload_download/#上传)上传

```java
/**
 * 用@DataFile注解修饰要上传的参数对象
 * OnProgress参数为监听上传进度的回调函数
 */
@Post(url = "/upload")
Map upload(@DataFile("file") String filePath, OnProgress onProgress);
```

1
2
3
4
5
6

调用上传接口以及监听上传进度的代码如下：

```java
Map result = myClient.upload("D:\\TestUpload\\xxx.jpg", progress -> {
    System.out.println("total bytes: " + progress.getTotalBytes());   // 文件大小
    System.out.println("current bytes: " + progress.getCurrentBytes());   // 已上传字节数
    System.out.println("progress: " + Math.round(progress.getRate() * 100) + "%");  // 已上传百分比
    if (progress.isDone()) {   // 是否上传完成
        System.out.println("--------   Upload Completed!   --------");
    }
});
```

1
2
3
4
5
6
7
8

在文件上传的接口定义中，除了可以使用字符串表示文件路径外，还可以用以下几种类型的对象表示要上传的文件:

```java
/**
 * File类型对象
 */
@Post(url = "/upload")
Map upload(@DataFile("file") File file, OnProgress onProgress);

/**
 * byte数组
 * 使用byte数组和Inputstream对象时一定要定义fileName属性
 */
@Post(url = "/upload")
Map upload(@DataFile(value = "file", fileName = "${1}") byte[] bytes, String filename);

/**
 * Inputstream 对象
 * 使用byte数组和Inputstream对象时一定要定义fileName属性
 */
@Post(url = "/upload")
Map upload(@DataFile(value = "file", fileName = "${1}") InputStream in, String filename);

/**
 * Spring Web MVC 中的 MultipartFile 对象
 */
@PostRequest(url = "/upload")
Map upload(@DataFile(value = "file") MultipartFile multipartFile, OnProgress onProgress);

/**
 * Spring 的 Resource 对象
 */
@Post(url = "/upload")
Map upload(@DataFile(value = "file") Resource resource);
```

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31

### [#](https://forest.dtflyx.com/pages/1.5.32/upload_download/#多文件批量上传)多文件批量上传

```java
/**
 * 上传Map包装的文件列表
 * 其中 ${_key} 代表Map中每一次迭代中的键值
 */
@PostRequest(url = "/upload")
ForestRequest<Map> uploadByteArrayMap(@DataFile(value = "file", fileName = "${_key}") Map<String, byte[]> byteArrayMap);

/**
 * 上传List包装的文件列表
 * 其中 ${_index} 代表每次迭代List的循环计数（从零开始计）
 */
@PostRequest(url = "/upload")
ForestRequest<Map> uploadByteArrayList(@DataFile(value = "file", fileName = "test-img-${_index}.jpg") List<byte[]> byteArrayList);

/**
 * 上传数组包装的文件列表
 * 其中 ${_index} 代表每次迭代List的循环计数（从零开始计）
 */
@PostRequest(url = "/upload")
ForestRequest<Map> uploadByteArrayArray(@DataFile(value = "file", fileName = "test-img-${_index}.jpg") byte[][] byteArrayArray);
```

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22

### [#](https://forest.dtflyx.com/pages/1.5.32/upload_download/#二进制上传)二进制上传

对于 `application/octect-stream` 等非form-data的Content-Type类型，直接用`@Body`修饰要上传的数据参数

```java
/**
 * 上传Byte数组类型数据
 */
@Post(
        url = "/upload/${filename}",
        contentType = "application/octet-stream"
)
String uploadByteArryr(@Body byte[] body, @Var("filename") String filename);

/**
 * 上传File类型数据
 */
@Post(
    url = "/upload/${filename}",
    contentType = "application/octet-stream"
)
String uploadFile(@Body File file, @Var("filename") String filename);

/**
 * 上传输入流类型数据
 */
@Post(
    url = "/upload/${filename}",
    contentType = "application/octet-stream"
)
String uploadInputStream(@Body InputStream inputStream, @Var("filename") String filename);
```

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27

### [#](https://forest.dtflyx.com/pages/1.5.32/upload_download/#下载)下载

```java
/**
 * 在方法上加上@DownloadFile注解
 * dir属性表示文件下载到哪个目录
 * filename属性表示文件下载成功后以什么名字保存，如果不填，这默认从URL中取得文件名
 * OnProgress参数为监听上传进度的回调函数
 */
@Get(url = "http://localhost:8080/images/xxx.jpg")
@DownloadFile(dir = "${0}", filename = "${1}")
File downloadFile(String dir, String filename, OnProgress onProgress);
```

1
2
3
4
5
6
7
8
9

调用下载接口以及监听上传进度的代码如下：

```java
File file = myClient.downloadFile("D:\\TestDownload", progress -> {
    System.out.println("total bytes: " + progress.getTotalBytes());   // 文件大小
    System.out.println("current bytes: " + progress.getCurrentBytes());   // 已下载字节数
    System.out.println("progress: " + Math.round(progress.getRate() * 100) + "%");  // 已下载百分比
    if (progress.isDone()) {   // 是否下载完成
        System.out.println("--------   Download Completed!   --------");
    }
});
```

1
2
3
4
5
6
7
8

如果您不想将文件下载到硬盘上，而是直接在内存中读取，可以去掉@DownloadFile注解，并且用以下几种方式定义接口:

```java
/**
 * 返回类型用byte[]，可将下载的文件转换成字节数组
 */
@GetRequest(url = "http://localhost:8080/images/test-img.jpg")
byte[] downloadImageToByteArray();

/**
 * 返回类型用InputStream，用流的方式读取文件内容
 */
@GetRequest(url = "http://localhost:8080/images/test-img.jpg")
InputStream downloadImageToInputStream();
```

1
2
3
4
5
6
7
8
9
10
11
12

以`InputStream`类型接受的数据在读取后一定别忘了关闭流

```java
// 使用 try-with-resource 机制自动关闭流
try (InputStream in = downloadImageToInputStream()) {
    String content = IOUtils.toString(object, StandardCharsets.UTF_8);
    ... ...
} catch(Exception ex) {
    ex.printStackTrace();
}

// 或者自行关闭流        
InputStream in = null;
try {
    in = downloadImageToInputStream();
    String content = IOUtils.toString(object, StandardCharsets.UTF_8);
    ... ...
} catch(Exception ex) {
    ex.printStackTrace();
} finally {
    if (in != null){
        in.close();
    }
}
```

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22

注意

用**File**类型定义的文件下载接口方法，一定要加上**@DownloadFile**注解

### 异常处理

发送HTTP请求不会总是成功的，总会有失败的情况。Forest提供多种异常处理的方法来处理请求失败的过程。

### [#](https://forest.dtflyx.com/pages/1.5.32/exception/#try-catch方式)try-catch方式

最常用的是直接用`try-catch`。Forest请求失败的时候通常会以抛异常的方式报告错误， 获取错误信息只需捕获`ForestNetworkException`异常类的对象，如示例代码所示：

```java
/**
 * try-catch方式：捕获ForestNetworkException异常类的对象
 */
try {
    String result = myClient.send();
} catch (ForestNetworkException ex) {
    int status = ex.getStatusCode(); // 获取请求响应状态码
    ForestResponse response = ex.getResponse(); // 获取Response对象
    String content = response.getContent(); // 获取请求的响应内容
    String resResult = response.getResult(); // 获取方法返回类型对应的最终数据结果
}
```

1
2
3
4
5
6
7
8
9
10
11

### [#](https://forest.dtflyx.com/pages/1.5.32/exception/#回调函数方式)回调函数方式

第二种方式是使用`OnError`回调函数，如示例代码所示：

```java
/**
 * 在请求接口中定义OnError回调函数类型参数
 */
@Request(
        url = "http://localhost:8080/hello/user",
        headers = {"Accept:text/plain"},
        data = "username=${username}"
)
String send(@Var("username") String username, OnError onError);
```

1
2
3
4
5
6
7
8
9

调用的代码如下：

```java
// 在调用接口时，在Lambda中处理错误结果
myClient.send("foo",  (ex, request, response) -> {
    int status = response.getStatusCode(); // 获取请求响应状态码
    String content = response.getContent(); // 获取请求的响应内容
    String result = response.getResult(); // 获取方法返回类型对应的最终数据结果
});
```

1
2
3
4
5
6

注意

加上**OnError**回调函数后便**不会再向上抛出异常**，所有错误信息均通过**OnError**回调函数的参数获得。

### [#](https://forest.dtflyx.com/pages/1.5.32/exception/#forestresponse)ForestResponse

第三种，用`ForestResponse`类作为请求方法的返回值类型，示例代码如下：

```java
/**
 * 用`ForestResponse`类作为请求方法的返回值类型, 其泛型参数代表实际返回数据的类型
 */
@Request(
        url = "http://localhost:8080/hello/user",
        headers = {"Accept:text/plain"},
        data = "username=${username}"
)
ForestResponse<String> send(@Var("username") String username);
```

1
2
3
4
5
6
7
8
9

调用和处理的过程如下：

```java
ForestResponse<String> response = myClient.send("foo");
// 用isError方法判断请求是否失败, 比如404, 500等情况
if (response.isError()) {
    int status = response.getStatusCode(); // 获取请求响应状态码
    String content = response.getContent(); // 获取请求的响应内容
    String result = response.getResult(); // 获取方法返回类型对应的最终数据结果
}
```

1
2
3
4
5
6
7

注意

以**ForestResponse**类为返回值类型的方法也**不会向上抛出异常**，错误信息均通过**ForestResponse**对象获得。

### [#](https://forest.dtflyx.com/pages/1.5.32/exception/#拦截器方式)拦截器方式

若要批量处理各种不同请求的异常情况，可以定义一个拦截器, 并在拦截器的`onError`方法中处理异常，示例代码如下：

```java
public class ErrorInterceptor implements Interceptor<String> {

    // ... ...

    @Override
    public void onError(ForestRuntimeException ex, ForestRequest request, ForestResponse response) {
        int status = response.getStatusCode(); // 获取请求响应状态码
        String content = response.getContent(); // 获取请求的响应内容
        Object result = response.getResult(); // 获取方法返回类型对应的返回数据结果
    }
}
```

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