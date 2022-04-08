### Redis的安装（15）

下载地址：http://redis.io/download，下载最新稳定版本。

- 在/opt下创建redis文件夹，并即将redis-5.0.7.tar.gz拷贝到该目录下，同时解压

  mkdir /opt/redis

  cd /opt/redis

  tar -zxvf xxx.ta.gz

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211018171622.png" alt="image-20211018171622439" style="zoom:50%;" />

- 安装gcc编译器

  yum -y install gcc gcc-c++ kernel-devel

如果出现下图所示的错误

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211018171641.png" alt="image-20211018171641105" style="zoom:50%;" />

可能是系统自动升级正在运行，yum在锁定状态中，可以通过输入以下命令强制关闭yum

  rm -f /var/run/yum.pid

 然后在执行yum -y install gcc gcc-c++ kernel-devel指令安装gcc

- 进入redis-5.0.7目录，输入make MALLOC=libc命令进行安装，出现以下结果表示安装成功

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211018171701.png" alt="image-20211018171701309" style="zoom:50%;" />

- 安装redis服务

  make install

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211018171720.png" alt="image-20211018171720272" style="zoom:50%;" />

- 创建存储redis文件目录

  mkdir -p /usr/local/redis

- 复制redis-server redis-cli redis-sentinel到新建立的文件夹

  cp /opt/redis/redis-5.0.7/src/redis-server /usr/local/redis/

  cp /opt/redis/redis-5.0.7/src/redis-cli /usr/local/redis/

   cp /opt/redis/redis-5.0.7/src/redis-sentinel /usr/local/redis

- 将redis配置文件复制一份到redis目录

  cp /opt/redis/redis-5.0.7/redis.conf /usr/local/redis/

- 然后切换到该目录下，编辑redis配置文件

  cd /usr/local/redis/

  vi /usr/local/redis/redis.conf

  在bind 127.0.0.1前加“#”将其注释掉（如果有注释）

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211018171801.png" alt="image-20211018171801171" style="zoom:50%;" />

默认为保护模式，把 protected-mode yes 改为 protected-mode no

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211018171817.png" alt="image-20211018171817717" style="zoom:50%;" />

默认为不守护进程模式，把daemonize no 改为daemonize yes

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211018171830.png" alt="image-20211018171830885" style="zoom:50%;" />

- 启动redis

  redis-server redis.conf

- 通过客户端连接redis

  redis-cli -h 127.0.0.1 -p 6379

### Redis基本配置及通用命令（45）

#### 启动、停止Redis

- 启动redis

  redis-server redis.conf

  检查是否正常与运行

  ps -ef|grep redis

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211018171905.png" alt="image-20211018171905308" style="zoom:50%;" />

- 停止redis

  redis-cli shutdown



#### 常用配置

- daemonize  

指定redis是否为后台运行，为true表示后台运行redis，false表示前台运行redis

 

- port

指定端口号，默认为6379

 

- logfile

指定日志文件，可以记录Redis运行时信息

 

- requirepass

  指定登录密码

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211018171945.png" alt="image-20211018171945567" style="zoom:50%;" />

登录时可以不需要指定密码，能够登录上，但是没有权限进行任何操作

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211018171958.png" alt="image-20211018171958727" style="zoom:50%;" />

可以使用auth指令进行授权

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211018172010.png" alt="image-20211018172010581" style="zoom:50%;" />



#### Redis 客户端的基本语法为：

​	$ redis-cli

在远程服务上执行命令

如果需要在远程 redis 服务上执行命令，同样我们使用的也是 redis-cli 命令。

语法

​	$ redis-cli -h host -p port -a password

实例

以下实例演示了如何连接到主机为 127.0.0.1，端口为 6379 ，密码为 mypass 的 redis 服务上。

​	$redis-cli -h 127.0.0.1 -p 6379 -a "mypass"

​	redis 127.0.0.1:6379>



### redis主从复制（45）

#### 什么是主从复制

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019171458.png" alt="image-20211019171458006" style="zoom:50%;" />

