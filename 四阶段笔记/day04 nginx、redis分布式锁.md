1、切换工作目录到 /usr/local/nginx下

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220324093530.png" alt="image-20220324093530432" style="zoom:67%;" />

2、目录介绍

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220324093622.png" alt="image-20220324093622206" style="zoom:67%;" />

3、修改配置文件

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220324093701.png" alt="image-20220324093701749" style="zoom:67%;" />

vi /usr/local/nginx/conf/nginx.conf

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220324093909.png" alt="image-20220324093909290" style="zoom:67%;" />

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220324094350.png" alt="image-20220324094350569" style="zoom:67%;" />

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220324094619.png" alt="image-20220324094619305" style="zoom:67%;" />

得到windows的ip地址

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220324094837.png" alt="image-20220324094837217" style="zoom:67%;" />

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220324094908.png" alt="image-20220324094908024" style="zoom:67%;" />

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220324095047.png" alt="image-20220324095047320" style="zoom: 67%;" />



5、在项目中创建NginxController类，添加以下代码

```java
@RestController
@RequestMapping("/nginx")
public class NginxController {

    @Value("${server.port}")
    private String port;

    @GetMapping("/port")
    public String port(){
        return port;
    }
}
```



6、双开idea

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220324100617.png" alt="image-20220324100617208" style="zoom: 67%;" />

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220324100113.png" alt="image-20220324100113373" style="zoom:67%;" />

在VM options中添加以下配置

-Dserver.port=8081



7、分别启动两个项目，然后在Windows浏览器分别访问这两个项目，确保没有问题



8、启动nginx

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220324102226.png" alt="image-20220324102226794" style="zoom:67%;" />



9、在Windows的浏览器上访问nginx

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220324102401.png" alt="image-20220324102401489" style="zoom:67%;" />

10、配置负载均衡策略

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220324103645.png" alt="image-20220324103645662" style="zoom:67%;" />

权重

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220324103843.png" alt="image-20220324103843659" style="zoom:67%;" />



11、保存退出，并重启nginx

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220324103924.png" alt="image-20220324103924379" style="zoom:67%;" />



12、在Windows浏览器上访问nginx进行测试





### redis分布式锁

#### 什么是分布式锁？

要介绍分布式锁，首先要提到与分布式锁相对应的是线程锁、进程锁。

 

线程锁：主要用来给方法、代码块加锁。当某个方法或代码使用锁，在同一时刻仅有一个线程执行该方法或该代码段。线程锁只在同一JVM中有效果，因为线程锁的实现在根本上是依靠线程之间共享内存实现的，比如synchronized是共享对象头，显示锁Lock是共享某个变量（state）。

 

进程锁：为了控制同一操作系统中多个进程访问某个共享资源，因为进程具有独立性，各个进程无法访问其他进程的资源，因此无法通过synchronized等线程锁实现进程锁。

 

分布式锁：当多个进程不在同一个系统中，用分布式锁控制多个进程对资源的访问。

分布式锁是控制分布式系统或不同系统之间共同访问共享资源的一种锁实现，如果不同的系统或同一个系统的不同主机之间共享了某个资源时，往往需要互斥来防止彼此干扰来保证一致性。

 

#### 分布式锁的使用场景。

线程间并发问题和进程间并发问题都是可以通过分布式锁解决的，但是强烈不建议这样做！因为采用分布式锁解决这些小问题是非常消耗资源的！分布式锁应该用来解决分布式情况下的多进程并发问题才是最合适的。

有这样一个情境，线程A和线程B都共享某个变量X。

如果是单机情况下（单JVM），线程之间共享内存，只要使用线程锁就可以解决并发问题。

如果是分布式情况下（多JVM），线程A和线程B很可能不是在同一JVM中，这样线程锁就无法起到作用了，这时候就要用到分布式锁来解决。

 

#### 实现方式

分布式锁一般有三种实现方式：1.数据库乐观锁；2.基于Redis的分布式锁；3.基于ZooKeeper的分布式锁。

 ##### Redis实现分布式锁

