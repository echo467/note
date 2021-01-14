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

#### 基本命令

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
hadoop dfs  yu hdfs差别
```

#### HDFS数据流

##### 写数据流程

###### 总体流程

1. client向NameNode请求上传文件，NameNode响应可以上传
2. client请求上传第一个block，NameNode返回三个DataNode节点，表示用这三个DN存储数据，最近的依次为DN1、DN2、DN3
3. client向DN1请求建立传输通道，DN1向DN2请求建立传输通道，DN2向DN3请求建立传输通道，当应答成功，即表示通道建立
4. client向DN1传输数据，DN1将收到的数据复制一份，一份存储到本地磁盘，另一份发送给DN2，DN2亦然

###### 节点距离计算

节点距离：两个节点到达最近的共同祖先的距离总和

###### 副本节点选择

1. 第一个副本位于client所在节点上，如果客户端在集群外随机选取一个
2. 第二个副本和第一个副本位于相同机架，随机节点
3. 第三个副本位于不同机架，随机节点

##### 读数据流程

1. client向NameNode请求下载数据，NameNode返回目标文件的元数据
2. client先去距离最近的节点读取block1，完成之后再去最近的节点读取block2，依次类推

#### NN与2NN

##### 工作机制

+ NN将在**内存**中保存元数据，使用FsImage在**磁盘**中备份元数据
+ 当数据发生更新时，FsImage同步更新效率过低，不更新会造成一致性问题
+ 使用Edits文件，采用追加（追加效率高）的形式记录内存中元数据的变化情况
+ 当NN断电时，通过合并FsImage和Edits文件成元数据
+ 如果长时间追加数据到Edits，会导致该文件过于庞大，效率过低，所以引入2NN定期进行FsImage和Edits合并

##### 示意图

![image-20201216152949115](https://gitee.com/ly10208/images/raw/master/img/20201216152949.png)

##### 详解

+ NN上电时，在自动将fsImage文件和edits文件加载到内存中
+ 当client对NN进行操作时，会将操作记录到edits文件中然后再修改内存中的数据
+ 当checkPoint条件触发时，2NN会向NN请求执行checkPoint
+ NN收到请求会新建一个edits文件，以后就在其中记录操作日志
+ 2NN会将NN的fsImage和Edits文件拷贝过来，并加载到内存中进行合并，和并完成生成生成新的fsImage文件，传输到NN替换原来的fsImage文件。

##### checkPoint时间设置

+ 通常情况下，2NN每小时、或检查到NN操作次数达到100万时，执行一次checkPoint。

+ 但亦可以在 hdfs-default.xml文件中设置。

  ```xml
  <property>
      <name>dfs.namenode.checkpoint.period</name>
      <value>3600</value>
      <description>操作时间间隔</description>
  </property>
  
  <property>
      <name>dfs.namenode.checkpoint.txns</name>
      <value>1000000</value>
      <description>操作动作次数</description>
  </property>
  <property>
      <name>dfs.namenode.checkpoint.check.period</name>
      <value>60</value>
      <description>检查到达100万与否时间间隔</description>
  </property>
  ```

##### NN故障处理

法一：将2NN中的数据拷贝到NN中

法二：使用 -importCheckpoint选项启动NN守护进程，从而将2NN拷贝到NN中

##### 集群安全模式

NameNode启动

+ 会先将镜像文件载入内存，并执行编辑日志中的各项操作。在这个过程中，**NameNode处于安全模式，对客户端只读**。
+ 当内存中成功创建元数据映像，则会创建一个新的Fsimage和一个空的编辑日志。

DataNode启动

+ 在安全模式下，各DataNode会向NameNode发送最新的块列表信息，NameNode了解到做够多的块信息之后，即可高效运行文件系统

安全模式退出

+ 当满足最小副本条件，NameNode会在30s之后退出安全模式
+ 最小副本条件是在整个文件系统中99.9%的块满足最小副本级别。

#### DataNode

##### DN工作机制

+ DN启动后，向NN**注册**，并上传块信息
+ 注册成功后，DN每小时**上报**一次所有块信息
+ NN与DN之间每三秒一次**心跳**，心跳返回的结果包括NN给DN的命令
+ 超过**十分钟+三十秒**没有收到DN的心跳信息，则认为DN掉线

##### 数据完整性

总体上说是利用校验机制（奇偶检验，CRC校验）

+ DN读取Block时，会计算CheckSum
+ 如果CheckSum和Block创建时的不一致，则说明Block已经损坏
+ 则Client读取其他DN上的Block

##### DN掉线时限参数

在hdfs-site.xml文件中配置

+ 当DN十分三十秒之内内没有向NN发送心跳，则判断DN掉线，这段时间叫做超时时间（TimeOut)。
+ 超时时间（TimeOut) = 2 * dfs.namenode.heartbeat.recheck-interval + 10 * dfs.heartbeat.interval
+ dfs.namenode.heartbeat.recheck-interval一般为5分钟；dfs.heartbeat.interval为3秒。

### MapReduce

#### 概念

+ MapReduce即MR，是一个**分布式计算**的编程框架
+ 核心功能将用户编写的业务逻辑和自带默认组件整个成一个完整的分布式计算程序

#### 优缺点

##### 优点

+ 易于编程，简单实现一些接口就能完成一个分布式程序
+ 良好的扩展性，可以通过简单增加机器来增加算力
+ 高容错性，系统自动
+ 适合PB级别以上的海量数据的离线处理

##### 缺点

+ 不擅长实时计算
+ 不擅长流式计算
+ 不擅长有向图（DAG）计算（迭代之类的）

#### 核心思想

+ MapReduce运算程序分成Map阶段和Reduce阶段。
+ Map只负责切分，Reduce只负责合并
+ Map阶段并发的MapTask，完全并行，互不相干。
  + 读数据、按行处理
  + 按空格切分行内数据
  + 并输出一个KV键值对
+ Reduce阶段并发的ReduceTask，也是完全并行，互不相干。
  + 按K值读入V数组
  + 对V进行处理
  + 并输出处理后的KV键值对

#### 相关进程

+ MRAppMaster，负责整个程序的过程调度和状态协调
+ MapTask，负责Map阶段的数据处理流程
+ ReduceTask，负责reduce阶段的数据处理

#### 编程规范

##### Map阶段

+ 用户自定义的Map类要继承父类Mapper
+ 输入输出数据都是KV键值对（其中类型可自定义）
+ 业务逻辑代码写在map()方法中
+ map()方法对每一个kV调用一次

##### Reduce阶段

+ 用户自定义的reduce类要继承父类
+ 输入数据的类型对应Map的输出数据类型
+ 业务逻辑代码写在reduce()方法中
+ reduce()方法对每一个kV调用一次

#### 手撸WordCount代码

![image-20210104112248429](https://gitee.com/ly10208/images/raw/master/img/20210104112248.png)

##### 环境

+ 需要先导入 hadoop-common、hadoop-client和hadoop-hdfs三个包

```xml
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-common</artifactId>
    <version>2.7.3</version>
