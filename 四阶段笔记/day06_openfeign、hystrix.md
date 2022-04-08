### OpenFeign声明式通信（中 15）

#### 声明式通信

 <img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210715152316.png" alt="image-20210715152316596" style="zoom:50%;" />

之前我们通过RestTemplate调用其它服务的API时，所需要的参数须在请求的URL中进行拼接，如果参数少的话或许我们还可以忍受，一旦有多个参数的话，这时拼接请求字符串就会效率低下。

 

那么有没有更好的解决方案呢？答案是确定的有，Netflix已经为我们提供了一个框架：Feign。

 

Feign是一个声明式的Web Service客户端，它的目的就是让Web Service调用更加简单。Feign提供了HTTP请求的模板，通过编写简单的接口和插入注解，就可以定义好HTTP请求的参数、格式、地址等信息。

 

而Feign则会完全代理HTTP请求，我们只需要像调用方法一样调用它就可以完成服务请求及相关处理。Feign整合了Ribbon和Hystrix(关于Hystrix我们后面再讲)，可以让我们不再需要显式地使用这两个组件。

 

总起来说，Feign具有如下特性：

- 可插拔的注解支持，包括Feign注解和JAX-RS注解;

- 支持可插拔的HTTP编码器和解码器;

- 支持Hystrix和它的Fallback;

- 支持Ribbon的负载均衡;

- 支持HTTP请求和响应的压缩。



#### 创建consumer-feign子模块



<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210715155655.png" alt="image-20210715155655429" style="zoom: 33%;" />

##### 导入web、eureka依赖

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210715155813.png" alt="image-20210715155813007" style="zoom:33%;" />

##### 在consumer-feign的pom.xml中指定父项目

```xml
<parent>
    <artifactId>springcloud-teach</artifactId>
    <groupId>com.woniuxy</groupId>
    <version>1.0</version>
</parent>
```

##### 在父项目中引入子模块

```xml
<modules>
    <module>eureka-server</module>
    <module>provider</module>
    <module>consumer</module>
    <module>commons</module>
    <module>consumer-feign</module>
</modules>
```

##### 在父项目中导入openfeign依赖

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-openfeign-core</artifactId>
            <version>2.1.3.RELEASE</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

##### 在consumer-feign的pom.xml中导入feign依赖和公共模块

```xml
<!--openfeign-->
<dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-openfeign-core</artifactId>
</dependency>
<!--公共模块-->
<dependency>
     <groupId>com.woniuxy</groupId>
     <artifactId>commons</artifactId>
     <version>1.0</version>
</dependency>
```



##### 编辑application.yml配置feign

```yaml
spring:
  application:
    name: consumer-feign
server:
  port: 80
ribbon:
  eureka:
    enabled: true
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:9001/eureka
  instance:
    instance-id: consumer-feign  #注册中心显示出来的微服务名称
```



##### 在commons模块中导入openfeign依赖、并创建service接口

**<font color='red'>spring-cloud-starter-openfeign</font>**

```xml
<!--openfeign-->
<dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```



##### 添加openfeign接口

```java
package com.commons.service;

import com.commons.entity.Goods;
import com.commons.result.ResponseResult;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.*;

import java.util.List;

//指定调用微服务的名字
@FeignClient(name = "PROVIDER")
public interface ProviderService {
    //接口与被调用的controller一致
    @GetMapping("/all")
    public List<Goods> all();
}
```

**<font color='red'>将commons模块重新打包</font>**



##### 在consumer-feign中创建controller，并注入service

```java
@RestController
public class FeignController {
    @Resource
    private ProviderService providerService;

    //接口与被调用的controller一致
    @GetMapping("/all")
    public List<Goods> all(){
        return providerService.all();
    }
}
```



##### 在consumer-feign的主启动类上开启eureka和openfeign

```java
@SpringBootApplication
@EnableEurekaClient
@EnableFeignClients(basePackages = "com.commons.service")
public class ConsumerFeignApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsumerFeignApplication.class, args);
    }

}
```

basePackages扫描openfeign service所在的包



