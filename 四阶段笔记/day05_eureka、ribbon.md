### 搭建Eureka Client（高 20）


在父项目上再创建一个provider子模块，导入**<font color='red'>Eureka Discovery Client、Spring Web</font>**依赖

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210708103648.png" style="zoom:33%;" />

在provider的pom中配置父项目

```xml
<parent>
   <artifactId>springcloud-teach</artifactId>
   <groupId>com.woniuxy</groupId>
   <version>1.0</version>
</parent>
    
<modelVersion>4.0.0</modelVersion>
```



在父项目的pom.xml中配置子模块

```xml
<modules>
   <module>eureka-server</module>
   <module>provider</module>
</modules>
```



在provider子项目的主配置文件中配置以下信息

```yaml
server:
  port: 8080
eureka:
  client: #客户端注册到eureka列表中
    service-url:
      defaultZone: http://127.0.0.1:9001/eureka
  instance:
    instance-id: provider-8080  #注册中心显示出来的微服务名称
    prefer-ip-address: true #显示访问url
spring:
  application:
    name: provider

```

**<font color='red'>注意：service-url使用IP地址，最好不要使用localhost</font>**



在provider的主启动类上开启eureka-client功能**<font color='red'>@EnableEurekaClient</font>**

```java
@SpringBootApplication
@EnableEurekaClient
public class ProviderApplication {
    public static void main(String[] args) {
        SpringApplication.run(ProviderApplication.class, args);
    }
}
```



启动provider模块，然后到eureka控制台查看注册情况

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210708104232.png" alt="image-20210708104232021" style="zoom: 25%;" />



在provider的pom文件中添加监控依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

在其主配置文件中配置微服务信息

```yaml
info:
  app.name: SpringCloud
  company.name: woniuxy
  build.artifactId: $project.artifactId$
  build.version: $project.version$
```



重启provider子模块，然后点击eureka控制台子模块连接，查看子模块信息

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210708104633.png" alt="image-20210708104632817" style="zoom:25%;" />



**<font color='red'>同样的方式再创建一个消费者consumer子模块 端口号80</font>**



#### 常见概念

##### Register	服务注册

当Eureka客户端向Eureka Server注册时，它提供自身的元数据，比如IP地址、端口，运行状况指示符URL，主页等。

 

##### Renew	服务续约（心跳机制）

Eureka客户会每隔30秒发送一次心跳来续约。通过续约来告知Eureka Server该Eureka客户仍然存在，没有出现问题。正常情况下，如果Eureka Server在90秒没有收到Eureka客户的续约，它会将实例从其注册表中删除。建议不要更改续约间隔。

- 心跳机制是每隔30秒发送一个自定义的结构体(心跳包)，让对方知道自己还活着，以确保连接的有效性的机制。

- 心跳机制是每隔30秒发送一个固定信息给服务端，服务端收到后回复一个固定的信息。如果服务端90秒内没有收到客户端消息则视客户端断开。

- 发送方可以是客户端或服务端，根据实际情况，一般是客户端；因为一个服务端可能有很多客户端，服务端作为发送方的比较耗费性能。

 

在provider微服务的application.yml配置心跳时间和下线时间

```yaml
server:
  port: 8080
eureka:
  client: #客户端注册到eureka列表中
    service-url:
      defaultZone: http://127.0.0.1:9001/eureka
  instance:
    instance-id: provider-8080  #注册中心显示出来的微服务名称
    prefer-ip-address: true #显示访问url
    lease-renewal-interval-in-seconds: 20   #心跳时间
    lease-expiration-duration-in-seconds: 60 #下线时间
```



心跳信息可以从eureka微服务的控制台看到

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210715151123.png" alt="image-20210715151122927" style="zoom: 33%;" />

要看到信息，可以在eureka微服务的application.yml中设置日志级别为debug

```yaml
logging:
  level:
    root: debug
```

也可以在provider的application.yml设置日志级别，观察provider发送服务续约请求信息

```yaml
logging:
  level:
    root: debug
```



##### Fetch Registries	获取注册列表信息

