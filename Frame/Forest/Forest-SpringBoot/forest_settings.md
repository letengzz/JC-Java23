# Forest 配置

Forest 遵循约定大于配置的理念，大多数情况下不需要进行配置，或填写非常简单的配置即可。但不同项目环境配置方式各有不同，需要根据项目需要，选择不同的环境进行配置。

## Springboot环境配置

若项目依赖`Spring Boot`，并加入了`spring-boot-starter-forest`依赖，就可以通过 `application.yml`/`application.properties` 方式定义配置。

**配置后端 HTTP API**：

目前 Forest 支持`okhttp3`和`httpclient`两种后端 HTTP API，若不配置该属性，默认为`okhttp3`：

> application.yml

```yaml
forest:
  #backend: okhttp3 # 配置后端HTTP API为 okhttp3
  backend: httpclient # 配置后端HTTP API为 httpclient
```

> application.properties

```properties
# 配置后端HTTP API为 httpclient
forest.backend=httpclient
# 配置后端HTTP API为 okhttp3
#forest.backend=okhttp3
```

**全局基本配置**：

在`application.yaml` / `application.properties`中配置的 HTTP 基本参数。

**注意**：**max-retry-count**只是简单机械的请求失败后的重试次数，所以一般建议设置为**0**。如果一定要多次重试，请一定要在保证服务端的**幂等性**的基础上进行重试，否则容易引发生产事故！

> application.yaml

```yaml
forest:
  backend: okhttp3             # 后端HTTP框架（默认为 okhttp3）
  max-connections: 1000        # 连接池最大连接数（默认为 500）
  max-route-connections: 500   # 每个路由的最大连接数（默认为 500）
  max-request-queue-size: 100  # [自v1.5.22版本起可用] 最大请求等待队列大小
  max-async-thread-size: 300   # [自v1.5.21版本起可用] 最大异步线程数
  max-async-queue-size: 16     # [自v1.5.22版本起可用] 最大异步线程池队列大小
  timeout: 3000                # [已不推荐使用] 请求超时时间，单位为毫秒（默认为 3000）
  connect-timeout: 3000        # 连接超时时间，单位为毫秒（默认为 timeout）
  read-timeout: 3000           # 数据读取超时时间，单位为毫秒（默认为 timeout）
  max-retry-count: 0           # 请求失败后重试次数（默认为 0 次不重试）
  ssl-protocol: TLS            # 单向验证的HTTPS的默认TLS协议（默认为 TLS）
  log-enabled: true            # 打开或关闭日志（默认为 true）
  log-request: true            # 打开/关闭Forest请求日志（默认为 true）
  log-response-status: true    # 打开/关闭Forest响应状态日志（默认为 true）
  log-response-content: true   # 打开/关闭Forest响应内容日志（默认为 false）
  async-mode: platform         # [自v1.5.27版本起可用] 异步模式（默认为 platform）
```

> application.properties

```properties
# 连接池最大连接数
forest.max-connections=1000
# 连接超时时间，单位为毫秒
forest.connect-timeout=3000
# 数据读取超时时间，单位为毫秒
forest.read-timeout=3000
```

**全局变量定义**：

Forest 可以在`forest.variables`属性下自定义全局变量。

其中 key 为变量名，value 为变量值。

全局变量可以在任何模板表达式中进行数据绑定。

> application.yaml

```yaml
forest:
  variables:
    username: foo      # 声明全局变量，变量名: username，变量值: foo
    userpwd: bar       # 声明全局变量，变量名: userpwd，变量值: bar
```

> application.properties

```properties
# 声明全局变量，变量名: username，变量值: foo
forest.variables.username=foo
# 声明全局变量，变量名: userpwd，变量值: bar
forest.variables.userpwd=bar
```

**配置 Bean ID**：

Forest 允许您在 yaml 文件中配置 Bean Id，它对应着`ForestConfiguration`对象在 Spring 上下文中的 Bean 名称。

> application.yaml

```yaml
forest:
  bean-id: config0 # 在spring上下文中bean的id，默认值为forestConfiguration
```

> application.properties

```properties
# 在spring上下文中bean的id，默认值为forestConfiguration
forest.bean-id: config0
```

然后便可以在 Spring 中通过 Bean 的名称引用到它

```java
@Resource(name = "config0")
private ForestConfiguration config0;
```

## Spring环境配置

Forest是基于约定大于配置的理念进行设计的，如果已经添加好了Forest Spring环境相关依赖，只要进行些简单的配置即可使用

若您的项目依赖的是`Spring`，而非`Spring Boot`，或者使用`xml`方式进行`Spring Bean`的配置，那么可以通过`Spring xml`的方式定义配置。