**分别按顺序启动eureka、provider、consumer-feign**



### OpenFeign请求传递对象（高 30）

#### 在commons模块的ProviderService接口中添加以下方法

```java
@FeignClient(name = "PROVIDER")
public interface ProviderService {
    //接口与被调用的controller一致
    @GetMapping("/all")
    public List<Goods> all();

    @GetMapping("find/{id}")
    public Goods findById(@PathVariable("id") int id);

    @PostMapping("/add")
    public ResponseResult<Boolean> add(@RequestBody Goods goods);

    @PutMapping("/update")
    public void update(@RequestBody Goods goods);

    @DeleteMapping("/del/{id}")
    public void del(@PathVariable("id") int id);
}
```

##### <font color="red">将commons模块重新打包</font>

#### 在consumer-feign的controller类上添加对应的方法

```java
@RestController
public class FeignController {
    @Resource
    private ProviderService providerService;

    //接口与被调用的controller一致
    @GetMapping("/all")
    public List<Goods> all(){
        return providerService.all();
    }

    @GetMapping("find/{id}")
    public Goods findById(@PathVariable("id") int id){
        return providerService.findById(id);
    }

    @PostMapping("/add")
    public ResponseResult<Boolean> add(@RequestBody Goods goods){
        return providerService.add(goods);
    }

    @PutMapping("/update")
    public void update(@RequestBody Goods goods){
        providerService.update(goods);
    }

    @DeleteMapping("/del/{id}")
    public void del(@PathVariable("id") int id){
        providerService.del(id);
    }
}
```



##### 运行程序通过postman进行测试

postman发送json

1、选择请求方式、设置请求头

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210715205729.png" alt="image-20210715205729068" style="zoom: 33%;" />

2、设置body的编码方式为raw，application/json,  raw是发送纯文本，不包含任何空格的编码方式

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210715205833.png" alt="image-20210715205833518" style="zoom: 33%;" />



### OpenFeign配置（高 20）

#### 负载均衡

openfeign 默认采用ribbon为负载均衡器，可以在配置类中指定负载均衡策略

```java
@Configuration
public class RibbonConfiguration {

    @Bean
    public IRule rule(){
        return new RandomRule();
    }
}
```



#### feign通信日志

Feign 提供了日志打印功能，可以通过配置来调整日志级别，从而了解 Feign 中 Http 请求的细节。

说白了就是对接口的调用情况进行监控和输出。

 

##### 日志级别

| 级别    | 解释                                                        |
| ------- | ----------------------------------------------------------- |
| NONE    | 默认的，不显示任何日志                                      |
| BASIC   | 仅记录请求方法、URL、响应状态码及执行时间                   |
| HEADERS | 除了 BASIC 中定义的信息之外，还有请求和响应的头信息         |
| FULL    | 除了 HEADERS 中定义的信息之外，还有请求和响应的正文及元数据 |

 

在consumer-feign中添加feign日志配置类，指定日志输出级别以显示指定的内容

注意Level的包为：**feign.Logger**

```java
import feign.Logger;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class LogConfiguration {
    @Bean
    public Logger.Level level(){
        return Logger.Level.FULL;
    }
}
```

因为feign日志的输出级别都是debug级别，因此如果想要看到日志信息，还需要设置service包的日志级别

```yaml
logging:
  level:
    com.commons.service: debug
```

运行consumer-feign，在其控制台会显示以下信息

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210715202905.png" alt="image-20210715202905651" style="zoom:33%;" />



### Hystrix概述（中 15）

Hystrix简介、雪崩效应与熔断机制

#### 雪崩效应

在微服务架构中通常会有多个服务层调用，基础服务的故障可能会导致级联故障，进而造成整个系统不可用的情况，这种现象被称为服务雪崩效应。服务雪崩效应是一种因“服务提供者"的不可用导致“服务消费者”的不可用，并将不可用逐渐放大的过程。

如果下图所示: A作为服务提供者，B为A的服务消费者，C和D是B的服务消费者。A不可用引起了B的不可用，并将不可用像滚雪球一样放大到C和D时，雪崩效应就形成了。

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210715210009.png" alt="image-20210715210009270" style="zoom: 67%;" />

