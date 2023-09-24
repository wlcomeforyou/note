# RocketMQ 消息中间件
Message Queue
### 应用场景

+ 解决业务耦合的问题
+ 进行流量的削峰
+ 数据分发

### 常见消息中间件
+ ActiveMQ
+ Kafka
+ RabbitMQ
+ RocketMQ
  **RocketMq优势**
    + 支持事务型消息（消息发送和 DB 操作保持两方的最终一致性，RabbitMQ 和 Kafka 不支持）
    + 支持结合 RocketMQ 的多个系统之间数据最终一致性（多方事务，二方事务是前提）
    + 支持 18 个级别的延迟消息（Kafka 不支持）
    + 支持指定次数和时间间隔的失败消息重发（Kafka 不支持，RabbitMQ 需要手动确认）
    + 支持 Consumer 端 Tag 过滤，减少不必要的网络传输（即过滤由MQ完成，而不是由消费者完成。RabbitMQ 和 Kafka 不支持）
    + 支持重复消费（RabbitMQ 不支持，Kafka 支持）

### RocketMQ核心概念
+ **生产者Producer**
也称为消息发布者，负责生产并发送消息至 Topic。
生产者向brokers发送由业务应用程序系统生成的消息。RocketMQ提供了发送：同步、异步和单向（one-way）的多种范例
    + **同步发送**
      同步发送指消息发送方发出数据后会在收到接收方发回响应之后才发下一个数据包。一般用于重要通知消息，例如重要通知邮件、营销短信。

    + **异步发送**
      异步发送指发送方发出数据后，不等接收方发回响应，接着发送下个数据包，一般用于可能链路耗时较长而对响应时间敏感的业务场景，例如用户视频上传后通知启动转码服务。假如过一段时间检测到某个信息发送失败，可以选择重新发送。

    + **单向发送**
      单向发送是指只负责发送消息而不等待服务器回应且没有回调函数触发，适用于某些耗时非常短但对可靠性要求并不高的场景，例如日志收集。

    + **生产者组**
      生产者组（Producer Group）是一类 Producer 的集合，这类 Producer 通常发送一类消息并且发送逻辑一致，所以将这些 Producer 分组在一起。从部署结构上看生产者通过 Producer Group 的名字来标记自己是一个集群。

    + **高可用保障**
      Producer与Name Server集群中的其中一个节点(随机选择)建立长连接，定期从Name Server取Topic路由信息，并向提供Topic服务的Master建立长连接，且定时向Master发送心跳。Producer完全无状态，可集群部署。
      Producer每隔30s（由ClientConfig的pollNameServerInterval）从Name server获取所有topic队列的最新情况，这意味着如果Broker不可用，Producer最多30s能够感知，在此期间内发往Broker的所有消息都会失败。
      Producer每隔30s（由ClientConfig中heartbeatBrokerInterval决定）向所有关联的broker发送心跳，Broker每隔10s中扫描所有存活的连接，如果Broker在2分钟内没有收到心跳数据，则关闭与Producer的连接

+ 消费者Consumer
  也称为消息订阅者，负责从 Topic 接收并消费消息。
  消费者从brokers那里拉取信息并将其输入应用程序。
    + **消费者组**
      消费者组（Consumer Group）一类 Consumer 的集合名称，这类 Consumer 通常消费同一类消息并且消费逻辑一致，所以将这些 Consumer 分组在一起。消费者组与生产者组类似，都是将相同角色的分组在一起并命名。
      RocketMQ中的消息有个特点，同一条消息，只能被某一消费组其中的一台机器消费，但是可以同时被不同的消费组消费
    + **高可用保障**
      Consumer与Name Server集群中的其中一个节点(随机选择)建立长连接，定期从Name Server取Topic路由信息，并向提供Topic服务的Master、Slave建立长连接，且定时向Master、Slave发送心跳。Consumer既可以从Master订阅消息，也可以从Slave订阅消息，订阅规则由Broker配置决定。

    Consumer每隔30s从Name server获取topic的最新队列情况，这意味着Broker不可用时，Consumer最多最需要30s才能感知。

    Consumer每隔30s（由ClientConfig中heartbeatBrokerInterval决定）向所有关联的broker发送心跳，Broker每隔10s扫描所有存活的连接，若某个连接2分钟内没有发送心跳数据，则关闭连接；并向该Consumer Group的所有Consumer发出通知，Group内的Consumer重新分配队列，然后继续消费。

    当Consumer得到master宕机通知后，转向slave消费，slave不能保证master的消息100%都同步过来了，因此会有少量的消息丢失。但是一旦master恢复，未同步过去的消息会被最终消费掉
+ 名称服务NameServer
    Name Server是一个几乎无状态节点，可集群部署，节点之间无任何信息同步。
    NameServer 是整个 RocketMQ 的“大脑” ，它是 RocketMQ 的服务注册中心，所以 RocketMQ 需要先启动 NameServer 再启动 Rocket 中的 Broker。
    + **作用**
    名称服务器（NameServer）用来保存 Broker 相关元信息并给 Producer 和 Consumer 查找Broker 信息。NameServer 被设计成几乎无状态的，可以横向扩展，节点之间相互之间无通信，通过部署多台机器来标记自己是一个伪集群。
    每个 Broker 在启动的时候会到 NameServer 注册，Producer 在发送消息前会根据 Topic 到NameServer 获取到 Broker 的路由信息，进而和Broker取得连接。Consumer 也会定时获取 Topic 的路由信息。所以从功能上看应该是和 ZooKeeper 差不多，据说 RocketMQ 的早期版本确实是使用的ZooKeeper ，后来改为了自己实现NameServer。
    + **和zooKeeper区别**
    Name Server和ZooKeeper的作用大致是相同的，从宏观上来看，Name Server做的东西很少，就是保存一些运行数据，Name Server之间不互连，这就需要broker端连接所有的Name Server，运行数据的改动要发送到每一个Name Server来保证运行数据的一致性（这个一致性确实有点弱），这样就变成了Name Server很轻量级，但是broker端就要做更多的东西了。
    + **高可用保障**
    Broker 在启动时向所有 NameServer 注册（主要是服务器地址等） ，生产者在发送消息之前先从NameServer 获取 Broker 服务器地址列表（消费者一样），然后根据负载均衡算法从列表中选择一台服务器进行消息发送。
    NameServer 与每台 Broker 服务保持长连接，并间隔 30S 检查 Broker 是否存活，如果检测到Broker 宕机，则从路由注册表中将其移除，这样就可以实现 RocketMQ 的高可用。