在进行`Spring`方式配置前，需要先确保项目在Maven中除了forest-core和spring外，还依赖forest-spring包

```xml
 <dependency>
     <groupId>com.dtflys.forest</groupId>
     <artifactId>forest-spring</artifactId>
     <version>1.5.32</version>
 </dependency>
```

**配置 XML SCEHEMA**：打开`spring`的上下文配置文件，在`beans`开头定义的属性中加入Forest的`Schema`

```xml
xmlns:forest="http://forest.dtflyx.com/schema/forest"
   ...
xsi:schemaLocation=" ...
http://forest.dtflyx.com/schema/forest
http://forest.dtflyx.com/schema/forest/forest-spring.xsd
..."
```

加入完成后：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:forest="http://forest.dtflyx.com/schema/forest"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://forest.dtflyx.com/schema/forest
       http://forest.dtflyx.com/schema/forest/forest-spring.xsd">
    
   ...

</beans>
```

**添加Forest基本配置的定义**：

1. 使用`forest:configuration`标签创建在Spring中的ForestConfiguration Bean

2. 使用`forest:var`标签定义变量

   **注意**：变量的作用域为该ForestConfiguration之下，所有跟这个配置对象绑定的Client都能访问到其下的变量，而别的ForestConfiguration下定义的变量不能访问。

```xml
<!-- Forest 全局配置 -->
<!-- id 在spring上下文中bean的id, 默认值为forestConfiguration -->
<!-- backend 后端HTTP API： okhttp3 -->
<!-- timeout 请求超时时间，单位为毫秒, 默认值为3000 -->
<!-- connectTimeout 连接超时时间，单位为毫秒, 默认值为2000 -->
<!-- retryCount 请求失败后重试次数，默认为0次不重试 -->
<!-- retryer 重试器类 -->
<!-- sslProtocol 单向验证的HTTPS的默认SSL协议，默认为SSLv3 -->
<!-- maxConnections 每个路由的最大连接数，默认为500 -->
<!-- maxRouteConnections 每个路由的最大连接数，默认为500 -->
<!-- maxRequestQueueSize [自v1.5.22版本起可用] 最大请求等待队列大小 -->
<!-- maxAsyncThreadSize [自v1.5.21版本起可用] 最大异步线程数 -->
<!-- maxAsyncQueueSize [自v1.5.22版本起可用] 最大异步线程池队列大小 -->
<!-- logEnabled 打开或关闭日志总开关，默认为true -->
<!-- logRequest 打开/关闭Forest请求日志（默认为 true） -->
<!-- logResponseStatus 打开/关闭Forest响应状态日志（默认为 true） -->
<!-- logResponseContent 打开/关闭Forest响应内容日志（默认为 false） -->
<forest:configuration
    id="config0"
    backend="httpclient"
    timeout="30000"
    connectTimeout="10000"
    retryCount="3"
    retryer="com.dtflys.forest.retryer.NoneRetryer"
    charset="UTF-8"
    sslProtocol="SSLv3"
    maxConnections="500"
    maxRouteConnections="500"
    maxRequestQueueSize="100"
    maxAsyncThreadSize="256"
    maxAsyncQueueSize="128"
    logEnabled="true"
    logRequest="false"
    logResponseStatus="false"
    logResponseContent="true">

   <!-- forest变量定义 开始 -->
   <forest:var name="baseUrl" value="http://www.xxx.com"/>
   <forest:var name="x" value="0"/>
   <forest:var name="y" value="1"/>
   <!-- forest变量定义 结束 -->

    <!-- SSL KeyStore定义 开始 -->
    <forest:ssl-keystore id="keystore1" file="test.keystore" keystorePass="123456" certPass="123456"/>
    <forest:ssl-keystore id="keystore2" file="test2.keystore" keystorePass="foo" certPass="bar"/>
    <!-- SSL KeyStore定义 结束 -->
    
    <!-- Forest转换器定义 开始 -->
    <!-- 设置JSON转换器 -->
    <forest:converter dataType="json" class="com.dtflys.forest.converter.json.ForestGsonConverter">
        <forest:parameter name="dateFormat" value="yyyy/MM/dd hh:mm:ss"/>
    </forest:converter>

    <!-- 设置XML转换器 -->
    <forest:converter dataType="xml" class="com.dtflys.forest.converter.xml.ForestJaxbConverter">
    </forest:converter>
    <!-- Forest转换器定义 结束 -->
    