Eureka客户端从服务器获取注册表信息，并将其缓存在本地。客户端会使用该信息查找其他服务，从而进行远程调用。该注册列表信息定期（每30秒钟）更新一次。每次返回注册列表信息可能与Eureka客户端的缓存信息不同， Eureka客户端自动处理。如果由于某种原因导致注册列表信息不能及时匹配，Eureka客户端则会重新获取整个注册表信息。 Eureka服务器缓存注册列表信息，整个注册表以及每个应用程序的信息进行了压缩，压缩内容和没有压缩的内容完全相同。Eureka客户端和Eureka 服务器可以使用JSON / XML格式进行通讯。在默认的情况下Eureka客户端使用压缩JSON格式来获取注册列表的信息。

 

##### Cancel		服务下线

Eureka客户端在程序关闭时向Eureka服务器发送取消请求，发送请求后，该客户端实例信息将从服务器的实例注册表中删除，该下线请求不会自动完成，它需要调用以下内容：

DiscoveryManager.getInstance().shutdownComponent()；

 

##### Eviction  服务剔除

在默认的情况下，当Eureka客户端连续90秒没有向Eureka服务器发送服务续约（心跳），Eureka服务器会将该服务实例从服务注册列表删除，即服务剔除。



### RestTemplate使用（中 30）

RestTmeplate使用GET、POST、PUT、DELETE请求方法

#### 实体类

 在provider子模块中添加Goods实体类及GoodsController接口

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Accessors(chain = true)
public class Goods {
    private int id;
    private String name;
}
```

#### controller

```java
@RestController
public class GoodsController {
    @RequestMapping("/all")
    public List<Goods> all(){
        return Arrays.asList(
                new Goods(1001,"手机"),
                new Goods(1002,"电脑")
        );
    }
}
```

#### 实体类

在consumer子模块中添加goods实体类、RestTemplate配置类及controller

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Accessors(chain = true)
public class Goods {
    private int id;
    private String name;
}
```

##### 配置类

```java
@Configuration
public class WebConfiguration {
    @Bean
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```

##### controller

```java
@RestController
public class ConsumerController {
    @Resource
    private RestTemplate restTemplate;

    @RequestMapping("/all")
    public List<Goods> all(){
        String url = "http://127.0.0.1:8080/all";
        //
        List<Goods> goods = restTemplate.getForObject(url,List.class);
        //
        return goods;
    }
}
```

先后启动provider、consumer两个子模块，一般情况下先启动被调用方


##### 测试

在浏览器中访问consumer中的接口

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210714172935.png" alt="image-20210714172934889" style="zoom: 25%;" />

得到数据说明OK



#### GET请求（restful）

##### 不带参数

```java
@GetMapping("/all")
public List<Goods> all(){
    String url = "http://127.0.0.1:8080/all";
    //
    List<Goods> goods = restTemplate.getForObject(url,List.class);
    //
    return goods;
}
```

##### 带参数（<font color='red'>restful风格</font>）

在provider模块controller添加以下接口

```java
@GetMapping("find/{id}")
public Goods findById(@PathVariable("id") int id){
    System.out.println(id);
    return new Goods(1003,"平板");
}
```

在consumer 的controller中添加接口

```java
@RequestMapping("/find")
public Goods find(){
    String url = "http://127.0.0.1:8080/find/1003";
    Goods goods = restTemplate.getForObject(url,Goods.class);
    return goods;
}
```



#### POST请求（restful）

##### 在provider的controller中添加接口，注意接收参数时需要加**<font color="red">@RequestBody</font>**注解

```java
@PostMapping("/add")
    public ResponseResult<Boolean> add(@RequestBody Goods goods){
        System.out.println(goods);
        //
        return new ResponseResult<>(200,"success",true);
    }
```

##### 在consumer的controller中添加接口

```java
@RequestMapping("/add")
    public ResponseResult<Boolean> add(){
        String url = "http://127.0.0.1:8080/add";
        //
        Goods goods = new Goods(1004,"耳机");
        //
        ResponseResult<Boolean> result = 
            restTemplate.postForObject(url,goods,ResponseResult.class);
        //
        return result;
    }
```



#### PUT请求（restful）

**<font color='red'>注意PUT请求没有返回值</font>**

##### 在provider的controller中添加接口

```java
@PutMapping("/update")
public void update(@RequestBody Goods goods){
    System.out.println(goods);
}
```

