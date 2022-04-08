### 链路追踪简介（中 15）

Sleuth是Spring Cloud的组件之一，它为Spring Cloud实现了一种分布式追踪解决方案，兼容Zipkin，HTrace和其他基于日志的追踪系统，例如 ELK（Elasticsearch 、Logstash、 Kibana）。

 

Seuth主要功能是在分布式系统中提供追踪解决方案，并且兼容支持了Zipkin(提供了链路追踪的可视化功能)利用这些信息，可以可视化地分析服务调用链路和服务间的依赖关系。


#### 相关术语

Sleuth引入了许多 Dapper中的术语：

 

- Span

基本的工作单元。无论是发送一个RPC或是向RPC发送一个响应都是一个Span。每一个Span通过一个64位ID来进行唯一标识，并通过另一个64位ID对Span所在的Trace进行唯一标识。

Span能够启动和停止，他们不断地追踪自身的时间信息，当你创建了一个Span，你必须在未来的某个时刻停止它。

提示：启动一个Trace的初始化Span被叫作 Root Span ，它的 Span ID 和 Trace Id 相同。

 

- Trace

由一系列Span 组成的一个树状结构。例如，如果你要执行一个分布式大数据的存储操作，这个Trace也许会由你的PUT请求来形成。

 

- Annotation：用来及时记录一个事件的存在。通过引入Brave库，我们不用再去设置一系列的特别事件，从而让 Zipkin 能够知道客户端和服务器是谁、请求是从哪里开始的、又到哪里结束。出于学习的目的，还是把这些事件在这里列举一下：

  a）CS （Client Sent） - 客户端发起一个请求，这个注释指示了一个Span的开始。

  b）SR （Server Received） - 服务端接收请求并开始处理它，如果用 SR 时间戳减去CS时间戳便能看出有多少网络延迟。

  c）SS（Server Sent）- 注释请求处理完成(响应已发送给客户端)，如果用 SS 时间戳减去SR 时间戳便可得出服务端处理请求耗费的时间。

  d）CR（Client Received）- 预示了一个Span的结束，客户端成功地接收到了服务端的响应，如果用CR时间戳减去CS时间戳便可得出客户端从服务端获得响应所需耗费的整个时间。

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720212314.png" alt="image-20210720212313914" style="zoom: 50%;" />

Sleuth是对Zipkin的封装，对应Span,Trace等信息的生成、接入HttpRequest,以及向Zipkin Server发送采集信息等全部自动化完成。

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720212620.png" alt="image-20210720212619907" style="zoom: 50%;" />

一次单独的调用链也可以称为一个Span，Dapper记录的是Span的名称，以及每个Span的ID和父ID，以重建在一次追踪过程中不同Span之间的关系，上图中一个矩形框就是一个Span，前端从发出请求到收到回复就是一个Span。

再细化到一个Span的内部，如下图：

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720212402.png" alt="image-20210720212401964" style="zoom:50%;" />



对于一个特定的Span，记录从Start到End，首先经历了客户端发送数据，然后Server接收数据，然后Server执行内部逻辑，这中间可能去访问另一个应用。执行完了Server将数据返回，然后客户端接收到数据。

 

一个Span的内容就能构成Trace上面的一个基本元素，可以在这个Span中埋点打上各种各样的Trace类型，比如，一般将客户端发送记录成依赖（Dependency），服务端接收客户端以及回复给客户端这两个时间统一记录成请求（Request），如果打上这两种，那么在运行完这个Span之后，日志库中就会多出两条日志，一条是Dependency的日志，一条是Request的日志。

 

现在的Trace SDK，都可以进行配置去自动记录一些事件，比如数据库调用依赖，Http调用依赖，记录上游的请求等等，也可以自己手动埋点，在需要打上记录点的地方写上记录的代码即可。

 

Dapper中给出的是一张这样的图

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720212430.png" alt="image-20210720212430783" style="zoom: 50%;" />

首先各个日志收集点按照一定的采样率将日志写进数据文件，然后通过管道将这些日志文件按照一定的TraceId排定输出到BigTable中去。

如果一个系统完成了上面阐述的架构，基本可以构成一个简单的Trace系统。



