redis

### NoSql

#### 概念

+ not noly sql，不仅仅是sql
+ 泛指非关系型数据库，也就是数据之间**不再依靠关系模型**来组织

#### 特点

+ 数据之间没有关系，方便扩展
+ 数据量大、高性能（redis一秒读11w，写8w）
+ 数据模型灵活，不需提前设计数据库

#### 分类

+ 基于kv键值对的，redis、memacache
  + redis、memacache都是基于内存的
+ 文档型数据库，MongoDb
  + MongoDb是一个基于分布式文件存储的数据库，主要用来处理大量文档
  + MongoDB是一个介于关系型数据库和非关系型数据库中间的产品，是最像关系型数据库的非关系型数据库
+ 列存储数据库，Hbase、Cassandra
+ 图关系数据库，Neo4j

![image-20201117104405775](images.assets\image-20201117104405775.png)

### redis基础

+ 官网：https://redis.io/  
+ 中文网：http://redis.cn/

#### 基础知识

+ 全称 remote dictionary server，远程字典服务，基于kv键值对
+ redis默认有16个数据库，通过`select n`选择数据库
+ redis是**单线程**，**基于内存**操作，cpu不是性能瓶颈，机器的内存和网络带宽才是

#### 用途

+ 数据库
+ 高速缓存
+ 发布订阅系统（消息队列）
+ 地图信息分析
+ 计数器

#### redis-key

+ redis中无论什么数据类型都是以kv键值对形式存储

+ key的基本操作

  ```bash
  keys *       #查看当前数据库所有的key
  exists key   #判断键是否存在
  del key      #删除某个key 
  move key db  #将某个key移动到某个数据库
  expire key second #设置某个key的过期时间
  ttl key      #查看某个key的剩余过期时间
  type key     #查看某个key的类型
  
  flushdb      #清空当前数据库的所有key
  flushall     #清空所有数据库的所有key
  ```

  

### 五大基础数据类型

#### 字符串 String

+ 底层数据结构是(简单动态字符串)SDS，以字节数组形式保存字符串将其长度

```bash
set key value 
get key
getset key value    #先get再set
del key             #删除

setnx key value     #当key不存在时，才设置value
setex key second value #设置value及其有效期

getrange key start end #按起止位置获取字符串（闭区间，起止位置都取，0 -1 则为all）
setrange key offset value #从指定下标开始替换字符串，但不改变字符串长度

mset k1 v1 k2 v2    #批量set
mget k1 v1 k2 v2    #批量get

msetnx k1 v1 k2 v2  #当不存在时批量set，原子操作

apppend key value   #追加字符串
strlen key          #获取字符串的长度

#value为整数时
incr/decr key       #自增自减
inceby/secrby key n #按步长增减

#设置对象
mset user:1:name ly user:1:age 18
```

#### 列表 list

+ redis中list，本质上是一个双向链表，是可以双向操作得，命令分成 lxx和rxx两类
+ list为空时会自动删除list

```bash
lpush/rpush key value1 value2 #从左边/右边向列表中PUSH值(一个或者多个)
lpushx/rpushx key value1 value2 #只有当列表中已有数据时，才放入数据

lpop/rpop key  #从左边/右边在列表中取值，并删除

linsert key before/after pivot value #在指定列表元素的前/后 插入value

llen key  #查看长度

lindex key index #通过索引获取列表元素
lset key index value #通过索引为元素设值

lrange key start end #获取列表中从start到end的几个值
ltrim key start end  #截取列表中，start到end的值，其他数值丢掉

lrem key count value #从前到后删除列表中，count个值为value的元素

RPOPLPUSH source destination #将列表的尾部(右)最后一个值弹出，并返回，然后加到另一个列表的头部
```

可以用作**消息排队、消息队列、栈**

#### 集合 set

+ set是**去重的无序**集合，成员唯一
+ 通过哈希表来实现的
+ 所有命令以 s 开头

