# kafka
## 1.1 消息队列（Message Queue)
Message Queue消息传送系统提供传送服务。消息传送依赖于大量支持组件，这些组件负责处理连接服务、消息的路由和传送、持久性、安全性以及日志记录。消息服务器可以使用一个或多个代理实例。

JMS（Java Messaging Service）是Java平台上有关面向消息中间件(MOM)的技术规范，它便于消息系统中的Java应用程序进行消息交换,并且通过提供标准的产生、发送、接收消息的接口简化企业应用的开发，翻译为Java消息服务。

## 1.2 MQ消息模型
Sender (Application) ---> MessageQueue ---> Receiver (Application)

## 1.3 MQ消息队列分类
消息队列分类：点对点和发布/订阅两种
* 点对点：
  消息生产者生产消息发送到queue中，然后消息消费者从queue中取出并且消费消息。

  消息被消费以后，queue中不再有存储，所以消息消费者不可能消费到已经被消费的消息。Queue支持存在多个消费者，但是对一个消息而言，只会有一个消费者可以消费。
* 发布/订阅：
  消息生产者（发布）将消息发布到topic中，同时有多个消息消费者（订阅）消费该消息。和点对点方式不同，发布到topic的消息会被所有订阅者消费。

## 1.4 MQ消息队列对比
* RabbitMQ：支持的协议多，非常重量级消息队列，对路由(Routing)，负载均衡(Loadbalance)或者数据持久化都有很好的支持。
* ZeroMQ：号称最快的消息队列系统，尤其针对大吞吐量的需求场景，擅长的高级/复杂的队列，但是技术也复杂，并且只提供非持久性的队列。
* ActiveMQ：Apache下的一个子项，类似ZeroMQ，能够以代理人和点对点的技术实现队列。
* Redis：是一个key-Value的NOSql数据库，但也支持MQ功能，数据量较小，性能优于RabbitMQ，数据超过10K就慢的无法忍受。

# 1.5 Kafka简介
Kafka是分布式发布-订阅消息系统,它最初由 LinkedIn 公司开发，使用 Scala语言编写,之后成为 Apache 项目的一部分。在Kafka集群中，没有“中心主节点”的概念，集群中所有的服务器都是对等的，因此，可以在不做任何配置的更改的情况下实现服务器的的添加与删除，同样的消息的生产者和消费者也能够做到随意重启和机器的上下线。

# 1.6 Kafka术语介绍
* 消息生产者：即：Producer，是消息的产生的源头，负责生成消息并发送到Kafka服务器上。
* 消息消费者：即：Consumer，是消息的使用方，负责消费Kafka服务器上的消息。
* 主题：即：Topic，由用户定义并配置在Kafka服务器，用于建立生产者和消息者之间的订阅关系：生产者发送消息到指定的Topic下，消息者从这个Topic下消费消息。
* 消息分区：即：Partition，一个Topic下面会分为很多分区，例如：“kafka-test”这个Topic下可以分为6个分区，分别由两台服务器提供，那么通常可以配置为让每台服务器提供3个分区，假如服务器ID分别为0、1，则所有的分区为0-0、0-1、0-2和1-0、1-1、1-2。Topic物理上的分组，一个 topic可以分为多个 partition，每个 partition 是一个有序的队列。partition中的每条消息都会被分配一个有序的 id（offset）。
* Broker：即Kafka的服务器，用户存储消息，Kafa集群中的一台或多台服务器统称为 broker。
* 消费者分组：Group，用于归组同类消费者，在Kafka中，多个消费者可以共同消息一个Topic下的消息，每个消费者消费其中的部分消息，这些消费者就组成了一个分组，拥有同一个分组名称，通常也被称为消费者集群。
* Offset：消息存储在Kafka的Broker上，消费者拉取消息数据的过程中需要知道消息在文件中的偏移量，这个偏移量就是所谓的Offset。

# 1.7 Kafka中Broker
* Broker：即Kafka的服务器，用户存储消息，Kafa集群中的一台或多台服务器统称为 broker。
* Message在Broker中通Log追加的方式进行持久化存储。并进行分区（patitions)。
* 为了减少磁盘写入的次数,broker会将消息暂时buffer起来,当消息的个数(或尺寸)达到一定阀值时,再flush到磁盘,这样减少了磁盘IO调用的次数。
* Broker没有副本机制，一旦broker宕机，该broker的消息将都不可用。Message消息是有多份的。
* Broker不保存订阅者的状态，由订阅者自己保存。
* 无状态导致消息的删除成为难题（可能删除的消息正在被订阅），kafka采用基于时间的SLA(服务水平保证)，消息保存一定时间（通常为7天）后会被删除。
* 消息订阅者可以rewind back到任意位置重新进行消费，当订阅者故障时，可以选择最小的offset(id)进行重新读取消费消息。

# 1.8 Kafka的Message组成
* Message消息：是通信的基本单位，每个 producer 可以向一个 topic（主题）发布一些消息。
* Kafka中的Message是以topic为基本单位组织的，不同的topic之间是相互独立的。每个topic又可以分成几个不同的partition(每个topic有几个partition是在创建topic时指定的)，每个partition存储一部分Message。
* partition中的每条Message包含了以下三个属性：
  * offset 即：消息唯一标识:对应类型：long
  * MessageSize 对应类型：int32
  * data：是message的具体内容。

# 1.9 Kafka的Partitions分区
* Kafka基于文件存储.通过分区，可以将日志内容分散到多个server上,来避免文件尺寸达到单机磁盘的上限，每个partiton都会被当前server(kafka实例)保存。
* 可以将一个topic切分多任意多个partitions，来消息保存/消费的效率。
* 越多的partitions意味着可以容纳更多的consumer，有效提升并发消费的能力。

# 1.10 Kafka的Consumers
* 消息和数据消费者，订阅 topics并处理其发布的消息的过程叫做 consumers。
* 在 kafka中,我们可以认为一个group是一个“订阅者”，一个Topic中的每个partions，只会被一个“订阅者”中的一个consumer消费，不过一个 consumer可以消费多个partitions中的消息（消费者数据小于Partions的数量时）。
  * 注意：kafka的设计原理决定，对于一个topic，同一个group中不能有多于partitions个数的consumer同时消费，否则将意味着某些consumer将无法得到消息。
* 一个partition中的消息只会被group中的一个consumer消息。每个group中consumer消息消费互相独立。
