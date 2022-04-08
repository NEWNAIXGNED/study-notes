### Zuul概述及使用（中 30）

#### 介绍

 <img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210716153731.png" alt="image-20210716153730904" style="zoom:50%;" />

外部的应用如何来访问内部各种各样的微服务呢?在微服务架构中， 后端服务往往不直接开放给调用端,而是通过于个API网关根据请求的url ,路由到相应的服务。当添加API网关后,在第三方调用端和服务提供方之间就创建了一面墙,这面墙直接与调用方通信进行权限控制,后将请求均衡分发给后台服务端。

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210716153756.png" alt="image-20210716153756292" style="zoom:67%;" />

Spring Cloud Zuul路由是微服务架构中不可或缺的一部分提供动态路由,监控,弹性,安全等的边缘服务。Zuul 是Netflix出品的一个基于JVM路由和服务端的负载均衡器。

 

作用:就是服务转发,接收并转发所有内外部的客户端调用。使用Zuul可以作为资源的统一访问入口,同时也可以在网关做一些权限校验等类似的功能。



#### 路由的应用场景

##### 简化客户端调用复杂度

在微服务架构模式下,后端服务的实例数一般是动态的,对于客户端而言很难发现动态改变的服务实例的访问地址信息。因此在基于微服务的项目中为了简化前端的调用逻辑,通常会引入API Gateway作为轻量级网关,同时API Gateway中也会实现相关的认证逻辑从而简化内部服务之间相互调用的复杂度。

 

##### 数据裁剪以及聚合

通常而言不同的客户端对于显示时对于数据的需求是不一致的。比如手机端或者Web端又或者在低延迟的网络环境或者高延迟的网络环境。

因此为了优化客户端的使用体验,API Gateway可以对通用性的响应数据进行裁剪以适应不同客户端的使用需求。同时还可以将多个API调用逻辑进行聚合,从而减少客户端的请求数,优化客户端用户体验。

 

##### 多渠道支持

当然我们还可以针对不同的渠道和客户端提供不同的API Gateway,手机端、pc端和云端都可以提供不同的API Gateway.

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210716153856.png" alt="image-20210716153856496" style="zoom:67%;" />

#### 代码实现

##### 创建zuul微服务

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210716154057.png" alt="image-20210716154057097" style="zoom:50%;" />

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
   <module>zuul</module>
</modules>
```



##### 在zuul的application.yml中配置eureka、路由等信息

```yaml
spring:
  application:
    name: zuul
server:
  port: 9500
eureka:
  client: #客户端注册到eureka列表中
    service-url:
      defaultZone: http://127.0.0.1:9001/eureka
  instance:
    instance-id: zuul-9500  #注册中心status显示出来的微服务id
    prefer-ip-address: true #显示访问url