```bash
sadd key v1 v2 v3    #向集合中增加元素
srem key v1          #删除集合中的v1元素

spop key count       #随机移除并返回集合中count个成员
srandmember key count#随机返回集合中count个成员，count缺省值为1

smove sour dest v1   #将sour集合的成员v1移动到dest集合

scard key            #获取集合中元素的个数
smembers key         #获取集合中所有的元素
sismember key value  #判断集合中是否存在某个值

sdiff key1 key2      #返回key1、key2的差集
sinter key1 key2     #返回key1、key2的交集
sunion key1 key2     #返回key1、key2的并集

```

#### 哈希 hash

+ 一个**key和一个field共同映射一个value**，特别适合**存储对象**。
+ key作为对象，field作为属性，value作为属性的值
+ 命令以 h 开头

```bash
hset key field value    #将哈希表 key 中的字段 field 的值设为 value 
hmset key f1 v1 f2 v2   #设置多个
hsetnx key field value  #不存在时才设置
hget key field
hmget key f1 f2

hlen key                #获取key下属性的数量
hgetall key             #获取key下所有属性和值
hkeys key               #获取key下所有属性
hvals key               #获取key下所有值

hexists key field       #判断是否存在
hdel key f1 f2          #删除key下的属性
```

#### 有序集 zset

+ 与集合set相似，但是zset每个元素都会关联一个double类型的分数（score）
+ 正是通过这个分数来实现集合中成员的排序
+ 集合中成员不能相同，但分数可以相同
+ 当分数相同时，按字典序排序
+ 所有命令都是以 z 开头

```bash
zadd key score1 member1 score2 member2 #向有序集合添加一个或多个成员，或者更新已存在成员的分数
zcard key          #有序集的大小
zcount key min max #计算在有序集合中指定区间score的成员数
zincrby key n member #对指定成员的分数加上增量 n
zscore key member #获取指定成员的分数

zrange key start end #返回分数在该区间内的成员
zrangebyscore key min max #通过分数返回有序集合指定区间内的成员(-inf +inf)

zrem key member1 member2 #删除元素
```

### 三种特殊数据类型

#### geospatial 

+ 地理位置
+ 用一个有序集合zset保存经纬度坐标
+ 可以用来实现 查找**附近的人**的功能

```bash
geoadd key longitude latitude member #将具体经纬度的坐标存入一个有序集合

geopos key member1 member2 #获取集合中的一个/多个成员坐标

geodist key member1 member2 [unit]#返回两个给定位置之间的距离,默认以米作为单位。

georadius key longitude latitude radius m|km|mi|ft [WITHCOORD][WITHDIST] [COUNT count]  
#以给定的经纬度为中心， 返回集合包含的位置元素当中， 与中心的距离不超过给定最大距离的所有位置元素

georadiusbymember key member radius #以数据集中某个点为中心
```

#### Hyperloglog

+ 基数统计（数据集中不重复元素的个数）
+ 使用很小的内存空间，计算基数
  + （12kb内存，可以计算接近2^64个不同元素的基数）
+ 数据量较大时可能有一定的误差

```bash
pfadd key v1 v2 v3 v4  #添加元素到 HyperLogLog 中
pfcount key            #返回基数
pfmerge key key1 key2  #将key1、key2合并成一个key Hyperloglog 
```

#### BitMap

+ 位图，使用位存储，信息状态只有 0 和 1
+ bitmap是一串连续的二进制数字，每一位所在的位置为偏移(offset)
+ 每一位所在的位置为偏移(offset)
+ 常用于**签到统计、状态统计**
+ 也可以用来**无差统计基数**

```bash
setbit key offset value             #为指定key的offset位设置值
gitbit key offset                   #获取指定key的offset位的值
bitcount key [start end]            #计算为1的个数

```

### 事务

#### 本质

+ redis事务是，一组命令的集合。放到队列中，然后统一执行
+ 每条命令会被序列化，保证执行顺序，不被其他命令干扰
+ 满足 一次性、顺序性、排他性
+ **不满足原子性，无隔离级别**

注意：redis事务不满足原子性、但单条命令满足原子性

#### 操作过程

+ 开启事务  `multi`
+ 命令入队
+ 执行事务 `exec`

##### 注意