</dependency>
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-client</artifactId>
    <version>2.7.3</version>
</dependency>
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-hdfs</artifactId>
    <version>2.7.3</version>
</dependency>
```

##### Mapper类

```java
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;
import java.io.IOException;

public class WordCountMapper extends Mapper<LongWritable,Text, Text, IntWritable> {
    //定义输出kv格式
    Text k = new Text();
    IntWritable v = new IntWritable(1);
    @Override
    protected void map(IntWritable key, Text value,
                       Context context) throws IOException, InterruptedException {
        //获取一行
        String line = value.toString();
        //按空格切分
        String[] words = line.split(" ");
        
        for (String word : words) {
            k.set(word);
            context.write(k,v);
        }
    }
}
```

##### Reducer类

```java
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;
import java.io.IOException;

public class WordCountReducer extends Reducer<Text, IntWritable,Text,IntWritable> {
    //输出value的类型
    IntWritable v = new IntWritable();
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context)
            throws IOException, InterruptedException {
        //求和变量
        int sum = 0;
        for (IntWritable value:values) {
            sum += value.get();
        }
        v.set(sum);
        context.write(key,v);
    }
}
```

##### Master类

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import java.io.IOException;

public class WordCountMaster {
    public static void main(String[] args)
            throws IOException, ClassNotFoundException, InterruptedException {
        //1.获取Job对象
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        //2.关联Mapper、Reducer和Master类
        job.setJarByClass(WordCountMaster.class);
        job.setMapperClass(WordCountMapper.class);
        job.setReducerClass(WordCountReducer.class);

        //3.设置Mapper阶段输出数据的kv类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);

        //4.设置最终输出数据的kv类型
        job.setOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);

        //5.设置参数输入路径和输出路径
        FileInputFormat.setInputPaths(job,new Path(args[0]));
        FileOutputFormat.setOutputPath(job,new Path(args[1]));

        //6.提交作业
        boolean result = job.waitForCompletion(true);
        System.exit(result?0:1);
    }
}
```