#### TraceId和ParentId的生成

在整个过程中，TraceId和ParentId的生成至关重要。首先解释下TraceId和ParentId。TraceId是标识这个调用链的Id，整个调用链，从浏览器开始放完，到A到B到C，一直到调用结束，所有应用在这次调用中拥有同一个TraceId，所以才能把这次调用链在一起。

 

既然知道了这次调用链的整个Id，那么每次查找问题的时候，只要知道某一个调用的TraceId，就能把所有这个Id的调用全部查找出来，能够清楚的知道本地调用链经过了哪些应用，产生了哪些调用。但是还缺一点，那就是链。

 

在Java中的LinkedList、Tree等数据结构，可以通过父节点就能够知道子节点，或者通过子节点能够知道父节点是谁（双向链表），那么我们想知道应用A调用了哪些应用，而又有哪些应用调用了应用A，单纯从TraceId里面根本看不出来，必须要指定自己的父节点才行，这就是ParentId的作用。



### 微服务与Zipkin的整合（中 20）

#### 下载安装Zipkin

​		https://dl.bintray.com/openzipkin/maven/io/zipkin/java/zipkin-server/

选择2.12.9版本

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720212714.png" alt="image-20210720212714225" style="zoom: 50%;" />

下载jar包

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720212733.png" alt="image-20210720212733264" style="zoom: 50%;" />

##### 将下载下来的jar包放到任意目录下

打开命令行工具，切换工作路径到jar包所在的位置，输入以下命令运行Zipkin

​	java -jar zipkin-server-2.12.9-exec.jar

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720212751.png" alt="image-20210720212750929" style="zoom: 50%;" />

##### 打开浏览器输入：localhost:9411/zipkin打开Zipkin管理页面

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720212805.png" alt="image-20210720212805200" style="zoom: 50%;" />

##### 在provider中加入依赖

```xml
<!--zipkin-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

##### 在provider的application.yml文件中添加配置

```yaml
spring:
  application:
    name: provider
  zipkin:
    base-url: http://localhost:9411
  sleuth:
    sampler:
      probability: 1
```



##### 在consumer模块的pom.xml中加入依赖

```xml
<!--zipkin-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

##### 在consumer的application.yml文件中添加配置

```yaml
spring:
  application:
    name: consumer
  zipkin:
    base-url: http://localhost:9411
  sleuth:
    sampler:
      probability: 1
```

依次启动redis、zipkin、eureka、provider、consumer



##### 访问consumer的all方法

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720214855.png" alt="image-20210720214855214" style="zoom: 50%;" />

**<font color='red'>刷新Zipkin管理页面</font>**

点击all旁边的下拉按钮，可以看到刚才请求过程中涉及到的微服务，选择comsumer

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720215031.png" alt="image-20210720215030856" style="zoom: 33%;" />

点击”查找”

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720215108.png" alt="image-20210720215108302" style="zoom:33%;" />

可以看到页面下方有一个时间提示，点击

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720215154.png" alt="image-20210720215154709" style="zoom:33%;" />

在新的页面中可以看到刚才请求中的微服务和各微服务消耗的时间

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720215255.png" alt="image-20210720215255622" style="zoom:33%;" />

 

### RabbitMQ概述（中 20）

常用消息队列介绍概述及应用场景，RabbitMQ简介

 <img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720215457.png" alt="image-20210720215456906" style="zoom:33%;" />

#### 什么是MQ

消息队列（Message Queue，简称MQ），从字面意思上看，本质是个队列，FIFO先入先出，只不过队列中存放的内容是message而已。

其主要用途：不同进程Process/线程Thread之间通信。

 

为什么会产生消息队列？有几个原因：

- 不同进程（process）之间传递消息时，两个进程之间耦合程度过高，改动一个进程，引发必须修改另一个进程，为了隔离这两个进程，在两进程间抽离出一层（一个模块），所有两进程之间传递的消息，都必须通过消息队列来传递，单独修改某一个进程，不会影响另一个；

 

- 不同进程（process）之间传递消息时，为了实现标准化，将消息的格式规范化了，并且，某一个进程接受的消息太多，一下子无法处理完，并且也有先后顺序，必须对收到的消息进行排队，因此诞生了事实上的消息队列；

 

