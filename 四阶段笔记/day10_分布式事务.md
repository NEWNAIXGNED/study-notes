### SpringBoot整合Seata（高 50）

#### 在commons模块中创建Bank实体类

```java
package com.commons.entity;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class Bank {
    private int id;
    private int money;
    private String user;
}
```



#### 创建bankB模块

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720160831.png" alt="image-20210720160831510" style="zoom: 50%;" />

##### 在pom.xml中导入druid连接池、父项目依赖，以及改造seata依赖

```xml
<parent>
    <artifactId>springcloud-teach</artifactId>
    <groupId>com.woniuxy</groupId>
    <version>1.0</version>
</parent>
```

```xml
<!--公共模块-->
<dependency>
    <groupId>com.woniuxy</groupId>
    <artifactId>commons</artifactId>
    <version>1.0</version>
</dependency>

<!--druid-->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.15</version>
</dependency>
<!--seata-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
    <version>2.1.0.RELEASE</version>
    <exclusions>
        <exclusion>
            <groupId>io.seata</groupId>
            <artifactId>seata-all</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>io.seata</groupId>
    <artifactId>seata-all</artifactId>
    <version>0.9.0</version>
</dependency>
```



##### 将seata-server中的file.conf文件拷贝到resources目录下，并修改代码，如下：

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720161312.png" alt="image-20210720161312144" style="zoom:50%;" />

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720161346.png" alt="image-20210720161346144" style="zoom: 50%;" />



##### 将seata-server中的registry.conf文件拷贝到resources目录下，并修改代码，如下：

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720161444.png" alt="image-20210720161443901" style="zoom:50%;" />



##### 在application.yml中添加以下配置

```yaml
server:
  port: 8182
mybatis:
  mapperLocations: classpath:mapper/*.xml
  typeAliasesPackage: com.commons.entity
spring:
  application:
    name: bank-b
  cloud:
    alibaba:
      seata:
        tx-service-group: fsp_tx_group
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/bank_b?useUnicode=true&characterEncoding=utf8&serverTimezone=UTC
    username: root
    password: root
eureka:
  instance:
    hostname: localhost
    prefer-ip-address: true
  client:
    serviceUrl:
      defaultZone: http://127.0.0.1:9001/eureka/
logging:
  level:
    io:
      seata: info
```



##### 创建数据源配置类

```java
package com.woniuxy.bankb.configuration;

import com.alibaba.druid.pool.DruidDataSource;
import io.seata.rm.datasource.DataSourceProxy;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.transaction.SpringManagedTransactionFactory;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;

import javax.sql.DataSource;


@Configuration
public class DataSourceConfiguration {
    /*
     * @ConfigurationProperties用于application.yml中的配置信息
     * prefix：前缀定义了读取到application.yml中哪些属性，在创建完对象之后，在加入IOC
     * 容器里面之后会自动给该对象的属性赋值
     */
    @Bean
    @ConfigurationProperties(prefix = "spring.datasource")
    public DataSource druidDataSource(){
        DruidDataSource druidDataSource = new DruidDataSource();
        return druidDataSource;
    }
    /*
     * 设置DataSource代理，主要目的是让mybatis在获取datasource时获取到由druid代理的dataSource
     * 能够让seata管理到数据库的操作，实现分布式事务
     * 通过DataSourceProxy能在业务代码的事务提交时，seata通过这个切入点，来给TC发送RM的处理结果
     *
     * 当一个接口有2个不同实现时,使用@Autowired注解时会报
     * org.springframework.beans.factory.NoUniqueBeanDefinitionException异常信息
     * Primary用于高速spring在不知道该注入哪个bean时，优先使用选择该bean
     *
     * DataSourceProxy extends AbstractDataSourceProxy
     * AbstractDataSourceProxy implements javax.sql.DataSource
     *
     */
    @Primary
    @Bean("dataSource")
    public DataSourceProxy dataSource(DataSource druidDataSource){
        return new DataSourceProxy(druidDataSource);
    }

    /*
     * 创建SqlSessionFactory
     */
    @Bean
    public SqlSessionFactory sqlSessionFactory(DataSourceProxy dataSourceProxy)throws Exception{
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        //设置数据源   javax.sql.DataSource
        sqlSessionFactoryBean.setDataSource(dataSourceProxy);
        //加载mapper文件
        sqlSessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver()
                .getResources("classpath*:/mapper/*.xml"));
        //设置事务工厂，用于创建事务
        sqlSessionFactoryBean.setTransactionFactory(new SpringManagedTransactionFactory());
        //创建工厂对象
        return sqlSessionFactoryBean.getObject();
    }
}
```