##### 在consumer的controller中添加接口

```java
@RequestMapping("/update")
public void update(){
    String url = "http://127.0.0.1:8080/update";
    //
    Goods goods = new Goods(1005,"手表");

    //PUT请求没有返回值
    restTemplate.put(url,goods);
}
```



#### DELETE请求（restful）

**<font color='red'>注意DELETE请求也没有返回值</font>**

##### 在provider的controller中添加接口

```java
@DeleteMapping("/del/{id}")
public void del(@PathVariable("id") int id){
    System.out.println(id);
}
```

##### 在consumer的controller中添加接口

```java
@RequestMapping("/del")
public void del(){
    String url = "http://PROVIDER/del/1006";
    restTemplate.delete(url);
}
```



#### provider集群

再开启一个provider服务器

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210714172353.png" alt="image-20210714172353486" style="zoom: 50%;" />

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210714172449.png" alt="image-20210714172449021" style="zoom:50%;" />



<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210714174822.png" alt="image-20210714174822010" style="zoom:33%;" />

-Dserver.port=8081 -Deureka.instance.instance-id=provider-8081



#### 负载均衡器

在consumer的WebConfiguration配置类中，在创建RestTemplate对象的方法上添加 **<font color='red'>@LoadBalanced</font>**注解，开启负载均衡器

```java
@Configuration
public class WebConfiguration {
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```

将consumer接口方法中url的ip地址改为provider子模块的名字

```java
String url = "http://PROVIDER/all";
String url = "http://PROVIDER/find/1003";
String url = "http://PROVIDER/add";
String url = "http://PROVIDER/update";
```


##### controller

在provider的controller上添加以下代码获取到当前项目端口号，并在all方法中打印

**<font color='red'>@Value("${server.port}")</font>**

```java
@RestController
public class GoodsController {
    @Value("${server.port}")
    private String port;
    @GetMapping("/all")
    public List<Goods> all(){
        System.out.println(port);
        return Arrays.asList(
                new Goods(1001,"手机"),
                new Goods(1002,"电脑")
        );
    }
}
```



### Ribbon配置（中 30）

Ribbon的常用配置、全局配置、指定服务进行配置

 <img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210708121320.jpg" alt="img" style="zoom:33%;" />

Ribbon是一个基于HTTP和TCP客户端的负载均衡器，默认使用轮询的方式

SpringCloud Ribbon是基于Netfix Ribbon实现的一套客户端负载均衡工具



**导入了eureka就导入了ribbon，不需要单独再导入**

#### 开启ribbon

在application.yml文件中开启ribbon

```yaml
ribbon:
  eureka:
    enabled: true #开启ribbon，并从eureka中获取其它微服务信息
```


#### 全局配置

修改consumer的配置类WebConfiguration，全局配置ribbon

```java
@Configuration
public class WebConfiguration {
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate(){
        //全局设置超时时间
        SimpleClientHttpRequestFactory factory = new SimpleClientHttpRequestFactory();
        factory.setReadTimeout(1000);       //读取超时时间  默认1秒
        factory.setConnectTimeout(2000);    //连接超时时间  默认1秒
        //
        return new RestTemplate(factory);
    }
}
```

参考：org.springframework.cloud.netflix.ribbon.RibbonClientConfiguration.ribbonClientConfig()方法


#### 局部配置

也可以在调用方的application.yml中对某个微服务进行局部设置

```yaml
#局部配置有效
provider:   #微服务名字
  ribbon:
    ReadTimeout: 1000 
    ConnectTimeout: 1000 
```



#### ribbon重试机制

##### 导入依赖

```xml
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
</dependency>
```

##### 开启重试

在主启动类上添加**<font color='red'>@EnableRetry</font>**注解开启重试机制

```java
@SpringBootApplication
@EnableEurekaClient
@EnableRetry
public class ConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }

}
```

##### 指定策略

在application.yml中指定全局重试策略

```yaml
ribbon:
  eureka:
    enabled: true #开启ribbon，并从eureka中获取其它微服务信息
  MaxAutoRetries: 1 # 切换实例后重试最大次数 默认0
  OkToRetryOnAllOperations: true #对所有超时的请求启用重试机制  默认false
  MaxAutoRetriesNextServer: 1 #切换重试实例的最大个数  默认1
```

