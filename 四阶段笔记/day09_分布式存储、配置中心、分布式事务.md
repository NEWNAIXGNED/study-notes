### 分布式存储介绍（中 15）

分布式存储的特点及解决方案

分布式存储是一种数据存储技术，通过网络使用企业中的每台机器上的磁盘空间，并将这些分散的存储资源构成一个虚拟的存储设备，数据分散的存储在企业的各个角落。

分布式网络存储系统采用可扩展的系统结构，利用多台存储服务器分担存储负荷，利用位置服务器定位存储信息，它不但提高了系统的可靠性、可用性和存取效率，还易于扩展。



#### 专业术语

##### OSS

Object Storage Service俗称对象存储，主要提供图片、文档、音频、视频等二进制文件的海量存储功能。目前除了公有云提供对象存储服务外，一般私有云比较关心一些开源的分布式对象存储解决方案，本文列举了一些常见的技术方案供参考。



##### 块存储

通常SAN（Storage Area Network）结构的产品属于块存储，比如我们常见的硬盘、磁盘阵列等物理盘。



##### 文件存储

一般NAS（Network Attached Storage）产品都是文件级存储，如Ceph的CephFS，另外GFS、HDFS等也属于文件存储。



##### 对象存储

同时兼顾着SAN高速直接访问磁盘特点及NAS的分布式共享特点的一类存储，一般是通过RESTful接口访问。

 

#### 常见解决方案

##### Swift

Swift 是 OpenStack 社区核心子项目，是一个弹性可伸缩、高可用的分布式对象存储系统，使用Python语言实现，采用 Apache 2.0 许可协议。

Swift 提供一个基于RESTful HTTP接口的 Object Storage API，用于创建，修改和获取对象和元数据。用户可以使用 Swift 高效、安全且廉价地存储大量数据。Swift 整体架构：

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210718181204.png" alt="image-20210718181203768" style="zoom:50%;" />

总的来说，企业如果想要建立可扩展的分布式对象存储集群，可以考虑 Swift。



##### Ceph

Ceph是一种高性能、高可用、可扩展的分布式存储系统，统一的对外提供对象存储、块存储以及文件存储功能，底层使用C/C++语言。

其中对象存储功能支持 2 种接口：
1、兼容S3：提供了对象存储接口，兼容 S3 RESTful 接口的一个大子集。
2、兼容Swift：提供了对象存储接口，兼容 Openstack Swift 接口的一个大子集。

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210718181305.png" alt="image-20210718181304987" style="zoom:50%;" />

Ceph是一个企业级分布式存储系统，功能强大，不仅可以为企业建立对象存储服务，还可以帮助企业建立自己的云平台，具有广泛的应用场景特别是在云环境下使用广泛。



##### Minio

Minio是一个企业级、兼容S3接口的对象存储系统。Minio基于 Apache 2.0 许可协议，采用Go语言实现，客户端支持Java、Python、Go等多种语言，是一种轻量级、高并发的开源解决方案，可以作为云存储方案用来保存海量的图片，视频，文档等。

大数据集成方面，Minio支持各种常见的查询计算引擎，比如Spark、Presto、Hive以及Flink等，可以使用这些处理框架查询分析对象数据，此外，Minio支持Parquet，Json、Csv格式等多种文件存储格式，包括压缩与编码。更多特性可以参考官网 地址https://min.io。Minio架构：

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210718181353.png" style="zoom:50%;" />

Minio主要为人工智能、机器学习而设计，并适用于其他大数据负载。从架构与功能方面考虑，Minio是一个比较好的开源对象存储解决方案。



##### HBase MOB

这是利用HBase的MOB特性支持对象存储功能。Apache HBase2.0 版本开始支持中等对象存储（Medium Object Storage，简称 MOB），这个特性使得HBase能够非常良好的存储大小在100KB-10M的图片、文档、音频、短视频等二进制数据。

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210718181616.png" alt="image-20210718181615972" style="zoom:50%;" />