##### 创建mapper接口

```java
package com.woniuxy.bankb.mapper;

import com.commons.entity.Bank;
import org.apache.ibatis.annotations.Mapper;

@Mapper
public interface BankMapper {
    //加钱
    public int update(Bank bank);
}
```



##### 在resources下创建mapper目录，并创建mapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.woniuxy.bankb.mapper.BankMapper" >
    <update id="update">
    update bank_b set money = money + #{money} where id=#{id};
  </update>
</mapper>
```



##### 创建service接口及实现类

```java
package com.woniuxy.bankb.service;

import com.commons.entity.Bank;

public interface BankService {
    public int update(Bank bank);
}
```

##### 实现类

```java
package com.woniuxy.bankb.service.impl;

import com.commons.entity.Bank;
import com.woniuxy.bankb.service.BankService;
import com.woniuxy.bankb.mapper.BankMapper;
import io.seata.spring.annotation.GlobalTransactional;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;

@Service
public class BankServiceImpl implements BankService {
    @Resource
    private BankMapper bankMapper;

    @Override
    public int update(Bank bank) {
        int res = bankMapper.update(bank);
        return res;
    }
}
```



##### 创建controller

```java
package com.woniuxy.bankb.controller;

import com.commons.entity.Bank;
import com.woniuxy.bankb.service.BankService;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

@RestController
@RequestMapping("/bankb")
public class BankBController {
    @Resource
    private BankService bankService;

    @RequestMapping("/update/{id}/{money}")
    public String update(@PathVariable("id") int id,@PathVariable("money") int money){
        Bank bank = new Bank(id,money,null);
        if (bankService.update(bank) > 0){
            return "success";
        }
        return "error";
    }
}
```



##### 在主启动类上开启eureka、禁止springboot自动注入数据源配置**<font color='red'>DataSourceAutoConfiguration</font>**

```java
package com.woniuxy.bankb;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

/*
 * 禁止springboot自动注入数据源配置，不让springboot给mybatis注入默认数据源
 */
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
@EnableEurekaClient
public class BankbApplication {

    public static void main(String[] args) {
        SpringApplication.run(BankbApplication.class, args);
    }

}
```



##### 在commons模块中添加BankB的service接口

```java
package com.commons.service;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;

@FeignClient(name = "BANK-B")
public interface BankBService {
    @RequestMapping("/bankb/update/{id}/{money}")
    public String update(@PathVariable("id") int id, @PathVariable("money") int money);
}
```



#### 创建bankA模块

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720160831.png" alt="image-20210720160831510" style="zoom: 50%;" />

在pom.xml中导入druid连接池、父项目依赖，以及改造seata依赖

```xml
<parent>
    <artifactId>springcloud-teach</artifactId>
    <groupId>com.woniuxy</groupId>
    <version>1.0</version>
</parent>
```

```xml
<!--公共模块-->
<dependency>
    <groupId>com.woniuxy</groupId>
    <artifactId>commons</artifactId>
    <version>1.0</version>
</dependency>

<!--druid-->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.15</version>
</dependency>
<!--seata-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
    <version>2.1.0.RELEASE</version>
    <exclusions>
        <exclusion>
            <groupId>io.seata</groupId>
            <artifactId>seata-all</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>io.seata</groupId>
    <artifactId>seata-all</artifactId>
    <version>0.9.0</version>
</dependency>
```



##### 将seata-server中的file.conf文件拷贝到resources目录下，并修改代码，如下：

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720161312.png" alt="image-20210720161312144" style="zoom:50%;" />

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720162543.png" alt="image-20210720162543602" style="zoom:50%;" />



##### 将seata-server中的registry.conf文件拷贝到resources目录下，并修改代码，如下：

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720161444.png" alt="image-20210720161443901" style="zoom:50%;" />

##### 在application.yml中添加以下配置

```yaml
server:
  port: 8181