zuul:
  routes:
    provider: /goods/**    #goods开头的请求都交给provider微服务处理
info:
  app.name: SpringCloud
  company.name: woniuxy
  build.artifactId: $project.artifactId$
  build.version: $project.version$
```



##### 在zuul的主启动类上通过**<font color='red'>@EnableZuulProxy</font>**注解开启zuul

```java
@SpringBootApplication
@EnableEurekaClient //开启eureka
@EnableZuulProxy //开启路由
public class ZuulApplication {

    public static void main(String[] args) {
        SpringApplication.run(ZuulApplication.class, args);
    }

}
```



##### 分别启动eureka、provider、zuul



##### 通过zuul访问provider中的方法

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210716164425.png" alt="image-20210716164425478" style="zoom:50%;" />

也可以通过微服务的名字来访问

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210716164530.png" alt="image-20210716164529847" style="zoom:50%;" />

在application.yml中配置忽略微服务名字调用

```yaml
zuul:
  ignored-services: "*"
  routes:
    provider: /goods/**
```



再通过微服务名调用时报404

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210716164709.png" alt="image-20210716164709396" style="zoom:50%;" />



### Zuul配置（中 40）

配置路由规则、负载均衡、网关限流

#### 路由规则

##### 默认路由规则

如果不定义路由规则使用默认路由规则。

```yaml
#zuul:
#  ignored-services: "*"
#  routes:
#    provider: /goods/**
```

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210716165535.png" alt="image-20210716165535347" style="zoom:50%;" />



##### 自定义微服务访问URL

需要在zuul的配置文件中添加如下定义
比如：定义SERVICE-ORDER服务的URL

```yaml
zuul:
  ignored-services: "*"
  routes:
    provider: /goods/**
```



##### 定义微服务名与对应URL，需要在zuul的配置文件中添加如下定义

比如：定义SERVICE-ORDER服务的URL

```yaml
zuul:
  routes:
    abc: #路由名，用户自定义
      service-id: provider #调用的微服务名字，最好小写
      path: /goods/**  #匹配的路径，此处所有的请求都被provider微服务处理
    aaa:
      service-id: consumer #调用的微服务名字，最好小写
      path: /order/**  #匹配的路径，此处所有的请求都被provider微服务处理
```

访问

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210716170028.png" alt="image-20210716170027983" style="zoom:50%;" />



##### 指定微服务与具体访问地址

需要在zuul-proxy的配置文件中添加如下定义
比如：定义SERVICE-ORDER服务的URL

```yaml
zuul:
  routes:
    abc: #路由名，用户自定义
      path: /goods/**  #匹配的路径，此处所有的请求都被provider微服务处理
      url: http://localhost:8080/
```

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210716170028.png" alt="image-20210716170027983" style="zoom:50%;" />

**<font color='red'>注意：使用具体的微服务地址，无法使用Ribbon的负载均衡、服务降级、熔断功能</font>**



#### 负载均衡

zuul自动实现负载均衡，zuul实现负载均衡很简单，使用serviceId进行绑定后，如果有多个相同的serviceid，则会进行轮询的方式进行访问。而且实现是基于客户端负载均衡。



##### 客户端负载均衡

基于客户端的负载均衡，简单的说就是在客户端程序里面，自己设定一个调度算法，在向服务器发起请求的时候，先执行调度算法计算出向哪台服务器发起请求，然后再发起请求给服务器。

特点：

1. 由客户端内部程序实现，不需要额外的负载均衡器软硬件投入。
2. 程序内部需要解决业务服务器不可用的问题，服务器故障对应用程序的透明度小。
3. 程序内部需要解决业务服务器压力过载的问题。

使用场景：

1. 可以选择为初期简单的负载均衡方案，和DNS负载均衡一样。
2. 比较适合于客户端具有成熟的调度库函数，算法以及API等
3. 毕竟适合对服务器入流量较大的业务，如HTTP POST文件上传，FTP文件上传，Memcache大流量写入。
4. 可以结合其他负载均衡方案进行架构。



#### 网关限流（网关Filter）

Zuul的核心是一系列的filters, 其作用类似Servlet框架的Filter，Zuul把客户端请求路由到业务处理逻辑的过程中，这些filter在路由的特定时期参与了一些过滤处理，比如实现鉴权、流量转发、请求统计等功能。Zuul的整个运行机制，可以用下图来描述。

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210717133732.png" alt="image-20210717133732577" style="zoom:50%;" />

##### 过滤器的生命周期

Filter的生命周期有4个，分别是“PRE”、“ROUTING”、“POST”、“ERROR”，整个生命周期可以用下图来表示。

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210717133755.png" alt="image-20210717133755709" style="zoom:50%;" />

基于Zuul的这些过滤器，可以实现各种丰富的功能，而这些过滤器类型则对应于请求的典型生命周期。

a）PRE： 这种过滤器在请求被路由之前调用。我们可利用这种过滤器实现身份验证、在集群中选择请求的微服务、记录调试信息等。

 

b）ROUTING：这种过滤器将请求路由到微服务。这种过滤器用于构建发送给微服务的请求，并使用

Apache HttpClient或Netfilx Ribbon请求微服务。

 

c）POST：这种过滤器在路由到微服务以后执行。这种过滤器可用来为响应添加标准的HTTP Header、收集统计信息和指标、将响应从微服务发送给客户端等。

 

d）ERROR：在其他阶段发生错误时执行该过滤器。

 

除了默认的过滤器类型，Zuul还允许我们创建自定义的过滤器类型。例如，我们可以定制一种STATIC类型的过滤器，直接在Zuul中生成响应，而不将请求转发到后端的微服务。



##### Zuul中默认实现的Filter

Zuul默认实现了很多Filter，这些Filter如下面表格所示。

| **类型** | **顺序** | **过滤器**              | **功能**                   |
| -------- | -------- | ----------------------- | -------------------------- |
| pre      | -3       | ServletDetectionFilter  | 标记处理Servlet的类型      |
| pre      | -2       | Servlet30WrapperFilter  | 包装HttpServletRequest请求 |
| pre      | -1       | FormBodyWrapperFilter   | 包装请求体                 |
| route    | 1        | DebugFilter             | 标记调试标志               |
| route    | 5        | PreDecorationFilter     | 处理请求上下文供后续使用   |
| route    | 10       | RibbonRoutingFilter     | serviceId请求转发          |
| route    | 100      | SimpleHostRoutingFilter | url请求转发                |
| route    | 500      | SendForwardFilter       | forward请求转发            |
| post     | 0        | SendErrorFilter         | 处理有错误的请求响应       |
| post     | 1000     | SendResponseFilter      | 处理正常的请求响应         |



##### 自定义Filter

实现自定义滤器需要继承ZuulFilter，并实现ZuulFilter中的抽象方法。

```java
/**
 * 自定义过滤器
 */
@Component
public class CustomFilter extends ZuulFilter {

    //过滤器类型
    @Override
    public String filterType() {
        return "pre";
    }

    //执行顺序
    @Override
    public int filterOrder() {
        return 0;
    }

    //是否过滤
    @Override
    public boolean shouldFilter() {
        return true;
    }

    //对请求进行处理
    @Override
    public Object run() throws ZuulException {
        System.out.println("正在执行过滤...");
        return null;
    }
}
```



#### 常见限流算法

##### 漏桶算法

又称leaky bucket。为了理解漏桶算法，我们看一下对于该算法的示意图：

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210717135526.png" alt="image-20210717135526354" style="zoom: 33%;" />

从图中我们可以看到，整个算法其实十分简单。首先，我们有一个固定容量的桶，有水流进来，也有水流出去。对于流进来的水来说，我们无法预计一共有多少水会流进来，也无法预计水流的速度。但是对于流出去的水来说，这个桶可以固定水流出的速率。而且，当桶满了之后，多余的水将会溢出。

我们将算法中的水换成实际应用中的请求，我们可以看到漏桶算法天生就限制了请求的速度。当使用了漏桶算法，我们可以保证接口会以一个常速速率来处理请求。所以漏桶算法天生不会出现临界问题。

漏桶算法可以粗略的认为就是注水漏水过程，往桶中以一定速率流出水，以任意速率流入水，当水超过桶流量则丢弃，因为桶容量是不变的，保证了整体的速率。



##### 令牌桶算法

对于很多应用场景来说，除了要求能够限制数据的平均传输速率外，还要求允许某种程度的突发传输。这时候漏桶算法可能就不合适了，令牌桶算法更为适合。如图所示，令牌桶算法的原理是系统会以一个恒定的速度往桶里放入令牌，而如果请求需要被处理，则需要先从桶里获取一个令牌，当桶里没有令牌可取时，则拒绝服务。



<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210717142953.png" alt="image-20210717142953331" style="zoom: 33%;" />



#### 基于令牌桶算法的网关限流

##### 为了提高对zuul的保护，先在zuul的pom.xml导入hystrix依赖并开启

```xml
<dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