#### RabbitMQ介绍

RabbitMQ是由Erlang语言开发的AMQP的开源实现。

 

AMQP：Advanced Message Queue，高级消息队列协议。它是应用层协议的一个开放标准，为面向消息的中间件设计，基于此协议的客户端与消息中间件可传递消息，并不受产品、开发语言等条件的限制。

 

#### RabbitMQ特点

- 可靠性(Reliablity)

  使用了一些机制来保证可靠性，比如持久化、传输确认、发布确认。

- 灵活的路由(Flexible Routing)

  在消息进入队列之前，通过Exchange来路由消息。对于典型的路由功能，Rabbit已经提供了一些内置的Exchange来实现。针对更复杂的路由功能，可以将多个Exchange绑定在一起，也通过插件机制实现自己的Exchange。

- 消息集群(Clustering)

  多个RabbitMQ服务器可以组成一个集群，形成一个逻辑Broker。

- 高可用(Highly Avaliable Queues)

  队列可以在集群中的机器上进行镜像，使得在部分节点出问题的情况下队列仍然可用。

- 多种协议(Multi-protocol)

  支持多种消息队列协议，如STOMP、MQTT等。

- 多种语言客户端(Many Clients)

  几乎支持所有常用语言，比如Java、.NET、Ruby等。

- 管理界面(Management UI)

  提供了易用的用户界面，使得用户可以监控和管理消息Broker的许多方面。

- 跟踪机制(Tracing)

  如果消息异常，RabbitMQ提供了消息的跟踪机制，使用者可以找出发生了什么。

- 插件机制(Plugin System)

  提供了许多插件，来从多方面进行扩展，也可以编辑自己的插件。

#### 应用场景

##### 1、异步处理

场景说明：用户注册后，需要发注册邮件和注册短信,传统的做法有两种：串行方式、并行方式

- 串行方式:将注册信息写入数据库后,发送注册邮件,再发送注册短信,以上三个任务全部完成后才返回给客户端。 这有一个问题是,邮件,短信并不是必须的,它只是一个通知,而这种做法让客户端等待没有必要等待的东西.

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720215709.png" alt="image-20210720215709121" style="zoom:50%;" />

- 并行方式:将注册信息写入数据库后,发送邮件的同时,发送短信,以上三个任务完成后,返回给客户端,并行的方式能提高处理的时间。

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720215737.png" alt="image-20210720215737666" style="zoom:50%;" />

假设三个业务节点分别使用50ms,串行方式使用时间150ms,并行使用时间100ms。虽然并性已经提高的处理时间,但是,前面说过,邮件和短信对我正常的使用网站没有任何影响，客户端没有必要等着其发送完成才显示注册成功,英爱是写入数据库后就返回。



- 消息队列 

  引入消息队列后，把发送邮件,短信不是必须的业务逻辑异步处理

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720215814.png" alt="image-20210720215814677" style="zoom:50%;" />

由此可以看出,引入消息队列后，用户的响应时间就等于写入数据库的时间+写入消息队列的时间(可以忽略不计),引入消息队列后处理后,响应时间是串行的3倍,是并行的2倍。



##### 2、应用解耦

场景：双11是购物狂节,用户下单后,订单系统需要通知库存系统,

传统的做法就是订单系统调用库存系统的接口

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720215900.png" alt="image-20210720215900697" style="zoom:50%;" />

这种做法有一个缺点:

当库存系统出现故障时,订单就会失败。

订单系统和库存系统高耦合。

 

引入消息队列 

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720215915.png" alt="image-20210720215915705" style="zoom:50%;" />

订单系统:用户下单后,订单系统完成持久化处理,将消息写入消息队列,返回用户订单下单成功。

库存系统:订阅下单的消息,获取下单消息,进行库操作。 

就算库存系统出现故障,消息队列也能保证消息的可靠投递,不会导致消息丢失



##### 3、流量削峰

流量削峰一般在秒杀活动中应用广泛 

场景:秒杀活动，一般会因为流量过大，导致应用挂掉,为了解决这个问题，一般在应用前端加入消息队列。 