+ 事务中的命令在加入时都没有被执行，直到提交时才会开始执行(Exec)一次性完成
+ 取消事务 `discard`

#### 事务错误

+ 语法错误（键入命令报错），所有命令全不执行
+ 逻辑错误（键入不报错），其他命令正常执行

#### 监控

+ `watch key` 监控某个值有没有被修改，没修改事务可正常执行，修改就不能执行
+ 相当于**乐观锁**的操作
+ 可以通过 `unwatch` 进行解锁
+ 执行事务后都会自动释放锁，无论是否成功

### redis.conf

+ redis核心配置文件，涉及到redis的配置和优化

#### 网络配置

```properties
bind 127.0.0.1      #绑定ip端口，注释掉才能远程访问redis
protected-mode yes  #保护模式，禁止公网访问redis
port 6379           #配置端口

```

#### 通用配置

```properties
daemonize yes       #守护线程，后台运行
loglevel notice     #日志级别，debug、verbose、notice、warning
logfile ""          #日志文件位置
databases 16        #数据库数量
```

#### 持久化规则

通过**RDB或AOF**进行持久化

```properties
save 900 1
save 300 10
save 60 10000
# 分别表示 900 秒（15 分钟）内有 1 个更改，300 秒（5 分钟）内有 10 个更改以及 60 秒内有 10000 个更改。就持久化到文件
```

##### RDB

```properties
dbfilename dump.rdb   #生成rdb文件位置
rebcompression yes    #压缩rdb文件
dir ./                #文件保存路径
```

##### AOF

```properties
appendonly no                     #默认关闭aof
appendfilename "appendonly.aof"   #aof文件名
appendfsync everysec              #同步规则，always、everysec、no
```

#### 密码

```properties
requirepass ""                    #设置密码
```

也可以通过命令行设置

```bash
config get requirepass            #查看密码
config set requirepass "123456"   #设置密码
```

### 持久化

+ redis是基于内存的数据库，断电数据丢失所以需要持久化
+ 常用持久化方式有两种：**RDB和AOF**
+ **默认RDB**的方式

#### RDB

+ 全称 redis database
+ 主要原理：
  + 持久化时，将内存中的数据集快照写入dump.rdb的二进制文件中；
  + 在恢复时，直接读取快照文件，进行数据的恢复 ；

##### 工作机制

+ 在进行rdb时，redis主线程会 fork 一个子线程将数据写入到临时rdb文件中
+ 当写入完成后，redis会用新的rdb文件替换原来的rdb文件
+ 主线程不会进行IO操作，依旧可以接受来自客户端的请求

##### 触发机制

+ redis.conf文件中，save规则满足
+ 执行flushall、flushdb命令，会触发rdb
+ 退出redis
+ 客户端发出`save`命令，也会立刻持久化（阻塞式的）
+ 客户端发出`bgsave`命令，也会后台持久化，但是不阻塞

##### 优缺点

+ 数据恢复速度快，适合大规模数据恢复
+ 可能会丢失最后一次的数据，数据完整性不高

#### AOF

+ 全称：Append only file
+ 主要原理
  + 持久化时，将所有的写操作都以日志的形式记录下来
  + 恢复时，逐条执行这些写操作

##### 工作机制

默认不开启，需要手动配置

```properties
appendonly no                     #默认关闭aof
appendfilename "appendonly.aof"   #aof文件名
```

![image-20201117211931480](images.assets\image-20201117211931480.png)

+ 每次会将写入命令追加到aof文件的末尾
+ aof文件会越来越庞大，超过一定阈值将会触发重写操作（将命令精简，多条合成一条）

注意：恢复数据时，如果AOF文件出错，redis将无法启动。此时可以通过`redis-check-aof --fix`自动校验修改aof文件

##### 触发机制

```properties
appendfsync everysec           #同步规则，always、everysec、no
```

+ 在redis.conf配置文件中，修改同步规则

  ```
  always       #每次写操作
  everysec     #每秒一次
  no           #永不
  ```

##### 优缺点

+ 可以保证数据的完整性
+ 但AOF文件远大于RDB文件，且速度远慢于rdb

