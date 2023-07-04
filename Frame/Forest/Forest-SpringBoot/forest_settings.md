# Forest 配置

Forest 遵循约定大于配置的理念，大多数情况下不需要进行配置，或填写非常简单的配置即可。

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

# 配置优先级/作用域

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