##### 开启熔断

```java
@SpringBootApplication
@EnableEurekaClient 
@EnableZuulProxy 
@EnableCircuitBreaker  //开启熔断
public class ZuulApplication {

    public static void main(String[] args) {
        SpringApplication.run(ZuulApplication.class, args);
    }

}
```



##### 自定义zuulfilter，添加实现限流代码

限流器：**<font color='red'>com.google.common.util.concurrent.RateLimiter</font>**

```java
package com.woniuxy.zuul.filter;

import com.commons.result.ResponseResult;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.google.common.util.concurrent.RateLimiter;
import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import com.netflix.zuul.exception.ZuulException;
import org.springframework.stereotype.Component;

import javax.servlet.http.HttpServletResponse;

/**
 * 自定义过滤器
 */
@Component
public class CustomFilter extends ZuulFilter {
    private static int count = 1;

    //创建限流器  500个令牌
    private static final RateLimiter RATE_LIMITER = RateLimiter.create(500);

    //过滤器类型
    @Override
    public String filterType() {
        return "pre";
    }

    //执行顺序
    @Override
    public int filterOrder() {
        return 0;
    }

    //是否过滤
    @Override
    public boolean shouldFilter() {
        return true;
    }

    //对请求进行处理
    @Override
    public Object run() throws ZuulException {
        //获取上下文对象
        RequestContext context = RequestContext.getCurrentContext();
        //获取response对象
        HttpServletResponse response = context.getResponse();
        //如果无法获取令牌
        if (!RATE_LIMITER.tryAcquire()){
            System.out.println("令牌不足，停止服务......." + count++);
            //停止访问
            context.setSendZuulResponse(false);

            // 返回消息
            response.setContentType("application/json;charset=utf-8");
            ResponseResult<Object> result = new ResponseResult<>(500,"系统繁忙,请稍后再试",null);
            try {
                response.getWriter().write(new ObjectMapper().writeValueAsString(result));
            }catch (Exception e){
                e.printStackTrace();
            }
        }
        return null;
    }
}

```

##### 运行eureka、两个provider、zuul，利用jemiter向zuul发送**<font color='red'>1000</font>**个请求



##### 如果请求失败的太多可以通过增加zuul超时时间解决

```yaml
zuul:
  host:
    connect-timeout-millis: 30000  #连接超时
    socket-timeout-millis: 10000  #请求超时
```



**<font color='red'>注意：第一次jemiter发送请求时请求失败的次数可能会比较多（原因暂不清楚），可以再执行一次查看结果</font>**



### Gateway简介（中 20）

#### 介绍

SpringCloud Gateway 是 Spring Cloud 的一个全新项目，该项目是基于 Spring 5.0，Spring Boot 2.0 和 Project Reactor 等技术开发的网关，它旨在为微服务架构提供一种简单有效的统一的 API 路由管理方式。

SpringCloud Gateway 作为 Spring Cloud 生态系统中的网关，目标是替代 Zuul，在Spring Cloud 2.0以上版本中，没有对新版本的Zuul 2.0以上最新高性能版本进行集成，仍然还是使用的Zuul 2.0之前的非Reactor模式的老版本。而为了提升网关的性能，SpringCloud Gateway是基于WebFlux框架实现的，而WebFlux框架底层则使用了高性能的Reactor模式通信框架Netty。

Spring Cloud Gateway 的目标，不仅提供统一的路由方式，并且基于 Filter 链的方式提供了网关基本的功能，例如：安全，监控/指标，和限流。



##### SpringCloud Gateway 特征

SpringCloud官方，对SpringCloud Gateway 特征介绍如下：

（1）基于 Spring Framework 5，Project Reactor 和 Spring Boot 2.0

（2）集成 Hystrix 断路器

（3）集成 Spring Cloud DiscoveryClient

（4）Predicates（谓词、断言） 和 Filters 作用于特定路由，易于编写的 Predicates 和 Filters

（5）具备一些网关的高级功能：动态路由、限流、路径重写

从以上的特征来说，和Zuul的特征差别不大。SpringCloud Gateway和Zuul主要的区别，还是在底层的通信框架上。



##### 专业术语

a）Filter（过滤器）：

和Zuul的过滤器在概念上类似，可以使用它拦截和修改请求，并且对上游的响应，进行二次处理。过滤器为org.springframework.cloud.gateway.filter.GatewayFilter类的实例。



b）Route（路由）：

网关配置的基本组成模块，和Zuul的路由配置模块类似。一个Route模块由一个 ID，一个目标 URI，一组断言和一组过滤器定义。如果断言为真，则路由匹配，目标URI会被访问。



c）Predicate（谓词、断言）：

这是一个 Java 8 的 Predicate，可以使用它来匹配来自 HTTP 请求的任何内容，例如 headers 或参数。断言的输入类型是一个 ServerWebExchange。



##### 工作流程

客户端向 Spring Cloud Gateway 发出请求。然后在 Gateway Handler Mapping 中找到与请求相匹配的路由，将其发送到 Gateway Web Handler。Handler 再通过指定的过滤器链来将请求发送到我们实际的服务执行业务逻辑，然后返回。过滤器之间用虚线分开是因为过滤器可能会在发送代理请求之前（“pre”）或之后（“post”）执行业务逻辑。