#### 小结

+ redis默认用RDB方式持久化
+ 如果两个都开启，优先使用AOF恢复数据，因为AOF数据完整性更好

### 发布与订阅

+ redis发布订阅是一种消息通信模式，**发送者发送消息，订阅者接收消息**

![image-20201119135631673](images.assets\image-20201119135631673.png)

#### 命令

##### 订阅端

```bash
#通过通配符订阅多个频道
psubscribe pattern      #订阅一个或多个符合给定模式的频道，
# 比如说 psubscribe news.* 则订阅所有以news.开头的频道
punsubscribe pattern    #退订符合该模式的频道

#通过频道名订阅频道
subscribe channel      #订阅给定的频道
unsubscribe channel    #退订
```

##### 发送端

```bash
publish channel message  #向指定频道发送指定消息
pubsub channels          #列出活跃的频道（至少有一个订阅者的频道）
```

#### 原理

+ redis维护一个pubsub_channels的字典，用来保存频道及其订阅的信息
+ 其中字典的键则是**频道名**，值是一个保存了所有**订阅者的链表**
+ 用户订阅时，则放到链表尾部；退订则将其移除

![image-20201119140031259](images.assets\image-20201119140031259.png)

#### 缺点

+ 数据可靠性无法保证，无法判断数据是否被接受
+ 扩展性太差，当发布者的消息很多，订阅者来不及处理，会导致redis速度变慢
+ 资源消耗较高，订阅者需要单独占用一个redis连接

#### 应用

+ 消息订阅，公众号，关注
+ 多人聊天

### 主从复制

#### 概念

+ 将主节点的数据复制到其他节点。
+ 数据复制是单向的，不能从从节点到主节点
+ 没有配置的情况下，每台redis服务器默认自己为主节点

#### 作用

+ 数据冗余，主从复制实现了数据的备份，安全性更高
+ 故障恢复，主节点故障时，从节点可代替主节点工作
+ 负载均衡，主从复制实现读写分离，分担服务器的负担
+ 高可用，主从复制是集群搭建的基石

#### 配置

##### conf文件配置

+ 可以在redis.conf文件中配置，replication模块中

  ```properties
  slaveof ip port  #将当前redis服务器作为从机，并绑定主机的地址和端口
  masterauth <password> #从机访问主机的密码
  ```

##### 命令行配置

```bash
slaveof ip port  #将当前redis服务器作为从机，并绑定主机的地址和端口
```

##### 注意

+ 通过命令行配置的主从结构，会在redis重启后消失
+ 通过conf文档配置的则不会消失

#### 原理

+ 主从复制分成三个阶段：建立连接、数据同步、命令传播

##### 建立连接

+ 在执行`slaveof`命令后，从机根据主机的ip和port，**创建连向主服务器socket**，连接成功后，从服务器为这个socket关联一个专门处理器
+ 连接成功后，向主机**发送`ping`命令**，确认主机是否可用；如果收到主机回复的`pong`，则连接可用，否则断开重新连接
+ 身份验证，如果主机设置了密码，则还需要**验证从机的身份**
+ 身份验证完成后，从机会向主机发送自己监听的端口，**主机保存端口**信息

##### 数据同步

+ 双方确认各自身份后，开始数据同步，从机向主机发送`psync`命令，执行同步操作
+ 同步操作分成：完整重同步和部分重同步

###### 完整重同步

+ 从机第一次连接上主节点会执行完整重同步，主从断线重连接也有可能完整重同步
+ 详细过程
  + 从机向主机发送同步命令
  + 主机接收到同步命令，执行`bgsave`生成快照文件，并使用缓冲区记录此后所有写命令
  + `bgsave`执行完后，主机向从机发送快照，在发送期间继续记录写命令
  + 从机接收到快照文件丢弃所有旧数据，并载入快照
  + 主机向从机发送缓冲区中的写命令
  + 从机完成快照载入，执行来自主机缓冲区的写命令

###### 部分重同步