mybatis:
  mapperLocations: classpath:mapper/*.xml
  typeAliasesPackage: com.commons.entity
spring:
  application:
    name: bank-a
  cloud:
    alibaba:
      seata:
        tx-service-group: fsp_tx_group
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/bank_a?useUnicode=true&characterEncoding=utf8&serverTimezone=UTC
    username: root
    password: root
eureka:
  instance:
    hostname: localhost
    prefer-ip-address: true
  client:
    serviceUrl:
      defaultZone: http://127.0.0.1:9001/eureka/
logging:
  level:
    io:
      seata: info
```



##### 创建数据源配置类

```java
package com.woniuxy.bankb.configuration;

import com.alibaba.druid.pool.DruidDataSource;
import io.seata.rm.datasource.DataSourceProxy;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.transaction.SpringManagedTransactionFactory;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;

import javax.sql.DataSource;


@Configuration
public class DataSourceConfiguration {
    /*
     * @ConfigurationProperties用于application.yml中的配置信息
     * prefix：前缀定义了读取到application.yml中哪些属性，在创建完对象之后，在加入IOC
     * 容器里面之后会自动给该对象的属性赋值
     */
    @Bean
    @ConfigurationProperties(prefix = "spring.datasource")
    public DataSource druidDataSource(){
        DruidDataSource druidDataSource = new DruidDataSource();
        return druidDataSource;
    }
    /*
     * 设置DataSource代理，主要目的是让mybatis在获取datasource时获取到由druid代理的dataSource
     * 能够让seata管理到数据库的操作，实现分布式事务
     * 通过DataSourceProxy能在业务代码的事务提交时，seata通过这个切入点，来给TC发送RM的处理结果
     *
     * 当一个接口有2个不同实现时,使用@Autowired注解时会报
     * org.springframework.beans.factory.NoUniqueBeanDefinitionException异常信息
     * Primary用于高速spring在不知道该注入哪个bean时，优先使用选择该bean
     *
     * DataSourceProxy extends AbstractDataSourceProxy
     * AbstractDataSourceProxy implements javax.sql.DataSource
     *
     */
    @Primary
    @Bean("dataSource")
    public DataSourceProxy dataSource(DataSource druidDataSource){
        return new DataSourceProxy(druidDataSource);
    }

    /*
     * 创建SqlSessionFactory
     */
    @Bean
    public SqlSessionFactory sqlSessionFactory(DataSourceProxy dataSourceProxy)throws Exception{
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        //设置数据源   javax.sql.DataSource
        sqlSessionFactoryBean.setDataSource(dataSourceProxy);
        //加载mapper文件
        sqlSessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver()
                .getResources("classpath*:/mapper/*.xml"));
        //设置事务工厂，用于创建事务
        sqlSessionFactoryBean.setTransactionFactory(new SpringManagedTransactionFactory());
        //创建工厂对象
        return sqlSessionFactoryBean.getObject();
    }
}
```



##### 创建mapper

```java
package com.woniuxy.banka.mapper;

import com.commons.entity.Bank;
import org.apache.ibatis.annotations.Mapper;

@Mapper
public interface BankMapper {
    //减钱
    public int update(Bank bank);
}
```

##### resources下创建mapper/xml文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.woniuxy.banka.mapper.BankMapper" >
    <update id="update">
    update bank_a set money = money - #{money} where id=#{id};
  </update>
</mapper>
```



##### 创建service及实现类

```java
package com.woniuxy.banka.service;

import com.commons.entity.Bank;

public interface BankAService {
    public int update(Bank src, Bank dest);
}
```



实现类**<font color='red'>@GlobalTransactional，全局事务，只需要加在业务发起方即可</font>**

```java
package com.woniuxy.banka.service.impl;

import com.commons.entity.Bank;
import com.commons.service.BankBService;
import com.woniuxy.banka.mapper.BankMapper;
import com.woniuxy.banka.service.BankAService;
import io.seata.spring.annotation.GlobalTransactional;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;

@Service
public class BankServiceImpl implements BankAService {
    @Resource
    private BankMapper bankMapper;

    @Resource
    private BankBService bankBService;

    @Override
    @GlobalTransactional
    public int update(Bank src, Bank dest) {
        //1.先修改对方+钱
        String result = bankBService.update(dest.getId(), dest.getMoney());
        //模拟运行时异常
        int n = 1;
        if (n==1) {
            throw new RuntimeException();   //模拟异常
        }
        int res = 0;
        if (result.equals("success")) {
            //加钱成功
            //2.减自己的
            res = bankMapper.update(src);
        }
        return res;
    }
}

```



##### 创建controller

```java
package com.woniuxy.banka.controller;

import com.commons.entity.Bank;
import com.woniuxy.banka.service.BankAService;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

@RestController
@RequestMapping("/banka")
public class BankAController {
    @Resource
    private BankAService bankAService;

    @RequestMapping("/start")
    public String begin(){
        Bank src = new Bank(1001,100,"zhangsan");	    //转出账户
        Bank dest = new Bank(2001, 100, "lisi");        //转入账户
        if(bankAService.update(src,dest)>0){
            return "转账成功!";
        }
        return "转账失败!";
    }
}
```