![Spring Cloud Gateway Diagram](https://docs.spring.io/spring-cloud-gateway/docs/3.0.4-SNAPSHOT/reference/html/images/spring_cloud_gateway_diagram.png)

| 类型 | 作用                                                         |
| ---- | ------------------------------------------------------------ |
| pre  | 这种过滤器在请求被路由之前调用。我们可利用这种过滤器实现身份验证、在集群中选择请求的微服务、记录调试信息等。 |
| post | 这种过滤器在路由到微服务以后执行。这种过滤器可用来为响应添加标准的HTTP Header、收集统计信息和指标、将响应从微服务发送给客户端等。 |



Filter在“pre”类型过滤器中可以做参数校验、权限校验、流量监控、⽇志输出、协议转换等，在“post”类型的过滤器中可以做响应内容、响应头的修改、⽇志的输出、流量监控等。



### Gateway路由配置（高 40）

#### 创建gateway微服务

##### 引入eureka、gateway、hystrix，**<font color='red'>因为gateway是基于webflux，与spring web不兼容，因此不要导入spring web</font>**

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210717164514.png" alt="image-20210717164513949" style="zoom:50%;" />

 

##### 在pom.xml文件中指定父项目

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
    <module>zuul</module>
    <module>gateway</module>
</modules>
```



##### 主启动类上开启eureka、hystrix

```java
@SpringBootApplication
@EnableEurekaClient
@EnableCircuitBreaker
public class GatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }

}
```



##### 配置gateway的application.yml

```yaml
spring:
  application:
    name: gateway
  cloud:
    inetutils:
      default-ip-address: 127.0.0.1
    gateway:
      routes:
        - id: provider  #路由规则命名   自定义，不重复   一个id代表一个微服务，可以配置多个id
          uri: lb://provider   #lb loadbalance 负载均衡缩写
          predicates:
            - Path=/goods/**
#      discovery:
#        locator:
#          lower-case-service-id: true #eureka中微服务名字为大写，此处开启表示在gateway中所有微服务名称使用小写名称定义
#          enabled: true #开启从注册中心定位路由服务
server:
  port: 9600
eureka:
  client: #客户端注册到eureka列表中
    service-url:
      defaultZone: http://127.0.0.1:9001/eureka
  instance:
    instance-id: gateway-9600  #注册中心status显示出来的微服务id
    prefer-ip-address: true #显示访问url
```



**<font color='red'>说明：Path=/goods/... 是指以/goods/开头的url都交给provider微服务处理，但是在转发请求的时候，gateway不会去掉/goods/前缀，因此在provider的controller上需要加上基础url</font>**

**<font color='red'>@RequestMapping("/goods")</font>**

```java
@RestController
@RequestMapping("/goods")
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



##### 依次运行eureka、两个provider、gateway，通过gateway请求数据

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210717172010.png" alt="image-20210717172010427" style="zoom:50%;" />

#### 路由断言

Predicate（谓语、断言）：路由转发的判断条件，目前SpringCloud Gateway支持多种方式，常见如：Path、Host、Method、Query等。

##### Path 方式匹配转发

```yaml
routes:
   - id: provider  
     uri: lb://provider
     predicates:
        - Path=/goods/**
```



##### Host 方式匹配转发

Spring Cloud Gateway可以根据Host主机名进行匹配转发，如果我们的接口只允许**.woniuxy.com域名进行访问，那么配置如下所示：

```yaml
routes:
   - id: provider  
     uri: lb://provider
     predicates:
        - Host=**.woniuxy.com
```



##### Method断言

这个断言是专门验证HTTP Method的，在下面的例子中，当访问“/gateway/sample”并且HTTP Method是GET的时候，将适配下面的路由

```yaml
routes:
   - id: provider  
     uri: lb://provider
     predicates:
        - Path=/goods/**
        - Method=GET
```



##### Query断言

请求断言也是在业务中经常使用的，它会从ServerHttpRequest中的Parameters列表中查询指定的属性，有如下两种不同的使用方式

```yaml
routes:
   - id: provider  
     uri: lb://provider
     predicates:
        - Path=/goods/**
        - Query=name,zhangsan*
```

Query=name表示只要请求中包含 name 属性的参数即可匹配路由



Query也可以有两个参数，如：Query=name，zhangsan，表示请求中必须包含 name 属性而且name的值必须是zhangsan才能匹配路由

通配符

| 通配符 | 解释         |
| ------ | ------------ |
| .      | 任意一个字符 |
| *      | 任意多个字符 |



#### 跨域配置

由于gateway使用的是webflux，而不是springmvc，所以使用servlet中的filter全局设置跨域配置时无效的，因此需要先关闭webflux的cors，再从gateway的filter里边设置cors就行了。配置方式分为代码、yml两种

##### 方式一

在gateway中创建配置类

```java
package com.woniuxy.gateway.configuration;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.reactive.CorsWebFilter;
import org.springframework.web.cors.reactive.UrlBasedCorsConfigurationSource;
import org.springframework.web.util.pattern.PathPatternParser;

@Configuration
public class GatewayCorsConfiguration {
    @Bean
    public CorsWebFilter corsWebFilter(){
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.addAllowedMethod("*");
        configuration.addAllowedOrigin("*");
        configuration.addAllowedHeader("*");

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource(new PathPatternParser());
        source.registerCorsConfiguration("/**", configuration);
        //
        return new CorsWebFilter(source);
    }
}

```



##### 方式二

在application.yml中配置

```yaml
spring:
  application:
    name: gateway
  cloud:
    inetutils:
      default-ip-address: 127.0.0.1
    gateway:
      routes:
        - id: provider  #路由规则命名   自定义，不重复
          uri: lb://provider   #lb loadbalance 负载均衡缩写
          predicates:
            - Path=/goods/**
      globalcors:
        cors-configurations:
          '[/**]':
            allowedHeaders: "*"
            allowedOrigins: "*"
            allowedMethods:
              - GET
              - POST
              - DELETE
              - PUT
              - OPTION
```



### Route Filter路由过滤器（高 40）

Spring Cloud Gateway除了具备请求路由功能之外，也支持对请求的过滤。通过Zuul网关类似，也是通过过滤器的形式来实现的。那么接下来我们一起来研究一下Gateway中的过滤器



#### 过滤器的生命周期

Spring Cloud Gateway 的 Filter 的生命周期不像 Zuul 的那么丰富，它只有两个：“pre” 和 “post”

| 类型 | 作用                                                         |
| ---- | ------------------------------------------------------------ |
| pre  | 这种过滤器在请求被路由之前调用。我们可利用这种过滤器实现身份验证、在集群中选择请求的微服务、记录调试信息等。 |
| post | 这种过滤器在路由到微服务以后执行。这种过滤器可用来为响应添加标准的HTTP Header、收集统计信息和指标、将响应从微服务发送给客户端等。 |



#### 过滤器类型

Spring Cloud Gateway 的 Filter 从作用范围可分为另外两种GatewayFilter 与 GlobalFilter。

| 类型          | 作用                                 |
| ------------- | ------------------------------------ |
| GatewayFilter | 应用到单个路由或者一个分组的路由上。 |
| GlobalFilter  | 应用到所有的路由上。                 |



##### 局部过滤器

局部过滤器（GatewayFilter），是针对单个路由的过滤器。可以对访问的URL过滤，进行切面处理。在Spring Cloud Gateway中通过GatewayFilter的形式内置了很多不同类型的局部过滤器。这里简单将Spring Cloud Gateway内置的所有过滤器工厂整理成了一张表格，虽然不是很详细，但能作为速览使用。如下：

| 过滤器工厂                  | 作用                                                         | 参数                                                         |
| --------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| AddRequestHeader            | 为原始请求添加Header                                         | Header的名称及值                                             |
| AddRequestParameter         | 为原始请求添加请求参数                                       | 参数名称及值                                                 |
| AddResponseHeader           | 为原始响应添加Header                                         | Header的名称及值                                             |
| DedupeResponseHeader        | 剔除响应头中重复的值                                         | 需要去重的Header名称及去重策路                               |
| Hystrix                     | 为路由引入Hystrix的断路器保护                                | Hystrixcommand的名称                                         |
| FallbackHeaders             | 为fallbackUri的请求头中添加具体的异常信息                    | Header的名称                                                 |
| prefixPath                  | 为原始请求路径添加前缀                                       | 前缀路径                                                     |
| PreserveHostHeader          | 为请求添加一个preserveHostHeader=true的属性，路由过滤器会检查该属性以决定是否要发送原始的Host | 无                                                           |
| RequestRateLimiter          | 用于对请求限流，限流算法为令牌桶                             | KeyResolver、rateLimiter、statusCode、denyEmptyKey、emptyKeyStatus |
| RedirectTo                  | http状态码及重定向的URL                                      | 将原始请求重定向到指定的URL                                  |
| RemoveHopByHopHeadersFilter | 为原始请求册除IETF组织规定的一系列Header                     | 默认就会启用，可以通过配置指定仅出除比Header                 |
| RemoveRequestHeader         | 为原始请求删除某个Header                                     | Header名称                                                   |
| RemoveResponseHeader        | 为原始响应册除某个Header                                     | Header名称                                                   |
| RewritePath                 | 重写原始的请求路径                                           | 原始路径正则表达式以及重写后路径的正则表达式                 |
| RewriteResponseHeader       | 重写原始响应中的某个Header                                   | Header名称，值的正则表达式，重写后的值                       |
| SaveSession                 | 在转发请求之前，强制执行websession : :save操作               | 无                                                           |
| secureHeaders               | 为原始响应添加一系列起安全作用的响应头                       | 无，支持修改这些安全响应头的值                               |
| SetPath                     | 修改原始的请求路径                                           | 修改后的路径                                                 |
| SetResponseHeader           | 修改原始响应中某个Header的值                                 | Header名称，修改后的值                                       |
| SetStatus                   | 修改原始响应的状态码                                         | HTTP状态码，可以是数字，也可以是字符串                       |
| StripPrefix                 | 用于截断原始请求的路径                                       | 使用数字表示要截断的路径的数量                               |
| Retry                       | 针对不同的响应进行重试                                       | retries、statuses、methods、series                           |
| RequestSize                 | 设置允许接收最大请求包的大小。如果请求包大小超过设置的值，则返回413 Payload TooLarge | 请求包大小，单位为字节，默认值为5M                           |
| wModifyRequestBody          | 在转发请求之前修改原始请求体内容                             | 修改后的请求体内容                                           |
| ModifyResponseBody          | 修改原始响应体的内容                                         | 修改后的响应体内容                                           |



每个过滤器工厂都对应一个实现类，并且这些类的名称必须以 GatewayFilterFactory 结尾，这是Spring Cloud Gateway的一个约定，例如 AddRequestHeader 对应的实现类为AddRequestHeaderGatewayFilterFactory 。

 

##### 全局过滤器

全局过滤器（GlobalFilter）作用于所有路由，Spring Cloud Gateway 定义了Global Filter接口，用户可以自定义实现自己的Global Filter。通过全局过滤器可以实现对权限的统一校验，安全性验证等功能，并且全局过滤器也是程序员使用比较多的过滤器。Spring Cloud Gateway内部也是通过一系列的内置全局过滤器对整个路由转发进行处理如下：

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210718163636.png" alt="image-20210718163636616" style="zoom: 50%;" />

##### 自定义全局过滤器：实现GlobalFilter和Ordered接口

```java
package com.woniuxy.gateway.filter;

import org.reactivestreams.Publisher;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.core.io.buffer.DataBuffer;
import org.springframework.http.HttpStatus;
import org.springframework.http.server.reactive.ServerHttpRequest;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

import java.nio.charset.StandardCharsets;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;

@Component
public class CustomFilter implements GlobalFilter, Ordered {
    /**
     * 过滤器执行过滤代码
     */
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        //获取request对象
        ServerHttpRequest request = exchange.getRequest();
        //获取请求头
        String token = request.getHeaders().get("authorization").get(0);

        //继续执行
		return chain.filter(exchange);
    }

    /*
     * 指定过滤器的顺序，值越小越先执行
     */
    @Override
    public int getOrder() {
        return 0;
    }
}
```



##### 通过response对象返回JSON数据

```java
//获取response对象
ServerHttpResponse response = exchange.getResponse();