服务雪崩的过程可以分为三个阶段：

a）服务提供者不可用

b）重试加大请求流量

c）服务调用者不可用

 

应对策略：

a）应用扩容，包括增加机器数量、升级硬件规格等

b）流量控制，限流、关闭重试

c）缓存，缓存预加载

d）服务降级，服务接口拒绝服务、页面拒绝服务、延迟持久化、随机拒绝服务

e）服务熔断



#### 熔断器（Hystrix）

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210715210124.png" alt="image-20210715210124778" style="zoom: 67%;float: left;" />

在分布式系统中，单个应用通常会有多个不同类型的外部依赖服务，内部通常依赖于各种RPC服务，外部则依赖于各种HTTP服务。这些依赖服务不可避免的会出现调用失败，比如超时、异常等情况，如何在外部依赖出问题的情况，仍然保证自身应用的稳定，就是Hystrix这类服务保障框架的工作了。

 

Hystrix是一个用于分布式系统的延迟和容错的开源库。在分布式系统里，许多依赖不可避免的调用失败，比如超时、异常等，Hystrix能够保证在一个依赖出问题的情况下，不会导致整个服务失败，避免级联故障，以提高分布式系统的弹性。

 

生活中举个例子，如电力过载保护器，当电流过大的时候，出问题，过载器会自动断开，从而保护电器不受烧坏。因此Hystrix请求熔断的机制跟电力过载保护器的原理很类似。

 

举个例子来说，某个应用中依赖了30个外部服务，实际应用中通常比这还要多，假设每个服务的可用性为99.99%，4个9的可用性，算是不错了，但99.99%的30次幂≈ 99.7%，也就是有0.3%的请求不可用，假设总共有1亿次请求，那么就意味着有300万次请求会失败，如果一切正常每个月就有2个小时服务是不可用的。

 

现实通常是更糟糕

当一切正常时，请求看起来是这样的

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210715210437.png" alt="image-20210715210436869" style="zoom:67%;" />

当其中有一个系统有延迟时，它可能阻塞整个用户请求：

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210715210455.png" alt="image-20210715210455534" style="zoom:67%;" />

在高流量的情况下，一个后端依赖项的延迟可能导致所有服务器上的所有资源在数秒内饱和（PS：意味着后续再有请求将无法立即提供服务）

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210715210511.png" alt="image-20210715210511685" style="zoom:67%;" />

#### Hystrix设计原则是什么

·防止任何单个依赖项耗尽所有容器（如Tomcat）用户线程。

·甩掉包袱，快速失败而不是排队。

·在任何可行的地方提供回退，以保护用户不受失败的影响。

·使用隔离技术（如隔离板、泳道和断路器模式）来限制任何一个依赖项的影响。

·通过近实时的度量、监视和警报来优化发现时间。

·通过配置的低延迟传播来优化恢复时间。

·支持对Hystrix的大多数方面的动态属性更改，允许使用低延迟反馈循环进行实时操作修改。

·避免在整个依赖客户端执行中出现故障，而不仅仅是在网络流量中。



#### Fallback

Fallback相当于是降级操作。对于查询操作,我们可以实现一个fallback方法，当请求后端服务出现异常的时候,可以使用fallback方法返回的值。fallback方法的返回值一般是设置的默认值或者来自缓存。

 

·资源隔离

在Hystrix中,主要通过线程池来实现资源隔离。通常在使用的时候我们会根据调用的远程服务划分出多个线程池。例如调用产品服务的Command放入A线程池，调用账户服务的Command放入B线程池。这样做的主要优点是运行环境被隔离开了。这样就算调用服务的代码存在bug或者由于其他原因导致自己所在线程池被耗尽时，不会对系统的其他服务造成影响。但是带来的代价就是维护多个线程池会对系统带来额外的性能开销。如果是对性能有严格要求而且确信自己调用服务的客户端代码不会出问题的话，可以使用Hystrix的信号模式(Semaphores)来隔离资源。