##### 分别启动eureka、seata、bankb、banka，确保后三者都注册到seata中，后两者都注册到seata中

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720163237.png" alt="image-20210720163236771" style="zoom: 33%;" />

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720163304.png" alt="image-20210720163303773" style="zoom: 33%;" />







### AT模式（高 30）

#### 基于XA协议的两阶段提交方案（2pc）

X/Open DTP(X/Open Distributed Transaction Processing Reference Model) 是X/Open 这个组织定义的一套分布式事务的标准，也就是了定义了规范和API接口，由各个厂商进行具体的实现。 X/Open DTP 定义了三个组件： AP，TM，RM

<div align="center">
<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720165131.jpg" alt="img" style="zoom:50%;" /> 
</div>

 

·AP(Application Program)	

也就是应用程序，可以理解为使用DTP(分布式事务处理)的程序

·RM(Resource Manager)

资源管理器，这里可以理解为一个DBMS系统，或者消息服务器管理系统，应用程序通过资源管理器对资源进行控制。资源必须实现XA定义的接口

·TM(Transaction Manager)

事务管理器，负责协调和管理事务，提供给AP应用程序编程接口以及管理资源管理器

 

其中在DTP定义了以下几个概念：

| 概念     | 解释                                                         |
| -------- | ------------------------------------------------------------ |
| 事务     | 一个事务是一个完整的工作单元，由多个独立的计算任务组成，这多个任务在逻辑上是原子的 |
| 全局事务 | 对于一次性操作多个资源管理器的事务，就是全局事务。           |
| 分支事务 | 在全局事务中，某一个资源管理器有自己独立的任务，这些任务的集合作为这个资源管理器的分支任务。 |
| 控制线程 | 用来表示一个工作线程，主要是关联AP,TM,RM三者的一个线程，也就是事务上下文环境。<br />简单的说，就是需要标识一个全局事务以及分支事务的关系 |

消息中间件与数据库通过 XA 接口规范，使用两阶段(2PC)提交来完成一个全局事务， XA 规范的基础是两阶段提交协议。

·第一阶段是准备阶段，所有参与者都将本事务能否成功的信息反馈发给协调者；

·第二阶段是提交阶段，协调者根据所有参与者的反馈，通知所有参与者，步调一致地在所有分支上提交或者回滚。


<div align="center">
<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720165156.png" alt="image-20210720165156022" style="zoom: 50%;" />
</div>

#### seata AT模式

Seata AT模式是基于XA事务演进而来的一个分布式事务中间件，XA是一个基于数据库实现的分布式事务协议，本质上和两阶段提交一样，需要数据库支持，Mysql5.6以上版本支持XA协议，其他数据库如Oracle，DB2也实现了XA接口

 

基本处理逻辑如下：

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720163932.png" alt="image-20210720163932837" style="zoom:67%;" />

##### 第一阶段

Seata 的 JDBC 数据源代理通过对业务 SQL 的解析，把业务数据在更新前后的数据镜像组织成回滚日志，利用本地事务 的 ACID 特性，将业务数据的更新和回滚日志的写入在同一个 本地事务 中提交。

 

这样，可以保证：**任何提交的业务数据的更新一定有相应的回滚日志存在**

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720163955.png" alt="image-20210720163955686" style="zoom:67%;" />

基于这样的机制，分支的本地事务便可以在全局事务的第一阶段提交，并马上释放本地事务锁定的资源。

 

这也是Seata和XA事务的不同之处，两阶段提交往往对资源的锁定需要持续到第二阶段实际的提交或者回滚操作，而有了回滚日志之后，可以在第一阶段释放对资源的锁定，降低了锁范围，提高效率，即使第二阶段发生异常需要回滚，只需找对undolog中对应数据并反解析成sql来达到回滚目的。

 

同时Seata通过代理数据源将业务sql的执行解析成undolog来与业务数据的更新同时入库，达到了对业务无侵入的效果。



##### 第二阶段

如果决议是全局提交，此时分支事务此时已经完成提交，不需要同步协调处理（只需要异步清理回滚日志），第二节段可以非常快速地完成。

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720164033.png" alt="image-20210720164033242" style="zoom:67%;" />

如果决议是全局回滚，RM 收到协调器发来的回滚请求，通过 XID 和 Branch ID 找到相应的回滚日志记录，通过回滚记录生成反向的更新 SQL 并执行，以完成分支的回滚。

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20210720164053.png" alt="image-20210720164052932" style="zoom:67%;" />

