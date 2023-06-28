# Nacos 更加全能的注册中心

Nacos(**Na**ming **Co**nfiguration **S**ervice)是一款阿里巴巴开源的服务注册与发现、配置管理的组件，相当于是[Eureka](../Netflix/Eureka.md)+[Config](../Config.md)的组合形态。

## 安装与部署

Nacos服务器是独立安装部署的，因此我们需要下载最新的Nacos服务端程序。

**下载地址**：https://github.com/alibaba/nacos

![image-20230316185713790](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301649736.png)

**注意版本问题**：https://github.com/alibaba/spring-cloud-alibaba/wiki/%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E

下载1.4.1版本并将文件进行解压：

![image-20230316190310059](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301649268.png)

直接将其拖入到项目文件夹下，便于在IDEA内部启动，接着添加运行配置：

![image-20230316191505185](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301649622.png)

其中`-m standalone`表示单节点模式，Mac和Linux下记得将解释器设定为`/bin/bash`，由于Nacos在Mac/Linux默认是后台启动模式，我们修改一下它的bash文件，让它变成前台启动，这样IDEA关闭了Nacos就自动关闭了，否则开发环境下很容易忘记关：

```bash
# 注释掉 nohup $JAVA ${JAVA_OPT} nacos.nacos >> ${BASE_DIR}/logs/start.out 2>&1 &
# 替换成下面的
$JAVA ${JAVA_OPT} nacos.nacos
```

接着我们点击启动：

![image-20230316191531850](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301650788.png)

**注意**：如果想在命令行中启动，只需要进入nacos/bin目录，输入命令即可启动：

```bash
startup.cmd -m standalone
```

启动成功，访问这个地址：http://localhost:8848/nacos/index.html

![image-20230316191644571](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301650931.png)

默认的用户名和管理员密码都是`nacos`，直接登陆即可，可以看到进入管理页面：

![image-20230316191724638](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301650564.png)

至此，Nacos的安装与部署完成。

## 服务注册与发现

实现基于Nacos的服务注册与发现，需要导入SpringCloudAlibaba相关的依赖。

在父工程将依赖进行管理：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.2.0</version>
        </dependency>
      
      	<!-- 引入SpringCloud依赖 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>2020.0.1</version>
          	<type>pom</type>
            <scope>import</scope>
        </dependency>

     	  <!-- 引入SpringCloudAlibaba依赖 -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2021.1</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

接着在子项目中添加服务发现依赖了，以图书服务为例：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

和注册到Eureka一样，需要在配置文件中配置Nacos注册中心的地址：

```yaml
server:
	# 之后所有的图书服务节点就81XX端口
  port: 8101
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/springcloud?useSSL=false&useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai
    username: root
    password: 123123
  # 应用名称 bookservice
  application:
    name: bookservice
  cloud:
    nacos:
      discovery:
        # 配置Nacos注册中心地址
        server-addr: localhost:8848
```

接着启动图书服务，可以在Nacos的服务列表中找到：

![image-20230321201102673](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301652292.png)

按照同样的方法，接着将另外两个服务也注册到Nacos中：

![image-20230321203348753](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301652127.png)

接着使用OpenFeign，实现服务发现远程调用以及负载均衡，导入依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
<!-- 这里需要单独导入LoadBalancer依赖 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```

编写接口：

```java
@FeignClient("userservice")
public interface UserClient {
    
    @RequestMapping("/user/{uid}")
    User getUserById(@PathVariable("uid") int uid);
}
```

```java
@FeignClient("bookservice")
public interface BookClient {

    @RequestMapping("/book/{bid}")
    Book getBookById(@PathVariable("bid") int bid);
}
```

```java
@Service
public class BorrowServiceImpl implements BorrowService{

    @Resource
    BorrowMapper mapper;

    @Resource
    UserClient userClient;

    @Resource
    BookClient bookClient;

