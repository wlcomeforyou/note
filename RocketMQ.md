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
+ RocketMq

### RocketMQ核心概念
+ 生产者Producer
+ 消费者Consumer
+ 名称服务NameServer
    精简版的注册中心，只有注册功能
+ 代理服务器BrokerServer
    高可用，集群，与生产、消费者者建立长连接(Netty)
+ 消息主题Topic
    消息的分类
+ 消息队列MessageQueue
    一个Topic有4个队列，为了解决并发
+ 消息内容Message
    存储在queue里
+ 标签Tag
    用于消息的过滤