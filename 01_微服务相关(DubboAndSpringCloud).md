### 版本号 ###
version=1.0-20201224-001

### 基本介绍 ###
总结原理


### Dubbo相关 ###
** springCloud和Dubbo区别 **
1. Dubbo是Alibaba，SpringCloud是Spring团队
2. 注册中心：Dubbo默认使用zookeeper，SpringCloud是eureka；
3. 服务之间通信：Dubbo使用RPC，SpringCloud使用REST
4. 模块区别：1、Dubbo主要分为服务注册中心，服务提供者，服务消费者，还有管控中心；
			2、相比起Dubbo简单的四个模块，SpringCloud则是一个完整的分布式一站式框架，他有着一样的服务注册中心，服务提供者，服务消费者，管控台，断路器，分布式配置服务，消息总线，以及服务追踪等


** RPC和REST区别 **
1. RPC远程过程调用，就是像调用本地方法一样调用远程方法。可以基于TCP/HTTP协议，大部分基于TCP协议，性能高，灵活度比较低
2. REST是一种架构风格，满足约定规则的称为RESTFUL，基于HTTP协议的，比较灵活，rest对资源的操作获取，新增，修改，删除正好对应Http协议的get，post，put，delete，性能没有rpc高

**  Dubbo的应用场景 **
dubbo是一个分布式、高性能、透明化的RPC服务框架，提供服务注册、自动发现等高效服务治理方案，与spring框架集成。


**  Dubbo底层通信组件 ** 
RPC、netty、NIO

RPC基于基于传输层的传输协议，效率高，优于http
RPC层级（从下到上）：通信层，序列化层，代理层（https://blog.csdn.net/heyeqingquan/article/details/78006587）
RPC原理：两个客户端传输数据
		 发送方先将消息发送到消费方存根（client stub）
		 通过网络发送到接收方存根
		 接收方将受到的消息加工再依次返回给接收方存根–消费方存根–发送方
RPC底层原理
	包含3个角色：提供者，消费者，注册中心
	1、提供者向注册中心注册自己的服务
	2、消费者向注册中心订阅自己所需的服务
	3、注册中心将提供者的服务地址列表等发送给消费者，如有变更，注册中心会通过长连接把变更数据发送给消费者，消费者会缓存到本地

RPC框架底层使用的netty作为通信组件
netty是一个dubbo中的网络框架，基于NIO来进行，用来快速开发高性能，高可靠的网络IO程序。
应用场景：
1.RPC框架的底层使用Netty作为基础通信组件，用来实现各进程节点间的内部通信
2.网络游戏多玩家通信与对战
3.地图服务器间的通讯
4.大数据领域的高性能通讯与序列化


NIO
优点：同步非阻塞。
NIO的三大核心组件：
Selector（选择器）
Channel（通道）
Buffer（缓冲区）
NIO面向缓冲区，数据读取到一个后会进入到缓冲区，需要时可在缓冲区中前后移动进行处理。增加了灵活性。


和BIO的区别
BIO：同步并阻塞
工作流程：
服务器端启动一个ServerSocket
客户端启动socket跟服务器进行通信，默认情况下服务器端需要对每个客户的请求建立一个线程与之通信
客户端发送请求后，先咨询服务器是否有线程响应，如果没有则等待，或者被拒绝
如果有响应，客户端线程会等待请求结束后继续执行

NIO和BIO比较
BIO以流的方式处理数据，而NIO以块的方式处理数据。块I/O的效率 比 流I/O的效率高
BIO是阻塞的，NIO是非阻塞的
BIO基于字节流和字符流传输。NIO基于Channel和Buffer进行传输
BIO单个线程只能监听单个客户端，NIO单个线程可以监听多个客户端

NIO和BIO适用场景
NIO适用于连接数多而且连接比较短的架构。比如聊天，弹幕等
BIO适用于连接数目小且固定的架构。这种方式对服务器资源要求比较高，并发局限在应用内，但程序简单易理解
AIO适用于连接数目多且连接比较长的架构。比如相册等

**  Dubbo服务降级 **

解决方法：
dubbo提供了mock配置，可以很好的实现dubbo服务降级，
mock主要有两种配置方式，

