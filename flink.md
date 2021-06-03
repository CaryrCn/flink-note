# Flink 

## 1. 基本介绍

### 1.1 流处理技术概览

#### 1.1.1 大数据计算模式

* **批量计算**

  * • MapReduce

    • Apache Spark

    • Hive

    • Flink

    • Pig

* **流式计算**

  * • Storm

    • Spark Streaming

    • Apache Flink

    • Samza

* 交互计算

  * • Presto

    • Impala

    • Druid

    • Drill

* 图计算

  * • Giraph（Facebook）

    • Graphx（Spark）

    • Gelly（Flink）

#### 1.1.2 流计算与批计算差异

* **数据时效性**
  * 流：实时、低延迟；批：非实时，高延迟
* 数据特征
  * 流：动态数据，没有边界
* **应用场景**
  * 流：时效性高；批：离线计算
* 运行方式
  * 流：持续进行；批：一次性完成
* **存储、计算**
  * 批：计算、存储成本高

#### 1.1.3 流计算框架差异

* **apache storm**
  * 优势
    * 最早使用的流处理框架，社区成熟
    * 支持原生流处理，即单事件来处理流数据（所有记录一个接一个处理）
    * 延迟低（毫秒级）
  * 劣势
    * 消息保证能力弱，消息传输可能重复但不会丢失
    * 吞吐量比较低
* **spark streaming**
  * 流程
    * ![image-20210529122451042](flink.assets/image-20210529122451042.png)
  * 劣势
    * 以固定间隔（几秒钟），处理一段批处理作业，延迟性较高
  * 优势
    * 保证消息不会丢失也不会重复
    * 吞吐高
* flink
  * 真正的流处理框架（dataflow model）
  * 延长性低
  * 高吞吐
  * 保证消息不会丢失也不会重复
  * 支持原生流处理
* 差异比对图
  * ![企业微信截图_7058ef9c-6392-4d9c-a08b-f408f756d951](flink.assets/企业微信截图_7058ef9c-6392-4d9c-a08b-f408f756d951.png)

### 1.2 Flink 发展历史及应用

* 发展历史
  * ![image-20210529123537107](flink.assets/image-20210529123537107.png)
* 应用场景
  * 实施监控
    * 用户行为预警
    * 用户行为或相关事件实时监测分析，基于风控规则预警
  * 实时报表
    * 双11直播大屏
    * 数据化运营
  * 流数据分析
    * 内容投放、个性化推荐、智能推送
  * 实时数据仓库
    * 数据实时清洗、归并、结构化
    * 数仓的补充和优化

### 1.3 Flink核心特性

#### 1.3.1统一数据处理组件栈，处理不同需求（batch、stream、machine learning、gragh）

* ![image-20210529124424026](flink.assets/image-20210529124424026.png)

#### 1.3.2 支持时间概念。事件时间、接入时间、处理时间

* ![image-20210529124515325](flink.assets/image-20210529124515325.png)

#### 1.3.3 基于轻量级分布式快照实现容错

* ![image-20210529124553348](flink.assets/image-20210529124553348.png)

#### 1.3.4 支持有状态计算

* support for very large state
* querable state支持
* 灵活的state-backend（HDFS、rocksdb、内存）

#### 1.3.5 带反压的连续流模型

* <img src="flink.assets/image-20210529124812373.png" alt="image-20210529124812373" style="zoom:67%;" />

#### 1.3.6 内存管理特性

* 基于jvm实现独立内存管理（flink自己的内存管理）
* 对象序列化二进制存储
* 应用可超出主内存的大小限制，承受更少的垃圾收集开销

## 2. Flink部署与应用

### 2.1 Flink集群架构

* **集群总体架构**
  * ![image-20210601205926537](flink.assets/image-20210601205926537.png)

#### 2.1.1 JobManager