作用: 

- 可以控制活动人数，超过此一定阀值的订单直接丢弃 

- 可以缓解短时间的高流量压垮应用(应用程序按自己的最大处理能力获取订单)

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720215941.png" alt="image-20210720215941237" style="zoom:50%;" />

处理过程

- 用户的请求,服务器收到之后,首先写入消息队列,加入消息队列长度超过最大值,则直接抛弃用户请求或跳转到错误页面. 

- 秒杀业务根据消息队列中的请求信息，再做后续处理.



##### 4、消息驱动的系统



#### Rabbit体系架构

系统分为消息队列、消息生产者、消息消费者，生产者负责产生消息，消费者负责对消息进行处理

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720220041.png" alt="image-20210720220041493" style="zoom:67%;" />



### 安装配置（低 20）

RabbitMQ安装、启动、停止服务、启用管理控制台

将以下几个rpm文件上传到centos上，放到自己的一个文件夹中

​	erlang-20.1.7-1.el7.centos.x86_64.rpm

​	rabbitmq-server-3.7.0-1.el7.noarch.rpm

​	socat-1.7.3.2-2.el7.x86_64.rpm



输入**<font color='red'>yum install *.rpm</font>**命令进行安装

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720220129.png" alt="image-20210720220129541" style="zoom:67%;" />

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720220139.png" alt="image-20210720220139303" style="zoom:67%;" />

启动rabbit mq

执行**<font color='red'>sudo service rabbitmq-server start</font>**

提示如图所示则表示启动成功

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720220154.png" alt="image-20210720220154184" style="zoom:67%;" />

配置rabbitmq管理账户。

执行命令 **<font color='red'>rabbitmqctl add_user admin admin</font>**，设置账户密码为admin admin

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720220211.png" alt="image-20210720220211182" style="zoom:67%;" />

执行命令 **<font color='red'>rabbitmqctl set_user_tags admin administrator</font>**，设置admin为管理员权限

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720220224.png" alt="image-20210720220224179" style="zoom:67%;" />

执行命令**<font color='red'> rabbitmq-plugins enable rabbitmq_management</font>**，打开rabbitmq web管理

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720220240.png" alt="image-20210720220240197" style="zoom:67%;" />

开放端口

firewall-cmd --zone=public --add-port=5672/tcp --permanent
firewall-cmd --zone=public --add-port=15672/tcp --permanent
sudo service firewalld restart



测试rabbit mq是否可以，打开浏览器在地址栏中输入：http://虚拟机ip:15672,出现登录页面，登陆账户密码为设置的admin admin

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720220254.png" alt="image-20210720220254230" style="zoom:67%;" />

查看用户权限,默认状态下权限是不允许访问(此时程序访问5672端口是连接被拒绝)

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720220306.png" alt="image-20210720220306204" style="zoom:67%;" />

点击用户名，进入用户页面，直接点击设置权限。此时刷新页面回到Users页面，权限变成可访问

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720220324.png" alt="image-20210720220324701" style="zoom:67%;" />

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720220333.png" alt="image-20210720220333577" style="zoom:67%;" />

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720220339.png" alt="image-20210720220339572" style="zoom: 67%;" />

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720220333.png" alt="image-20210720220333577" style="zoom:67%;" />

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720220339.png" alt="image-20210720220339572" style="zoom:67%;" />



#### 常见问题

 http://ip:15672不能访问，确认两点：

1、 添加用户、给用户设置管理员权限、rabbitmq-plugins这三步是否执行成功。

2、使用firewall打开5672/15672端口。具体步骤如下：

```
firewall-cmd --zone=public --add-port=5672/tcp --permanent
firewall-cmd --zone=public --add-port=15672/tcp --permanent
sudo service firewalld restart（如果系统不要求开启防火墙，可以在设置完以后再关闭它）
备注：即使防火墙处于关闭状态，也应该先打开端口再关闭，否则在有些机器上会仍然端口不通。
```



### 相关概念及常用命令（低 20）

#### 常见概念

##### 生产者

​	创建消息，发布到代理服务器（Message Broker）。

##### 消费者