//准备数据
ResponseResult<Object> result = new ResponseResult<>(500,"error",null);
//将数据转换成byte数组
byte[] data = null;
try {
    data = new ObjectMapper().writeValueAsString(result).getBytes(StandardCharsets.UTF_8);
}catch (Exception e){
    e.printStackTrace();
}

//创建数据缓存对象
DataBuffer buffer = response.bufferFactory().wrap(data);

//设置响应头
response.getHeaders().add("Content-Type","application/json;charset=utf-8");

//返回数据
return response.writeWith(Mono.just(buffer));
```



### 微服务的认证方案简介（中 25）

基于Session的认证简介，基于Token的认证简介，JWT基本含义及其应用领域，传统Session认证优缺点

#### 传统的session认证

我们知道，http协议本身是一种无状态的协议，而这就意味着如果用户向我们的应用提供了用户名和密码来进行用户认证，那么下一次请求时，用户还要再一次进行用户认证才行，因为根据http协议，我们并不能知道是哪个用户发送的请求，所以为了让我们的应用能识别是哪个用户发出的，我们只能在服务器存储一份用户登陆的信息，这份登陆信息会在响应时传递给服务器，告诉其保存为cookie，以便下次请求时发送给我们的应用，这样我们的应用就能识别请求来自哪个用户了，这就是传统的基于sessino认证

 

但是这种基于session的认证使应用本身很难得扩展，随着不用客户端的增加，独立的服务器已无法承载更多的用户，而这个时候基于session认证应用的问题就会暴露出来



#### 基于session认证所显露的问题

- Session：每个用户经过我们的应用认证之后，我们的应用都要在服务端做一次记录，以便用户下次请求的鉴别，通常而言session都是保存在内存中，而随着认证用户的增多，服务端的开销会明显增大。

 

- 扩展性：用户认证之后，服务端做认证记录，如果认证的记录被保存在内存的话，这意味着用户下次请求还必须要请求在这台服务器上，这样才能拿到授权的资源，这样在分布式的应用上，响应的限制了负载均衡器的能力，也意味着限制了应用的扩展性。



#### 基于token的鉴权机制

基于token的鉴权机制类似于http协议也是无状态的，它不需要在服务端去保留用户的认证信息或会话信息。这也就意味着基于token认证机制的应用不需要去考虑用户在哪一台服务器登陆了，这就为应用的扩展提供了便利。

 

流程

- 用户使用用户名密码请求服务器

- 服务器进行验证用户信息

- 服务器通过验证发送给用户一个token

- 客户端存储token，并在每次请求时附加这个token值

- 服务器验证token，并返回数据

 

这个token必须要在每次请求时发送给服务器，它应该保存在请求头中，另外服务器要支持CORS（跨来源资源共享）策略，一般我们在服务端这么做就可以了： Access-Control-Allow-Origin：*

 

（Access-Control-Allow-Origin是HTML5中定义的一种解决资源跨域的策略。他是通过服务器端返回带有Access-Control-Allow-Origin标识的Response header，用来解决资源的跨域权限问题。）



### CSRF（中 20）

CSRF基本原理、GET和POST型CSRF

跨站请求伪造（英语：Cross-site request forgery），也被称为 one-click attack 或者 session riding，通常缩写为 CSRF 或者 XSRF， 是一种挟制用户在当前已登录的Web应用程序上执行非本意的操作的攻击方法。

 

#### CSRF原理

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210719093005.png" alt="image-20210719093005022" style="zoom: 50%;" />

A并不知道(5)中的请求是C发出的还是B发出的，由于浏览器会自动带上用户C的Cookie，所以A会根据用户的权限出(5)的请求，这样B就达到了模拟用户操作的目的

 

从上图能够看出，要完毕一次CSRF攻击，受害者必须依次完毕两个步骤：

- 登录受信任站点A，并在本地生成Cookie。

- 在不登出A的情况下，訪问危急站点B。



#### 常见的攻击类型

##### GET类型的CSRF

仅仅须要一个HTTP请求。就能够构造一次简单的CSRF。

案例：

```html
<!-- 银行站点A：它以GET请求来完毕银行转账的操作，如：-->
http://www.mybank.com/Transfer.php?toBankId=11&money=1000