主从复制，是指将一台Redis服务器的数据，复制到其他的Redis服务器。前者称为主节点(master)，后者称为从节点(slave),数据的复制是单向的，只能由主节点到从节点。

默认情况下，每台Redis服务器都是主节点；且一个主节点可以有多个从节点(或没有从节点)，但一个从节点只能有一个主节点。

#### 主从复制的作用

- 数据冗余		主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。

- 故障恢复		当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复；实际上是一种服务的冗余。

- 负载均衡		在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务（即写Redis数据时应用连接主节点，读Redis数据时应用连接从节点），分担服务器负载；尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高Redis服务器的并发量。

- 读写分离		可以用于实现读写分离，主库写、从库读，读写分离不仅可以提高服务器的负载能力，同时可根据需求的变化，改变从库的数量；

- 高可用基石	除了上述作用以外，主从复制还是哨兵和集群能够实施的基础，因此说主从复制是Redis高可用的基础。

 

#### 常见主从结构

- 一主一从：用于主节点故障转移从节点，当主节点的“写”命令并发高且需要持久化，可以只在从节点开启AOF（主节点不需要），这样即保证了数据的安全性，也避免持久化对主节点的影响

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019171600.png" alt="image-20211019171600775" style="zoom:50%;" />

- 一主多从：针对“读”较多的场景，“读”由多个从节点来分担，但节点越多，主节点同步到多节点的次数也越多，影响带宽，也加重主节点的稳定

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019171612.png" alt="image-20211019171612581" style="zoom:50%;" />

- 树状主从：一主多从的缺点（主节点推送次数多压力大）可用些方案解决，主节点只推送一次数据到从节点B，再由从节点B推送到C，减轻主节点推送的压力。

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019171627.png" alt="image-20211019171627037" style="zoom:50%;" />

#### 实例演示

- 在一台电脑上模拟多个redis服务器。

一般至少需要一主两从，将redis.conf拷贝两份，分别打开修改以下内容

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019171645.png" alt="image-20211019171645240" style="zoom:50%;" />

修改以下配置

```
92 		prot	6379
158 	pidfile /var/run/redis_6379.pid
171		logfile "redis6379.log"
253		dbfilename dump6379.rdb
```



- 分别启动三台服务器

  redis-server redis6379.conf

  redis-server redis6380.conf

  redis-server redis6381.conf

- 开启三个终端连接不同的redis服务器

  redis-cli -h 127.0.0.1 -p 6379		或者		redis-cli -p 6379

  redis-cli -h 127.0.0.1 -p 6380		或者		redis-cli -p 6380

  redis-cli -h 127.0.0.1 -p 6381		或者		redis-cli -p 6381

现在三台服务器是互相独立的，没有任何联系

输入info replication 查看当前服务器的角色

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019171805.png" alt="image-20211019171805779" style="zoom:50%;" />

- 在6380、6381下输入：slaveof 127.0.0.1 6379 设置当前服务器的主机是谁

6380

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019171830.png" alt="image-20211019171830496" style="zoom:50%;" />

6379

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019171842.png" alt="image-20211019171842593" style="zoom:50%;" />

- 主机中存放内容时，会自动将数据同步到从机上

6379

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019171904.png" alt="image-20211019171904691" style="zoom:50%;" />

6380/6381

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019171914.png" alt="image-20211019171914705" style="zoom:50%;" />

在从机上不能进行set key value的操作，因为在redis中主从策略为主机做写的操作，从机做读的操作

6380/6381

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019171925.png" alt="image-20211019171925713" style="zoom:50%;" />

当从机关闭然后重新启动时，不会自动变成从机，还是需要再次指定其为从机

6380/6381

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019171943.png" alt="image-20211019171943033" style="zoom:50%;" />

当主机关闭或遇到问题停止运行，然后再次启动之后还是会做为主机，同时与从机保持主从关系

6379

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019171959.png" alt="image-20211019171959397" style="zoom:50%;" />