    @Override
    public UserBorrowDetail getUserBorrowDetailByUid(int uid) {
        List<Borrow> borrow = mapper.getBorrowsByUid(uid);
        User user = userClient.getUserById(uid);
        List<Book> bookList = borrow
                .stream()
                .map(b -> bookClient.getBookById(b.getBid()))
                .collect(Collectors.toList());
        return new UserBorrowDetail(user, bookList);
    }
}
```

```java
@EnableFeignClients
@SpringBootApplication
public class BorrowApplication {
    public static void main(String[] args) {
        SpringApplication.run(BorrowApplication.class, args);
    }
}
```

测试：

![image-20230321203411697](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301652462.png)

测试正常，可以自动发现服务，接着多配置几个实例，图书服务和用户服务的端口配置：

![image-20230321204338767](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301653743.png)

> BorrowController.java
>

```java
@RestController
public class BorrowController {
    @Resource
    BorrowService service;

    @Resource
    private LoadBalancerClient loadBalancerClient;
    

    @RequestMapping("/borrow/{uid}")
    public UserBorrowDetail findUserBorrows(@PathVariable("uid") Integer uid){
        return service.getUserBorrowDetailByUid(uid);
    }


    @GetMapping("/test-load-balancer")
    public String testLoadBalancer() {
        ServiceInstance instance = loadBalancerClient.choose("userservice");
        return instance.getHost() + ":" + instance.getPort();
    }
}
```

现在将全部服务启动：

![image-20230330165835594](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301658605.png)

可以看到Nacos中的实例数量已经显示为3：

![image-20230321205814610](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301653254.png)

浏览器访问http://localhost:8301/test-load-balancer，刷新发现浏览器轮流显示：

![image-20230321205723401](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301653104.png)

这样就实现了基于Nacos的服务的注册与发现，实际上大致流程与Eureka一致。

**注意**：Nacos区分了临时实例和非临时实例：

![image-20220326155010841](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301701860.png)

- **临时实例**：和Eureka一样，采用心跳机制向Nacos发送请求保持在线状态，一旦心跳停止，代表实例下线，不保留实例信息。
- **非临时实例**：由Nacos主动进行联系，如果连接失败，那么不会移除实例信息，而是将健康状态设定为false，相当于会对某个实例状态持续地进行监控。

可以通过配置文件进行修改临时实例：

```yaml
spring:
  application:
    name: borrowservice
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
        # 将ephemeral修改为false，表示非临时实例
        ephemeral: false
```

在Nacos中查看，可以发现实例已经不是临时的：

![image-20230321210336483](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301701057.png)

关闭此实例：

![image-20230321210414605](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301701216.png)

只是将健康状态变为false，而不会删除实例的信息。

## 集群分区

在一个分布式应用中，相同服务的实例可能会在不同的机器、位置上启动，比如我们的用户管理服务，可能在北京有1台服务器部署、上海有一台服务器部署，而这时，在北京的服务器上启动了借阅服务，如果借阅服务要调用用户服务，就应该优先选择同一个区域的用户服务进行调用，这样会使得响应速度更快。

![image-20220326150024118](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301702028.png)

因此，可以对部署在不同机房的服务进行分区，可以看到实例的分区是默认：

![image-20230321211323808](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301702799.png)

直接在配置文件中进行修改：

```yaml
spring:
  application:
    name: borrowservice
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
        # 修改为北京地区的集群
        cluster-name: Beijing
```

由于使用的是不同的启动配置，直接在启动配置中添加环境变量`spring.cloud.nacos.discovery.cluster-name`，将用户服务和图书服务两个区域都分配一个，借阅服务就配置为北京地区：

![image-20230330170316076](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301703634.png)

修改完成之后，重新启动服务（Nacos也要重启），观察Nacos中集群分布情况：

![image-20230321212629691](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301703735.png)

现在有三个集群，并且都有一个实例正在运行。接着去调用借阅服务，但是发现并没有按照区域进行优先调用，而依然使用的是轮询模式的负载均衡调用。

必须要提供Nacos的负载均衡实现才能开启区域优先调用机制，只需要在配制文件中进行修改即可：

```yaml
spring:
  application:
    name: borrowservice
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
        cluster-name: Beijing
    # 将loadbalancer的nacos支持开启，集成Nacos负载均衡
    loadbalancer:
      nacos:
        enabled: true