<!-- 危急站点B：它里面有一段HTML的代码例如以下：-->
<img src=http://www.mybank.com/Transfer.php?toBankId=11&money=1000>
```

首先。你登录了银行站点A，然后訪问危急站点B，这时你会发现你的银行账户少了1000块

为什么会这样呢？原因是银行站点A违反了HTTP规范，使用GET请求更新资源。
在访问危急站点B的之前。你已经登录了银行站点A，而B中的 一个合法的请求，但这里被不法分子利用了）。所以你的浏览器会带上你的银行站点A的Cookie发出Get请求，去获取资源以GET的方式请求第三方资源（这里的第三方就是指银行站点了，原本这是http://www.mybank.com/Transfer.php?toBankId=11&money=1000 ，结果银行站点服务器收到请求后，觉得这是一个更新资源操作（转账操作），所以就立马进行转账操作。



##### POST类型的CSRF

这种类型的CSRF危害没有GET型的大，利用起来通常使用的是一个自动提交的表单

案例：

```html
<form action=http://www.mybank.com/Transfer.php method=POST>
	<input type="text" name="toBankId" value="11" />
	<input type="text" name="money" value="1000" />
</form>
<script> document.forms[0].submit(); </script> 
```

访问该页面后，表单会自动提交，相当于模拟用户完成了一次POST操作。



**<font color='red'>目前主流的做法是使用Token抵御CSRF攻击。</font>**



#### JWT

官网：https://jwt.io/

一般用于身份验证和数据信息交换

Json web token（JWT）是为了在网络应用环境间传递声明而执行的一种基于JSON的开发标准（RFC 7519），该token（令牌）被设计为紧凑且安全的，特别适用于分布式站点的单点登陆（SSO）场景。JWT的声明一般被用来在身份提供者和服务提供者间传递被认证的用户身份信息，以便于从资源服务器获取资源，也可以增加一些额外的其它业务逻辑所必须的声明信息，该token也可直接被用于认证，也可被加密。



### 基于JWT的认证流程及JWT组成结构（高 15）

Header的基本作用、构成；payload的基本作用、构成，共有声明、私有声明；signature的基本含义及其作用

#### JWT的构成

JWT是由三部分构成，将这三段信息文本用链接构成了JWT字符串。就像这样

 	<font color='red'>eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9</font>.<font color='green'>eyJVc2VySWQiOjEyMywiVXNlck5hbWUiOiJhZG1pbiJ9</font>.<font color='blue'>Qjw1epD5P6p4Yy2yju3-fkq28PddznqRj3ESfALQy_U</font>

第一部分我们称它为头部（header）第二部分我们称其为载荷（payload，类似于飞机上承载的物品），第三部分是签证（signature）

 

##### header

JWT的头部承载的两部分信息：

- 声明类型，这里是jwt

- 声明加密的算法，通常直接使用HMAC SHA256或RSA

 

完整的头部就像下面这样的JSON

```json
{ 'typ':'JWT', 'alg':'HSA256' }
```

然后将头部进行base64加密（该加密是可以对称解密的），构成了第一部分	

​	eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9



##### plyload

也称为JWT Claims，包含用户的一些非隐私数据

- 标准中注册的声明

- 公共的声明

- 私有的声明

 

a）标注中注册的声明（建议不强制使用）

​	·iss (issuer)			签发人

​	·exp (expiration time)		过期时间

​	·sub (subject)			主题

​	·aud (audience)			受众用户

​	·nbf (Not Before)			在此之前不可用

​	·iat (Issued At)			签发时间

​	·jti (JWT ID)			JWT唯一标识，能用于防止JWT重复使用

 

b）公共的声明（用户自定义）：

公共的声明可以添加任何的信息，一般添加用户的相关信息或其它业务需要的必要信息，但不建议添加敏感信息，因为该部分在客户端可解密；

 

c）私有的声明

私有的声明是提供者和消费者功能定义的声明，一般不建议存放敏感信息，因为base64是对称解密的，意味着该部分信息可以归类为名文信息。

 

定义一个payload

```json
{ "sub": "1234567890", "name": "John Doe", "admin": true }
```

然后将其base64加密，得到jwt的一部分

​	eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9



##### Signature

jwt的第三部分是一个签证信息，这个签证信息由三部分组成：

​	·header(base64后的)

​	·payload(base64后的)

​	·secred

 

这个部分需要base64加密后的header和base64加密后的payload使用“.”连接组成的字符串，然后通过header中声明的加密方式进行加secret组合加密，然后就构成了jwt的第三部分。将这三部分用“.”连接成一个完整的字符串，构成了最终的jwt：

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ

 

注意：secret是保存在服务器端的，jwt的签发也是在服务端的，secret就是用来进行jwt的签发和jwt的验证，所以它就是你服务端的私钥，在任何场景都不应该流露出去，一旦客户端得知这个secret，那就意味着客户端可以自我签发jwt了



##### 流程

·一种做法是放在HTTP请求的头信息字段里面，格式如下：

Token: <token>

 

·另一种做法是，JWT就放在POST请求的数据体里面。

服务端会验证token，如果验证通过就会返回相应的资源，整个流程就是这样

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210719094230.png" alt="image-20210719094230501" style="zoom:50%;" />

详细流程(时序图)

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210719094259.png" alt="image-20210719094259072" style="zoom:50%;" />



### SpringBoot中使用JWT（高  40）

#### 在父项目中引入redis依赖

```xml
<!--redis-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <version>2.3.7.RELEASE</version>
</dependency>
```



#### 在commons模块中导入jwt、Redis、jackson依赖

```xml
<!-- JWT -->
<dependency>
	<groupId>com.auth0</groupId>
	<artifactId>java-jwt</artifactId>
	<version>3.4.0</version>