第1种
在远程调用异常时，服务端直接返回一个固定的字符串(也就是写死的字符串)
具体配置：
在服务调用方配置mock参数：
`<dubbo:reference id="xxxService" interface="com.x..service.xxxxService" check="false" mock="return 123456..." />`
说明：配置了mock参数之后，假设在调用服务的时候，远程服务没有启动，或者各种网络异常了，那远程服务会把这个mock配置的值返回，也就是会返回123456...
通过这种方式就可以避免了因为服务调用不了而出现异常错误而带来的程序不可用(起码是有值返回的，然后可以根据值进行业务逻辑处理判断等等)。

第2种
在远程调用异常时，服务端根据自定义mock业务处理类进行返回
具体配置：
在服务调用方配置mock参数：
`<dubbo:reference id="xxxService" interface="com.x..service.xxxxService" check="false" mock="true" />`

说明：配置了mock参数之后，假设在调用服务的时候，远程服务没有启动，或者各种网络异常了，那远程服务会去寻找自定义的mock业务处理类进行业务处理。
因此还需配置一个自定义mock业务处理类
在接口服务xxxxService的目录下创建相应的mock业务处理类，同时实现业务接口xxxxService()，接口名要注意命名规范：接口名+Mock后缀，mock实现需要保证有无参的构造方法。
`public class xxxxServiceMock implements xxxxService {`
`    @Override
`    public String getXXXX(int id) {
`        return "this is exception 自定义....";
`    }
`}
配置完成后，此时如果调用失败会调用自定义的Mock业务实现。

**  Dubbo负载均衡 **
Dubbo内置了4种负载均衡策略:

RandomLoadBalance:随机负载均衡。随机的选择一个。是Dubbo的默认负载均衡策略。
RoundRobinLoadBalance:轮询负载均衡。轮询选择一个。
LeastActiveLoadBalance:最少活跃调用数，相同活跃数的随机。活跃数指调用前后计数差。使慢的 Provider 收到更少请求，因为越慢的 Provider 的调用前后计数差会越大。
ConsistentHashLoadBalance:一致性哈希负载均衡。相同参数的请求总是落在同一台机器上。

**  Dubbo接口暴露 **
1.通过注解暴露       
第一行写入服务的package
使用Dubbo的`@Service`注解在实现类的上面
<!-- 使用注解方式暴露接口  --> 
`<dubbo:annotation package="com.dotoyo.dsframe.form" />    


2.非注解暴露
<!-- 接口的位置 -->
` <dubbo:service interface="com.dotoyo.dsframe.form.interf.ITfPreform" ref="demoService" executes="10" />`
<!-- 实现bean，客户端应用的bean就以这个id名称为主 -->
`<bean id="demoService" class="com.dotoyo.dsframe.form.service.TfPreformServiceImpl" />`

 

**  Dubbo序列化方式 **
序列化，就是把数据结构或者是一些对象，转换为二进制串的过程
dubbo 支持 hession、Java 二进制序列化、json、SOAP 文本序列化多种序列化协议。但是 hessian 是其默认的序列化协议。

说一下 Hessian 的数据结构？
Hessian 的对象序列化机制有 8 种原始类型：
原始二进制数据
boolean
64-bit date（64 位毫秒值的日期）
64-bit double
32-bit int
64-bit long
null
UTF-8 编码的 string

另外还包括 3 种递归类型：
list for lists and arrays
map for maps and dictionaries
object for objects

还有一种特殊的类型：
ref：用来表示对共享对象的引用。

** DUBBO支持哪些协议，以及序列化协议 **
1、dubbo协议，单一长连接，NIO，序列化协议是hessian
2、rmi协议，短连接，序列化协议是java二进制
3、hessian协议，短连接，序列化协议hessian
4、http协议，短连接，序列化协议json
5、webservice协议，短连接，序列化协议soap


**  让你实现一个简单的RPC框架你会怎么做？ **
Client Code：客户端调用方代码，负责发起RPC调用，为调用方提供接口
Serialization：负责对RPC调用通过网络传输的内容进行序列化和反序列化
Stub Proxy：一个代理对象，屏蔽RPC调用过程中复杂的网络处理逻辑
Transport：作为RPC框架底层的通信传输模块，一般通过Socket在客户端和服务器端进行信息交互
Server Code：服务端服务业务逻辑具体的实现

https://blog.csdn.net/yanghan1222/article/details/80456455


**  DUBBO注册中心集群挂掉，提供者和消费者之间还能通信吗？ **
可以，消费者会到从注册中心拉取的提供者的地址接口等数据缓存到本地，按照本地存储地址调用


** DUBBO通信框架 **
NIO netty框架