</forest:configuration>
```

**创建Client Bean**：

创建Client Bean有两种方式

1. 通过`forest:client`标签创建单个Client Bean：

   ```xml
    <forest:client id="siteAClient" configuration="config0" class="com.xxx.client.SiteAClient"/>
   ```

2. 通过`forest:scan`标签制定back-package的方式批量创建Client Bean：

   ```xml
   <forest:scan configuration="config0" base-package="com.xxx.client"/>
   ```

## Solon环境配置

若您的项目依赖`Solon`，并加入了`forest-solon-plugin`依赖，就可以通过 `app.yml`/`app.properties` 方式定义配置。

**配置后端 HTTP API**：

目前 Forest 支持`okhttp3`和`httpclient`两种后端 HTTP API，若不配置该属性，默认为`okhttp3`

> app.yaml

```yaml
forest:
  backend: okhttp3 # 配置后端HTTP API为 okhttp3
  #backend: httpclient # 配置后端HTTP API为 httpclient
```

> app.properties

```properties
# 配置后端HTTP API为 okhttp3
forest.backend=okhttp3
# 配置后端HTTP API为 httpclient
#forest.backend=httpclient
```

**全局基本配置**：

在`app.yaml` / `app.properties` 中配置的 HTTP 基本参数：

**注意**：max-retry-count只是简单机械的请求失败后的重试次数，所以一般建议设置为0。如果一定要多次重试，请一定要在保证服务端的幂等性的基础上进行重试，否则容易引发生产事故！

> app.yaml

```yaml
forest:
  backend: okhttp3             # 后端HTTP框架（默认为 okhttp3）
  max-connections: 1000        # 连接池最大连接数（默认为 500）
  max-route-connections: 500   # 每个路由的最大连接数（默认为 500）
  max-request-queue-size: 100  # [自v1.5.22版本起可用] 最大请求等待队列大小
  max-async-thread-size: 300   # [自v1.5.21版本起可用] 最大异步线程数
  max-async-queue-size: 16     # [自v1.5.22版本起可用] 最大异步线程池队列大小
  timeout: 3000                # [已不推荐使用] 请求超时时间，单位为毫秒（默认为 3000）
  connect-timeout: 3000        # 连接超时时间，单位为毫秒（默认为 timeout）
  read-timeout: 3000           # 数据读取超时时间，单位为毫秒（默认为 timeout）
  max-retry-count: 0           # 请求失败后重试次数（默认为 0 次不重试）
  ssl-protocol: TLS            # 单向验证的HTTPS的默认TLS协议（默认为 TLS）
  log-enabled: true            # 打开或关闭日志（默认为 true）
  log-request: true            # 打开/关闭Forest请求日志（默认为 true）
  log-response-status: true    # 打开/关闭Forest响应状态日志（默认为 true）
  log-response-content: true   # 打开/关闭Forest响应内容日志（默认为 false）
  async-mode: platform         # [自v1.5.27版本起可用] 异步模式（默认为 platform）
```

> app.properties

```properties
# 后端HTTP框架（默认为 okhttp3）
forest.backend=okhttp3
# 连接池最大连接数（默认为 500）
forest.max-connections=1000
# 每个路由的最大连接数（默认为 500）
forest.max-route-connections=500
# [自v1.5.22版本起可用] 最大请求等待队列大小
forest.max-request-queue-size=800
# [自v1.5.21版本起可用] 最大异步线程数
forest.max-async-thread-size=300
# [自v1.5.22版本起可用] 最大异步线程池队列大小
forest.max-async-queue-size=16
# (已不推荐使用) 请求超时时间，单位为毫秒（默认为 3000）
forest.timeout=3000
# 连接超时时间，单位为毫秒（默认为 timeout）
forest.connect-timeout=3000
# 数据读取超时时间，单位为毫秒（默认为 timeout）
forest.read-timeout=3000
# 请求失败后重试次数（默认为 0 次不重试）
forest.max-retry-count=0
# 单向验证的HTTPS的默认TLS协议（默认为 TLS）
forest.ssl-protocol=TLS
# 打开或关闭日志（默认为 true）
forest.log-enabled=true
# 打开/关闭Forest请求日志（默认为 true）
forest.log-request=true
# 打开/关闭Forest响应状态日志（默认为 true）
forest.log-response-status=true
# 打开/关闭Forest响应内容日志（默认为 false）
forest.log-response-content=true
# [自v1.5.27版本起可用] 异步模式（默认为 platform）
forest.async-mode=platform
```

**全局变量定义**：

Forest 可以在`forest.variables`属性下自定义全局变量。

其中 key 为变量名，value 为变量值。

全局变量可以在任何模板表达式中进行数据绑定。

> app.yaml

```yaml
forest:
  variables:
    username: foo      # 声明全局变量，变量名: username，变量值: foo
    userpwd: bar       # 声明全局变量，变量名: userpwd，变量值: bar
