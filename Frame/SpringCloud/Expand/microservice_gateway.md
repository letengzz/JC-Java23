# 微服务网关

各种微服务通过REST API接口提供给外部应用调用，外部客户端直接调用各个微服务是没有问题的。但出于种种原因，这并不是一个好的选择。让客户端直接与各个微服务通讯，会存在以下几个问题：

- 客户端会多次请求不同的微服务，增加了客户端的复杂性。
- 存在跨域请求，在一定场景下处理会变得相对比较复杂。
- 实现认证复杂，每个微服务都需要独立认证。
- 难以重构，项目迭代可能导致微服务重新划分。如果客户端直接与微服务通讯，那么重构将会很难实施。
- 如果某些微服务使用了防火墙、浏览器不友好的协议，直接访问会有一定困难。

在微服务系统中微服务资源一般不直接暴露给我外部客户端访问，这样做的好处是将内部服务隐藏起来，从而解决上述问题。

网关有很多重要的意义，具体体现：

- 网关可以做一些身份认证、权限管理、防止非法请求操作服务等，对服务起一定保护作用。
- 网关将所有微服务统一管理，对外统一暴露，外界系统不需要知道微服务架构个服务相互调用的复杂性，同时也避免了内部服务一些敏感信息泄露问题。
- 易于监控。可在微服务网关收集监控数据并将其推送到外部系统进行分析。
- 客户端只跟服务网关打交道，减少了客户端与各个微服务之间的交互次数。
- 多渠道支持，可以根据不同客户端（WEB端、移动端、桌面端...）提供不同的API服务网关。
- 网关可以用来做流量监控。在高并发下，对服务限流、降级。
- 网关把服务从内部分离出来，方便测试。

微服务网关能够实现，路由、负载均衡等多种功能。类似Nginx，反向代理的功能。在微服务架构中，后端服务往往不直接开放给调用端，而是通过一个API网关根据请求的URL，路由到相应的服务。当添加API网关后，在第三方调用端和服务提供方之间就创建了一面墙，在API网关中进行权限控制，同时API网关将请求以负载均衡的方式发送给后端服务。

**微服务网关架构**：

![image.png](https://cdn.nlark.com/yuque/0/2021/png/12830737/1627378943430-afa9c253-892e-4b7c-953c-4e0851025f87.png?x-oss-process=image)