架构如上，HBase MOB的设计类似于HBase + HDFS的方式，中等对象在写入HDFS之前同样是先写入MemStore，但是刷写与其他写入数据不同，MOB数据被刷写到MOB File中，MOB File被存放在特殊的Region中。

MOB特性在Apache HBase 2.0、CDH 5.4.x 或 HDP 2.5.x 及以上版本支持，用户可以基于HBase MOB特性设计自己的对象存储服务。


### 对象存储实现（高 40）

#### 利用阿里云OSS实现文件存储

阿里云对象存储*OSS*（Object Storage Service）是阿里云提供的海量、安全、低成本、高持久的云存储服务。其数据设计持久性不低于99.9999999999%（12个9），服务可用性...有关*阿里云存储服务*的客户案例、解决方案等，请参见阿里云存储产品家族。



##### 开通阿里云OSS服务

点击“产品”->“存储”->“对象存储OSS”

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210718203230.png" alt="image-20210718203230530" style="zoom: 33%;" />

点击“立即开通”

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210718203600.png" alt="image-20210718203600333" style="zoom:33%;" />

勾选协议，点击立即开通

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210718203638.png" alt="image-20210718203638118" style="zoom:33%;" />

开通完毕

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210718203710.png" alt="image-20210718203709866" style="zoom:33%;" />

#### 创建桶

bucket列表->创建bucket

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210718203840.png" alt="image-20210718203840668" style="zoom:33%;" />

点击“开通”

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210718203922.png" alt="image-20210718203922369" style="zoom:33%;" />



指定桶名字以及存储区域

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220331110115.png" alt="image-20220331110115666" style="zoom:50%;" />

#### 设置AccessKey

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210718210715.png" alt="image-20210718210715686" style="zoom: 25%;" />

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220331110137.png" alt="image-20220331110137680" style="zoom:50%;" />

点击“开始使用子用户AccessKey”

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210718210835.png" alt="image-20210718210835130" style="zoom: 25%;" />

点击“创建用户”

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210718210911.png" alt="image-20210718210910861" style="zoom: 25%;" />

指定登录名和显示名称，选择“编程访问”，点击“确定创建”

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220331110530.png" alt="image-20220331110530134" style="zoom:50%;" />



**<font color='red'>创建完毕之后，在用户列表下游用户信息和accesskey id和accesskey secret，请立即将access key保存</font>**

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210718211413.png" alt="image-20210718211412744" style="zoom: 25%;" />



##### 指定用户权限

点击用户信息

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210718211821.png" alt="image-20210718211821017" style="zoom: 25%;" />

选择”权限管理“选项卡，点击”添加权限“

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210718211900.png" alt="image-20210718211859661" style="zoom: 25%;" />

授予全部权限

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220331111005.png" alt="image-20220331111005416" style="zoom:50%;" />



#### 新建oss微服务

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210718213144.png" alt="image-20210718213144561" style="zoom:33%;" />

##### 导入阿里云sdk

```xml
<!--aliyun-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>aliyun-oss-spring-boot-starter</artifactId>
    <version>1.0.0</version>
</dependency>
<dependency>
    <groupId>com.aliyun</groupId>
    <artifactId>aliyun-java-sdk-core</artifactId>
    <version>4.5.0</version>
</dependency>
```



##### 导入公共模块

```xml
<parent>
    <artifactId>springcloud-teach</artifactId>
    <groupId>com.woniuxy</groupId>
    <version>1.0</version>
</parent>
```



##### oss模块中创建实体类

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class UploadResult implements Serializable {
    private String fileName;
    private String url;
}
```



##### oss模块中创建controller

```java
package com.woniuxy.oss.controller;

import com.aliyun.oss.OSSClient;
import com.commons.result.ResponseResult;
import com.woniuxy.oss.entity.UploadResult;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