```

现在重启借阅服务，会发现优先调用的是同区域的用户和图书服务，现在可以将北京地区的服务下线：

![image-20230321213948924](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301704657.png)

可以看到，在下线之后，由于本区域内没有可用服务了，借阅服务将会调用上海区域的用户服务。

除了根据区域优先调用之外，同一个区域内的实例也可以单独设置权重，Nacos会优先选择权重更大的实例进行调用，我们可以直接在管理页面中进行配置：

![image-20230321214135160](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301704177.png)

或是在配置文件中进行配置：

```yml
spring:
  application:
    name: borrowservice
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
        cluster-name: Chengdu
        # 权重大小，越大越优先调用，默认为1
        weight: 0.5
```

通过配置权重，某些性能不太好的机器就能够更少地被使用，而更多的使用那些网络良好性能更高的主机上的实例。

## 配置中心

[Spring Cloud Config](../Config.md)在`bootstrap.yml`中配置远程配置文件获取，然后再进入到配置文件加载环节，而Nacos也支持这样的操作，使用方式与Config比较类似，比如想要将借阅服务的配置文件放到Nacos进行管理，那么这个时候就需要在Nacos中创建配置文件：

![image-20230324190930429](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301705971.png)

将借阅服务的配置文件全部复制过来注意**Data ID**的格式跟之前一样，`应用名称-环境.yml`，如果只编写应用名称，那么代表此配置文件无论在什么环境下都会使用，然后每个配置文件都可以进行分组，也算是一种分类方式：

![image-20230324191144386](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301705393.png)

完成之后点击发布即可：

![image-20220326162122134](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301705265.png)

在项目中导入依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

在借阅服务中添加`bootstrap.yml`文件：

```yaml
spring:
  application:
  	# 服务名称和配置文件保持一致
    name: borrowservice
  profiles:
  	# 环境也是和配置文件保持一致
    active: dev
  cloud:
    nacos:
      config:
      	# 配置文件后缀名
        file-extension: yml
        # 配置中心服务器地址，也就是Nacos地址
        server-addr: localhost:8848
```

启动服务：

![image-20230330170752677](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301707107.png)

可以看到成功读取配置文件并启动了，实际上使用上来说跟Config是基本一致的。

Nacos还支持配置文件的热更新，比如在配置文件中添加了一个属性，而这个时候可能需要实时修改，并在后端实时更新：

创建一个新的Controller：

```java
@RestController
@RefreshScope   //添加此注解实现自动刷新
public class TestController {
    
    @Value("${test.txt}")  //我们从配置文件中读取test.txt的字符串值，作为test接口的返回值
    String txt;
    
