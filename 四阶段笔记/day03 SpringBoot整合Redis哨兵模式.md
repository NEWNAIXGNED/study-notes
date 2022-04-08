### SpringBoot整合Sentinel

注意：

主从复制时slaveof指令后的ip应该为网络ip（虚拟机IP），不要写成127

哨兵配置文件中主机的IP也应该是网络ip（虚拟机IP），不要写127

 

配置哨兵

需要注意的是：所有哨兵的配置文件中一定要关闭保护模式

protected-mode no	#关闭保护模式，保护redis服务器

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211020173929.png" alt="image-20211020173929242" style="zoom:50%;" />

在项目主配置文件中配置哨兵

```yaml
mybatis:
  config-location: classpath:mybatis/mybatis-config.xml
  type-aliases-package: com.woniuxy.main.pojo
  
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/mybatis?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=true
    username: root
    password: root
  redis: 
    #database: 0
    #host: 192.168.41.226  # redis服务器ip
    #password: 123456  # 密码，如果redis没有设置密码，这一行千万别写，不然会报错
    #port: 6379        # 端口
    pool:
      max-active: 8   # 连接池最大连接数（使用负值表示没有限制）
      max-wait: -1    # 连接池最大阻塞等待时间（使用负值表示没有限制）
      max-idle: 8     # 连接池中的最大空闲连接
      min-idle: 1     # 连接池中的最小空闲连接
    timeout: 1000       # 连接超时时间（毫秒）
    sentinel:
      master: mymaster
      nodes: 192.168.41.226:26379,192.168.41.226:26380,192.168.41.226:26381
server:
  port: 8081
```



### 哨兵操作流程

#### 停止服务

停止所有哨兵

停掉所有redis

删除所有从机中最后一行关于主机的配置信息

删除哨兵中所有多余的数据



#### 修改哨兵配置

哨兵配置文件中主机ip一定要用网络ip（虚拟机ip），不要使用127.0.0.1



#### 启动redis

分别启动三台Redis，然后形成主从结构

注意：在使用slaveof指令时，主机ip使用网络ip（虚拟机ip）



#### 启动哨兵

分别启动三台哨兵



#### 开启端口

开启所有redis、哨兵所使用到的端口

firewall-cmd --zone=public --add-port=6379/tcp --permanent

firewall-cmd --zone=public --add-port=6380/tcp --permanent

firewall-cmd --zone=public --add-port=6381/tcp --permanent

firewall-cmd --zone=public --add-port=16379/tcp --permanent

firewall-cmd --zone=public --add-port=16380/tcp --permanent

firewall-cmd --zone=public --add-port=16381/tcp --permanent

sudo service firewalld restart



#### 修改项目配置文件

注释掉redis的host、port配置

添加哨兵配置

```yaml
sentinel:
	master: mymaster
	nodes: 192.168.74.145:16379,192.168.74.145:16380,192.168.74.145:16381
```

