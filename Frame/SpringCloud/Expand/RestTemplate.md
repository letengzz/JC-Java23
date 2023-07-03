# RestTemplate

RestTemplate是Spring Resources中一个访问第三方RESTful API接口的网络请求框架。RestTemplate的设计原则和其他的Spring Template（例如JdbcTemplate）类似，都是为了执行复杂任务提供了一个具有默认行为的简单方法。

RestTemplate是用来消费REST服务的，所以RestTemplate的主要方法都与REST的HTTP协议的一些方法紧密相连，例如HEAD、GET、POST、PUT、DELETE、OPTIONS等方法，这些方法在RestTemplate类对应的方法为`headForHeaders()`，`getForObject()`、`postForObject()`、`put()`、`delete()`等。

举例说明，在订单服务通过RestTemplate的getForObject方法调用支付服务，并且将调用结果反序列化成Payment对象：

```java
    @GetMapping("/payment/{id}")
    public ResponseEntity<Payment> getPaymentById(@PathVariable("id") Integer id) {
        String url = "http://localhost:9001/payment/" + id;

        List<ServiceInstance> serviceInstances = discoveryClient.getInstances("cloud-payment-service");
        ServiceInstance serviceInstance = serviceInstances.get(0);
        url = "http://" + serviceInstance.getHost() + ":" + serviceInstance.getPort() + "/payment/" + id;


        Payment payment = restTemplate.getForObject(url, Payment.class);
        return ResponseEntity.ok(payment);
    }
```

RestTemplate支持常见的Http协议请求方法，例如post, get, delete等，所以用RestTemplate很容易构建RESTful API。上述案例结果返回json对象，使用jackson框架完成。

下面RestTemplate一个综合的应用案例，在案例中用户服务通过RestTemplate发送HTTP请求给授权中心申请令牌，过程中需要封装请求参数，请求头，然后对相应结果Oauth2AccessToken进行序列化：

```java
@RequestMapping("/login")
public ResponseEntity<OAuth2AccessToken> login(String username, String password) {
    // 1:验证用户
    User user = service.getUserByUserName(username);
    if (null == user) {
        return new ResponseEntity<>(HttpStatus.UNAUTHORIZED);
    }
    if (!BPwdEncoderUtil.matches(password, user.getPassword())) {
        return new ResponseEntity<>(HttpStatus.UNAUTHORIZED);
    }
    // 2:使用restTemplate发送请求到授权服务器，申请令牌
    // 请求头 "basic auth"
    String client_secret = oAuth2ClientProperties.getClientId() + ":" + oAuth2ClientProperties.getClientSecret();
    client_secret = "Basic " + Base64.getEncoder().encodeToString(client_secret.getBytes());
    HttpHeaders headers = new HttpHeaders();
    headers.set("Authorization", client_secret);

    //请求参数
    MultiValueMap<String, String> map = new LinkedMultiValueMap<>();
    map.put("username", Collections.singletonList(username));
    map.put("password", Collections.singletonList(password));
    map.put("grant_type", Collections.singletonList(oAuth2ProtectedResourceDetails.getGrantType()));
    map.put("scope",oAuth2ProtectedResourceDetails.getScope());

    // HttpEntity(请求参数头)
    HttpEntity httpEntity = new HttpEntity(map, headers);

    return restTemplate.exchange(oAuth2ProtectedResourceDetails.getAccessTokenUri(), HttpMethod.POST, httpEntity, OAuth2AccessToken.class);

}
```

> PaymentController.java

```java
    @RequestMapping("/http")
    public String testHttp(
            @RequestParam("username") String username,
            @RequestParam("password")  String password,
            @RequestHeader("Authorization") String token) {
        log.info("username:{}, password:{}, Authorization: {}", username, password, token);
        return username + "\t" + password + "\t" + token;
    }
```

> OrderController.java

```java
    @RequestMapping("/http")
    public ResponseEntity<String> testHttp() {

        HttpHeaders headers = new HttpHeaders();
        headers.set("Authorization", "my-token-123456");

        //请求参数
        MultiValueMap<String, String> map = new LinkedMultiValueMap<>();
        map.put("username", Collections.singletonList("zhangsan"));
        map.put("password", Collections.singletonList("654321"));

        HttpEntity httpEntity = new HttpEntity(map, headers);

        ResponseEntity<String> result = restTemplate.exchange("http://localhost:9001/payment/http", HttpMethod.POST, httpEntity, String.class);

        return result;
    }
```