```

> app.properties

```properties
# 声明全局变量，变量名: username，变量值: foo
forest.variables.username=foo
# 声明全局变量，变量名: userpwd，变量值: bar
forest.variables.userpwd=bar
```

**配置 Bean Name**：

Forest 允许您在 yaml 文件中配置 Bean Id，它对应着`ForestConfiguration`对象在 Solon 上下文中的 Bean 名称。

> app.yaml

```yaml
forest:
  bean-id: config0 # 在Solon上下文中bean的id，默认值为forestConfiguration
```

> app.properties

```properties
# 在Solon上下文中bean的id，默认值为forestConfiguration
forest.bean-id: config0
```

然后便可以在 Solon 中通过 Bean 的名称引用到它

```java
@Jnject("config0")
private ForestConfiguration config0;
```

## 原生Java环境配置

**创建 ForestConfiguration 对象**：

`ForestConfiguration`为 Forest 的全局配置对象类，所有的 Forest 的全局基本配置信息由此类进行管理。

`ForestConfiguration`对象的创建方式：调用静态方法`Forest.config()`，此方法会创建/获取全局唯一的 ForestConfiguration 对象并初始化默认值。

```java
ForestConfiguration configuration = Forest.config();
```

**配置后端 HTTP API**：

目前 Forest 支持`okhttp3`和`httpclient`两种后端 HTTP API，若不配置该属性，默认为`okhttp3`。

```java
//configuration.setBackendName("okhttp3"); //配置okhttp3
configuration.setBackendName("httpclient"); //配置httpclient
```

**全局基本配置**：

**注意**：setRetryCount只是简单机械的请求失败后的重试次数，所以一般建议设置为**0**。如果一定要多次重试，请一定要在保证服务端的**幂等性**的基础上进行重试，否则容易引发生产事故！

```java
// 连接池最大连接数，默认值为500
configuration.setMaxConnections(123);
// 每个路由的最大连接数，默认值为500
configuration.setMaxRouteConnections(222);
// [自v1.5.22版本起可用] 最大请求等待队列大小
configuration.setMaxRequestQueueSize(100);
// [自v1.5.21版本起可用] 最大异步线程数
configuration.setMaxAsyncThreadSize(300);
// [自v1.5.22版本起可用] 最大异步线程池队列大小
configuration.setMaxAsyncQueueSize(16);
// 请求超时时间，单位为毫秒, 默认值为3000
configuration.setTimeout(3000);
// 连接超时时间，单位为毫秒, 默认值为2000
configuration.setConnectTimeout(2000);
// 设置重试器
configuration.setRetryer(BackOffRetryer.class);
// 请求失败后重试次数，默认为0次不重试
configuration.setMaxRetryCount(0);
// 单向验证的HTTPS的默认SSL协议，默认为SSLv3
configuration.setSslProtocol(SSLUtils.SSLv3);
// 打开或关闭日志，默认为true
configuration.setLogEnabled(true);
// [自v1.5.27版本起可用] 异步模式（默认为 platform）
configuration.setAsyncMode(ForestAsyncMode.PLATFORM);
```

**全局变量定义**：

Forest 可以通过`ForestConfiguration`对象的`setVariableValue`方法自定义全局变量。

其中第一个参数为变量名，第二个为变量值。

全局变量可以在任何模板表达式中进行数据绑定。

```java
ForestConfiguration configuration = ForestConfiguration.configuration();
...
configuration.setVariableValue("username", "foo");
configuration.setVariableValue("userpwd", "bar");
```

## 配置优先级/作用域

`application.yml` / `application.properties`配置以及通过`ForestConfiguration`对象设置的配置都是全局配置。

除了全局配置，Forest 还提供了接口配置和请求配置。

这三种配置的作用域和读取优先级各不相同：

**作用域**： 配置作用域指的是配置所影响的请求范围。

**优先级**： 优先级值的是是否优先读取该配置。比如您优先级最高`@Request`中定义了`timeout`为`500`，那么即便在全局配置中定了`timeout`为`1000`，最终该请求实际的`timeout`为优先级配置最高的`@Request`中定义的`500`。

**配置层级**：

![](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/202307031615720.png)

**Forest 的配置层级介绍**：

1. **全局配置**：针对全局所有请求，作用域最大，配置读取的优先级最小。
2. **接口配置**： 作用域为某一个`interface`中定义的请求，读取的优先级最小。您可以通过在`interface`上修饰`@BaseRequest`注解进行配置。
3. **请求配置**： 作用域为某一个具体的请求，读取的优先级最高。您可以在接口的方法上修饰`@Request`注解进行 HTTP 信息配置的定义。