* 功能：管理节点，管理整个集群计算资源，job管理与调度执行以及checkpoint协调
* 功能明细：
  * **Checkpoint Coordinator** 
    *  Checkpoint协调相关
  * **Actor System**
    * RPC通信
  * **Job Dispatch**
    * **接收Job**，提供了一个 REST 接口，用来提交 Flink 应用程序执行，并为每个提交的作业启动一个新的 **JobMaster**
    * 运行 Flink WebUI 用来提供作业执行信息
  * **ResourceManager**：
    * 集群资源管理，负责 Flink 集群中的资源提供、回收、分配 - 它管理 **task slots**
    * 不同的环境和资源提供者（例如 YARN、Mesos、Kubernetes 和 standalone 部署）实现了对应的 ResourceManager
    * 只能分配可用 TaskManager 的 slots，而不能自行启动新的 TaskManager
  * JobGraph-> Execution Graph
  * Task部署与调度
  * TaskManager注册与管理
  * ![image-20210601222315302](flink.assets/image-20210601222315302.png)

#### 2.1.2 TaskManager

* 功能：提供计算资源
* 功能明细：
  * Task Execution
  * Network Manager
  * Shuffle Environment管理
  * Rpc通信（Actor system）
  * heartbeat with jobmanager And RM
  * Data Exchange
  * Memory Management
  * Register To RM
  * Offer Slots To JobManager
  * ![image-20210602200618966](flink.assets/image-20210602200618966.png)

#### 2.1.3 Client

* 功能：本地执行main方法解析JobGraph对象，并提交到JobManager运行
* 功能明细：
  * app main方法的执行
  * JobGraph 生成
  * Execution Environment管理
  * Job 和依赖包提交
  * RPC with JobManager
  * 集群部署（Cluster Deploly）**。？？这个跟Client什么关系？？？**

#### 2.1.3 Job Graph介绍

* 说明
  * 通过有向无环图（Dag）方式，表达程序
  * flink1.11之前只能在client生成
  * 客户端与集群间Job描述的载体
  * 不同接口程序的抽象表达
  * ![image-20210602201507421](flink.assets/image-20210602201507421.png)
  * ![image-20210602201514134](flink.assets/image-20210602201514134.png)

### 2.2 Flink集群部署模式（Session、Per-Job、Application）

#### 2.2.1 集群部署模式分类指标

1. **集群的生命周期与资源隔离**
2. **程序的main在Client还是JobManager执行**

#### 2.2.2 Session 模式

* 特点
  * JobManager与TaskManager 所有Job共享
  * 客户端通过RPC或Rest API连接集群管理节点
  * 客户端需上传依赖jar
  * 客户端生成JobGraph，提交到JobManager
  * JobManager生命周期不受Job影响，会长期运行
  * ![image-20210602201904075](flink.assets/image-20210602201904075.png)
* 优势
  * 资源充分共享，提升资源利用率
  * Job在集群中管理，运维简单
* 劣势
  * 资源隔离相对较差
  * 非Native类型部署，TM不易扩展，slot计算资源收缩性差

#### 2.2.3 Per-Job 模式

* 特点
  * 单个Job独享 JobManager与 Task Manager
  * TM中Slot资源根据Job指定
  * 客户端需上传依赖jar
  * 客户端生成JobGraph，提交到JobManager
  * JobManager的生命周期和Job生命周期绑定
  * ![image-20210602203124479](flink.assets/image-20210602203124479.png)
* 优势
  * Job和Job之间资源隔离充分
  * 资源根据Job需要进行申请, TM Slots数量可以不同
* 劣势
  * 资源相对比较浪费, JobManager需要消耗资源
  * Job管理完全交给 ClusterManagement,管理复杂

#### 2.2.4 Application模式

