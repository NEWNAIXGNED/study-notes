### SpringBoot整合RabbitMQ开发（高 45）

#### 导入依赖

```xml
<!-- rabbid mq -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
<!-- rabbit test -->
<dependency>
	<groupId>org.springframework.amqp</groupId> 
	<artifactId>spring-rabbit-test</artifactId>
	<scope>test</scope>
</dependency>
<!-- lombok -->
<dependency>
	<groupId>org.projectlombok</groupId>
	<artifactId>lombok</artifactId>
	<optional>true</optional>
</dependency>
<!-- 热部署 -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-devtools</artifactId>
	<optional>true</optional>
	<scope>true</scope>
</dependency>
```

#### 编辑application.yml文件

```yaml
spring:
  rabbitmq:
    host: 192.168.41.226
    port: 5672    # java程序访问时用到的端口
    username: admin   #账号
    password: admin   #密码
    virtual-host: /test    #虚拟主机
    publisher-confirm-type: correlated   #开启RabbitMQ的生产者确认模式
    publisher-returns: true    #返回确认消息
    template:
      mandatory: true  #启用强制信息，必须确认，消息无法达到队列时将消息返回给生产者，否则丢弃消息
```

#### 编写配置类

```java
package com.woniuxy.main.configuration;

import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RabbitConfiguration {
	@Bean
	public Queue helloQueue() {
		return new Queue("hello");  //队列名
	}
}
```



#### 直接模式

直接向消息队列发送消息，包括simple、work模式

#### 生产者

在direct包中创建SingletonHello类发送消息，在SpringBoot中直接使用AmqpTemplate去发送消息，代码如下:

```java
package com.woniuxy.main.direct;

import org.springframework.amqp.core.AmqpTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class SingletonHello {
    
    @Autowired
    private AmqpTemplate amqpTemplate;
    
    public void send() {
        String sendMsg = "hello rabbit";
        System.out.println("sender :" + sendMsg);
        amqpTemplate.convertAndSend("hello", sendMsg);
    }
}
```

##### 消费者

在direct文件夹中创建SingletonHelloReceiver类来接受消息，代码如下所示：

```java
package com.woniuxy.main.direct;

import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
@RabbitListener(queues =  "hello")
public class SingletonHelloReceiver {
    
    @RabbitHandler
    public void receiver(String hello) {
        System.out.println("SingletonHelloReceiver" + hello);
    }
}
```

##### 测试类

验证测试代码，在controller文件夹中创建RabbitTestController类来测试

```java
package com.woniuxy.main.handler;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;

import com.woniuxy.main.direct.SingletonHello;

@Controller
@RequestMapping("/rabbit")
public class RabbitTestHandler {
	@Autowired
    private SingletonHello singletonHello;
     
    @GetMapping("/hello")
    public void hello () {
        singletonHello.send();
    }
}
```

##### 测试

运行程序在浏览器地址栏中输入：http://localhost:8080/rabbit/hello 验证，如果在控制台上输出以下信息表示成功

```
sender :hello rabbit
SingletonHelloReceiver:hello rabbit
```

#### 分类

- 一对一：一个生产者一个消费者

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210729103559.png" alt="image-20210729103559413" style="zoom:50%;" />

- 一对多：一个生产者多个消费者

  再新建一个消费者，同样监听

  注意：当其中一个消费获取到数据之后，消息队列会将对应的消息删除，其他消费者就无法获取到该消息

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210729103658.png" alt="image-20210729103658713" style="zoom:50%;" />

- 多对多：多个生产者多个消费者

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210729103647.png" alt="image-20210729103647507" style="zoom:50%;" />

#### Fanout模式（分发模式）

- 任何发送到Fanout Exchange上的消息都会转发到与Exchange绑定（binding）的queue上。

- 可以理解为路由表的模式

- 这种模式需要提前将Exchange与Queue进行绑定，一个Exchange可以绑定多个Queue，一个Queue可以同多个Exchange进行绑定。

- 如果接受到消息的Exchange没有与任何Queue绑定，则消息会被抛弃。

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210729103739.png" alt="image-20210729103739683" style="zoom:50%;" />

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210729103750.png" alt="image-20210729103749931" style="zoom:50%;" />

##### 配置类

在RabbitConfiguration配置类中添加如下配置

```java
/*
 * fanout 模式
 */
@Bean
public Queue fanoutMessageA() {
    return new Queue("fanout.A");
}

@Bean
public Queue fanoutMessageB() {
    return new Queue("fanout.B");
}

@Bean
public Queue fanoutMessageC() {
    return new Queue("fanout.C");
}

@Bean
public FanoutExchange fanoutExchange() {
    return new FanoutExchange("fanoutExchange");
}
@Bean
public Binding bindingExchangeA(Queue fanoutMessageA,FanoutExchange fanoutExchange) {
    return BindingBuilder.bind(fanoutMessageA).to(fanoutExchange);
}

@Bean
public Binding bindingExchangeB(Queue fanoutMessageB,FanoutExchange fanoutExchange) {
    return BindingBuilder.bind(fanoutMessageB).to(fanoutExchange);
}

@Bean
public Binding bindingExchangeC(Queue fanoutMessageC,FanoutExchange fanoutExchange) {
    return BindingBuilder.bind(fanoutMessageC).to(fanoutExchange);
}
```

##### 生产者

创建一个fanout的包，在包下创建一个生产者和三个消费者

生产者负责向fanout的交换机发送消息，消费者分别负责监听对应的队列，从队列中取出数据编写FanoutSender类，负责发送消息

```java
package com.woniuxy.main.fanout;

import org.springframework.amqp.core.AmqpTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import lombok.Data;

@Data
@Component
public class FanoutSender {
	@Autowired
	private AmqpTemplate amqpTemplate;
	
	//发送消息的方法
	public void send() {
		String message = "fanout message";
		amqpTemplate.convertAndSend("fanoutExchange","",message);//发送消息
	}
}
```

##### 消费者

编写FanoutReceiveA、B、C三个消费者类，代码基本一致

```java
package com.woniuxy.main.fanout;

import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
@RabbitListener(queues = "fanout.A")
public class FanoutReceiveA {
	@RabbitHandler
	public void receive(String message) {
		System.out.println("A:"+message);
	}
}
```

##### 测试类

修改RabbitTestController

```java
@Data
@Controller
@RequestMapping("/rabbit")
public class RabbitTestHandler {
	@Autowired
    private SingletonHello singletonHello;        
	@Autowired
	private FanoutSender fanoutSender;
	
	
    @GetMapping("/hello")
    public void hello () {
        singletonHello.send();
    }
    //fanout模式    
    @GetMapping("/fanout")
    public void fanout() {
		fanoutSender.send();
	}  
}
```

##### 测试

发送请求测试，控制台输入以下结果表示成功!

```
A:fanout message
B:fanout message
C:fanout message
```



#### Direct（路由模式）