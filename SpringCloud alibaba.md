# Spring cloud Alibaba 组件
## 远程调用Feign
#### 动态代理介绍
动态代理：是使用反射和字节码的技术，在运行期创建指定接口或类的子类以及其实例对象的技术，通过这个技术可以无侵入的为代码进行增强。

#### 动态代理的实现方式
Java中动态代理主要有JDK和CGLIB两种方式。

#### JDK实现的动态代理
##### JDK实现动态代理介绍。
JDK实现的动态代理由两个重要的成员组成，分别是Proxy、InvocationHandler

+ Proxy： 是所有动态代理的父类，它提供了一个静态方法来创建动态代理的class对象和实例

+ InvocationHandler： 每个动态代理实例都有一个关联的InvocationHandler，在代理实例上调用方法是，方法调用将被转发到InvocationHandler的invoke方法
	启动类上开启FeignClient注解，会扫描当前包及其子包下有FeignClient注解的接口
	并创建这些接口实现代理类，创建代理类对象放入容器中。
	调用时，实际上是代理类对象调用，
	1、代理类对象可以获取到当前类的实现的接口，
	2、根据接口获取到接口上的注解，
	3、根据注解获取到注解的属性值
	4、获取当前方法的引用
	5、获取方法上的注解
	6、获取传入的参数
	7、然后拼接url和参数
	8、通过feign集成的ribbon,拿到本地缓存列表将服务名替换
	9、内部使用restTemplate进行网络调用
	10、根据方法的返回值将应答进行解析

##  熔断降级Sentinel
#### 服务的雪崩
某个服务出现异常，导致调用，请求阻塞，从而导致调用方也崩溃，出现连锁反应，导致一大片服务都崩溃。
#### 常见的容错组件
+ Hystrix:Netflfix
+ Resilience4J:Hystrix
+ Sentinel:Alibaba
#### Sentinel流量卫兵
+ 集成
	1. 添加依赖
	2. 配置文件中指定Sentinel地址
	3. 启动Sentinel控制台
#### 常见的容错的思路
##### 隔离
给不同的服务分配固定的线程，不要把所有的线程都分配给某个服务，这样进行资源的隔离
##### 超时
+ 在上游服务调用调用下游服务的时候，设置一个最大相应时间没如果超时未相应，则断开请求，释放线程。
+ 在调用第三方服务时，应为第三方的接口相应时间不可控，一般会新起一个线程去调用第三方线程，自己应用会先返回响应给请求方。
##### 限流
限制系统的输入和输出流量来达到保护系统的目的，保证系统的稳固运行。
##### 熔断
当下游服务因访问压力过大而响应变慢或者崩掉，上游服务为了保护系统整体可用，可以先暂时切断对下游服务的调用。
服务熔断一般有三种状态
+ 熔断关闭状态(closed)
服务没有故障时，熔断器所处的状态，对调用方的调用不做任何限制
+ 熔断关闭状态(closed)
后续对该服务接口的调用不在经过网络，直接执行本地的fallback方法
+ 半熔断状态(Half-Open)
尝试恢复服务调用，允许有限的流量调用，并监控调用成功率。若成功率达到预期，则说明服务已经恢复，若果成功率很低，则重新进入熔断开启状态。
##### 降级
为服务提供一个默认响应。
#### Sentinel
Sentinel(分布式系统的流量防卫兵)，以流量为切入点，从流量控制、熔断降级、系统负载保护维持稳定。
Sentinel分为两个部分：
+ 核心库(Java客户端)不依赖任何框架/库，能够运行于所有jre
+ 控制台(Dashboard)基于springboot开发，打包后可以直接运行。
##### Sentinel规则
+ 流控(在请求业务之前进行控制)
	+ 线程数: 超过指定线程数
	+ QPS:每秒查询量
		1. 直接-流控模式
		+ 快速失败: 直接拒绝
		+ 预热: 前期QPS是设置QPS的三分之一,随着时间的慢慢达到设置的QPS
		+ 排队: 当达到设置的QPS,后面的请求会进行有限时长的排队
		2. 关联-流控模式: 当两个资源出现竞争资源的情况，为了保证核心资源正常，在其到达阈值后，将对其他资源进行限流
		3. 链路-流控模式: 在业务层可以指定某个链路进行流控
+ 降级(执行业务之后进行控制)
	+ 慢比例: 指定判定为慢请求的时长，当慢请求比例达到阈值，进行熔断
	+ 异常比例: 当出现异常的请求达到阈值，进行熔断
	+ 异常数: 当请求异常的数量达到阈值，进行熔断
+ 热点
	+ 查询某些热点数据进行限流
+ 授权
	Sentinel来源访问控制(黑白名单控制)
	+ 系统中需要添加RequestOriginParser接口实现类，解析请求中的权限信息
	+ 控制台中设置白名单或者黑名单
+ 系统规则
	1. LOAD
	2. RT比例
	3. 线程数
	4. 入口QPS
	5. CPU使用率
##### Sentinel自定义异常
实现BlockExceptionHandler接口
+ 限流异常FlowException
+ 降级异常DegradeException
+ 参数限流异常ParamFlowException
+ 授权异常AuthorityException
+ 系统负载异常SystemBlockException
##### @SentinelResource注解
可以针对某一个地址做特殊响应，可以定制化返回
##### Feign集成Sentinel
+ 默认没有开启 需要在配置文件中开启
+ FeignClient注解添加fallback属性，值为Feign接口的实现类
	实现接口中的方法达到返回默认数据

##  网关Gateway
##### Gateway概念
路由(Route) 是 gateway 中最基本的组件之一，表示一个具体的路由信息载体。主要定义了下面的几个
信息:
+ id，路由标识符，区别于其他 Route。 
+ uri，路由指向的目的地 uri，即客户端请求最终被转发到的微服务。
+ order，用于多个 Route 之间的排序，数值越小排序越靠前，匹配优先级越高。
+ predicate，断言的作用是进行条件判断，只有断言都返回真，才会真正的执行路由。
+ filter，过滤器用于修改请求和响应信息

# 微服务拆分
### 拆分原则
+ 基于业务逻辑
	+ 将徐听众的业务按照职责范围进行识别，职责相同的划分为一个单独的服务
+ 基于稳定性
	+ 将系统中的业务模块按照稳定性进行排序。稳定的、不经常修改划分一起，不稳定的，经常修改的划分为一起。比如日志服务、监控服务都是相对稳定的服务。
+ 基于可靠性
	+ 按照可靠性排序，
+ 基于高性能
	+ 按照对性能的要求进行优先级排序，比如全文搜索，商品查询，秒杀属于高性能的核心模块。