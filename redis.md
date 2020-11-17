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



### 主从复制

### 缓存穿透与雪崩

https://blog.csdn.net/DDDDeng_/article/details/108118544?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522160557961219724835835601%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=160557961219724835835601&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~baidu_landing_v2~default-10-108118544.first_rank_ecpm_v3_pc_rank_v2&utm_term=%E7%8B%82%E7%A5%9E+redis&spm=1018.2118.3001.4449



