import javax.annotation.Resource;
import java.io.IOException;
import java.io.InputStream;
import java.net.URL;
import java.util.Date;

@RestController
@RequestMapping("/file")
public class OssController {
    @Resource
    private OSSClient ossClient;

    private static final String BUCKETNAME = "woniuxy-79";
    private String baseUrl = "https://woniuxy-79.oss-cn-chengdu.aliyuncs.com/";

    @PostMapping("/upload")
    public ResponseResult<UploadResult> upload(MultipartFile file){
        //
        ResponseResult<UploadResult> responseResult = new ResponseResult<>(500,"fail",null);

        //1.获得文件名字
        String fileName = file.getOriginalFilename();

        //2.获取文件流
        InputStream inputStream = null;
        try {
            inputStream = file.getInputStream();
            //

            ossClient.putObject(BUCKETNAME,fileName,inputStream);

            //设置文件有效期  60天
            Date date = new Date(new Date().getTime()+ 1000*3600*24*60);
            //获取url
            URL url = ossClient.generatePresignedUrl(BUCKETNAME,fileName,date);

            //返回数据
            UploadResult fileResult = new UploadResult(fileName,baseUrl+fileName);
            //
            responseResult.setStatus(200);
            responseResult.setMessage("success");
            responseResult.setData(fileResult);
            //
            return responseResult;
        }catch (IOException e){
            e.printStackTrace();
        }
        return responseResult;
    }

    //删除文件
    @PostMapping("/del")
    public ResponseResult<String> del(String fileName){
        ossClient.deleteObject(BUCKETNAME,fileName);
        return new ResponseResult<>(200,"success",null);
    }
}
```



##### 在oss的application.yml配置文件中配置eureka、阿里云oss等信息

```yaml
alibaba:
  cloud:
    access-key: LTAI5tHmB7zi4kM8Azw5xqxU
    secret-key: TscBRtri6x1KUXGhdbPYOArFJVALDX
    oss:
      endpoint: oss-cn-chengdu.aliyuncs.com
spring:
  application:
    name: oss
server:
  port: 9999
eureka:
  client: #客户端注册到eureka列表中
    service-url:
      defaultZone: http://127.0.0.1:9001/eureka
  instance:
    instance-id: oss-9600  #注册中心status显示出来的微服务id
    prefer-ip-address: true #显示访问url