* 特点
  * 每个 Application对应一个 JobManager,且可以运行多个Job
  * 客户端无需将 Dependencies上传到JobManager,仅负责管理Job的提交与管理
  * main(方法运行 JobManager中,将JobGraph的生成放在集群上运行,客户端压力降低
* 优势
  * 有效降低带宽消耗和客户端负载
  * Application实现资源隔离, Application中实现资源共享
* 劣势
  * 功能太新,还未经过生产验证
  * 仅支持Yan和 Kubunetes

### 2.3 集群资源管理器支持（Standalone、Yarn、Kubernates）

#### 2.3.1 集群部署对比

* ![企业微信截图_3e8a3704-f498-4de5-8a7d-2d252c7fea89](flink.assets/企业微信截图_3e8a3704-f498-4de5-8a7d-2d252c7fea89.png)

#### 2.3.2 Native集群部署（仅Session模式存在Native）

* 特点
  * Session集群根据根据实际提交的Job资源动态申请和启动 Taskmanager计算资源
  * 支持 Native部署模式的有Yarn, Kubernetes, Mesos资源管理器
  * Standalone不支持 Native部署

* ![企业微信截图_114e88ce-577a-4345-a919-22fc67ac7e58](flink.assets/企业微信截图_114e88ce-577a-4345-a919-22fc67ac7e58.png)

### 2.4 Standalone集群原理

#### 2.4.1 特点

* 分布式多台物理主机部署
* 依赖于Java8或Java11 JDK环境
* 仅支持 Session模式提交 Job
* 支持高可用配置

#### 2.4.2 Job提交流程

* ![image-20210603135716067](flink.assets/image-20210603135716067.png)

#### 2.4.3 多机模式注意事项

* 每台节点都必须安装 JDK,并配置 JAVA_HOME路径和环境变量
* 如果需要用hdfs,则需要在每台节点需配置 HADOOP CONFIG_DIR环境变量
* 每台节点 Flink package路径保持一致,且版本保持一致
* 相关配置修改
  * 主要配置修改
  * 修改conf/fink-conf.yam文件,配置 jobmanager rpc. address: ${地址}
  * 修改conf/ master文件,设定 master节点为主节点地址
  * 修改conf/ Worker文件,设定work节点地址
  * 将修改后的配置文件同步到每台节点的$ FLINK HOME}/conf路径下

### 2.5 Flink On Yarn集群原理

#### 2.5.1 Yarn 架构

* 组件
  * Resource Manager (NM)
    * 负责处理客户端请求
    * 监控 Nodemanager
    * 启动和监控 APPLicationMaster
    * 资源的分配和调度
  * Node Manager
    * 管理单个 Worker节点上的资源;
    * 处理来自 ResourceManager的命令
    * 处理来自 Application Master的命令
    * 汇报资源状态
  * Application Master
    * 负责数据的切分
    * 为应用申请计算资源,并分配给Task
    * 任务的监控与容错
    * 运行在 Worker节点上
  * Container
    * 资源抽象,封装了节点上的多维度资源,如CPU,内存,网络资源等;
* 架构图
  * ![image-20210603140722999](flink.assets/image-20210603140722999.png)

#### 2.5.2 Session模式提交（On Yarn ）

* 特点
  * 多 Jobmanager共享 Dispatcher和 YarnResource manager
  * 支持 Native模式,TM动态申请
* 流程图
  * ![image-20210603141102410](flink.assets/image-20210603141102410.png)

#### 2.5.3 Per-Job模式提交（On Yarn）

* 特点
  * 单个 JobManager独享 YarnResource manager和 Dispatcher
  * Application Master与 Flink master节点处于同一个 Container

#### 2.5.4 优劣势

* 优势
  * 与现有大数据平台无缝对接( Hadoop2.4+)。
  * 部署集群与任务提交都非常简单
  * 资源管理统一通过Yarn管理,提升整体资源利用率类。
  * 基于 Native方式, TaskManager资源按需申请和启动,防止资源浪费。
  * 容错保证借助于 Hadoop Yarn提供的自动 failover机制,能保证JobManager, TaskManager节点异常恢复
* 劣势
  * 资源隔离问题,尤其是网络资源的隔离,Yarn做的还不够完善。
  * 离线和实时作业同时运行相互干扰等问题需要重视。
  * Kerberos认证超期问题导致 Checkpoint无法持久化。

#### 2.5.5 部署及注意事项

* 注意事项

  * Hadoop Yarn版本需要在2.4.1以上,且具有HDFS文件系统;
  * 该节点需要配置 HADOOP_CoNF_DR环境变量,并指向 Hadoop客户端配置路径;
  * 下载 Hadoop依赖包配置
    * FLink1.11版本后将不再在主版本中支持 flink- shaded- hadoop-2-uber包,用户需要自己
      指定 HADOOP CLASSPATH,或者继续到flink官网下载 Hadoop依赖包。