#### 序列化

##### 概念

+ 序列化：将内存中的对象，转换成字节序列，便于持久化和网络传输
+ 反序列化：将字节序列转换成内存中的对象

##### Hadoop序列化

java序列化是重量级序列化框架，过于臃肿，不便于在网络中传输。

hadoop开发了一套序列化机制（Writable)

+ 紧凑，高效使用存储空间
+ 快速，读写数据的额外开销小
+ 可扩展，随通信协议的升级而升级
+ 互操作，支持多语言的交互

##### 常用数据序列化类型

| Java类型   | hadoop writable类型 |
| ---------- | ------------------- |
| byte       | ByteWritable        |
| int        | IntWritable         |
| float      | FloatWritable       |
| double     | DoubleWritable      |
| boolean    | BooleanWritable     |
| long       | LongWritable        |
| **String** | **Text**            |
| map        | MapWritable         |
| array      | ArrayWritable       |

##### 自定义bean对象序列化

+ 实现Writable接口
+ 有空参构造函数，反序列化时调用
+ 重写序列化方法
+ 重写反序列化方法
+ 反序列化的顺序要和序列化的顺序完全一致
+ 要把结果显示在文件中，需要重写toString()
+ 如果自定义对象放在key中，需要实现Comaprable接口。

### MapReduce框架原理

#### InputFormal数据输入

##### 切片与MapTask并行度

数据块：block是HDFS物理上把数据分块

切片：逻辑上对输入进行分片，并不会在磁盘上切分

+ 一个job的map阶段并行度由客户端在提交job时的切片书决定
+ 每一个切片分配一个MapTask并行处理
+ 默认情况下，切片大小等于blockSize
+ 切片时，不考虑数据集整体性，而是对每一个文件单独切分

##### Job提交流程

```java
waitForCompletion(verbose);

submit()
  //1. 建立连接
  connect()
    //创建提交job的代理
    new Cluster(getConfiguration())
        //本地还是集群
        initialize(jobTrackAddr, conf)
  //2. 提交job
  submitter.submitJobInternal(Job.this, cluster)
    //2.1 创建提交资源的路径stage
    Path jobStagingArea = JobSubmissionFiles.getStagingDir(cluster, conf)
    //2.2 获取jobid，并创建job路径
    JobID jobId = submitClient.getNewJobID()
    Path submitJobDir = new Path(jobStagingArea, jobId.toString())
    //2.3 拷贝jar包到集群
    copyAndConfigureFiles(job, submitJobDir);
    //2.4 计算并提交切片信息
    int maps = writeSplits(job, submitJobDir);
    conf.setInt(MRJobConfig.NUM_MAPS, maps);
    //2.5 将job相关参数写到xml文件中并提交
    writeConf(conf, submitJobFile);
```