#### 主从复制原理

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019172013.png" alt="image-20211019172013563" style="zoom:50%;" />

### redis哨兵模式（45）

#### 哨兵模式

当主机在停机期间怎么实现功能能正常使用呢？使用哨兵机制（哨兵模式）

主从切换技术的方法是：当主服务器宕机后，需要手动把一台从服务器切换为主服务器，这就需要人工干预，费事费力，还会造成一段时间内服务不可用。这不是一种推荐的方式，更多时候，我们优先考虑哨兵模式。

##### 哨兵模式概述

哨兵模式是一种特殊的模式，首先Redis提供了哨兵的命令，哨兵是一个独立的进程，作为进程，它会独立运行。其原理是哨兵通过发送命令，等待Redis服务器响应，从而监控运行的多个Redis实例。

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019172038.png" alt="image-20211019172038879" style="zoom:50%;" />

这里的哨兵有两个作用

- 通过发送命令，让Redis服务器返回监控其运行状态，包括主服务器和从服务器。

- 当哨兵监测到master宕机，会自动将slave切换成master，然后通过发布订阅模式通知其他的从服务器，修改配置文件，让它们切换主机。

然而一个哨兵进程对Redis服务器进行监控，可能会出现问题，为此，我们可以使用多个哨兵进行监控。各个哨兵之间还会进行监控，这样就形成了多哨兵模式。

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019172058.png" alt="image-20211019172058582" style="zoom:50%;" />

用文字描述一下故障切换（failover）的过程。假设主服务器宕机，哨兵1先检测到这个结果，系统并不会马上进行failover过程，仅仅是哨兵1主观的认为主服务器不可用，这个现象成为主观下线。当后面的哨兵也检测到主服务器不可用，并且数量达到一定值时，那么哨兵之间就会进行一次投票，投票的结果由一个哨兵发起，进行failover操作。切换成功后，就会通过发布订阅模式，让各个哨兵把自己监控的从服务器实现切换主机，这个过程称为客观下线。这样对于客户端而言，一切都是透明的。

 

为了方便测试，请将redis6379.conf、redis6380.conf、redis6381.conf中bind 127.0.0.1中的IP地址改为0.0.0.0

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019172111.png" alt="image-20211019172111102" style="zoom:50%;" />

##### 单个哨兵配置操作步骤

- 所有redis配置文件中的bind 127.0.0.1中的IP地址改为0.0.0.0

- 分别在所有的redis配置文件中搜索查看是否包含有 slave xxx.xxx.xxx.xxx的配置，如果有则删除。搜索字符串方式：在命令模式下输入”/要搜索的字符串”，按n搜索下一个

- 启动主机6379

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019172147.png" alt="image-20211019172146924" style="zoom:50%;" />

启动从机6380

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019172157.png" alt="image-20211019172157594" style="zoom:50%;" />

启动从机6381

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019172209.png" alt="image-20211019172209103" style="zoom:50%;" />

分别在从机6380、6381中设置主机

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019172221.png" alt="image-20211019172221283" style="zoom:50%;" />

- 在redis目录下创建sentinel6379.conf，并添加以下内容

```
protected-mode no				# 关闭保护模式，方便测试
port 26379						# 哨兵的端口
sentinel monitor mymaster 192.168.41.226 6379 1		# 192.168.41.226：主机ip 6379：端口 1：至少几个哨兵认为主机下线时进行故障切换
```

注意：不要添加注释

- 输入redis-sentinel sentinel6379.conf 启动哨兵，看到以下界面表示成功

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019172325.png" alt="image-20211019172325633" style="zoom:50%;" />

此时哨兵已经对主机以及从机进行监控

- 在主机6379中输入shutdown命令关闭主机，然后注意观察哨兵的反应（反应会有点慢）

- 哨兵

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019172343.png" alt="image-20211019172343834" style="zoom:50%;" />

可以看到现在的主机已经变成6381了，我们可以在6381上查看一下自己的身份

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019172355.png" alt="image-20211019172355607" style="zoom:50%;" />