- 利用Redis界面管理工具设置一个key，模拟商品剩余数量

创建springboot项目

项目结构：

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220324165227.png" alt="image-20220324165227049" style="zoom:67%;" /> 

在pom中引入以下依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

编写application.yml文件

```yaml
spring:
  redis:
    port: 6379
    host: 192.168.74.145   # 虚拟机ip
    jedis:
      pool:
        max-idle: 8   # 最大发呆（空闲）的连接数
        max-wait: -1ms   # 最大等待时间  -1代表一直登陆
        max-active: 8 # 最大活跃的连接数  最大连接数
        min-idle: 2  # 最小发呆的连接数
```

创建redis配置类

```java
import org.springframework.cache.annotation.CachingConfigurerSupport;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.cache.interceptor.KeyGenerator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;

import java.lang.reflect.Method;

/*
 * 开启redis缓存，设置key的生产策略。主要用于注解情况下，默认使用方法的全限定名作为key    自动生成key
 */
@EnableCaching
@Configuration
public class RedisConfiguration extends CachingConfigurerSupport{

    /**
     * redisTemplate相关配置
     * @param factory
     * @return
     */
    @Bean
    @SuppressWarnings("all")
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> template  = new RedisTemplate<>();
        //设置工厂
        template.setConnectionFactory(factory);
        //使用Jackson2JsonRedisSerializer来序列化和反序列化redis的value值
        //（默认使用JDK的序列化方式）
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer =
                new Jackson2JsonRedisSerializer<>(Object.class);
        //创建对象映射
        ObjectMapper mapper = new ObjectMapper();
        //指定要序列化的域，field,get和set,以及修饰符范围，ANY是都有包括private和public
        mapper.setVisibility(PropertyAccessor.ALL,JsonAutoDetect.Visibility.ANY);
        //指定序列化输入的类型，类必须是非final修饰的，final修饰的类，比如String,Integer
        //等会抛出异常
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

创建controller

```java
package com.woniuxy.controller;


import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/sale")
public class RedisController {
	
	@Autowired
	private RedisTemplate<String, Object> redisTemplate;
	
	@GetMapping("/kill")
	public boolean kill() {
		boolean flag = false;		
		int num = (Integer)redisTemplate.opsForValue().get("num");
		if (num>0) {
			num--;
			System.out.println("删减库存成功,剩余库存:"+num);
			//设置回去
			redisTemplate.opsForValue().set("num", num+"");
			flag = true;
		}
		return flag;
	}
}
```

改进：通过setnx命令实现简单的分布式锁

修改controller

```java
@RestController
@RequestMapping("/sale")
public class RedisController {
	
	@Autowired
	private RedisTemplate<String, String> redisTemplate;
	