### RestTemplate与Hystrix整合（低 20）

#### 在**<font color='red'>consumer</font>**模块中导入hystrix依赖

注：hystrix导入的依赖如下

```xml
<!--hystrix-->
<dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

#### 再导入公共模块

```xml
<!--公共模块-->
<dependency>
    <groupId>com.woniuxy</groupId>
    <artifactId>commons</artifactId>
    <version>1.0</version>
</dependency>
```



#### 在主启动类上添加**<font color="red">@EnableCircuitBreaker</font>**注解开启熔断器

```java
@SpringBootApplication
@EnableEurekaClient
@EnableRetry
@EnableCircuitBreaker  //开启降级
public class ConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }

}
```



#### 在controller需要保护的方法上添加**<font color="red">@HystrixCommand</font>**注解，并指定fallback方法

```java
@RestController
public class ConsumerController {
    @Resource
    private RestTemplate restTemplate;

    @RequestMapping("/all")
    @HystrixCommand(fallbackMethod = "fallback")
    public List<Goods> all(){
        String url = "http://PROVIDER/all";
        //
        List<Goods> goods = restTemplate.getForObject(url,List.class);
        //
        return goods;
    }
    //返回值、参数必须与对应的方法保持一致
    public List<Goods> fallback(){

        List<Goods> goods = Arrays.asList(
                new Goods(4001,"fallbackA"),
                new Goods(4002,"fallbackB")
        );
        //
        return goods;
    }
}
```

#### 启动eureka、consumer模块，然后访问all方法，得到以下结果表示成功

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210715214749.png" alt="image-20210715214749221" style="zoom:50%;" />



### OpenFeign与Hystrix整合（高 20）

#### 服务降级

<font color='red'>巨坑：一定要确保Comsumer的依赖中不要自己再导入ribbon了，如果有则删除，因为feign其实已经集成了ribbon，而我们导入的ribbon将feign中的ribbon给覆盖了，而我们又没有给新的ribbon指定http请求、超时时间等配置原因导致的，所以在使用feign调用微服务时就不要再自己引入ribbon，反而破坏了feign与ribbon的整合关系。</font>

 

什么是服务降级？当服务器压力剧增的情况下，根据实际业务情况及流量，对一些服务和页面有策略的不处理或换种简单的方式处理，从而释放服务器资源以保证核心交易正常运作或高效运作。

 

如果还是不理解，那么可以举个例子：假如目前有很多人想要给我付钱，但我的服务器除了正在运行支付的服务之外，还有一些其它的服务在运行，比如搜索、定时任务和详情等等。然而这些不重要的服务就占用了JVM的不少内存与CPU资源，为了能把钱都收下来（钱才是目标），我设计了一个动态开关，把这些不重要的服务直接在最外层拒掉，这样处理后的后端处理收钱的服务就有更多的资源来收钱了（收钱速度更快了），这就是一个简单的服务降级的使用场景。

 

所谓降级，就是一般是从整体层面考虑，就是当某个服务熔断之后，服务器将不再被调用，此刻客户端可以自己准备一个本地的fallback回调，返回一个缺省值，这样做，虽然服务水平下降，但好歹可用，比直接挂掉要强。

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210715215026.png" alt="image-20210715215026501" style="zoom:67%;" />



#### 使用场景

服务降级主要用于什么场景呢？当整个微服务架构整体的负载超出了预设的上限阈值或即将到来的流量预计将会超过预设的阈值时，为了保证重要或基本的服务能正常运行，我们可以将一些 不重要 或 不紧急 的服务或任务进行服务的 延迟使用 或 暂停使用。

 

实现方式：<font color='red'>在客户端实现服务降级，在客户端指定本地fallback</font>



##### 在**<font color='red'>commons</font>**模块创建fallback工厂类   <font color='red'>不需要导包</font>

注意：需要通过**<font color='red'>@Component</font>**将factory加入到IOC容器

```java
@Component
public class ProviderServiceFactory implements FallbackFactory<ProviderService> {