可以看到身份已经是master了

那么如果现在原来的主机6379又重新启动，会是什么样的情况呢？

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019172408.png" alt="image-20211019172408476" style="zoom:50%;" />

可用看到原来的主机现在变为从机了

保存，并重新启动6379

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019172421.png" alt="image-20211019172421310" style="zoom:50%;" />

此时查看6380的身份，可以看到6379已经成为它的从机了

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019172434.png" alt="image-20211019172433972" style="zoom:50%;" />

##### 多哨兵操作步骤：

- 拷贝两份sentinel6379.conf，分别为命名为sentinel6380.conf、sentinel6380.conf

- 分别打开两个文件，修改sentinel的运行端口为26380、26381，同时将选举哨兵数修改成2

- 分别运行两个哨兵



### 缓存穿透（15）

#### 产生原因

程序在处理缓存时，一般是先从缓存查询，如果缓存没有这个key获取为null，则会从DB中查询，并设置到缓存中去。

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019172511.png" alt="image-20211019172511807" style="zoom:33%;" />

按这种做法，那查询一个一定不存在的数据值，由于缓存是不命中时需要从数据库查询，查不到数据则不写入缓存，这将导致这个不存在的数据每次请求都要到数据库去查询，造成缓存穿透。

 

#### 解决办法

- 最好对于每一个缓存key都有一定的规范约束，这样在程序中对不符合parttern的key 的请求可以拒绝。（但一般key都是通过程序自动生成的）

- 将可能出现的缓存key的组合方式的所有数值以hash形式存储在一个很大的bitmap中<布隆过滤器>（需要考虑如何将这个可能出现的数据的hash值之后同步到bitmap中，后端每次新增一个可能的组合就同步一次，或者穷举），一个一定不存在的数据会被这个bitmap拦截掉，从而避免了对底层存储系统的查询压力

- 常用：如果对应在数据库中的数据都不存在，我们将此key对应的value设置为一个默认的值，比如“NULL”，并设置一个缓存的失效时间。当然这个key的时效比正常的时效要小的多

 

### 缓存雪崩（15）

#### 产生原因

指的是大量缓存集中在一段时间内失效，发生大量的缓存穿透，所有的查询都落在数据库上，造成了缓存雪崩。

正常情况

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019172550.png" alt="image-20211019172550805" style="zoom:33%;" />

当大量数据失效或者redis宕机

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019172601.png" alt="image-20211019172601397" style="zoom:33%;" />

#### 解决办法

- 这个没有完美解决办法，但可以分析用户行为，尽量让失效时间点均匀分布，设置不同的过期时间。缓存的过期时间用随机值，尽量让不同的key的过期时间不同（例如：定时任务新建大批量key，设置的过期时间相同）

- 可以把缓存层设计成高可用的，即使个别节点、个别机器、甚至是机房宕掉，依然可以提供服务。利用sentinel或cluster实现。

- 采用多级缓存，本地进程作为一级缓存，redis作为二级缓存，不同级别的缓存设置的超时时间不同，即使某级缓存过期了，也有其他级别缓存兜底

 

### 缓存击穿（15）

#### 产生原因

缓存击穿是指一个key非常热点，在不停的扛着大并发，大并发集中对这一个点进行访问，当这个key在失效的瞬间，持续的大并发就穿破缓存，直接请求数据库，瞬间对数据库的访问压力增大。

缓存击穿这里强调的是并发，造成缓存击穿的原因有以下两个：

- 该数据没有人查询过 ，第一次就大并发的访问。（冷门数据）

- 添加到了缓存，reids有设置数据失效的时间 ，这条数据刚好失效，大并发访问（热点数据）

 

对于缓存击穿的解决方案就是加锁，具体实现的原理图如下：

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019172643.png" alt="image-20211019172643238" style="zoom:50%;" />

当用户出现大并发访问的时候，在查询缓存的时候和查询数据库的过程加锁，只能第一个进来的请求进行执行，当第一个请求把该数据放进缓存中，接下来的访问就会直接集中缓存，防止了缓存击穿。

 