+ 主要处理断线重连接
  + runid，主机的运行id，标识当前节点
  + offset，复制偏移量，主从各自维护自己的offset，记录当前传输接受到哪里
  + replication backlog buffer，复制积压缓冲区。固定长度的FIFO队列，默认大小1M。有且仅有一个，所有slave共享，其作用在于备份最近主库发送给从库的数据
+ 当从机不是第一次连接到主机，会执行`psync runid offset`发送给主机，主机就可以只发送从机缺少的一部分，此时就是部分重同步。
+ 但是如果runid不对，或者复制积压缓冲区中没有足够的命令，就会进行完整重同步

![image-20201119151324366](images.assets\image-20201119151324366.png)

##### 命令传播

+ 数据同步完成后，主从数据一致，当主机执行了客户端的写命令，就会向从服务器发送同样的写命令
+ 从机通过每秒一次的心跳机制，检测主从的链接状态与命令传播情况

#### 哨兵模式

##### 概念

+ 当主机宕机后，需要手动把一台从机切换为主机，这就需要人工干预，费事费力，还会造成一段时间内服务不可用
+ 哨兵模式会，在主机挂掉之后，自动选举一个从机成为主机

##### 优缺点

+ 主节点挂掉之后可以自行切换，系统可用性更高
+ 和主从复制一样，redis容量受限于单机配置；还需要额外资源来启动哨兵

#### Cluster模式

+ Cluster模式实现了Redis的分布式存储，即每台节点存储不同的内容，来解决在线扩容的问题

##### 特点

+ 无中心结构
+ 所有redis节点两两相连，内部通过二进制协议优化传输速度和带宽
+ 节点的fail是通过集群中超过半数的节点检测失效时才生效
+ 客户端与redis节点直连,不需要中间代理层.客户端不需要连接集群所有节点,连接集群中任何一个可用节点即可

##### 机制

+ 在Redis的每个节点上，都有一个插槽（slot），取值范围为0-16383
+ 当我们存取key的时候，Redis会根据CRC16的算法得出一个结果，然后把结果对16384求余数，这样每个key都会对应一个编号在0-16383之间的哈希槽，通过这个值，去找到对应的插槽所对应的节点，然后直接自动跳转到这个对应的节点上进行存取操作
+ 为了保证高可用，Cluster模式也引入主从复制模式，一个主节点对应一个或者多个从节点，当主节点宕机的时候，就会启用从节点
+ 当其它主节点ping一个主节点A时，如果半数以上的主节点与A通信超时，那么认为主节点A宕机了。如果主节点A和它的从节点都宕机了，那么该集群就无法再提供服务了

#### 总结

+ 其中主从复制模式能实现读写分离，但是不能自动故障转移
+ 哨兵模式基于主从复制模式，能实现自动故障转移，达到高可用，但与主从复制模式一样，不能在线扩容，容量受限于单机的配置
+ Cluster模式通过无中心化架构，实现分布式存储，可进行线性扩展，也能高可用，但对于像批量操作、事务操作等的支持性不够好

### 缓存穿透、击穿、雪崩

#### 缓存穿透

查不到的数据

##### 概念

+ 查询缓存中和数据库中都没有的数据，这样每次请求都会绕开缓存直接查询数据库，请求量过大救护导致数据库宕机

##### 解决

+ 空缓存，数据库中查不到的数据，可以设置为空缓存
+ 布隆过滤器，利用高效的数据结构和算法快速判断这个key是否在数据库中存在
+ 接口层增加校验，不合法的参数直接返回
+ 网关给每个ip设置访问次数限制

#### 缓存击穿

某个热点数据过期

##### 概念

+ 一个热点数据，不停抗住大并发请求，当该**热点数据过期**瞬间，持续大并发直接击穿缓存，打在数据库上

##### 解决

+ 热点数据永不过期
+ 加互斥锁，一时间只能有一个用户访问

#### 缓存雪崩

缓存同时过期

##### 概念

+ 缓存同一时间全部失效，恰好那一时间有大并发，导致请求全部打在数据库上，数据库宕机

##### 解决

+ 给每个key的失效时间加个随机数，避免全部同时失效
+ 大并发前数据预热（先把数据都查一次，使缓存中有）