    @Override
    public ProviderService create(Throwable throwable) {
        return new ProviderService() {
            @Override
            public List<Goods> all() {

                return Arrays.asList(
                        new Goods(5001,"service-backA"),
                        new Goods(5002,"service-backB")
                );
            }

            @Override
            public Goods findById(int id) {
                return new Goods(5003,"service-backC");
            }

            @Override
            public ResponseResult<Boolean> add(Goods goods) {
                return new ResponseResult<>();
            }

            @Override
            public void update(Goods goods) {
                System.out.println("update服务降级");
            }

            @Override
            public void del(int id) {
                System.out.println("del服务降级");
            }
        };
    }
}
```



注意目录结构

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210715220232.png" alt="image-20210715220231879" style="zoom:50%;float: left;" />

##### 将commons模块重新打包



##### 在consumer-feign的application.yml中开启服务降级

```yaml
feign:
  hystrix:
    enabled: true  #开启服务降级
```



##### 在consumer-feign的主启动类上通过@SpringBootApplication改变默认扫描起始位置，以便扫描到factory所在的包

```java
@SpringBootApplication(scanBasePackages = "com")
@EnableEurekaClient
@EnableFeignClients(basePackages = "com.commons.service")
```



**<font color='red'>分别启动eureka、provider、consumer-feign模块，先正常请求，然后关闭provider再次请求查看服务是否降级</font>**，出现以下结果说明成功

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210715221936.png" alt="image-20210715221936403" style="zoom:50%;" />



### Hystrix超时设置（高 15）

#### 在provider的controller对应方法上添加sleep

```java
@GetMapping("find/{id}")
public Goods findById(@PathVariable("id") int id){
    try {
        Thread.sleep(5000);
    }catch (Exception e){
    }
    System.out.println(id);
    return new Goods(1003,"平板");
}
```



#### 运行eureka、provider、consumer-feign，访问consumer-feign的find方法，一秒钟之后自动熔断，得到factory中的结果

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210716103448.png" alt="image-20210716103448712" style="zoom:50%;" />



#### 在consumer-feign的application.yml中开启hystrix超时配置、设置超时时间

```yaml
hystrix:
  command:
    default:
      execution:
        timeout:
          enabled: true #开启超时管理
        isolation:
          thread:
            timeoutInMilliseconds: 10000  #设置超时时间
```



#### 重启consumer-feign，再次请求find，结果发现还是得不到数据，原因是openfeign默认采用了ribbon作为负载均衡器，而ribbon也是有超时时间的，因此此处由于超过了ribbon的超时时间导致的熔断，所以还需要修改ribbon的默认超时时间

```yaml
ribbon:
  eureka:
    enabled: true
  http:
    client:
      enabled: true   #开启超时管理
  ReadTimeout: 10000  #请求超时
  ConnectTimeout: 10000 #连接超时
```



#### 重启consumer-feign，请求find接口测试，5秒钟之后返回结果，说明超时设置成功



### Hystrix Dashboard（低 20）

#### 介绍

我们在公司开发过程中,微服务越来越多,为了保证系统的健壮性和安全性,我们加入了熔断机制。那现在我们想要知道每个微服务的运行状态,针对微服务进行优化。应该怎么办呢?

 

Hystrix-dashboard是一款针对Hystrix 进行实时监控的工具,通过Hystrix Dashboard我们可以在直观地看到各Hystrix Command的请求响应时间，请求成功率等数据。


#### 代码操作

##### 创建dashboard模块，该模块专门用来监控其他模块，而且它**<font color='red'>不用注册到注册中心</font>**

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210716110359.png" alt="image-20210716110359142" style="zoom:50%;" />

##### pom.xml中配置父项目

```xml
<parent>
    <artifactId>springcloud-teach</artifactId>
    <groupId>com.woniuxy</groupId>
    <version>1.0</version>
</parent>
```



##### 在父项目中引入子模块

```xml
<modules>
   <module>eureka-server</module>
   <module>provider</module>
   <module>consumer</module>
   <module>commons</module>
   <module>consumer-feign</module>
   <module>dashboard</module>