业界比价普遍的一种做法，即根据key获取value值为空时，锁上，从数据库中load数据后再释放锁。若其它线程获取锁失败，则等待一段时间后重试。



#### 解决办法

- 与缓存雪崩的解决方法类似： 用加锁或者队列的方式保证缓存的单线程（进程）写，在加锁方法内先从缓存中再获取一次，没有再查DB写入缓存。 

 

- 还有一种比较好用的（针对缓存雪崩与缓存击穿）：

  物理上的缓存是不设置超时时间的（或者超时时间比较长），但是在缓存的对象上增加一个属性来标识超时时间（此时间相对小）。当获取到数据后，校验数据内部的标记时间，判定是否快超时了，如果是，异步发起一个线程（控制好并发）去主动更新该缓存。

  这种方式会导致一定时间内，有些请求获取缓存会拿到过期的值，看业务是否能接受而定。

 

### 淘汰策略（15）

#### 热点数据

在现今的电商平台中都有会有些大卖的商品，我们称之为简爆品。这些商品会有个特点，就是访问量(查询)特别大。我们专业上面可以称之为热点数据，在处理这些热点商品时，系统需要做一些特殊的处理。我们常见的处理方式就是将这些数据放到redis缓存中去，但是redis是一种内存数据库，而内存的容量又是有限的。随着业务量的增长，我们放在Redis里面的数据越来越多了，导致内存严重不足，这个时候就需要我们将Redis缓存中一些不是那么常用的数据移除掉，为了更好的利用内存，使Redis存储的都是缓存的热点数据，Redis设计了相应的内存淘汰机制（也可以叫做缓存淘汰机制）。

 

#### 开启淘汰机制

修改redis缓存大小，单位是字节

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019172733.png" alt="image-20211019172733217" style="zoom:50%;" />

内存淘汰的过程

- 首先，客户端发起了需要申请更多内存的命令（如set）。

- 然后，Redis检查内存使用情况，如果已使用的内存大于maxmemory则开始根据用户配置的不同淘汰策略来淘汰内存（key），从而换取一定的内存。

- 最后，如果上面都没问题，则这个命令执行成功。

提示：maxmemory为0的时候表示我们对Redis的内存使用没有限制。



##### 淘汰策略

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019172806.png" alt="image-20211019172806084" style="zoom:50%;" />

LRU（Least Recently Used）

| 策略            | 解释                                                         |
| --------------- | ------------------------------------------------------------ |
| volatile-lru    | 从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰。注意：redis并不是保证取得所有数据集中最近最少使用的键值对，而只是随机挑选的几个键值对中的， 当内存达到限制的时候无法写入非过期时间的数据集。 |
| volatile-ttl    | 从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰。注意：redis 并不是保证取得所有数据集中最近将要过期的键值对，而只是随机挑选的几个键值对中的， 当内存达到限制的时候无法写入非过期时间的数据集。 |
| volatile-random | 从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰。当内存达到限制的时候无法写入非过期时间的数据集。 |
| allkeys-lru     | 从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰。当内存达到限制的时候，对所有数据集挑选最近最少使用的数据淘汰，可写入新的数据集。 |
| allkeys-random  | 从数据集（server.db[i].dict）中任意选择数据淘汰，当内存达到限制的时候，对所有数据集挑选随机淘汰，可写入新的数据集。 |
| no-enviction    | 当内存达到限制的时候，不淘汰任何数据，不可写入任何数据集，所有引起申请内存的命令会报错。 |

##### 淘汰策略选择

根据使用经验, 一般来说回收策略可以这样来配置:

- allkeys-lru：如果期望用户请求呈现幂律分布(power-law distribution)，也就是，期望一部分子集元素被访问得远比其他元素多时，可以使用allkeys-lru策略。在你不确定时这是一个好的选择。

- allkeys-random：如果期望是循环周期的访问，所有的键被连续扫描，或者期望请求符合平均分布(每个元素以相同的概率被访问)，可以使用allkeys-random策略。