​	连接到代理服务器，并订阅到队列（queue）上，代理服务器将发送消息给一个订阅的/监听的消费者

##### 队列

​	队列结构，先进先出（FIFO）

##### 虚拟主机

​	虚拟机概念是RabbitMQ的核心,在用户未自定义虚拟机前已经内置有虚拟机,在使用RabbitMQ中,可以进行自定义配置虚拟机。一个虚拟机中可以含有多个队列信息

​	虚拟机最大的好处在于可以根据不同的用户分配不同的操作空间

**创建虚拟主机**

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210810201848.png" alt="image-20210810201847824" style="zoom: 33%;" />

指定主机名，点击添加

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210810202050.png" alt="image-20210810202050237" style="zoom:33%;" />

指定能够访问虚拟主机的用户（默认guest）

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210810202337.png" alt="image-20210810202337442" style="zoom:33%;" />

删除不需要的用户

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210810202538.png" alt="image-20210810202537944" style="zoom:33%;" />

指定新用户

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210810202607.png" alt="image-20210810202607698" style="zoom:33%;" />

得到如下结果表明成功

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210810202633.png" alt="image-20210810202632788" style="zoom:33%;" />


### Java客户端访问RabbitMQ（低 30）

#### 创建springboot项目

<img src="C:\Users\Administrator\Desktop\20210721153235.png" alt="image-20210721153235607" style="zoom: 33%;" />

导入依赖

```xml
<!--rabbit mq-->
<dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>5.1.1</version>
</dependency>
```

### Simple简单模式（中 20）

#### RabbitMQ工作模式的介绍

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210721154205.png" alt="image-20210721154205876" style="zoom: 67%;" />

- 消息产生者将消息放入队列

- 消息的消费者(consumer) 监听(while) 消息队列,如果队列中有消息,就消费掉,消息被拿走后,自动从队列中删除(隐患 消息可能没有被消费者正确处理,已经从队列中消失了,造成消息的丢失)应用场景:聊天(中间有一个过度的服务器;p端,c端)

#### 编写生产者代码

```java
package com.woniuxy.rabbitteach.simple;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

public class Productor {
    public static void main(String[] args) throws IOException, TimeoutException {
        //1.创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        //2.设置参数
        factory.setHost("192.168.74.146");
        factory.setPort(5672);
        factory.setUsername("admin");
        factory.setPassword("admin");
        factory.setVirtualHost("/test");  //设置虚拟主机   默认/
        //3.创建连接
        Connection connection = factory.newConnection();
        //4.创建通道
        Channel channel = connection.createChannel();
        //5.创建消息队列
        //参数1：队列名
        //参数2：是否持久化
        //参数3：是否排外，一个队列只允许一个消费者连接
        //参数4：是否自动删除
        //参数5：初始化参数

        channel.queueDeclare("test",false,false,false,null);
        //6.发送消息
        channel.basicPublish("","test",null,"hello world".getBytes());
        //7.关闭资源
        channel.close();
        connection.close();
    }
}
```

#### 消费者代码

```java
package com.woniuxy.rabbitteach.simple;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

public class Received {
    public static void main(String[] args) throws IOException, TimeoutException {
        //1.创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        //2.设置参数
        factory.setHost("192.168.74.146");
        factory.setPort(5672);
        factory.setUsername("admin");
        factory.setPassword("admin");
        factory.setVirtualHost("/test");  //设置虚拟主机   默认/
        //3.创建连接
        Connection connection = factory.newConnection();
        //4.创建通道
        Channel channel = connection.createChannel();
        //5.创建消息队列
        channel.queueDeclare("test",false,false,false,null);
        //6.创建消费者
        Consumer consumer = new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String message = new String(body,"UTF-8");
                System.out.println(message);
            }
        };
        //7.消费者确认消息并进行处理
        //参数1：队列名
        //参数2：true 接收到传递过来的消息后acknowledged（应答服务器），false 接收到消息后不应答服务器
        //参数3：消费者对象的回调接口
        channel.basicConsume("test",true,consumer);
    }
}
```

依次运行rabbitmq、Received、Productor

**<font color='red'>注意：每次测试时，最好将之前的消息队列删除</font>**