```



其中endpoint可以在控制台找到，如下图所示

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210718214120.png" alt="image-20210718214120421" style="zoom: 25%;" />



##### gateway模块中配置oss路由信息

```yaml
spring:
  application:
    name: gateway
  cloud:
    inetutils:
      default-ip-address: 127.0.0.1
    gateway:
      routes:
        - id: oss
          uri: lb://oss
          predicates:
            - Path=/file/**
```



##### 前端代码

```java
<template>
  <div>
   <input type="file" name="file" id="file"><br>
   <button @click="upload()">上传</button>
  </div>
</template>

<script>
export default {
  methods:{
    upload:function(){
      //获取文件
      let file = document.getElementById("file").files[0];
      console.log(file);
      //将file封装到formdata
      let formData = new FormData();
      formData.append("file",file);
      //上传
      this.axios.post("http://localhost:9600/file/upload",formData,{
        headers:{
          'Content-Type':'multipart/form-data'
        }
      }).then(res => {
        console.log(res.data);
      })
    }
  }
}
</script>

<style scoped>

</style>
```



##### 依次启动eureka、oss、gateway微服务，前端向gateway发送请求，浏览器控制台显示以下信息表示成功

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210719203919.png" alt="image-20210719203918891" style="zoom:33%;" />



### Config概述（中 10）

Config分布式配置中心的概念及作用

 <img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210719210621.png" alt="image-20210719210621454" style="zoom:33%;" />

#### 基本概念

随着线上项目变的日益庞大,每个项目都散落着各种配置文件，如果采用分布式的开发模式，需要的配置文件随着服务增加而不断增多。某一个基础服务信息变更,都会引起一系列的更新和重启, 运维苦不堪言也容易出错。配置中心便是解决此类问题的灵丹妙药。

 

·简化集群yml文件配置

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210719210647.png" alt="image-20210719210647481" style="zoom: 67%;" />

目前支持本地存储、Git 以及SVN。

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210719210707.png" alt="image-20210719210707304" style="zoom:67%;" />



Spring Cloud Config分服务端和客户端,服务端负责将git( svn )中存储的配置文件发布成REST接口,客户端可以从服务端REST接口获取配置。但客户端并不能主动感知到配置的变化,从而主动去获取新的配置，这需要每个客户端通过POST方法触发各自的/refresh。



### Config搭建（高 20）

搭建Config配置中心，微服务与配置中心通信

#### 配置文件

##### 先在gitee上创建一个远程仓库

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210719210859.png" alt="image-20210719210859320" style="zoom: 25%;" />

##### 本地创建一个文件夹，将远程仓库克隆到其中

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210719211145.png" alt="image-20210719211145266" style="zoom:33%;" />

##### 在克隆下来的文件夹中创建application.yml文件，添加以下内容并保存

```yaml
spring: 
    profiles: dev   #开发环境
    application: 
        name: provider-dev
 
---
 
 spring: 
    profiles: test   #测试环境
    application: 
        name: provider-test

---

spring: 
    profiles: pro   #生产环境
    application: 
        name: provider-pro 
```

**<font color='red'>注意：编码集一定要设置成UTF-8</font>**

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210719211527.png" alt="image-20210719211527305" style="zoom: 50%;" />

##### 添加application.yml到缓存区中

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210719211340.png" alt="image-20210719211340601" style="zoom:33%;" />

##### 提交	git commit -m "add file application.yml

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210719211721.png" alt="image-20210719211721464" style="zoom:33%;" />

##### 推送到远程仓库	git push origin master

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210719211943.png" alt="image-20210719211943061" style="zoom:33%;" />

#### 创建config模块

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210719212339.png" alt="image-20210719212339402" style="zoom:33%;" />

##### 如果出现config-server引入失败，可以自定义引入版本

```xml
<!--config-server-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
    <version>2.1.1.RELEASE</version>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-client</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<!--config-client-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-client</artifactId>
    <version>2.1.1.RELEASE</version>
</dependency>
```

##### 在主启动类上开启eureka和config

```java
@SpringBootApplication
@EnableConfigServer
@EnableEurekaClient
public class ConfigApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigApplication.class, args);
    }

}
```

##### 在application.yml中添加配置信息

```yaml
server:
  port: 7000

spring:
  application:
    name: config
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/xiangweilll/config-79.git  #仓库地址
          #search-paths: config-repo      #仓库下的相对地址，可以有多个，用逗号隔开
          username: xiangweilll   #账号
          password: xw20200401    #密码
eureka:
  client: #客户端注册到eureka列表中
    service-url:
      defaultZone: http://127.0.0.1:9001/eureka
  instance:
    instance-id: config-7000  #注册中心status显示出来的微服务id
    prefer-ip-address: true #显示访问url
```

##### 依次运行eureka、config

##### 浏览器上访问配置中心，获取配置信息

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210719214657.png" alt="image-20210719214657078" style="zoom:33%;" />

或者

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210719214730.png" alt="image-20210719214730709" style="zoom:33%;" />

#### 配置中心访问规则

```
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```



#### 模拟需要从配置中心获取配置文件的微服务

##### 在本地仓库中添加application-client.yml配置文件

```yml
server: 
  port: 7500
 
spring: 
  profile: dev      #开发环境
  application:
    name: config-client-dev #当前项目的名字

eureka:
  client: #客户端注册到eureka列表中
    service-url:
      defaultZone: http://localhost:9001/eureka/
  instance:
    instance-id: config-client  #配置中心显示出来的微服务名称
    prefer-ip-address: true #显示访问url

---

server: 
  port: 7500
 
spring: 
  profile: test      #开发环境
  application:
    name: config-client-test #当前项目的名字

eureka:
  client: #客户端注册到eureka列表中
    service-url:
      defaultZone: http://localhost:9001/eureka/
  instance:
    instance-id: config-test  #配置中心显示出来的微服务名称
    prefer-ip-address: true #显示访问url 
```

##### 并推送到远程仓库

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210719215919.png" alt="image-20210719215919792" style="zoom:33%;" />



##### 创建config-client微服务

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210719215213.png" alt="image-20210719215212841" style="zoom:33%;" />

如果config依赖导入不了，自定义导入

```xml
<!--config-client-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-client</artifactId>
    <version>2.1.1.RELEASE</version>
</dependency>
```



在config-client下创建bootstrap.yml，并添加以下配置

```yaml
#连接config-server服务
spring:
  cloud:
    config:
      name: application-client    #要读取配置文件的名字
      profile: test                 #要加载的环境
      label: master                 #git上的哪个分支
      uri: http://localhost:7000  #连接到config-server的路径
```



为了方便观察效果，创建一个controller，用于得到从git上过去到的配置文件信息

```java
@RestController
public class ClientController {
    @Value("${spring.application.name}")
    private String applicationName;

    @RequestMapping("/name")
    public String name(){
        return applicationName;
    }
}
```



在主启动类上开启eureka

```java
@SpringBootApplication
@EnableEurekaClient
public class ConfigClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigClientApplication.class, args);
    }

}
```

分别启动eureka、config、config-client，通过浏览器访问config-client，正常情况下可以得到配置信息，如下图所示

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210719220403.png" alt="image-20210719220403570" style="zoom:33%;" />



### 分布式事务基本概念（中 15）

#### 产生的背景

在微服务环境下，因为会根据不同的业务将拆分成不同的服务，比如会员服务、订单服务、商品服务等等，每个服务都有自己独立的数据库并且是独立运行的，互不影响的。服务与服务之间通讯采用RPC远程调用技术，但是每个服务中都有自己独立的数据源，即自己独立的本地事务；两个服务相互进行通讯的时候，两个本地事务互不影响，从而出现分布式事务产生的原因。

 

传统项目大部分情况下，不会产生分布式事务，但是在项目中如果采用多数据源的方式就会产生分布式事务；

 

案例：

假设银行(bank)中有两个客户(name)张三和李四

我们需要将张三的1000元存款(sal)转到李四的账户上

目标就是张三账户减1000，李四账户加1000，不能出现中间步骤(张三减1000，李四没加)

 

#### 事务

·原子性（A）

所谓的原子性就是说，在整个事务中的所有操作，要么全部完成，要么全部不做，没有中间状态。对于事务在执行中发生错误，所有的操作都会被回滚，整个事务就像从没被执行过一样。

·一致性（C）

事务的执行必须保证系统的一致性，就拿转账为例，A有500元，B有300元，如果在一个事务里A成功转给B50元，那么不管并发多少，不管发生什么，只要事务执行成功了，那么最后A账户一定是450元，B账户一定是350元。

·隔离性（I）

所谓的隔离性就是说，事务与事务之间不会互相影响，一个事务的中间状态不会被其他事务感知。

·持久性（D）

所谓的持久性，就是说一单事务完成了，那么事务对数据所做的变更就完全保存在了数据库中，即使发生停电，系统宕机也是如此。

 

#### 微服务落地存在的问题

虽然微服务现在如火如荼，但对其实践其实仍处于探索阶段。很多中小型互联网公司，鉴于经验、技术实力等问题，微服务落地比较困难，目前存在的主要困难有如下几方面：

1）单体应用拆分为分布式系统后，进程间的通讯机制和故障处理措施变的更加复杂。

2）系统微服务化后，一个看似简单的功能，内部可能需要调用多个服务并操作多个数据库实现，服务调用的分布式事务问题变的非常突出。

3）微服务数量众多，其测试、部署、监控等都变的更加困难。

 

随着RPC框架的成熟，第一个问题已经逐渐得到解决。例如dubbo可以支持多种通讯协议，springcloud可以非常好的支持restful调用。对于第三个问题，随着docker、devops技术的发展以及各公有云paas平台自动化运维工具的推出，微服务的测试、部署与运维会变得越来越容易。

 

而对于第二个问题，现在还没有通用方案很好的解决微服务产生的事务问题。分布式事务已经成为微服务落地最大的阻碍，也是最具挑战性的一个技术难题。



### 解决方案（中  15）

#### 基于消息的最终一致性方案

例如：RocketMQ、rabbitMQ等

消息一致性方案是通过消息中间件保证上、下游应用数据操作的一致性。基本思路是将本地操作和发送消息放在一个事务中，保证本地操作和消息发送要么两者都成功或者都失败。下游应用向消息系统订阅该消息，收到消息后执行相应操作。

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720114130.png" alt="image-20210720114129917" style="zoom:50%;" />

- 实现：业务处理服务在业务事务提交之前，向实时消息服务请求发送消息，实时消息服务只记录消息数据，而不是真正的发送。业务处理服务在业务事务提交之后，向实时消息服务确认发送。只有在得到确认发送指令后，实时消息服务才会真正发送。
- 消息：业务处理服务在业务事务回滚后，向实时消息服务取消发送。消息发送状态确认系统定期找到未确认发送或者回滚发送的消息，向业务处理服务询问消息状态，业务处理服务根据消息ID或者消息内容确认该消息是否有效。被动方的处理结果不会影响主动方的处理结果，被动方的消息处理操作是幂等操作。
- 成本：可靠的消息系统建设成本，一次消息发送需要两次请求，业务处理服务需要实现消息状态回查接口。
- 优点：消息数据独立存储，独立伸缩，降低业务系统和消息系统之间的耦合。对最终一致性时间敏感度较高，降低业务被动方的实现成本。兼容所有实现JMS标准的MQ中间件，确保业务数据可靠的前提下，实现业务的最终一致性，理想状态下是准实时的一致性。



消息方案从本质上讲是将分布式事务转换为两个本地事务，然后依靠下游业务的重试机制达到最终一致性。基于消息的最终一致性方案对应用侵入性也很高，应用需要进行大量业务改造，成本较高。



#### TCC事务补偿型方案

<div align="center">
<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720110954.png" alt="image-20210720110954541" style="zoom:50%;;" />
</div>


- 实现：业务应用负责发起并完成整个业务活动。其它业务服务提供TCC型业务操作。事务协调器控制业务活动的一致性，它登记业务活动的操作，并在业务活动提交时确认所有的TCC型操作的Confirm操作，在业务活动取消时调用所有TCC型操作的Cancel操作。
- 成本：实现TCC操作的成本较高，业务活动结束的时候Confirm和Cancel操作的执行成本。业务活动的日志成本。
- 使用范围：强隔离性，严格一致性要求的业务活动。适用于执行时间较短的业务，比如处理账户或者收费等等。
- 特点：不与具体的服务框架耦合，位于业务服务层，而不是资源层，可以灵活的选择业务资源的锁定粒度。TCC里对每个服务资源操作的是本地事务，数据被锁住的时间短，可扩展性好，可以说是为独立部署的SOA服务而设计的。



#### 最大努力通知型

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720115527.png" alt="image-20210720115527200" style="zoom:50%;" />

- 实现：业务活动的主动方在完成处理之后向业务活动的被动方发送消息，允许消息丢失。业务活动的被动方根据定时策略，向业务活动的主动方查询，恢复丢失的业务消息。
- 约束：被动方的处理结果不影响主动方的处理结果。
- 成本：业务查询与校对系统的建设成本。
- 使用范围：对业务最终一致性的时间敏感度低。跨企业的业务活动。
- 特点：业务活动的主动方在完成业务处理之后，向业务活动的被动方发送通知消息。主动方可以设置时间阶梯通知规则，在通知失败后按规则重复通知，知道通知N次后不再通知。主动方提供校对查询接口给被动方按需校对查询，用户恢复丢失的业务消息。
- 适用范围：银行通知，商户通知。



### Seata基础（中 15）

#### 背景

2019年1月，阿里巴巴中间件团队发起了开源项目 Fescar（Fast & EaSy Commit And Rollback），和社区一起共建开源分布式事务解决方案。Fescar 的愿景是让分布式事务的使用像本地事务的使用一样，简单和高效，并逐步解决开发者们遇到的分布式事务方面的所有难题。

 

Fescar 开源后，蚂蚁金服加入 Fescar 社区参与共建，并在 Fescar 0.4.0 版本中贡献了 TCC 模式。

 

为了打造更中立、更开放、生态更加丰富的分布式事务开源社区，经过社区核心成员的投票，大家决定对 Fescar 进行品牌升级，并更名为 Seata，意为：Simple Extensible Autonomous Transaction Architecture，是一套一站式分布式事务解决方案。

 

Seata 项目地址：https://github.com/seata/seata

 

Seata 是一款开源的分布式事务解决方案，致力于提供高性能和简单易用的分布式事务服务。Seata 将为用户提供了 AT、TCC、SAGA 和 XA 事务模式，为用户打造一站式的分布式解决方案。



#### 专业术语

##### TC(Transaction Coordinator) - 事务协调器

​			维护全局和分支事务的状态，驱动全局事务提交或回滚。

##### TM(Transaction Manager) - 事务管理器

​			定义全局事务的范围：开始全局事务、提交或回滚全局事务。

##### RM(Resource Manager) - 资源管理器

​			管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720165056.png" alt="image-20210720165055913" style="zoom:67%;" />

#### 下载地址

https://github.com/seata/seata/releases



### Seata安装（中 15）

#### seata-server

a）备份seata-server中的conf/file.conf、registry.conf

b）修改file.conf

##### 设置自定义事务组名称

<div align="center">
<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720121915.png" alt="image-20210720121915344" style="zoom: 50%;" />
</div>


##### 修改日志存储模式为db,事务信息用db存储

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720122112.png" alt="image-20210720122112796" style="zoom: 50%;" />


##### 修改数据库配置

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720122209.png" alt="image-20210720122209354" style="zoom:50%;" />

#### 修改registry.conf文件，指定注册中心及其地址

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720122246.png" alt="image-20210720122246019" style="zoom:50%;" />

#### 准备数据库

```sql
-- seata
DROP DATABASE IF EXISTS seata;
CREATE DATABASE seata DEFAULT CHARACTER SET utf8;
USE seata;
-- the table to store GlobalSession data
DROP TABLE IF EXISTS `global_table`;
CREATE TABLE `global_table` (
  `xid` VARCHAR(128)  NOT NULL,
  `transaction_id` BIGINT,
  `status` TINYINT NOT NULL,
  `application_id` VARCHAR(32),
  `transaction_service_group` VARCHAR(32),
  `transaction_name` VARCHAR(128),
  `timeout` INT,
  `begin_time` BIGINT,
  `application_data` VARCHAR(2000),
  `gmt_create` DATETIME,
  `gmt_modified` DATETIME,
  PRIMARY KEY (`xid`),
  KEY `idx_gmt_modified_status` (`gmt_modified`, `status`),
  KEY `idx_transaction_id` (`transaction_id`)
);

-- the table to store BranchSession data
DROP TABLE IF EXISTS `branch_table`;
CREATE TABLE `branch_table` (
  `branch_id` BIGINT NOT NULL,
  `xid` VARCHAR(128) NOT NULL,
  `transaction_id` BIGINT ,
  `resource_group_id` VARCHAR(32),
  `resource_id` VARCHAR(256) ,
  `lock_key` VARCHAR(128) ,
  `branch_type` VARCHAR(8) ,
  `status` TINYINT,
  `client_id` VARCHAR(64),
  `application_data` VARCHAR(2000),
  `gmt_create` DATETIME,
  `gmt_modified` DATETIME,
  PRIMARY KEY (`branch_id`),
  KEY `idx_xid` (`xid`)
);

-- the table to store lock data
DROP TABLE IF EXISTS `lock_table`;
CREATE TABLE `lock_table` (
  `row_key` VARCHAR(128) NOT NULL,
  `xid` VARCHAR(96),
  `transaction_id` LONG ,
  `branch_id` LONG,
  `resource_id` VARCHAR(256) ,
  `table_name` VARCHAR(32) ,
  `pk` VARCHAR(36) ,
  `gmt_create` DATETIME ,
  `gmt_modified` DATETIME,
  PRIMARY KEY(`row_key`)
);
-- the table to store seata xid data  回滚日志表
DROP TABLE `undo_log`;
CREATE TABLE `undo_log` (
  `id` BIGINT(20) NOT NULL AUTO_INCREMENT,
  `branch_id` BIGINT(20) NOT NULL,
  `xid` VARCHAR(100) NOT NULL,
  `context` VARCHAR(128) NOT NULL,
  `rollback_info` LONGBLOB NOT NULL,
  `log_status` INT(11) NOT NULL,
  `log_created` DATETIME NOT NULL,
  `log_modified` DATETIME NOT NULL,
  `ext` VARCHAR(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

-- bank_a
DROP DATABASE IF EXISTS bank_a;
CREATE DATABASE bank_a DEFAULT CHARACTER SET utf8;
USE bank_a;
CREATE TABLE bank_a(
	id INT PRIMARY KEY,
	money INT,
	`user` CHAR(20)
); 
INSERT INTO bank_a VALUES(1001,5000,'zhangsan');
-- the table to store seata xid data
DROP TABLE `undo_log`;
CREATE TABLE `undo_log` (
  `id` BIGINT(20) NOT NULL AUTO_INCREMENT,
  `branch_id` BIGINT(20) NOT NULL,
  `xid` VARCHAR(100) NOT NULL,
  `context` VARCHAR(128) NOT NULL,
  `rollback_info` LONGBLOB NOT NULL,
  `log_status` INT(11) NOT NULL,
  `log_created` DATETIME NOT NULL,
  `log_modified` DATETIME NOT NULL,
  `ext` VARCHAR(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

-- bank_b
DROP DATABASE IF EXISTS bank_b;
CREATE DATABASE bank_b DEFAULT CHARACTER SET utf8;
USE bank_b;
CREATE TABLE bank_b(
	id INT PRIMARY KEY,
	money INT,
	`user` CHAR(20)
); 
INSERT INTO bank_b VALUES(2001,5000,'lisi');
-- the table to store seata xid data
DROP TABLE `undo_log`;
CREATE TABLE `undo_log` (
  `id` BIGINT(20) NOT NULL AUTO_INCREMENT,
  `branch_id` BIGINT(20) NOT NULL,
  `xid` VARCHAR(100) NOT NULL,
  `context` VARCHAR(128) NOT NULL,
  `rollback_info` LONGBLOB NOT NULL,
  `log_status` INT(11) NOT NULL,
  `log_created` DATETIME NOT NULL,
  `log_modified` DATETIME NOT NULL,
  `ext` VARCHAR(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```



##### 启动eureka，然后双击seata/bin/seata-server.bat运行seata服务器，出现以下信息运行成功

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720121646.png" alt="image-20210720121646560" style="zoom: 33%;" />