也可以在单个微服务下单独设置

```yaml
#局部配置有效
provider:   #微服务名字
  ribbon:
    ReadTimeout: 1000
    ConnectTimeout: 1000
    MaxAutoRetries: 1 # 切换实例后重试最大次数 默认0
    OkToRetryOnAllOperations: true #对所有超时的请求启用重试机制  默认false
    MaxAutoRetriesNextServer: 1 #切换重试实例的最大个数  默认1
```



**具体配置参考类：com.netflix.client.config.DefaultClientConfigImpl**



### Ribbon负载均衡（中 30）

#### 负载均衡策略

 常见负载均衡策略

| 策略                                               | 解释                                                         |
| -------------------------------------------------- | ------------------------------------------------------------ |
| com.netflix.loadbalancer.RandomRule                | 随机，在服务实例中随机选择请求                               |
| com.netflix.loadbalancer.RoundRobinRule            | 轮询，根据服务列表轮流请求                                   |
| com.netflix.loadbalancer.RetryRule                 | 在RoundRobinRule的基础上添加重试机制，即在指定的重试时间内，反复使用线性轮询策略来选择可用实例 |
| com.netflix.loadbalancer.WeightedResponseTimeRule  | 对RoundRobinRule的扩展，响应速度越快的实例选择权重越大，越容易被选择 |
| com.netflix.loadbalancer.BestAvailableRule         | 选择并发较小的实例                                           |
| com.netflix.loadbalancer.AvailabilityFilteringRule | 先过滤掉故障实例，再选择并发较小的实例                       |
| com.netflix.loadbalancer.ZoneAwareLoadBalancer     | 采用双重过滤，同时过滤不是同一区域的实例和故障实例，选择并发较小的实例 |

#### 全局设置

在WebConfiguration配置类中通过代码方式指定

```java
//全局设置负载均衡策略
@Bean
public IRule rule(){
    return new RandomRule();
}
```

#### 局部设置

**<font color='red'>注意：全局与局部同时存在，ribbon优先使用全局设置，因此需要先注释掉全局设置才有用</font>**

```yaml
#局部配置有效
provider:   #微服务名字
  ribbon:
#    ReadTimeout: 1000
#    ConnectTimeout: 1000
#    MaxAutoRetries: 1 # 切换实例后重试最大次数 默认0
#    OkToRetryOnAllOperations: true #对所有超时的请求启用重试机制  默认false
#    MaxAutoRetriesNextServer: 1 #切换重试实例的最大个数  默认1
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RoundRobinRule
```



### 公共模块

公共模块中定义各个子模块中共同的类，方便维护

#### 在父项目名上右键新建maven项目

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210715152700.png" alt="image-20210715152700234" style="zoom: 33%;" />

##### 指定项目名字

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210715152735.png" alt="image-20210715152734971" style="zoom: 33%;" />

##### 在公共模块的pom.xml中导入lombok依赖，**并指定打包方式为jar包**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>springcloud-teach</artifactId>
        <groupId>com.woniuxy</groupId>
        <version>1.0</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <packaging>jar</packaging>

    <artifactId>commons</artifactId>
    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.20</version>
        </dependency>
    </dependencies>
</project>
```

##### 创建Goods实体类

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Accessors(chain = true)
public class Goods {
    private int id;
    private String name;
}
```

目录结构如下

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210715155016.png" alt="image-20210715155016857" style="zoom:50%;" />

##### 将commons打包

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210715155132.png" alt="image-20210715155132415" style="zoom: 50%;" />

##### 在provider、consumer模块的pom.xml中分别引入commons模块

```xml
<!--导入公共模块-->
<dependency>
     <groupId>com.woniuxy</groupId>
     <artifactId>commons</artifactId>
     <version>1.0</version>
</dependency>
```

##### 将provider、consumer的controller中使用的Goods更换成commons模块中的Goods



##### 测试

**ResponseResult按照同样的方式提取到commons模块中**



**<font color='red'>注意：只要在commons中做了修改，请立即将commons进行打包，如果不打包其它模块无法得到最新的更改</font>**