** dubbo运作流程 **
包含：容器，注册中心，提供方，消费方，监控中心
运作流程：
1、容器负责服务启动，加载
2、提供方启动向注册中心注册自己的服务
3、消费方启动向注册中心订阅所需服务
4、注册中心返回服务提供者地址列表给消费者，如有变更，注册中心会基于长连接把表更数据推送消费者，消费者会缓存到本地
5、消费者通过负载均衡去选择一台提供者调用，如果调用失败会选择另一台
6、监控中心会定期统计调用的次数

### zookeeper相关 ###

** eureka和zookeeper区别 **
CAP理论，C  (数据一致性)，A（可用性），P（分区容错）
eureka确保的是可用性和分区容错（有一台机器正常就能运行，自我保护机制超过85%的没有了心跳，那么就认为eureka发生了网络瘫痪）
zookeeper确保的是一致性和分区容错（至少要一半的机器正常才能继续，每个机器的数据是一致的，leader宕机后会通过选举模式重新选择leader）

** zookeeper提供了什么 **
1、文件系统
2、通知机制（watcher事件，实现watcher接口重写process方法，当节点发送变化时，客户端收到zk的通知，然后可以感觉节点变化做出业务的变化）

** zookeeper文件系统 **
节点，每个节点都可以存储数据，不超过1M

4种节点类型
1、持久目录节点
2、持久顺序目录节点
3、临时目录节点
4、临时顺序目录节点

节点事件类型
1、EventType.NODECREATE   节点新增
2、EventType.NODEDELETE   节点删除
3、EventType.NODEDATACHANGED  节点数据改变
4、EventType.NODEChildrenChanged  节点孩子改变
5、EventType.None


** zookeeper可以做什么 **
1、命名服务
2、配置中心管理 （watcher监听，节点数据发生改变则处理）
3、集群管理
4、分布式锁
5、队列管理

** zookeeper集群管理，创建流程，选举Leader **
多个机器，每台机器一个zookeeper，修改conf文件下的配置文件，配置每台机器server* 
ip：端口：端口，第一个端口用来数据同步，第二个端口用来选举的，配置dataDir目录，在dataDir目录下创建myid文件以及对应myid
选举leader
1、每个server发出投票，使用（myid，zxid）表示
2、检查投票接受各个服务器的投票，检查是否是本次投票，以及是否是looking状态
3、处理投票
      优先检查zxid的大小，大的优先leader
      zxid相同，myid大的优先leader
server会根据自己的myid，zxid的大小确定是否要重新投票
4、统计投票，超过半数的机器收到相同的投票者当选leader，其他的为follower


** zookeeper节点宕机 **
如果是leader的话，要重新选择leader
如果是follower的话，只要还有一半以上机器正常，则不影响
集群要有一半机器以上正常才能正常工作

zookeeper节点为什么是奇数
为了满足选举需要
1、防止脑裂 例子4 > 2,2 集群彻底不能用了，集群要一半机器以上正常
2、奇数节省资源


** zookeeper分布式锁 **
保持独占
保持时序
采用的是保存独占的方式，每个节点当做一个锁，
假设用一个教学班表的主键创建节点，因为一个这个教学班表可以知道他有多少个选课的数量，以及这个学生是否符合的选课的要求，在获取锁方法里，创建节点，如果节点已经存在则用一个死循环等待释放锁并且创建获取锁，在watcher里写一个重连zk的方法，没有节点之后countdown一下。
保存时序大概的流程是
创建一个永久的父节点
然后子节点是临时有序的，最小顺序的可以获取到锁
下一个节点会监听上一节点的状态，并且同步堵塞

** zookeeper工作原理，zab协议 **
zookeeper核心是原子广播，这个机制保证了server之间的同步。实现这个机制的协议叫做ZAB协议
ZAB协议：恢复模式（选主）和广播模式（同步）

** zookeeper的server工作状态 **
looking（选举）
leading（主）
following（从）

** zookeeper同步流程 **
1、follower连接leader，并发送最大zxid给leader
2、leader根据zxid确定同步点
3、完成同步后通知follower已经成为uptodate状态（follower一半以上机器能够同步）
4、follower收到uptodate消息后，又可以重新接受client的服务


** 为什么要有leader呢 **
1.性能问题
2.有些业务逻辑只需要在一台机器执行，其他机器就能共享这个结果

** zookeeper事务一致性 **
采用的是zxid，是顺序递增事务id，两阶段提交，首先会向其他server发出事务执行请求，如果有一半机器以上能够执行并且成功，那么才会进行执行。

** JVM虚拟机为什么使用元空间替换了永久代 **