	@GetMapping("/kill")
	public boolean kill() {
		boolean flag = false;
		String lock = "key";  //通过setnx命令设置key
		boolean result = redisTemplate.opsForValue().setIfAbsent(lock, "woniuxy");
		if (!result) {
			//如果拿不到，直接返回
			return flag;
		}
		
		int num = Integer.parseInt(redisTemplate.opsForValue().get("num"));
		if (num>0) {
			num--;
			System.out.println("删减库存成功,剩余库存:"+num);
			//设置回去
			redisTemplate.opsForValue().set("num", num+"");
			flag = true;
		}
		//删除key
		redisTemplate.delete(lock);
		return flag;
	}
}
```

缺陷：在删除key之前的业务代码如果出现异常，则key一致都不会被释放，相当于死锁



改进：加try catch，在finally中释放锁

```java
public boolean kill() {
	boolean flag = false;
	String lock = "key";
		
	try {
		boolean result = redisTemplate.opsForValue().setIfAbsent(lock, "woniuxy");
		if (!result) {
			//如果拿不到，直接返回
			return flag;
		}
			
		int num = Integer.parseInt(redisTemplate.opsForValue().get("num"));
		if (num>0) {
			num--;
			System.out.println("删减库存成功,剩余库存:"+num);
			//设置回去
			redisTemplate.opsForValue().set("num", num+"");
			flag = true;
		}
	} finally{
		//删除key
		redisTemplate.delete(lock);
	}
	return flag;
}
```

缺陷：当前服务器宕机也无法释放锁



改进：设置过期时间

```java
public boolean kill() {
	boolean flag = false;
	String lock = "key";
		
	try {
		boolean result = redisTemplate.opsForValue().setIfAbsent(lock, "woniuxy", 10, TimeUnit.SECONDS);
		.....
	} finally{
		//删除key
		redisTemplate.delete(lock);
	}	
	return flag;
}
```

缺陷：如果完成业务时间的大于或者小于过期时间，都有可能出现问题，如下：

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220324165648.png" alt="image-20220324165648064" style="zoom:67%;" />

假设高并发下现在有三个线程，执行业务所需时间分别是13s、7s和7s，最开始线程A获取到key然后执行，当执行到第10秒时key过期，此时线程B获取到key，然后继续执行3秒，结果线程A执行完毕将key删除，此时线程C获取到key开始执行，线程B又执行5秒后删除key....依次类推，这样就会出现耗时长的线程会删除耗时短的线程的key，造成线程安全问题

 

改进：给每一个线程指定一个唯一id

```java
public boolean kill() {
	String id = UUID.randomUUID().toString();
	boolean flag = false;
	String lock = "key";
		
	try {
		boolean result = redisTemplate.opsForValue()
.setIfAbsent(lock, id, 10, TimeUnit.SECONDS);
		if (!result) {
			//如果拿不到，直接返回
			return flag;
		}
		int num = Integer.parseInt(redisTemplate.opsForValue().get("num"));
		if (num>0) {
			num--;
			System.out.println("删减库存成功,剩余库存:"+num);
			//设置回去
			redisTemplate.opsForValue().set("num", num+"");
			flag = true;
		}
	} finally{
		//先获取进行判断
		if (id.equals(redisTemplate.opsForValue().get(lock))) {
			//删除key
			redisTemplate.delete(lock);
		}
	}
	return flag;
}
```

缺陷：

过期时间固定，不太合适，因为可能存在业务超过过期时间的情况（不够时需要续命）

加锁的机器宕机，其他机器只能等待key过期才能执行（降低效率）

 

改进：在获取key时同时开启一个子线程，子线程的作用是每个一段时间查看一下当前线程是否执行完，并且判断是否需要进行续命，这种方式就可以有效的解决上面提到的缺陷，但是要完成相关的代码编写是非常麻烦的，而且很容易出现各种各样的问题，这个时候我们需要Redisson来解决该问题，redisson中帮我们实现了相关的逻辑。



#### Redisson

Redisson官网：

https://redisson.org

 

Redisson在基于NIO的Netty框架上，充分的利用了Redis键值数据库提供的一系列优势，在Java实用工具包中常用接口的基础上，为使用者提供了一系列具有分布式特性的常用工具类。使得原本作为协调单机多线程并发程序的工具包获得了协调分布式多机多线程并发系统的能力，大大降低了设计和研发大规模分布式系统的难度。同时结合各富特色的分布式服务，更进一步简化了分布式环境中程序相互之间的协作。

 

适用场景

分布式应用，缓存，分布式会话，分布式任务/服务/延迟执行服务，Redis客户端

 

在pom.xml中引入redisson依赖

```xml
<!-- redisson -->
<dependency>
	<groupId>org.redisson</groupId> 
	<artifactId>redisson</artifactId>
	<version>3.12.5</version>
</dependency>
```

编写RedissonConfiguration配置类

```java
@Configuration
public class RedissonConfiguration {
    @Bean
    public Redisson redisson(){
        Config config = new Config();
        config.useSingleServer().setAddress("redis://127.0.0.1:6379").setDatabase(0);
        return (Redisson)Redisson.create(config);
    }
}
```

修改controller代码

```java
public class RedisController {
	@Autowired
	private RedisTemplate<String, String> redisTemplate;
	