### Work模式（中 25）

#### 介绍

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210721154902.png" alt="image-20210721154902797" style="zoom:67%;" />

- 消息产生者将消息放入队列消费者可以有多个,消费者1,消费者2,同时监听同一个队列,消息被消费者C1 C2共同争抢当前的消息队列内容,谁先拿到谁负责消费消息(隐患,高并发情况下,默认会产生某一个消息被多个消费者共同使用,可以设置一个开关(syncronize,与同步锁的性能不一样) 保证一条消息只能被一个消费者使用)

- 应用场景:红包;大项目中的资源调度(任务分配系统不需知道哪一个任务执行系统在空闲,直接将任务扔到消息队列中,空闲的系统自动争抢)

#### 生产者，在simple模式代码基础上循环发送消息

```java
package com.woniuxy.rabbitteach.work;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

public class Productor {
    public static void main(String[] args) throws IOException, TimeoutException {
        //1.创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        //2.设置参数
        factory.setHost("192.168.74.146");
        factory.setPort(5672);
        factory.setUsername("admin");
        factory.setPassword("admin");
        //3.创建连接
        Connection connection = factory.newConnection();
        //4.创建通道
        Channel channel = connection.createChannel();
        //5.创建消息队列
        //参数1：队列名
        //参数2：是否持久化
        //参数3：是否排外，一个队列只允许一个消费者连接
        //参数4：是否自动删除
        //参数5：初始化参数

        channel.queueDeclare("test",false,false,false,null);
        //6.发送消息
        for(int i=0;i<10;i++){
            channel.basicPublish("","test",null,("hello world"+i).getBytes());
        }
        //7.关闭资源
        channel.close();
        connection.close();
    }
}
```



#### 创建两个消费者

#### 消费者同simple模式



利用Productor发送消息，可以看到两个消费者的控制台打印信息如下

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210721155421.png" alt="image-20210721155421839" style="zoom:67%;" />

### 订阅模式（中 25）

#### 介绍

publish/subscribe发布订阅(共享资源)

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210721155503.png" alt="image-20210721155503436" style="zoom:67%;" />

- X代表交换机rabbitMQ内部组件,erlang 消息产生者是代码完成,代码的执行效率不高,消息产生者将消息放入交换机,交换机发布订阅把消息发送到所有消息队列中,对应消息队列的消费者拿到消息进行消费

- 相关场景:邮件群发,群聊天,广播(广告)

#### 生产者

```java
public class Productor {
    public static String EXCHANGE_NAME = "exchange-1";

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("192.168.74.146");
        factory.setPort(5672);
        factory.setUsername("admin");
        factory.setPassword("admin");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        //创建交换机fanout
        channel.exchangeDeclare(EXCHANGE_NAME,"fanout");
        //发送消息
        for(int i=0;i<10;i++){
            channel.basicPublish(EXCHANGE_NAME,"", null, ("hello"+i).getBytes());
        }
        channel.close();
        connection.close();
    }
}
```

#### 消费者

```java
public class Consumer1 {
    public static String EXCHANGE_NAME = "exchange-1";

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("192.168.74.146");
        factory.setPort(5672);
        factory.setUsername("admin");
        factory.setPassword("admin");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        //声明交换机
        channel.exchangeDeclare(EXCHANGE_NAME,"fanout");
        //获取随机队列名
        String queueName = channel.queueDeclare().getQueue();
        //绑定交换机和队列
        channel.queueBind(queueName,EXCHANGE_NAME,"");
        Consumer consumer = new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String message = new String(body,"UTF-8");
                System.out.println(message);
            }
        };
        channel.basicConsume(queueName,true,consumer);
    }
}

```



### 路由模式（中 25）

#### 介绍

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210721155604.png" alt="image-20210721155604376" style="zoom:67%;" />

- 消息生产者将消息发送给交换机按照路由判断,路由是字符串(info) 当前产生的消息携带路由字符(对象的方法),交换机根据路由的key,只能匹配上路由key对应的消息队列,对应的消费者才能消费消息;

- 根据业务功能定义路由字符串