![image-20210105150121296](https://gitee.com/ly10208/images/raw/master/img/20210105150121.png)

##### FileInputFormat切片机制

+ 简单按照文件内容大小进行切片
+ 切片大小，默认等于block大小
+ 切片不考虑数据集整体，而是逐个针对每一个文件单独切片

**切片大小公式：** **Math.max(minSize, Math.min(maxSize,bolckSize))**

默认  minSize = 1 ; maxSize = Long.MAXValue

可以通过修改minSize和maxSize修改切片大小

**切片数量：文件大小除以切片大小** 

**每次切片时，都要判断当前剩余文件大小是否大于切片大小的 1.1 倍， 不大于 1.1 倍就划分一块切片**

eg: 文件大小129M，切片大小64M。

​      129/64 >1.1 ；先分64M一块，剩余65M；

​       65/64 < 1.1，剩余也只能分成一块；

​       故分成两块，64M和65M

##### CombineTextInputFormat切片机制

**hadoop默认的TextInputFormat**切片机制是对任务按文件切片，无论文件多小，都是一个单独的切片，每一个切片都是一个MapTask，效率较低。

CombineTextInputFormat适用于**小文件过多**的场景，可以将多个小文件划分到一个切片中，多个小文件交给一个MapTask处理。

通过**setMaxInputSplitSize设置切片大小**（默认128M）

整体分成两个阶段，虚拟存储过程和切片过程

虚拟存储过程：

+ 如果文件大小小于 setMaxInputSplitSize ，则划分成一块
+ 如果文件大小大于 setMaxInputSplitSize 、小于 2*setMaxInputSplitSize，则划分成大小相等的两块
+ 如果文件大小大于  2*setMaxInputSplitSize，先画出一块4M大小的。

切片过程：

+ 如果虚拟存储文件的大小大于等于 setMaxInputSplitSize，则单独划成一块
+ 如果不大于，则与下一个虚拟存储文件合并，直到大于等于为止。

![image-20210105155937167](https://gitee.com/ly10208/images/raw/master/img/20210105155937.png)

使用：

+ 需要在Master类中配置

  ```java
  job.setInputFormatClass(CombineFileInputFormat.class);
  CombineFileInputFormat.setMaxInputSplitSize(job,4194304);
  ```

##### FileInputFormat实现类

###### TextInputFormat

+ 默认实现类，按行读取每天记录
+ 键是该行在整个文件中的字节偏移量，是LongWritable类型
+ 值是这行的内容，不包括任何终止符，是Text类型

###### KeyValueTextInputFormat

+ 每一行都是一条记录，被分割成key、value。
+ 可以在Master类中设置，`conf.set(KeyValueLineRecordReader.KEY_VALUE_SEPERATOR,"\t");`来设置分隔符
+ 默认分隔符为 tab(\t)

###### NLineInputFormat

+ NLineInputFormat代表每个map进程处理的N行数据，也就是将文件划分成 总行数/N 个切片，如果不能整除，切片数等于商+1
+ 其中键值与TextInputFormat一样

###### 自定义InputFormat

+ 自定义一个类，继承FileInputFormat

  + 重写isSplitable()方法，返回false不可分割
  + 重写createRecordReader()，创建RecordReader对象，并初始化

+ 改写RecordReader，实现一次读取一个完整文件封装为KV

  + 采取IO流一次读取一个文件输出到value中
  + 获取文件的路径+名称，并设置key

+ 设置Master

  + 设置输入的inputFormat

    `job.setInputFormatClass(WholeFileInputformat.class)`

  + 在输出时使用SequenceFileOutPutFormat合并文件

    `job.setOutputFormatClass(SequenceFileOutputFormat.class)`

#### MR工作流程

##### MapTask工作机制

![image-20210110103841075](https://gitee.com/ly10208/images/raw/master/img/20210110103841.png)

##### ReduceTask工作机制

![image-20210110104220933](https://gitee.com/ly10208/images/raw/master/img/20210110104221.png)

### Shuffle机制

![image-20210110105035261](https://gitee.com/ly10208/images/raw/master/img/20210110105035.png)

#### Partition分区

##### 默认

+ 将统计结果输出到不同的文件中

+ 默认partition分区是hash分区

  `(key.hashCode() & Integer.MAX_VALUE) % numReduceTasks`

  默认hash分区是根据key的hashCode对ReduceTask个数取模得到的，用户无法控制。numReduceTasks默认为1，需要通过`job.setNumReduceTasks()` 来设置。

##### 自定义partition

+ 自定义类继承Partitioner方法，重写getPartition()方法
+ 在Job中，设置自定义Partitioner `job.setPartitionerClass()`
+ 根据自定义partition的逻辑设置相应数量的ReduceTask，`job.setNumReduceTasks()`

![image-20210107163155367](https://gitee.com/ly10208/images/raw/master/img/20210107163155.png)

#### 排 序

+ MapTask和ReduceTask都会对数据按照**key**进行排序。默认使用字典序排序（快排实现）
+ ![image-20210107163911846](https://gitee.com/ly10208/images/raw/master/img/20210107163912.png)

##### 排序分类：

1. 部分排序，保证每个输出文件内部有序
2. 全排序，最终输出结果只有一个，文件内部有序
3. 辅助排序，reduce对key进行分组排序
4. 二次排序，compareTo中的判断条件为两个

##### 自定义排序

+ bean对象作为key进行传输，需要实现WritableComparable接口、重写compareTo接口

#### Combiner合并

+ 位于MR程序之中Mapper和Reducer之外的一种组件
+ 其父类就是Reducer
+ 相当于运行在**Mapper阶段的Reducer**
+ 主要是对每一个MapTask的输出进行局部汇总，以**减小网络传输量**
+ 但是使用combiner不能影响最终的业务逻辑

##### 自定义combiner

法一：

+ 增加一个combiner类继承Reducer
+ 并在其中汇总输出
+ 在job中绑定combiner

法二：

+ 直接在job中将combiner的驱动类设置为自定义的reducer类

  `job.setCombinerClass(WordcountReducer.class)`

#### OutputFormal数据输出

+ 默认TextOutputFormat，**把每条记录写为文本行**。
+ SequenceFileOutputFormat，将其输出作为后续mapReduce任务的输入，格式紧凑，很容易被压缩
+ 自定义OutputFormat，自定义输出（写入redis、mysql）

##### 自定义

+ 自定义类继承FileOutputFormat
+ 改写RecordWriter,具体改写输出数据的方法write()

#### Join

##### Reduce join

+ 在reduce端，联结几张表的字段
+ map端负责，为不同表的KV对，打标签以区别不同来源的记录；然后用**连接字段作为key**，其余部分和新加的标签作为value
+ reduce负责，在每一个分组中将那些来源于不同文件的记录分开，最后进行合并

合并操作在reduce完成，处理压力太大，而map运算负载很低，资源利用率不高，且在reduce容易产生数据倾斜

##### Map join

+ 适用于一张大表一张小表的情况，需要提前将小表缓存到内存中
+ 在自定义mapper函数的setup阶段，将文件读到缓存集合中；
+ 并且在驱动函数中加载缓存`job.addCacheFile(new URI("file://"))`;
+ 将reduceTask数设置为0， `job.setNumReduceTasks(0)`。

### 数据压缩

+ 减少底层存储系统（HDFS)读写字节数，提高网络带宽和磁盘空间的效率。
+ 可以在mr的任意阶段进行压缩

#### MR支持的压缩编码

![image-20210112205001092](https://gitee.com/ly10208/images/raw/master/img/20210112205001.png)

![image-20210112205101702](https://gitee.com/ly10208/images/raw/master/img/20210112205101.png)

#### 压缩方式选择

##### Gzip

+ 优点：压缩率较高，压缩解压缩速度比较快；hadoop本身支持
+ 缺点：不支持切分
+ 应用场景：每个文件压缩后大小在130M以内（一个块）

##### Bzip2

+ 优点：支持切分，有很高的压缩率
+ 缺点：压缩解压速度慢
+ 应用场景：对速度要求不高，需要较大压缩率；处理后需要压缩存档并且用的比较少的情况

##### Lzo

+ 合理的压缩率，压缩解压缩速度较快，支持切分，是最流行的压缩格式
+ 缺点：hadoop本身不支持，需要安装，使用时还要建立索引
+ 应用场景：很大的文本文件，压缩后还大于一个块

##### Snappy

+ 优点：高速压缩速度和合理的压缩率
+ 缺点：不支持切分，Hadoop本身不支持，需要安装
+ 应用场景：作为map到reduce的中间数据的压缩格式，或一个MR和另一个MR的衔接

#### 压缩位置的选择

![image-20210112210344508](https://gitee.com/ly10208/images/raw/master/img/20210112210344.png)

### Yarn资源调度器

+ **资源调度**器平台，负责为运算程序提供运算资源，相当于一个**分布式操作系统**，mr程序是运行在yarn之上的应用程序。

#### 基本架构

![image-20210114133113613](https://gitee.com/ly10208/images/raw/master/img/20210114133113.png)

#### 工作机制

![image-20210114133522455](https://gitee.com/ly10208/images/raw/master/img/20210114133522.png)

#### 资源调度器

##### FIFO

+ 先进先出队列

##### 容量调度器

+ hadoop**默认**调度器，多条FIFO队列
+ 每个队列可配置一定的资源量，每个队列分配的资源也不一样
+ 首先，计算每个队列中正在运行的任务数与其应该分得的计算资源之间的比值，选择一个比值最小的队列
+ 其次，按照作业优先级和提交顺序，同时考虑用户资源两线制和内存限制对队列内任务排序
+ 三个队列中的任务可以同时执行

##### 公平调度器

+ 也是多条队列，每个队列中的job按照优先级分配资源，优先级越高分配的资源越多，但每个job都会分配到资源以确保公平。
+ 但是资源有限，每个job理论需要的资源和实际获得资源之间的差值，就叫做差额
+ job的资源缺额越大，越先获得资源优先执行。

#### 任务推测执行

+ 作业完成时间取决于最慢的任务完成时间
+ 为拖后腿的任务启动一个备份任务，同时运行，谁先运行完，采用谁的结果
+ 执行推测任务的条件
  + 每个task只能有一个备份任务
  + 当前job完成度不小于5%
+ 不能启动推测任务的情况
  + 任务间存在严重的负载倾斜
  + 特殊任务，写入数据库等

算法原理

![image-20210114140913533](https://gitee.com/ly10208/images/raw/master/img/20210114140913.png)

### hadoop优化

#### MR程序效率瓶颈

+ 计算机性能（cpu、内存、磁盘、网络）
+ IO操作
  + 数据倾斜
  + Map数和reduce数不合理
  + 小文件过多
  + 大量不可切分的超大文件
  + 溢写和合并次数过多

#### 优化方法

+ 数据输入，合并小文件
+ map，减少溢写和合并次数，使用combine
+ reduce，设置reduce可以在map没完成时运行
+ IO传输，数据压缩、使用SequenceFile二进制文件
+ 数据倾斜，自定义分区，采用map join

#### 小文件

+ hadoop archive ，可以将多个小文件打包成一个 HAR 文件，减少NameNode内存的使用
+ Sequence File ，将文件名作为key，文件内容作为value，将小文件合并成一个大文件
+ CombineFileInputFormat，将多个文件合并成一个单独的切片
+ 开启JVM重用，JVM可以运行完一个MAP程序，不关闭，继续运行其他Map程序