- volatile-ttl：如果你期望能让 Redis 通过使用你创建缓存对象的时候设置的TTL值，确定哪些对象应该是较好的清除候选项，可以使用volatile-ttl策略。

另外值得注意的是，为键设置过期时间需要消耗内存，所以使用像allkeys-lru这样的策略会更高效，因为在内存压力下没有必要为键的回收设置过期时间。



##### 淘汰个数

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019172901.png" alt="image-20211019172901697" style="zoom:50%;" />



### 删除策略（15）

数据删除策略有三种

- 被动删除：只有key被操作时(如GET)，Redis才会被动检查该key是否过期，如果过期则删除之并且返回NIL。

- 主动删除：定期删除过期的数据

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20211019172923.png" alt="image-20211019172923589" style="zoom:50%;" />

默认是10，表示serverCron每秒执行10次

注意：hz调大将会提高Redis主动淘汰的频率，如果Redis存储中包含很多冷数据占用内存过大的话，可以考虑将这个值调大，但Redis作者建议这个值不要超过100。

- 当前已用内存超过maxmemory限定时，触发数据淘汰策略

 

#### 被动删除特点

- 这种删除策略对CPU是友好的，删除操作只有在不得不的情况下才会进行，不会其他的expire key上浪费无谓的CPU时间。

- 但是这种策略对内存不友好，一个key已经过期，但是在它被操作之前不会被删除，仍然占据内存空间。如果有大量的过期键存在但是又很少被访问到，那会造成大量的内存空间浪费。



### 集群

从redis 3.0之后版本支持redis-cluster集群，Redis-Cluster采用无中心结构，每个节点保存数据和整个集群状态,每个节点都和其他所有节点连接。

sentinel模式基本可以满足一般生产的需求，具备高可用性。但是当数据量过大到一台服务器存放不下的情况时，主从模式或sentinel模式就不能满足需求了，这个时候需要对存储的数据进行分片，将数据存储到多个Redis实例中。cluster模式的出现就是为了解决单机Redis容量有限的问题，将Redis的数据根据一定的规则分配到多台机器。

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220322172356.png" alt="image-20220322172356353" style="zoom:80%;" />

其结构特点：

- redis是一个开源的key value存储系统，受到了广大互联网公司的青睐。redis3.0版本之前只支持单例模式，在3.0版本及以后才支持集群，我这里用的是redis5.0.7版本；

- redis集群采用P2P模式，是完全去中心化的，不存在中心节点或者代理节点；

- redis集群是没有统一的入口的，客户端（client）连接集群的时候连接集群中的任意节点（node）即可，集群内部的节点是相互通信的（PING-PONG机制），每个节点都是一个redis实例；

- 为了实现集群的高可用，即判断节点是否健康（能否正常使用），redis-cluster有这么一个投票容错机制：如果集群中超过半数的节点投票认为某个节点挂了，那么这个节点就挂了（fail）。这是判断节点是否挂了的方法；

- 那么如何判断集群是否挂了呢? 如果集群中任意一个节点挂了，而且该节点没有从节点（备份节点），那么这个集群就挂了。这是判断集群是否挂了的方法；

- 那么为什么任意一个节点挂了（没有从节点）这个集群就挂了呢？因为集群内置了16384个slot（哈希槽），并且把所有的物理节点映射到了这16384[0-16383]个slot上，或者说把这些slot均等的分配给了各个节点。当需要在Redis集群存放一个数据（key-value）时，redis会先对这个key进行crc16算法，然后得到一个结果。再把这个结果对16384进行求余，这个余数会对应[0-16383]其中一个槽，进而决定key-value存储到哪个节点中。所以一旦某个节点挂了，该节点对应的slot就无法使用，那么就会导致集群无法正常工作。

- 综上所述，每个Redis集群理论上最多可以有16384个节点。。

 

#### 集群实现

##### 集群搭建需要的环境