- 从系统的代码逻辑中获取对应的功能字符串,将消息任务扔到对应的队列中业务场景:error 通知;EXCEPTION;错误通知的功能;传统意义的错误通知;客户通知;利用key路由,可以将程序中的错误封装成消息传入到消息队列中,开发者可以自定义消费者,实时接收错误;

#### 生产者

```java
public class Productor {
    public static String EXCHANGE_NAME = "rout-1";
    public static List<String> routs = new ArrayList<>();
    static {
        routs.add("news");
        routs.add("weather");
    }

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("192.168.74.146");
        factory.setPort(5672);
        factory.setUsername("admin");
        factory.setPassword("admin");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        //声明交换机
        channel.exchangeDeclare(EXCHANGE_NAME,"direct");
        //发送消息
        Random random = new Random();
        for(int i = 0;i < 10; i++){
            String rout = routs.get(random.nextInt(2));
            channel.basicPublish(EXCHANGE_NAME, rout,null,("hello"+i).getBytes());
        }

        channel.close();
        connection.close();
    }
}
```

#### 消费者

```java
public class Consumer {

    public static String EXCHANGE_NAME = "rout-1";

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("192.168.74.146");
        factory.setPort(5672);
        factory.setUsername("admin");
        factory.setPassword("admin");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        //声明交换机
        channel.exchangeDeclare(EXCHANGE_NAME,"direct");
        //获取随机队列名
        String queueName = channel.queueDeclare().getQueue();
        //绑定交换机和队列
        channel.queueBind(queueName,EXCHANGE_NAME,"news");
        Consumer consumer = new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String message = new String(body,"UTF-8");
                System.out.println(message);
            }
        };
        channel.basicConsume(queueName,true,consumer);
    }
}
```



### 主题模式（中 25）

#### 介绍

topic主题模式(路由模式的一种)

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210721155705.png" alt="image-20210721155705717" style="zoom:67%;" />

- 星号井号代表通配符

- 星号代表一个单词,井号代表零个或多个单词       

- 路由功能添加模糊匹配

- 消息产生者产生消息,把消息交给交换机

- 交换机根据key的规则模糊匹配到对应的队列,由队列的监听消费者接收消息消费

#### 生产者

```java
public class Productor {
    public static String EXCHANGE_NAME = "topic-1";

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("192.168.74.146");
        factory.setPort(5672);
        factory.setUsername("admin");
        factory.setPassword("admin");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        //声明交换机
        channel.exchangeDeclare(EXCHANGE_NAME,"topic");
        //发送消息
        Random random = new Random();
        for(int i = 0;i < 10; i++){
            channel.basicPublish(EXCHANGE_NAME, "chengdu.news",null,("hello"+i).getBytes());
        }

        channel.close();
        connection.close();
    }
}
```

#### 消费者1

```java
public class Consumer1 {

    public static String EXCHANGE_NAME = "topic-1";

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("192.168.74.146");
        factory.setPort(5672);
        factory.setUsername("admin");
        factory.setPassword("admin");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        //声明交换机
        channel.exchangeDeclare(EXCHANGE_NAME,"topic");
        //获取随机队列名
        String queueName = channel.queueDeclare().getQueue();
        //绑定交换机和队列
        channel.queueBind(queueName,EXCHANGE_NAME,"*.news");
        Consumer consumer = new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String message = new String(body,"UTF-8");
                System.out.println("news:"+message);
            }
        };
        channel.basicConsume(queueName,true,consumer);
    }
}
```

#### 消费者2

```java
public class Consumer1 {

    public static String EXCHANGE_NAME = "topic-1";

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("192.168.74.146");
        factory.setPort(5672);
        factory.setUsername("admin");
        factory.setPassword("admin");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        //声明交换机
        channel.exchangeDeclare(EXCHANGE_NAME,"topic");
        //获取随机队列名
        String queueName = channel.queueDeclare().getQueue();
        //绑定交换机和队列
        channel.queueBind(queueName,EXCHANGE_NAME,"chengdu.*");
        Consumer consumer = new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String message = new String(body,"UTF-8");
                System.out.println("chengdu:"+message);
            }
        };
        channel.basicConsume(queueName,true,consumer);
    }
}

```