* 启动

  * Session集群启动

    * /bin/yarn- session. sh -im 1024m-tm 4096m

  * Job集群启动

    * /bin/flink run -m yarn-cluster-p4-yjm 1024m-ytm 4096m /examples/batch/Word Count jar

  * Application Mode集群启动

    * /bin/flink run -application-t yarn -application \

      -Djobmanager. memory. process size=2048m \
      -Dtaskmanager memory process size=4096m \
      -Dyarn provided. lib. dirs="hdfs: //node02: 8020/flink-training /flink-111" \
      ./MyApplication jar

### 2.6 Flink On Kubernates集群原理

#### 2.6.1 待补充

### 2.7 Flink集群高可用

#### 2.7.1 待补充

## 3.Flink DataStream API使用与说明

### 3.1分布式流处理模型（DataFlow）

* 数据流关注点
  * **正确性、延迟、大规模的成本、无界、无序的数据处理**
* 流模型图
  * ![image-20210603151101780](flink.assets/image-20210603151101780.png)
* 多并发度流模型的特性
  * 数据从上一个 Operation节点直接**Push**到下一个 Operation节点。
  * 各节点可以分布在不同的Task线程中运行,数据在 Operation之间传递。
  * 具有 Shuffle过程,但是数据不像 MapReduce模型, Reduce从Map端拉取数据（flink是上游push）。
  * 实现框架有 Apache Storn和 Apache Flink以及 Apache beam。
  * ![image-20210603151233305](flink.assets/image-20210603151233305.png)

### 3.2 Flink DataStream API架构及原理

#### 3.2.1 API架构

![image-20210603153728963](flink.assets/image-20210603153728963.png)

####  3.2.2 Flink demo 解读

* ![image-20210603154009705](flink.assets/image-20210603154009705.png)

* 相关类介绍

  * 运行环境（StreamExecutionEnvironment）

    * 功能

    ![image-20210603154901554](flink.assets/image-20210603154901554.png)

  * Data Stream数据源

    * 使用说明

      ![image-20210603155042397](flink.assets/image-20210603155042397.png)

    * Datastream基本数据源的使用

      ```
      /从给定的数据元素中转换
      DataStreamSource<OUT> fromElements(OUT. data)
      
      /从指定的集合中转换成 Datastream
      DataStreamSource<OUT> fromCollection( Collection<OUT> data)
      
      //读取文件并转换
      DataStreamSource<OUT> readFile(filelnputFormat<OUT> inputFormat, String filepath)
      
      //从 Socket端口中读取
      DataStreamSource<String> socketTextstream(string hostname int port String delimiter)
      
      //直接通过InputFormat创建
      DataStreamSource<OUT> createInput(InputFormat<OUT, ? inputFormat
      
      最终都是通过 Execution environment创建 from source(方法转换成 Datastream source)
      ```

    * DataStream 数据源连接器的使用

      * Flink 内置的Connector：

        • Apache Kafka (source/sink)

        • Apache Cassandra (sink)

        • Amazon Kinesis Streams (source/sink)

        • Elasticsearch (sink)

        • Hadoop FileSystem (sink)

        • RabbitMQ (source/sink)

        • Apache NiFi (source/sink)

        • Twitter Streaming API (source)

        • Google PubSub (source/sink)

        • JDBC (sink)

      * 例子：Kafka 连接器使用

        * 1.引入依赖

          * ```
            <dependency>
            
            <groupId>org.apache.flink</groupId>
            
            <artifactId>flink-connector-kafka_2.11</artifactId>
            
            <version>1.11.0</version>
            
            </dependency>
            ```

        * 2.配置属性，把数据源添加到env中

          * ```
            Properties properties = new Properties();
            properties.setProperty("bootstrap.servers", "localhost:9092");
            properties.setProperty("group.id", "test");
            DataStream<String> stream = env.addSource(new FlinkKafkaConsumer<>("topic",
            new SimpleStringSchema(), properties));
            ```

            * 这里的FlinkKafkaConsumer就是source operator

#### 3.2.3 DataStream 转换操作

* 转换操作类型
  * ![image-20210603165302420](flink.assets/image-20210603165302420.png)



* 转换具体操作
  * ![image-20210603165340285](flink.assets/image-20210603165340285.png)

* Data Steam间的转换
  * ![image-20210603165444148](flink.assets/image-20210603165444148.png)