</dependency>
<!--redis-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<!--jackson-->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.12.3</version>
</dependency>
```



##### 在commons模块中创建JWT用到的工具类和枚举

```java
package com.commons.enums;

public enum TokenEnum {
    TOKEN_EXPIRE,TOKEN_BAD,TOKEN_SUCCESS
}
```

```java
package com.commons.utils;

import java.util.Date;

import com.auth0.jwt.JWT;
import com.auth0.jwt.JWTVerifier;
import com.auth0.jwt.algorithms.Algorithm;
import com.auth0.jwt.exceptions.TokenExpiredException;
import com.commons.enums.TokenEnum;

public class JWTUtil {
    public static final String SECRET_KEY = "123456"; //秘钥
    public static final long TOKEN_EXPIRE_TIME = 1 * 60 * 1000; //token过期时间
    public static final long REFRESH_TOKEN_EXPIRE_TIME = 10 * 60 * 1000; //refreshToken过期时间
    private static final String ISSUER = "issuer"; //签发人

    /**
     * 生成签名
     */
    public static String generateToken(String uname){
        Date now = new Date();
        //创建签名算法对象
        Algorithm algorithm = Algorithm.HMAC256(SECRET_KEY); //算法

        String token = JWT.create()
                .withIssuer(ISSUER) //签发人
                .withIssuedAt(now)  //签发时间
                .withExpiresAt(new Date(now.getTime() + TOKEN_EXPIRE_TIME)) //过期时间
                .withClaim("uname", uname) //保存身份标识
                .sign(algorithm);
        return token;
    }