</modules>
```



##### 配置application.yml

```yaml
# 应用名称
spring:
  application:
    name: dashboard
server:
  port: 9999
hystrix:
  dashboard:
  	#添加服务监控白名单，可以监控哪些IP的服务，如：localhost
    proxyStreamAllowList: '127.0.0.1,localhost,192.168.41.161'  
```



##### 在主启动类上开启dashboard**<font color='red'>@EnableHystrixDashboard</font>**

```java
@SpringBootApplication
@EnableHystrixDashboard  //开启仪表盘
public class DashboardApplication {

    public static void main(String[] args) {
        SpringApplication.run(DashboardApplication.class, args);
    }

}
```



##### 启动项目在浏览器上访问：http://localhost:9999/hystrix，如下图所示

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210716113523.png" alt="image-20210716113523538" style="zoom: 33%;" />

##### 在需要被监控的微服务（provider）中添加hystrix依赖

```xml
<dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```



##### 在其主启动类上开启hystrix

```java
@SpringBootApplication
@EnableEurekaClient
@EnableCircuitBreaker
public class ProviderApplication {
    public static void main(String[] args) {
        SpringApplication.run(ProviderApplication.class, args);
    }
}
```



**<font color='red'>在需要被监控的方法上添加@HystrixCommand并指定fallback，只有加了@HystrixCommand注解的方法才能被dashboard监控</font>**

```java
@GetMapping("/all")
@HystrixCommand(fallbackMethod = "fallback")
public List<Goods> all(){
    System.out.println(port);
    return Arrays.asList(
            new Goods(1001,"手机"),
            new Goods(1002,"电脑")
    );
}
public List<Goods> fallback(){
    System.out.println(port);
    return Arrays.asList(
            new Goods(6001,"手机"),
            new Goods(6002,"电脑")
    );
}
```



##### 在provider 的application.yml文件中暴露监控端口

```yaml
management:
  endpoints:
    web:
      exposure:
        include: hystrix.stream
```



##### 启动provider，然后在dashboard的地址栏中输入：localhost:8080/actuator/hystrix.stream并指定名字，点击按钮进行监控

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210716114152.png" alt="image-20210716114152535" style="zoom: 33%;" />

如果一切正常，会得到如下页面



<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210716121859.png" alt="image-20210716121859144" style="zoom:33%;" />

①圆点：微服务的健康状态，颜色有绿色、黄色、橙色、红色，健康状态依次降低

②线条：流量变化

③请求的方法

④成功请求（绿色）

⑤短路请求（蓝色）

⑥坏请求（青色）

⑦超时请求（黄色）

⑧被拒绝的请求（紫色）

⑨失败请求（红色）

⑩最近10秒钟内请求错误的百分比

11请求频率

12熔断器状态

13数据延迟统计

14线程池



### Hystrix熔断实验（中 15）

#### 原理

Hystrix的作用就是保险箱的作用，如果它在一段时间内侦测到许多类似的错误，会强迫其以后的多个调用快速失败，不再访问远程服务器，从而防止应用程序不断地尝试执行可能会失败的操作，使得应用程序继续执行而不用等待修正错误，或者浪费CPU时间去等到长时间的超时产生。熔断器也可以使应用程序能够诊断错误是否已经修正，如果已经修正，应用程序会再次尝试调用操作。

熔断器模式就像是那些容易导致错误的操作的一种代理。这种代理能够记录最近调用发生错误的次数，然后决定使用允许操作继续，或者立即返回错误。熔断器开关相互转换的逻辑如下图:

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210715210631.png" alt="image-20210715210631335" style="zoom:67%;" />

#### 特点

·断路器机制

断路器很好理解,当Hystrix Command请求后端服务失败数量超过一定比例(默认50%),断路器会切换到开路状态(Open).这时所有请求会直接失败而不会发送到后端服务。断路器保持在开路状态一段时间后(默认5秒),自动切换到半开路状态(HAL F-OPEN)。这时会判断下一次请求的返回情况，如果请求成功，断路器切回闭路状态(CLOSED),否则重新切换到开路状态(OPEN)。



### Hystrix熔断设置（高 10）


详细参数设置参考com.netflix.hystrix.HystrixCommandProperties类



#### 在consumer-feign的application.yml文件中添加以下配置

```yaml
hystrix:
  command:
    default:
      execution:
        timeout:
          enabled: true #开启超时管理
        isolation:
          thread:
            timeoutInMilliseconds: 10000  #设置超时时间  默认1秒
      fallback:
        isolation:
          semaphore:
            maxConcurrentRequests: 10 #设置调用fallback函数的最大并发数（线程数）  默认10
        enabled: true  #开启fallback  默认true
      circuitBreaker:
        enabled: true  #设置断路器是否起作用 默认true
        sleepWindowInMilliseconds: 3000  #熔断器打开后多少秒内 熔断状态变成半开状态  默认5秒
        errorThresholdPercentage: 50 #错误百分比条件，
        requestVolumeThreshold: 10 #打开断路器的最少失败的请求数
        forceOpen: false  #强制打开断路器
        forceClosed: false #强制关闭断路器