    @RequestMapping("/test")
    public String test(){
        return txt;
    }
}
```

修改配置文件，然后重启服务器：

![image-20230324192733766](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301708021.png)

显示内容：

![image-20230324192811371](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301708849.png)

将配置文件的值进行修改：

![image-20230324192836776](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301708585.png)

再次访问接口：

![image-20230324193012675](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301708425.png)

## 数据持久化

当使用默认配置启动Nacos时(单节点)，所有配置文件都被Nacos保存在自带的一个嵌入式数据库中。

> 在0.7版本之前，在单机模式时nacos使用嵌入式数据库实现数据的存储，不方便观察数据存储的基本情况。0.7版本增加了支持mysql数据源能力。

![image-20230330173535247](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/202306281454201.png)

如果使用内嵌数据库，注定会有存储上限，所以要将Nacos中的数据实现持久化。

Nacos通过集中式存储来保证数据的持久化，同时也为Nacos集群部署奠定了基础

Nacos采用了单一数据源，直接解决了分布式和集群部署中的一致性问题。

**操作步骤**：

1. 创建数据库，用于存储数据。

2. 直接导入到数据库即可，文件在conf目录中：

   ![image-20230330173934813](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301739626.png)

   将其导入到数据库，可以看到生成了很多的表：

   ![image-20230628150410241](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/202306281504358.png)

3. 修改配置文件，Nacos-server其实就是一个Java工程或者说是一个Springboot项目，他的配置文件在`nacos\conf`目录下，名为 `application.properties`，在文件底部添加数据源配置：

   ```
   spring.datasource.platform=mysql
   
   db.num=1
   db.url.0=jdbc:mysql://127.0.0.1:3306/mynacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
   db.user=root
   db.password=123456
   ```

4. 验证是否持久化到数据库中：启动Nacos，进入Nacos控制台，此时的Nacos控制台中焕然一新，之前的数据都不见了

   > 因为加入了新的数据源，Nacos从mysql中读取所有的配置文件，而刚刚初始化的数据库是干干净净的，自然不会有什么数据和信息显示。

   在公共空间(public)中新建一个配置文件DataID: `nacos-config.yml`, 配置内容如下：

   ```yaml
   server: 
       port: 9989
   nacos:
       config: 配置文件已持久化到数据库中...
   ```

   ![image-20230628151454965](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/202306281514070.png)

   观察数据库`mynacos`中的数据库表 `config_info` ：

   ![image-20230628151542840](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/202306281515927.png)

## 多环境配置和服务

在实际开发中，通常一个系统会准备开发环境、测试环境、预发环境、正式环境。

需要保证指定环境启动时服务能正确读取到Nacos上相应环境的配置文件。

### Data ID方案

Data ID它的定义规则是：`${prefix}-${spring.profile.active}.${file-extension}`(`应用名称-环境.数据格式`)

- prefix：默认为 spring.application.name 的值，也可以通过配置项 `spring.cloud.nacos.config.prefix` 来配置。
- spring.profile.active：即为当前环境对应的 profile，可以通过配置项 `spring.profile.active` 来配置。
- file-exetension：为配置内容的数据格式，可以通过配置项 `spring.cloud.nacos.config.file-extension` 来配置。目前只支持 properties 和 yaml 类型。

注意：当 `spring.profile.active` 为空时，对应的连接符 `-` 也将不存在，Data ID 的拼接格式变成 `prefix.file-extension`，代表此配置文件无论在什么环境下都会使用。

### Group方案



### Namespace方案

可以将配置文件或是服务实例划分到不同的命名空间中，其实就是区分开发、生产环境或是引用归属之类的：

![image-20230324193205881](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301709165.png)

创建一个新的命名空间：

![image-20230324193150966](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301709558.png)

可以看到在dev命名空间下，没有任何配置文件和服务：

![image-20230324193324963](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301730758.png)

在不同的命名空间下，实例和配置都是相互之间隔离的。

可以在配置文件中指定当前的命名空间：

```yaml
spring:
  application:
    name: borrowservice
  profiles:
    active: dev   
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
        # 配置命名空间 namespace: 命名空间ID
        namespace: c8428b6d-744d-482f-8042-649b5045cef7
        cluster-name: Chengdu
        weight: 0.5
      config:
        file-extension: yml
        server-addr: localhost:8848
        # 配置命名空间 namespace: 命名空间ID
        namespace: c8428b6d-744d-482f-8042-649b5045cef7
```

## 多环境下管理及隔离配置和服务



## 实现高可用

搭建Nacos集群，实现高可用。

官方方案：https://nacos.io/zh-cn/docs/cluster-mode-quick-start.html

![deployDnsVipMode.jpg](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301730370.png)

> http://ip1:port/openAPI 直连ip模式，机器挂则需要修改ip才可以使用。
>
> http://SLB:port/openAPI 挂载SLB模式(内网SLB，不可暴露到公网，以免带来安全风险)，直连SLB即可，下面挂server真实ip，可读性不好。
>
> http://nacos.com:port/openAPI 域名 + SLB模式(内网SLB，不可暴露到公网，以免带来安全风险)，可读性好，而且换ip方便，推荐模式

Nacos推荐在所有的服务端之前建立一个负载均衡，通过访问负载均衡服务器来间接访问到各个Nacos服务器。实际上就是比如有三个Nacos服务器做集群，但是每个服务不可能把每个Nacos都去访问一次进行注册，实际上只需要在任意一台Nacos服务器上注册即可，Nacos服务器之间会自动同步信息，但是如果我们随便指定一台Nacos服务器进行注册，如果这台Nacos服务器挂了，但是其他Nacos服务器没挂，这样就没办法完成注册了，但是实际上整个集群还是可用的状态。

所以就需要在所有Nacos服务器之前搭建一个SLB(服务器负载均衡)，这样就可以避免上面的问题了。但是如果要实现外界对服务访问的负载均衡，就得用比如Gateway来实现，而这里实际上可以用一个更加方便的工具：Nginx来实现。

SLB最上方还有一个DNS是因为SLB是裸IP，如果SLB服务器修改了地址，那么所有微服务注册的地址也得改，所以这里是通过加域名，通过域名来访问，让DNS去解析真实IP，这样就算改变IP，只需要修改域名解析记录即可，域名地址是不会变化的。

在多节点集群模式下，数据肯定是不能各存各的，所以，Nacos提供了MySQL统一存储支持，只需要让所有的Nacos服务器连接MySQL进行数据存储即可，官方也提供好了SQL文件。

步骤：[实现持久化](#实现持久化)

创建两个Nacos服务器，做一个迷你的集群，这里使用`scp`命令将nacos服务端上传到Linux服务器（注意需要提前安装好JRE 8或更高版本的环境）：

![image-20220327115901662](https://img-blog.csdnimg.cn/img_convert/f4061d1124e410c38fad64f4937ce7ef.png)

解压之后，我们对其配置文件进行修改，首先是`application.properties`配置文件，修改以下内容，包括MySQL服务器的信息：

```properties
### Default web server port:
server.port=8801