- Redis集群至少需要3个节点，因为投票容错机制要求超过半数节点认为某个节点挂了该节点才是挂了，所以2个节点无法构成集群。

- 要保证集群的高可用，需要每个节点都有从节点，也就是备份节点，所以Redis集群至少需要6台服务器。因为我没有那么多服务器，也启动不了那么多虚拟机，所在这里搭建的是伪分布式集群，即一台服务器虚拟运行6个redis实例，修改端口号为（8001-8006），当然实际生产环境的Redis集群搭建和这里是一样的。

 

##### 实现步骤

说明：针对5.0版本以后的版本，之前的版本需要先单独去安装ruby，但是很麻烦

- 在/usr/local/redis下创建redis-cluster目录

- 拷贝redis6379到redis-cluster中并命名为redis8001.conf

- 修改redis8001.conf中的端口号

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220322172548.png" alt="image-20220322172548183" style="zoom:80%;" />

- 修改pidfile

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220322172603.png" alt="image-20220322172602941" style="zoom:80%;" />

- 修改日志文件

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220322172615.png" alt="image-20220322172615249" style="zoom:80%;" />

- 修改rdb文件

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220322172628.png" alt="image-20220322172628140" style="zoom:67%;" />

- 开启aof

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220322172640.png" alt="image-20220322172640803" style="zoom:80%;" />![image-20220322172657203](https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220322172657.png)

- 指定aof文件名

![image-20220322172657203](https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220322172657.png)

- 开启集群

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220322172712.png" alt="image-20220322172712784" style="zoom:80%;" />

- 指定nodes配置文件

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220322172724.png" alt="image-20220322172724671" style="zoom:80%;" />

- 将8001拷贝5份，分别为8002-8006,

- 依次修改8002-8006配置文件信息，修改的内容同8001



分别启动8001-8006显得十分麻烦，因此编写启动文件，用于一次性启动所redis

在/user/local/redis下创建start_all.sh文件，在文件中添加入下内容

```sh
redis-server redis-cluster/redis8001.conf
redis-server redis-cluster/redis8002.conf
redis-server redis-cluster/redis8003.conf
redis-server redis-cluster/redis8004.conf
redis-server redis-cluster/redis8005.conf
redis-server redis-cluster/redis8006.conf
ps -ef|grep redis
```

保存退出，为start_all.sh文件添加执行权限

chmod +x start_all.sh

 

控制台输入./start_all

看到以下界面表示所有redis启动成功

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220322172806.png" alt="image-20220322172806788" style="zoom:80%;" />

- 输入以下命令创建集群

```
redis-cli --cluster create 192.168.242.129:8000 192.168.242.129:8001 192.168.242.129:8002 192.168.242.129:8003 192.168.242.129:8004 192.168.242.129:8005 --cluster-replicas 1
```

**注：ip地址为自己虚拟机的ip地址**

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220322172909.png" alt="image-20220322172909789" style="zoom:80%;" />

输入yes继续

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220322172920.png" alt="image-20220322172920438" style="zoom:80%;" />

出现此界面表示集群创建成功

- 登录8001，设置key-value，**需要注意的是：redis-cli连接redis时需要使用-c参数来启动集群模式**

  redis-cli -c -p 8001

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220322173021.png" alt="image-20220322173021036" style="zoom:80%;" />

- 使用同样的方式登录8003，获取key对应的value

<img src="https://woniumd.oss-cn-hangzhou.aliyuncs.com/java/xiangwei/20220322173035.png" alt="image-20220322173035756" style="zoom:80%;" />

如果能获得值，表示集群已经开启，key正常使用了



如果想要一次性关闭所有redis，key在/usr/local/redis下创建stop_all.sh文件，并添加如下内容

```shell
#!/bin/bash
PORT=8001
ENDPORT=8006
while [ $((PORT <= ENDPORT)) != "0" ]; do
   echo "Stopping Redis $PORT"
   redis-cli -p $PORT shutdown
   PORT=$((PORT+1))
done
echo "done"
exit 0
```

添加执行权限

chmod +x stop_all.sh