	@Autowired
	private Redisson redisson;
	
	@GetMapping("/kill")
	public boolean kill() {
		boolean flag = false;
		String lock = "key";
		RLock rLock = redisson.getLock(lock);
		try {
			rLock.lock();
			int num = Integer.parseInt(redisTemplate.opsForValue().get("num"));
			if (num>0) {
				num--;
				System.out.println("删减库存成功,剩余库存:"+num);
				//设置回去
				redisTemplate.opsForValue().set("num", num+"");
				//
				flag = true;
			}
		} finally{
			rLock.unlock();
		}
		return flag;
	}
}
```



### 微服务概述（低 30）

单体应用到分布式，微服务的演变,微服务架构的定义及优缺点



#### 大型网络架构变迁

##### 单体架构

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210802094516.png" alt="image-20210802094515955" style="zoom:50%;" />

集群解决单服务器负载问题

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210801172038.png" alt="image-20210801172038458" style="zoom:50%;" />

##### SOA

面向服务的架构（SOA）是一个组件模型，它将应用程序的不同功能单元（称为服务）进行拆分，并通过这些服务之间定义良好的接口和协议联系起来。一个服务可能负责几个功能

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210801174815.png" alt="image-20210801174815174" style="zoom: 50%;" />

SOA 设计原则 
    在 SOA 架构中，继承了来自对象和构件设计的各种原则，例如，封装和自我包含等。那些保证服务的灵活性、松散耦合和复用能力的设计原则，对 SOA 架构来说同样是非常重要的。关于服务，一些常见的设计原则如下：

- 明确定义的接口。服务请求者依赖于服务规约来调用服务，因此，服务定义必须长时间稳定，一旦公布，不能随意更改；服务的定义应尽可能明确，减少请求者的不适当使用；不要让请求者看到服务内部的私有数据。

- 自包含和模块化。服务封装了那些在业务上稳定、重复出现的活动和构件，实现服务的功能实体是完全独立自主的，独立进行部署、版本控制、自我管理和恢复。

- 粗粒度。服务数量不应该太多，依靠消息交互而不是远程过程调用，通常消息量比较大，但是服务之间的交互频度较低。

- 松耦合。服务请求者可见的是服务的接口，其位置、实现技术、当前状态和私有数据等，对服务请求者而言是不可见的。

- 互操作性、兼容和策略声明。为了确保服务规约的全面和明确，策略成为一个越来越重要的方面。这可以是技术相关的内容，例如，一个服务对安全性方面的要求；也可以是与业务有关的语义方面的内容，例如，需要满足的费用或者服务级别方面的要求，这些策略对于服务在交互时是非常重要的。

但是这种集成方式开发代价大、通信效率低，且有单点故障的风险， 实际上在企业中并没有得到大规模应用。



##### 微服务

微服务顾名思义，就是很小的服务，所以它属于面向服务架构的一种。通俗一点来说，微服务类似于古代著名的发明，活字印刷术，每个服务都是一个组件，通过编排组合的方式来使用，从而真正做到了独立、解耦、组件化、易维护、可复用、可替换、高可用、最终达到提高交付质量、缩短交付周期的效果。

从专业的角度来看，微服务架构是一种架构模式，它提倡将单一应用程序划分成一组小的服务，服务之间互相协调、互相配合，为用户提供最终价值。每个服务运行在其独立的进程中，服务与服务间采用轻量级的通信机制互相沟通（通常是基于 HTTP  协议的 RESTful API）。每个服务都围绕着具体业务进行构建，并且能够被独立的部署到生产环境、类生产环境等。另外，应当尽量避免统一的、集中式的服务管理机制，对具体的一个服务而言，应根据业务上下文，选择合适的语言、工具对其进行构建。

所以总结起来，微服务的核心特点为：小, 且专注于做⼀件事情、轻量级的通信机制、松耦合、独立部署。



与SOA的主要区别

| 微服务                       | SOA                                       |
| ---------------------------- | ----------------------------------------- |
| 能拆分的就拆分               | 是整体的，服务能放一起的都放一起          |
| 业务逻辑存在于每一个服务中   | 业务逻辑横跨多个业务领域                  |
| 使用轻量级的通讯方式，如HTTP | 企业服务产总线(ESB)充当服务之间通讯的角色 |
| 细粒度                       | 粗粒度                                    |



微服务由SOA的发展而来。



#### 什么是分布式系统？

要理解分布式系统，主要需要明白一下2个方面：

1.分布式系统一定是由多个节点组成的系统。

其中，节点指的是计算机服务器，而且这些节点一般不是孤立的，而是互通的。

 

2.这些连通的节点上部署了我们的节点，并且相互的操作会有协同。

分布式系统对于用户而言，他们面对的就是一个服务器，提供用户需要的服务而已，而实际上这些服务是通过背后的众多服务器组成的一个分布式系统，因此分布式系统看起来像是一个超级计算机一样。

 

例如淘宝，平时大家都会使用，它本身就是一个分布式系统，我们通过浏览器访问淘宝网站时，这个请求的背后就是一个庞大的分布式系统在为我们提供服务，整个系统中有的负责请求处理，有的负责存储，有的负责计算，最终他们相互协调把最后的结果返回并呈现给用户。

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210708093744.png" alt="image-20210708093744032" style="zoom: 50%;" />

#### 分布式服务框架的发展

##### 第一代服务框架

　　代表：Dubbo(Java)、Orleans(.Net)等

　　特点：和语言绑定非常紧密

##### 第二代服务框架

　　代表：Spring Cloud

　　特点：适合混合式开发，非常成熟，市场占有率高

##### 第三代服务框架

　　代表：Service Mesh（服务网格），例如Service Fabric、lstio、Linkerd、Conduit等

　　特点：快速发展，更新迭代比较快不够成熟

 

Spring Cloud作为第二代微服务的代表性框架，已经在国内众多大中小型的公司有实际应用案例。许多公司的业务线全部拥抱Spring Cloud，部分公司选择部分拥抱Spring Cloud。

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210708093823.png" alt="image-20210708093823557" style="zoom:50%;" />

#### 微服务介绍

##### 什么是微服务

在介绍微服务时，首先得先理解什么是微服务，顾名思义，微服务得从两个方面去理解，什么是"微"、什么是"服务"，

 

微，狭义来讲就是体积小、著名的"2 pizza 团队"很好的诠释了这一解释（2 pizza 团队最早是亚马逊 CEO Bezos提出来的，意思是说单个服务的设计，所有参与人从设计、开发、测试、运维所有人加起来 只需要2个披萨就够了 ）。 而所谓服务，一定要区别于系统，服务一个或者一组相对较小且独立的功能单元，是用户可以感知最小功能集。

 

##### 微服务由来

微服务最早由Martin Fowler与James Lewis于2014年共同提出，微服务架构风格是一种使用一套小服务来开发单个应用的方式途径，每个服务运行在自己的进程中，并使用轻量级机制通信，通常是HTTP API，这些服务基于业务能力构建，并能够通过自动化部署机制来独立部署，这些服务使用不同的编程语言实现，以及不同数据存储技术，并保持最低限度的集中式管理。

 

##### 为什么需要微服务？

在传统的IT行业软件大多都是各种独立系统的堆砌，这些系统的问题总结来说就是扩展性差，可靠性不高，维护成本高。到后面引入了SOA服务化，但是，由于 SOA 早期均使用了总线模式，这种总线模式是与某种技术栈强绑定的，比如：J2EE。这导致很多企业的遗留系统很难对接，切换时间太长，成本太高，新系统稳定性的收敛也需要一些时间。最终 SOA 看起来很美，但却成为了企业级奢侈品，中小公司都望而生畏。



#### 早期的单体架构带来的问题

单体架构在规模比较小的情况下工作情况良好，但是随着系统规模的扩大，它暴露出来的问题也越来越多，主要有以下几点：

##### 复杂性逐渐变高

比如有的项目有几十万行代码，各个模块之间区别比较模糊，逻辑比较混乱，代码越多复杂性越高，越难解决遇到的问题。

 

##### 技术债务逐渐上升

公司的人员流动是再正常不过的事情，有的员工在离职之前，疏于代码质量的自我管束，导致留下来很多坑，由于单体项目代码量庞大的惊人，留下的坑很难被发觉，这就给新来的员工带来很大的烦恼，人员流动越大所留下的坑越多，也就是所谓的技术债务越来越多。

 

##### 部署速度逐渐变慢

这个就很好理解了，单体架构模块非常多，代码量非常庞大，导致部署项目所花费的时间越来越多，曾经有的项目启动就要一二十分钟，这是多么恐怖的事情啊，启动几次项目一天的时间就过去了，留给开发者开发的时间就非常少了。

 

##### 阻碍技术创新

比如以前的某个项目使用struts2写的，由于各个模块之间有着千丝万缕的联系，代码量大，逻辑不够清楚，如果现在想用spring mvc来重构这个项目将是非常困难的，付出的成本将非常大，所以更多的时候公司不得不硬着头皮继续使用老的struts架构，这就阻碍了技术的创新。

 

##### 无法按需伸缩

比如说电影模块是CPU密集型的模块，而订单模块是IO密集型的模块，假如我们要提升订单模块的性能，比如加大内存、增加硬盘，但是由于所有的模块都在一个架构下，因此我们在扩展订单模块的性能时不得不考虑其它模块的因素，因为我们不能因为扩展某个模块的性能而损害其它模块的性能，从而无法按需进行伸缩。

 

#### 微服务与单体架构区别

单体架构所有的模块全都耦合在一块，代码量大，维护困难，微服务每个模块就相当于一个单独的项目，代码量明显减少，遇到问题也相对来说比较好解决。

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210708093948.png" alt="image-20210708093948274" style="zoom:50%;" />

单体架构所有的模块都共用一个数据库，存储方式比较单一，微服务每个模块都可以使用不同的存储方式（比如有的用redis，有的用mysql等），数据库也是单个模块对应自己的数据库。

 

单体架构所有的模块开发所使用的技术一样，微服务每个模块都可以使用不同的开发技术，开发模式更灵活。

 

#### 微服务的本质

微服务，关键其实不仅仅是微服务本身，而是系统要提供一套基础的架构，这种架构使得微服务可以独立的部署、运行、升级，不仅如此，这个系统架构还让微服务与微服务之间在结构上“松耦合”，而在功能上则表现为一个统一的整体。这种所谓的“统一的整体”表现出来的是统一风格的界面，统一的权限管理，统一的安全策略，统一的上线过程，统一的日志和审计方法，统一的调度方式，统一的访问入口等等。

微服务的目的是有效的拆分应用，实现敏捷开发和部署 。

微服务提倡的理念团队间应该是 inter-operate, not integrate 。inter-operate是定义好系统的边界和接口，在一个团队内全栈，让团队自治，原因就是因为如果团队按照这样的方式组建，将沟通的成本维持在系统内部，每个子系统就会更加内聚，彼此的依赖耦合能变弱，跨系统的沟通成本也就能降低。

 

#### 什么样的项目适合微服务

微服务可以按照业务功能本身的独立性来划分，如果系统提供的业务是非常底层的，如：操作系统内核、存储系统、网络系统、数据库系统等等，这类系统都偏底层，功能和功能之间有着紧密的配合关系，如果强制拆分为较小的服务单元，会让集成工作量急剧上升，并且这种人为的切割无法带来业务上的真正的隔离，所以无法做到独立部署和运行，也就不适合做成微服务了。

 

能不能做成微服务，取决于四个要素：

小：微服务体积小，2 pizza 团队。

独：能够独立的部署和运行。

轻：使用轻量级的通信机制和架构。

松：为服务之间是松耦合的。

 

#### 微服务折分与设计

从单体式结构转向微服务架构中会持续碰到服务边界划分的问题：比如，我们有user 服务来提供用户的基础信息，那么用户的头像和图片等是应该单独划分为一个新的service更好还是应该合并到user服务里呢？如果服务的粒度划分的过粗，那就回到了单体式的老路；如果过细，那服务间调用的开销就变得不可忽视了，管理难度也会指数级增加。目前为止还没有一个可以称之为服务边界划分的标准，只能根据不同的业务系统加以调节。

 

拆分的大原则是当一块业务不依赖或极少依赖其它服务，有独立的业务语义，为超过2个的其他服务或客户端提供数据，那么它就应该被拆分成一个独立的服务模块。

 

#### 微服务设计原则

##### 单一职责原则

意思是每个微服务只需要实现自己的业务逻辑就可以了，比如订单管理模块，它只需要处理订单的业务逻辑就可以了，其它的不必考虑。

 

##### 服务自治原则

意思是每个微服务从开发、测试、运维等都是独立的，包括存储的数据库也都是独立的，自己就有一套完整的流程，我们完全可以把它当成一个项目来对待。不必依赖于其它模块。

 

##### 轻量级通信原则

首先是通信的语言非常的轻量，第二，该通信方式需要是跨语言、跨平台的，之所以要跨平台、跨语言就是为了让每个微服务都有足够的独立性，可以不受技术的钳制。

 

##### 接口明确原则

由于微服务之间可能存在着调用关系，为了尽量避免以后由于某个微服务的接口变化而导致其它微服务都做调整，在设计之初就要考虑到所有情况，让接口尽量做的更通用，更灵活，从而尽量避免其它模块也做调整。

 

#### 微服务优势与缺点

##### 特性

- 每个微服务可独立运行在自己的进程里；

- 一系列独立运行的微服务共同构建起了整个系统；

- 每个服务为独立的业务开发，一个微服务一般完成某个特定的功能，比如：订单管理，用户管理等；

- 微服务之间通过一些轻量级的通信机制进行通信，例如通过REST API或者RPC的方式进行调用。

 

##### 特点

- 易于开发和维护

由于微服务单个模块就相当于一个项目，开发这个模块我们就只需关心这个模块的逻辑即可，代码量和逻辑复杂度都会降低，从而易于开发和维护。

 

- 启动较快

这是相对单个微服务来讲的，相比于启动单体架构的整个项目，启动某个模块的服务速度明显是要快很多的。

 

- 局部修改容易部署

在开发中发现了一个问题，如果是单体架构的话，我们就需要重新发布并启动整个项目，非常耗时间，但是微服务则不同，哪个模块出现了bug我们只需要解决那个模块的bug就可以了，解决完bug之后，我们只需要重启这个模块的服务即可，部署相对简单，不必重启整个项目从而大大节约时间。

 

- 技术栈不受限

比如订单微服务和电影微服务原来都是用java写的，现在我们想把电影微服务改成nodeJs技术，这是完全可以的，而且由于所关注的只是电影的逻辑而已，因此技术更换的成本也就会少很多。

 

- 按需伸缩

上面说了单体架构在想扩展某个模块的性能时不得不考虑到其它模块的性能会不会受影响，对于我们微服务来讲，完全不是问题，电影模块通过什么方式来提升性能不必考虑其它模块的情况。

 

##### 缺点

- 运维要求较高

对于单体架构来讲，我们只需要维护好这一个项目就可以了，但是对于微服务架构来讲，由于项目是由多个微服务构成的，每个模块出现问题都会造成整个项目运行出现异常，想要知道是哪个模块造成的问题往往是不容易的，因为我们无法一步一步通过debug的方式来跟踪，这就对运维人员提出了很高的要求。

 

- 分布式的复杂性

对于单体架构来讲，我们可以不使用分布式，但是对于微服务架构来说，分布式几乎是必会用的技术，由于分布式本身的复杂性，导致微服务架构也变得复杂起来。

 

- 接口调整成本高

比如，用户微服务是要被订单微服务和电影微服务所调用的，一旦用户微服务的接口发生大的变动，那么所有依赖它的微服务都要做相应的调整，由于微服务可能非常多，那么调整接口所造成的成本将会明显提高。

 

- 重复劳动

对于单体架构来讲，如果某段业务被多个模块所共同使用，我们便可以抽象成一个工具类，被所有模块直接调用，但是微服务却无法这样做，因为这个微服务的工具类是不能被其它微服务所直接调用的，从而我们便不得不在每个微服

 

#### 分布式和微服务的区别

<font color='red'>简单的说，微服务是架构设计方式，分布式是系统部署方式，两者概念不同</font>



### SpringCloud介绍（中 20）

#### SpringCloud整体架构概览

子项目简介

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210708094138.png" alt="image-20210708094138045" style="zoom:50%;" />

Spring Cloud为开发人员提供了快速构建分布式系统中一些常见模式的工具（例如配置管理，服务发现，断路器，智能路由，微代理，控制总线，一次性令牌，全局锁，领导选举，分布式会话，集群状态）。分布式系统的协调导致了样板模式, 使用Spring Cloud开发人员可以快速地支持实现这些模式的服务和应用程序。他们将在任何分布式环境中运行良好，包括开发人员自己的笔记本电脑，裸机数据中心，以及Cloud Foundry等托管平台。

 

##### 注意：

首先，尽管Spring Cloud带有“Cloud”这个单词，但它并不是云计算解决方案，而是在Spring Boot基础之上构建的，用于快速构建分布式系统的通用模式的工具集。

 

其次，使用Spring Cloud开发的应用程序非常适合在Docker和PaaS（比如Pivotal Cloud Foundry）上部署，所以又叫做云原生应用（Cloud Native Application）。云原生可以简单地理解为面向云环境的软件架构。

 

##### 特征

- Spring Cloud专注于提供良好的开箱即用经验的典型用例和可扩展性机制覆盖。

- 分布式/版本化配置

- 服务注册和发现

- 路由

- service - to - service调用

- 负载均衡

- 断路器

- 全局锁

- Leadership选举与集群状态

- 分布式消息传递

 

#### Spring Cloud与Spring Boot的关系

Spring Boot用来开发项目

Spring Cloud用来管理项目，Spring Cloud管理的项目需要基于Spring Boot来开发

 

#### 版本对应关系

| Spring Cloud             | Spring Boot                                    |
| ------------------------ | ---------------------------------------------- |
| Angel版本                | 兼容Spring Boot 1.2.x                          |
| Brixton版本              | 兼容Spring Boot 1.3.x，也兼容Spring Boot 1.4.x |
| Camden版本               | 兼容Spring Boot 1.4.x，也兼容Spring Boot 1.5.x |
| Dalston版本、Edgware版本 | 兼容Spring Boot 1.5.x，不兼容Spring Boot 2.0.x |
| Finchley版本             | 兼容Spring Boot 2.0.x，不兼容Spring Boot 1.5.x |
| Greenwich版本            | 兼容Spring Boot 2.1.x                          |
| Hoxton版                 | 兼容Spring Boot 2.2.x                          |

在实际开发过程中，我们需要更详细的版本对应：

| Spring Boot   | Spring Cloud            |
| ------------- | ----------------------- |
| 1.5.2.RELEASE | Dalston.RC1             |
| 1.5.9.RELEASE | Edgware.RELEASE         |
| 2.0.2.RELEASE | Finchley.BUILD-SNAPSHOT |
| 2.0.3.RELEASE | Finchley.RELEASE        |
| 2.1.0.RELEASE | Greenwich.SR1           |
| 2.2.0.M4      | Hoxton.SR9              |
| 2.3.7         | Hoxton.BUILD-SNAPSHOT   |
| 2.4.0.M1      | 2020.0.0-M3             |





