#*************** Config Module Related Configurations ***************#
### If use MySQL as datasource:
spring.datasource.platform=mysql

### Count of DB:
db.num=1

### Connect URL of DB:
db.url.0=jdbc:mysql://cloudstudy.mysql.cn-chengdu.rds.aliyuncs.com:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user.0=nacos
db.password.0=nacos
```

然后修改集群配置，这里需要重命名一下：

![image-20220327120219022](https://img-blog.csdnimg.cn/img_convert/b31bb10a555eca191d4264450ab85e69.png)

端口记得使用内网IP地址：

![image-20220327142541523](https://img-blog.csdnimg.cn/img_convert/43cbdb2caa044c2ee963eb1119f21ad7.png)

最后我们修改一下Nacos的内存分配以及前台启动，直接修改`startup.sh`文件（内存有限，玩不起高的）：

![image-20220327125049013](https://img-blog.csdnimg.cn/img_convert/a1716649e8f4558a620e9b7581ca1586.png)

保存之后，将nacos复制一份，并将端口修改为8802，接着启动这两个Nacos服务器。

![image-20220327125201913](https://img-blog.csdnimg.cn/img_convert/ee39bbdbda4edeec3fa22ba0f077d7c5.png)

然后我们打开管理面板，可以看到两个节点都已经启动了：

![image-20220327125232238](https://img-blog.csdnimg.cn/img_convert/1bed88bdb27adf85aed9b7fac7fad0c7.png)

这样，我们第二步就完成了，接着我们需要添加一个SLB，这里我们用Nginx做反向代理：

> *Nginx* (engine x) 是一个高性能的[HTTP](https://baike.baidu.com/item/HTTP)和[反向代理](https://baike.baidu.com/item/反向代理/7793488)web服务器，同时也提供了IMAP/POP3/SMTP服务。它相当于在内网与外网之间形成一个网关，所有的请求都可以由Nginx服务器转交给内网的其他服务器。

这里我们直接安装：

```sh
sudo apt install nginx
```

可以看到直接请求80端口之后得到，表示安装成功：

![image-20220327130009391](https://img-blog.csdnimg.cn/img_convert/c0b2b7d7f3b244ceee97566f84f1d4d4.png)

现在我们需要让其代理我们刚刚启动的两个Nacos服务器，我们需要对其进行一些配置。配置文件位于`/etc/nginx/nginx.conf`，添加以下内容：

```conf
#添加我们在上游刚刚创建好的两个nacos服务器
upstream nacos-server {
        server 10.0.0.12:8801;
        server 10.0.0.12:8802;
}

server {
        listen   80;
        server_name  1.14.121.107;

        location /nacos {
                proxy_pass http://nacos-server;
        }
}
```

重启Nginx服务器，成功连接：

![image-20220327144441878](https://img-blog.csdnimg.cn/img_convert/3300584093bd95abc5c019dfea3401a8.png)

然后将所有的服务全部修改为云服务器上Nacos的地址，启动试试看。

![image-20220327145216771](https://img-blog.csdnimg.cn/img_convert/1d05106f2f09c69603ca94fe969bf7aa.png)

这样就搭建好了Nacos集群。