+ 代理服务器BrokerServer
    消息服务器（Broker）是消息存储中心，主要作用是接收来自 Producer 的消息并存储，Consumer 从这里取得消息。它还存储与消息相关的元数据，包括用户组、消费进度偏移量、队列信息等。从部署结构图中可以看出 Broker 有 Master 和 Slave 两种类型，Master 既可以写又可以读，Slave不可以写只可以读。
    + **部署方式**
      Broker部署相对复杂，Broker分为Master与Slave，一个Master可以对应多个Slave，但是一个Slave只能对应一个Master，Master与Slave的对应关系通过指定相同的Broker Name，不同的BrokerId来定义，BrokerId为0表Master，非0表示Slave。Master也可以部署多个。
      从物理结构上看 Broker 的集群部署方式有四种：单 Master 、多 Master 、多 Master 多Slave（同步刷盘）、多 Master多 Slave（异步刷盘）。
        + **单 Master**
          这种方式一旦 Broker 重启或宕机会导致整个服务不可用，这种方式风险较大，所以显然不建议线上环境使用。
        + **多 Master**
          所有消息服务器都是 Master ，没有 Slave 。这种方式优点是配置简单，单个 Master 宕机或重启维护对应用无影响。缺点是单台机器宕机期间，该机器上未被消费的消息在机器恢复之前不可订阅，消息实时性会受影响
        + **多 Master 多 Slave（异步复制）**
          每个 Master 配置一个 Slave，所以有多对 Master-Slave，消息采用异步复制方式，主备之间有毫秒级消息延迟。这种方式优点是消息丢失的非常少，且消息实时性不会受影响，Master 宕机后消费者可以继续从 Slave 消费，中间的过程对用户应用程序透明，不需要人工干预，性能同多 Master 方式几乎一样。缺点是 Master 宕机时在磁盘损坏情况下会丢失极少量消息
        + **多 Master 多 Slave（同步双写）**
          每个 Master 配置一个 Slave，所以有多对 Master-Slave ，消息采用同步双写方式，主备都写成功才返回成功。这种方式优点是数据与服务都没有单点问题，Master 宕机时消息无延迟，服务与数据的可用性非常高。缺点是性能相对异步复制方式略低，发送消息的延迟会略高
     + **高可用保障**
     每个Broker与Name Server集群中的所有节点建立长连接，定时(每隔30s)注册Topic信息到所有Name Server。Name Server定时(每隔10s)扫描所有存活broker的连接，如果Name Server超过2分钟没有收到心跳，则Name Server断开与Broker的连接。
+ 消息主题Topic
    主题（Topic）可以看做消息的规类，它是消息的第一级类型。比如一个电商系统可以分为：交易消息、物流消息等，一条消息必须有一个 Topic 。

    Topic 与生产者和消费者的关系非常松散，一个 Topic可以有0个、1个、多个生产者向其发送消息，一个生产者也可以同时向不同的 Topic 发送消息。一个Topic 也可以被 0个、1个、多个消费者订阅
+ 消息队列MessageQueue
    一个Topic有4个队列，为了解决并发
+ 消息内容Message
    消息（Message）就是要传输的信息。一条消息必须有一个主题（Topic），主题可以看做是你的信件要邮寄的地址。一条消息也可以拥有一个可选的标签（Tag）和额处的键值对，它们可以用于设置一个业务 key 并在 Broker 上查找此消息以便在开发期间查找问题
+ 标签Tag
    标签（Tag）可以看作子主题，它是消息的第二级类型，用于为用户提供额外的灵活性。使用标签，同一业务模块不同目的的消息就可以用相同 Topic 而不同的 Tag 来标识。
    比如交易消息又可以分为：交易创建消息、交易完成消息等，一条消息可以没有 Tag 。标签有助于保持您的代码干净和连贯，并且还可以为 RocketMQ 提供的查询系统提供帮助
### RocketMQ消息消费模式
消息消费模式有两种：集群消费（Clustering）和广播消费（Broadcasting）
默认情况下就是集群消费，该模式一条消息只能被某一消费者组中的某一台机器消费，如果某个消费者挂掉，分组内其它消费者会接替挂掉的消费者继续消费。
而广播消费消息会发给消费者组中的每一个消费者进行消费

### RocketMQ消息消费顺序
消息顺序（Message Order）有两种：顺序消费（Orderly）和并行消费（Concurrently）。

顺序消费表示消息消费的顺序和生产者为每个消息队列发送信息时候的顺序一致，所以如果正在处理全局顺序是强制性的场景，需要确保使用的主题只有一个消息队列。

并行消费不再保证消息顺序，消费的最大并行数量受每个消费者客户端指定的线程池限制