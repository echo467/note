大数据技术-Hadoop

### 概述

#### 大数据

##### 概念

无法在一定时间内使用常规工具进行处理的数据集合

##### 理解

大数据主要解决海量数据的存储和分析

##### 特征

+ （Volume）大量，数据量很大
+ （Velocity）高速，数据产生速度快
+ （Variety）多样，数据类型多，既有结构化也有非结构化数据
+ （Value）低价密，价值密度低，数据中有效信息少

#### 组织架构

##### 平台组

+ Hadoop、KafKa、Hbase等平台搭建
+ 集群性能监控
+ 集群性能调优

##### 数据仓库

+ 数据清洗
+ 数据分析，数据仓库建模

##### 数据挖掘

+ 推荐系统
+ 用户画像

##### 可视化

+ 数据可视化

### Hadoop

#### 概念

+ 分布式系统**基础架构**
+ 解决海量**数据的存储和分析计算**

#### 历史

+ google是hadoop的思想之源
  + GFS             -> HDFS
  + MapReduce -> MapReduce
  + BigTable       -> Hbase

#### 优势

+ 高可靠，Hadoop底层维护多个数据副本
+ 高扩展性，集群之间分配任务数据，方便扩展
+ 高效性，并行工作
+ 高容错，能够将失败的任务自动重新分配

#### 版本

![image-20201201205931228](https://gitee.com/ly10208/images/raw/master/img/20201201205931.png)

1.x版本，mapReduce既负责计算又负责资源的调度，耦合性较大

2.x版本增加了yarn，让mapReduce只负责计算，yarn负责资源调度

#### 生态体系

![image-20201201213324737](https://gitee.com/ly10208/images/raw/master/img/20201201213324.png)

### HDFS

#### 概述

##### 特点

+ 分布式文件管理系统，存储文件

+ 适合一次写入，多次读出的场景，且不支持文件的修改，适合做数据分析

##### 优点

+ 高容错，数据保存多个副本
+ 适合处理大数据，可以处理GB、TB、PB级的数据
+ 可构建在廉价设备上

##### 缺点

+ 不适合低时延数据访问
+ 对小文件不友好，会占用nameNode大量内存
+ 不支持并发写入、文件随即修改
  + 一个文件只能有一个写、不允许多个同时写
  + 仅支持数据追加，不支持修改

#### 架构

##### NameNode

+ 管理HDFS的空间
+ 配置副本策略
+ 管理数据块的映射关系
+ 处理客户端读写请求
+ 存储文件的元数据，如文件名、目录结构、文件属性和每个文件的块列表和DataNode

##### DataNode

+ 存储实际的数据块
+ 执行数据块的读写操作

##### client

+ 文件切分（上传文件的时候）
+ 与NameNode交互，读取或写入数据
+ 管理和访问HDFS

##### Secondary NameNode

+ 不是NameNode的热备份,监控HDFS状态
+ 辅助NameNode，分担其工作量，比如定期合并Fsimage和Edits，并推送给NameNode
+ 紧急情况下，可辅助恢复NameNode

#### HDFS文件块

+ 2.x版本默认为128M，可以自定义
+ 主要取决于磁盘的传输速率

##### 注意

+ 为啥不能太大也不能太小
+ 一般寻址时间为传输时间的1/100，状态最佳
+ 如果块太小，会增加寻址时间
+ 如果块太大，磁盘传输数据的时间会远大于寻址的时间，会导致数据处理时很慢

#### 命令

```bash
hadoop fs -mkdir 
hadoop fs -ls
hadoop fs -rm -r -f        
hadoop fs -append 源 目     #将源文件追加到目的文件的末尾
hadoop fs -cat              #显示文件内容

hadoop fs -cp               #从HDFS一个路径拷贝到另一个路径
hadoop fs -mv               #从HDFS一个路径移动到另一个路径
hadoop fs -get              #下载
hadoop fs -getmerage        #文件合并下载
hadoop fs -put              #上传
hadoop fs -tail             #显示一个文件的末尾
hadoop fs -do               #统计文件的大小

hadoop fs -setrep          #设置文件副本数量
```

```bash
hadoop dfs
```

### MR

### Yarn