```

##### hystrix.command.default

| 参数                                               | 解释                                                         |
| -------------------------------------------------- | ------------------------------------------------------------ |
| fallback.isolation.semaphore.maxConcurrentRequests | 设置调用fallback函数的最大并发数  默认10，如果并发数达到该设置值，请求会被拒绝和抛出异常并且fallback不会被调用 |
| fallback.enabled                                   | 开启fallback  默认true                                       |
| circuitBreaker.enabled                             | 设置断路器是否起作用 默认true                                |
| circuitBreaker.sleepWindowInMilliseconds           | 熔断器打开后多少秒内 熔断状态变成半熔断状态  默认5秒         |
| circuitBreaker.errorThresholdPercentage            | 错误百分比条件，达到熔断器最小请求数后错误率达到百分之多少后打开熔断器 默认50% |
| circuitBreaker.requestVolumeThreshold              | 打开断路器的最少请求数,比如：如果值是20，在一个窗口内（比如10秒），收到19个请求，即使这19个请求都失败了，断路器也不会打开。 |
| circuitBreaker.forceOpen                           | 是否强制打开断路器 默认false                                 |
| circuitBreaker.forceClosed                         | 是否强制关闭断路器 默认false                                 |

consumer-feign的pom文件中导入hystrix和actuator依赖

```xml
<!--hystrix依赖-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
<!--监控-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

在application.yml中暴露监控端口

```yaml
management:
  endpoints:
    web:
      exposure:
        include: hystrix.stream
```

在主启动类上开启熔断器

```java
package com.woniuxy.consumerfeign;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

//修改自动扫描的起始位置
@SpringBootApplication(scanBasePackages = "com.woniuxy")
@EnableEurekaClient //开启注册
@EnableFeignClients(basePackages = "com.woniuxy.commons.service")  //扫描service
@EnableCircuitBreaker
public class ConsumerFeignApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsumerFeignApplication.class, args);
    }

}
```

分别启动eureka、consumer-feign和dashboard

在dashboard中输入监控的url信息

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210804202547.png" alt="image-20210804202547080" style="zoom:33%;" />

另开一个窗口请求consumer-feign中的接口，然后观察dashboard，熔断器状态为closed

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210804203128.png" alt="image-20210804203128399" style="zoom:33%;" />

然后请求到10次的时候可以看到熔断器状态变为了open

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210804203249.png" alt="image-20210804203249517" style="zoom:33%;" />

然后启动provider，再次请求consumer-feign中的该接口，可以看到熔断器可以变为closed状态

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210804203513.png" alt="image-20210804203513766" style="zoom:33%;" />

<font color='red'>注意：启动provider之后如果立即访问consumer-feign的接口，可能不会正常得到数据，原因是此时consumer-feign微服务并不知道provider已经可用，其根本原因是微服务会每隔30秒钟去eureka中获取服务列表，只有获取到最新的服务列表consumer-feign才知道provider可用，才能调用成功</font>







