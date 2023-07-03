# Forest 概述

Forest 是一个开源的 Java HTTP 客户端框架，能够将 HTTP 的所有请求信息（包括 URL、Header 以及 Body 等信息）绑定到您自定义的 Interface 方法上，能够通过调用本地接口方法的方式发送 HTTP 请求。

使用 Forest 就像使用类似 Dubbo 那样的 RPC 框架一样，只需要定义接口，调用接口即可，不必关心具体发送 HTTP 请求的细节。同时将 HTTP 请求信息与业务代码解耦，方便您统一管理大量 HTTP 的 URL、Header 等信息。而请求的调用方完全不必在意 HTTP 的具体内容，即使该 HTTP 请求信息发生变更，大多数情况也不需要修改调用发送请求的代码。

Forest 不需要编写具体的 HTTP 调用过程，只需要定义一个接口，然后通过 Forest 注解将 HTTP 请求的信息添加到接口的方法上即可。请求发送方通过调用定义的接口便能自动发送请求和接受请求的响应。

**Forest官网**：http://forest.dtflyx.com/

## Forest 工作原理

Forest 会将定义好的接口通过动态代理的方式生成一个具体的实现类，然后组织、验证 HTTP 请求信息，绑定动态数据，转换数据形式，SSL 验证签名，调用后端 HTTP API(httpclient 等 API)执行实际请求，等待响应，失败重试，转换响应数据到 Java 类型等脏活累活都由这动态代理的实现类给包了。 请求发送方调用这个接口时，实际上就是在调用这个干脏活累活的实现类。

## Forest 的架构

![forest-architecture](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/202307031324179.png)

将 HTTP 发送请求的过程分为前端部分和后端部分，Forest 本身是处理前端过程的框架，是对后端 HTTP API 框架的进一步封装。

**前端部分：**

1. Forest 配置： 负责管理 HTTP 发送请求所需的配置。
2. Forest 注解： 用于定义 HTTP 发送请求的所有相关信息，一般定义在 interface 上和其方法上。
3. 动态代理： 用户定义好的 HTTP 请求的`interface`将通过动态代理产生实际执行发送请求过程的代理类。
4. 模板表达式： 模板表达式可以嵌入在几乎所有的 HTTP 请求参数定义中，它能够将用户通过参数或全局变量传入的数据动态绑定到 HTTP 请求信息中。
5. 数据转换： 此模块将字符串数据和`JSON`或`XML`形式数据进行互转。目前 JSON 转换器支持`Jackson`、`Fastjson`、`Gson`三种，XML 支持`JAXB`一种。
6. 拦截器： 用户可以自定义拦截器，拦截指定的一个或一批请求的开始、成功返回数据、失败、完成等生命周期中的各个环节，以插入自定义的逻辑进行处理。
7. 过滤器： 用于动态过滤和处理传入 HTTP 请求的相关数据。
8. SSL： Forest 支持单向和双向验证的 HTTPS 请求，此模块用于处理 SSL 相关协议的内容。

**后端部分：**

后端为实际执行 HTTP 请求发送过程的第三方 HTTP API，目前支持`okHttp3`和`httpclient`两种后端 API。