    /**
     * 验证token
     */
    public static TokenEnum verify(String token){
        try {
            //签名算法
            Algorithm algorithm = Algorithm.HMAC256(SECRET_KEY); //算法
            JWTVerifier verifier = JWT.require(algorithm)
                    .withIssuer(ISSUER)
                    .build();
            verifier.verify(token);
            return TokenEnum.TOKEN_SUCCESS;
        } catch (TokenExpiredException ex){
            return TokenEnum.TOKEN_EXPIRE;
            //ex.printStackTrace();
        } catch (Exception e) {
            return TokenEnum.TOKEN_BAD;
        }
    }

    /**
     * 从token获取uname
     */
    public static String getUname(String token){
        try{
            return JWT.decode(token).getClaim("uname").asString();
        }catch(Exception ex){
            ex.printStackTrace();
        }
        return "";
    }
}
```



##### 在commons模块中添加Redis配置类

```java
package com.commons.configuration;

import org.springframework.cache.annotation.CachingConfigurerSupport;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;

@EnableCaching
@Configuration
public class RedisConfiguration extends CachingConfigurerSupport{

    @Bean
    @SuppressWarnings("all")
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> template  = new RedisTemplate<>();
        //设置工厂
        template.setConnectionFactory(factory);
        //使用Jackson2JsonRedisSerializer来序列化和反序列化redis的value值（默认使用JDK的序列化方式）
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer =
                new Jackson2JsonRedisSerializer<>(Object.class);
        //创建对象映射
        ObjectMapper mapper = new ObjectMapper();
        //指定要序列化的域，field,get和set,以及修饰符范围，ANY是都有包括private和public
        mapper.setVisibility(PropertyAccessor.ALL,JsonAutoDetect.Visibility.ANY);
        //指定序列化输入的类型，类必须是非final修饰的，final修饰的类，比如String,Integer等会抛出异常
        mapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        //
        jackson2JsonRedisSerializer.setObjectMapper(mapper);
        //字符串序列化器
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        //key采用String的序列化方式
        template.setKeySerializer(stringRedisSerializer);
        //hash的key也采用String的方式
        template.setHashKeySerializer(stringRedisSerializer);
        //value采用jackson的方式
        template.setValueSerializer(jackson2JsonRedisSerializer);
        //hash的value也采用Jackson
        template.setHashValueSerializer(jackson2JsonRedisSerializer);

        template.afterPropertiesSet();
        return template;
    }
}
```



#### 创建auth模块，导入web、eureka依赖

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210719100717.png" alt="image-20210719100717522" style="zoom:33%;" />

##### auth中指定父项目

```xml
<parent>
   <artifactId>springcloud-teach</artifactId>
   <groupId>com.woniuxy</groupId>
   <version>1.0</version>
</parent>
```



##### 父项目指定子模块

```xml
<modules>
    <module>auth</module>
</modules>
```



##### 在auth中引入commons模块，主要是引入Redis

```xml
<dependency>
    <groupId>com.woniuxy</groupId>
    <artifactId>commons</artifactId>
    <version>1.0</version>
</dependency>
```



##### 在auth主启动类上开启eureka

```java
@SpringBootApplication
@EnableEurekaClient
public class AuthApplication {
    public static void main(String[] args) {
        SpringApplication.run(AuthApplication.class, args);
    }
}
```



##### 在auth中添加controller

```java
package com.woniuxy.auth.controller;

import com.commons.entity.User;
import com.commons.result.ResponseResult;
import com.commons.utils.JWTUtil;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;
import javax.servlet.http.HttpServletResponse;
import java.util.HashMap;
import java.util.Map;
import java.util.UUID;
import java.util.concurrent.TimeUnit;

@RestController
@RequestMapping("/auth")
public class AuthController {
    @Resource
    private RedisTemplate<String,Object> redisTemplate;

    @GetMapping("/login")
    private ResponseResult<String> login(User user, HttpServletResponse response){
        //假设登录成功，生成token和refreshToken
        String account = "zhangsan";
        String token = JWTUtil.generateToken(account);
        String refreshToken = UUID.randomUUID().toString();

        //以refreshToken为key将token和account存放到redis中
        Map<String,Object> data = new HashMap<>();
        data.put("token",token);
        data.put("account",account);
        redisTemplate.opsForHash().putAll(refreshToken,data);
        
        //设置refreshToken过期时间   1天内过期
        redisTemplate.expire(refreshToken,1, TimeUnit.DAYS);

        //设置响应头，将token和refreshToken返回给前端
        response.setHeader("authorization",token);
        response.setHeader("refreshToken",refreshToken);
        //暴露头
        response.setHeader("Access-Control-Expose-Headers","authorization,refreshToken");

        //返回结果
        return new ResponseResult<>(200,"success",null);
    }
}
```

#### 运行eureka、auth，请求login方法

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210719150917.png" alt="image-20210719150917101" style="zoom: 25%;" />

得到数据表明OK











