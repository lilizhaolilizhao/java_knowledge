#kafka相关
##1 什么是kafka？
kafka是分布式发布-订阅消息系统，是一种分布式的消息队列工具

kafka是一个分布式的，可分区的，可复制的消息系统.kafka对消息保存的时候根据topic进行分类，发送消息者称为Producer，消息接受者称为consumer，此外kafka集群由多个kafka实例组成，每个实例称为broker

依赖zookeeper来保证系统的可用性，保存元数据信息

##2.kafka主要概念

**topic主题**

一个topic是对一组消息的归纳

在一个Kafka集群中，可以创建多个topic主题，以topic主题为单位管理消息，kafka中多个topic主题之间是互相隔离互不影响，从而可以在一个Kafka集群中通过创建多个topic主题实现不同的使用者独立使用不同topic主题而互不影响。

**partition分区**

topic可以划分出多个分区，利用分区机制保证每个分区的数据量不会太大， 可以在单个服务器上保存

分区是kafka实现负载均衡和失败恢复分布式数据存储的基本单元

每个分区可以单独发布和消费，为并发操作topic提供了可能

**offset序号**

每个分区都由一系列有序的，不可变的消息组成，这些消息被连续追加到分区中

分区中的每个消息都由一个连续的序列号叫做offset，用来在分区中唯一的标识这个消息

在一个可配置的时间段内，Kafka集群保留所有发布的消息，不管这些消息有没有被消费。

可以设置消息的保存策略，制定保存期限，在期限到来之前，数据会一直存在，无论是否被消费国，当保存期限结束，消息会被连续的擦除，释放空间

一系列的机制保证了kafka当中数据的连续读写磁盘，保证了性能，从而使得kafka的性能与数据量无关，只和磁盘的性能是常量级的

**Replication复本**

每个分区拥有若干复本，这些复本存放在不同的服务器中

若干个副本中，有一个称为leader负责读写操作，而其他的作为Leader，负责同步leader中的数据，对外只提供读的能力

kafka不是以broker为单位划分leader，follwer，而是以副本为单位划分；这样，集群中的每一个broker是持有一部分分区的leader和另一部分分区的follwer，从而将写的压力分摊到不同的broken中取，利用分布式分摊写的压力，提升性能

**Producer生产者**

生产者将消息发布到制定的主题中，默认使用简单的负载均衡机制选择分区，如果需要可以通过特定的分区函数选择分区，制定发布到哪个分区

**Consumer消费者**

Consumer负责消费主题中的数据，消费时由Consumer自己来维护会话产生的数据，实际上每个consumer唯一需要维护的数据是消息在日志中的位置，也就是offset，一般情况下随着Consumer不断的读取消息，这offset的值不断增加,从而实现连续读取数据

**Broker**

集群汇中的一台或多台服务器统称为broker

**消费者消费数据的模式**

发布订阅模式：多个Consumer可以同时从服务端读取数据，Consumer之间互不影响，每个Consumer都可以读取到全量的数据。达成了多个Consumer之间共享数据的效果。

队列模式：多个Consumer可以同时从服务端读取消息，每个消息只被其中一个Consumer读到。达成多个Consumer之间竞争数据的效果。

**消费者组的概念**

在Kafka中可以将多个消费者组成一个消费者组。

在消费者组内，多个消费者而形成竞争状态，互相抢夺数据。同一份消息只能被一个消费者组内的消费者消费一次。

在消费者组之间，多个消费者形成共享状态，共享数据。同一份消息会同时被多个消费者组各自消费到

##3 kafka的设计
**吞吐量**

数据磁盘持久化：消息不在内存中cache，直接写入到磁盘，充分利用磁盘的顺序读写性能
zero\-copy：减少IO操作步骤
数据批量发送
数据压缩
Topic划分为多个partition，提高parallelism

**负载均衡**

producer根据用户指定的算法，将消息发送到指定的partition
存在多个partiiton，每个partition有自己的replica，每个replica分布在不同的Broker节点上
多个partition需要选取出lead partition，lead partition负责读写，并由zookeeper负责fail over
通过zookeeper管理broker与consumer的动态加入与离开

**拉取系统**

kafka broker会持久化数据，broker没有内存压力，因此，consumer非常适合采取pull的方式消费数据
consumer根据消费能力自主控制消息拉取速度
consumer根据自身情况自主选择消费模式，例如批量，重复消费，从尾端开始消费等

**可扩展性**

当需要增加broker结点时，新增的broker会向zookeeper注册，而producer及consumer会根据注册在zookeeper上的watcher感知这些变化，并及时作出调整。

## 4.kafka的主要特点

1.  同时为发布和订阅提供高吞吐量。据了解，Kafka每秒可以生产约25万消息（50 MB），每秒处理55万消息（110 MB）。
2.  可进行持久化操作。将消息持久化到磁盘，因此可用于批量消费，例如ETL，以及实时应用程序。通过将数据持久化到硬盘以及replication防止数据丢失。
3.  分布式系统，易于向外扩展。所有的producer、broker和consumer都会有多个，均为分布式的。无需停机即可扩展机器。
4.  消息被处理的状态是在consumer端维护，而不是由server端维护。当失败时能自动平衡。
5.  支持online和offline的场景。

### 5.kafka文件存储机制

同一个topic下有多个不同的partition，每个partition为一个目录，partition命名的规则是topic的名称加上一个序号，序号从0开始。

![](https://img2018.cnblogs.com/blog/1451255/201907/1451255-20190726094119930-414232030.png)

每一个partition目录下的文件被平均切割成大小相等（默认一个文件是500兆，可以手动去设置）的数据文件，
每一个数据文件都被称为一个段（segment file），但每个段消息数量不一定相等，这种特性能够使得老的segment可以被快速清除。
默认保留7天的数据。

![](https://img2018.cnblogs.com/blog/1451255/201907/1451255-20190726094135648-1431349434.png)

每个partition下都会有这些每500兆一个每500兆一个（当然在上面的测试中我们将它设置为了1G一个）的segment段。

![](https://img2018.cnblogs.com/blog/1451255/201907/1451255-20190726094149118-1913223110.png)

另外每个partition只需要支持顺序读写就可以了，partition中的每一个segment端的生命周期是由我们在配置文件中指定的一个参数觉得的。
比如它在默认情况下，每满500兆就会创建新的segment段（segment file），每满7天就会清理之前的数据。

它的一个特点就是支持顺序写。如下图所示：

![](https://img2018.cnblogs.com/blog/1451255/201907/1451255-20190726094207218-1894091880.png)

首先00000000000000000000.log文件是最早产生的文件，该文件达到1G（因为我们在配置文件里面指定的1G大小，默认情况下是500兆）
之后又产生了新的0000000000000672348.log文件，新的数据会往这个新的文件里面写，这个文件达到1G之后，数据就会再往下一个文件里面写，
也就是说它只会往文件的末尾追加数据，这就是顺序写的过程，生产者只会对每一个partition做数据的追加（写）的操作。

**问题：**如何保证消息消费的有序性呢？比如说生产者生产了0到100个商品，那么消费者在消费的时候安装0到100这个从小到大的顺序消费，
那么kafka如何保证这种有序性呢？难度就在于，生产者生产出0到100这100条数据之后，通过一定的分组策略存储到broker的partition中的时候，
比如0到10这10条消息被存到了这个partition中，10到20这10条消息被存到了那个partition中，这样的话，消息在分组存到partition中的时候就已经被分组策略搞得无序了。
那么能否做到消费者在消费消息的时候全局有序呢？遇到这个问题，我们可以回答，在大多数情况下是做不到全局有序的。但在某些情况下是可以做到的。

比如我的partition只有一个，这种情况下是可以全局有序的。那么可能有人又要问了，只有一个partition的话，哪里来的分布式呢？哪里来的负载均衡呢？
所以说，全局有序是一个伪命题！全局有序根本没有办法在kafka要实现的大数据的场景来做到。但是我们只能保证当前这个partition内部消息消费的有序性。

**结论：**一个partition中的数据是有序的吗？回答：间隔有序，不连续。

针对一个topic里面的数据，只能做到partition内部有序，不能做到全局有序。特别是加入消费者的场景后，如何保证消费者的消费的消息的全局有序性，
这是一个伪命题，只有在一种情况下才能保证消费的消息的全局有序性，那就是只有一个partition！

**Segment file是什么？**
生产者生产的消息按照一定的分组策略被发送到broker中partition中的时候，这些消息如果在内存中放不下了，就会放在文件中，
partition在磁盘上就是一个目录，该目录名是topic的名称加上一个序号，在这个partition目录下，有两类文件，一类是以log为后缀的文件，
一类是以index为后缀的文件，每一个log文件和一个index文件相对应，这一对文件就是一个segment file，也就是一个段。
其中的log文件就是数据文件，里面存放的就是消息，而index文件是索引文件，索引文件记录了元数据信息。

说到segment file的索引文件和数据文件的一一对应，我们应该能想到storm中的Ack File机制，在spout发出去的时候要发一个Ack Tuple，
在下游的bolt处理完之后，它也要发一个Ack Tuple，这两个Ack Tuple里面包含了同样一份数据，这个数据叫做MessageId，它是一个对象，
这个对象里面包含两个比较重要的字段，一个是RootId，另一个是TupleId（也叫锚点Id），这个锚点Id会在我们发送数据的时候进行异或一下，
异或的结果才会发送给Ack那个Bolt。

![](https://img2018.cnblogs.com/blog/1451255/201907/1451255-20190726094258987-1353867435.png)

**Segment文件命名的规则：**partition全局的第一个segment从0（20个0）开始，后续的每一个segment文件名是上一个segment文件中最后一条消息的offset值。

那么这样命令有什么好处呢？假如我们有一个消费者已经消费到了368776（offset值为368776），那么现在我们要继续消费的话，怎么做呢？
看上图，分2个步骤，第1步是从所有文件log文件的的文件名中找到对应的log文件，第368776条数据位于上图中的“00000000000000368769.log”这个文件中，
这一步涉及到一个常用的算法叫做“二分查找法”（假如我现在给你一个offset值让你去找，你首先是将所有的log的文件名进行排序，然后通过二分查找法进行查找，
很快就能定位到某一个文件，紧接着拿着这个offset值到其索引文件中找这条数据究竟存在哪里）；第2步是到index文件中去找第368776条数据所在的位置。

索引文件（index文件）中存储这大量的元数据，而数据文件（log文件）中存储这大量的消息。

索引文件（index文件）中的元数据指向对应的数据文件（log文件）中消息的物理偏移地址。

![](https://img2018.cnblogs.com/blog/1451255/201907/1451255-20190726094318610-1176622693.png)

上图的左半部分是索引文件，里面存储的是一对一对的key\-value，其中key是消息在数据文件（对应的log文件）中的编号，比如“1,3,6,8……”，
分别表示在log文件中的第1条消息、第3条消息、第6条消息、第8条消息……，那么为什么在index文件中这些编号不是连续的呢？
这是因为index文件中并没有为数据文件中的每条消息都建立索引，而是采用了稀疏存储的方式，每隔一定字节的数据建立一条索引。
这样避免了索引文件占用过多的空间，从而可以将索引文件保留在内存中。
但缺点是没有建立索引的Message也不能一次定位到其在数据文件的位置，从而需要做一次顺序扫描，但是这次顺序扫描的范围就很小了。

其中以索引文件中元数据3,497为例，其中3代表在右边log数据文件中从上到下第3个消息(在全局partiton表示第368772个消息)，
其中497表示该消息的物理偏移地址（位置）为497。

##6.kafka存储策略

kafka通过topic来分主题存放数据，主题内又有分区，分区还可以有多个副本 。

从物理结构来看，分区本身是kafka存储目录下的一个文件夹，文件夹名称是主题名加分区编号，编号从0开始

分区的内部还有segment的概念，其实就是在分区对应的文件夹下产生的文件，

一个分区会被划分为大小相等的若干个segment，一方面保证了分区的数据被划分到多个文件中（保证了文件的体积不会太大），另一方面可以基于这些segment文件进行历史数据的删除，提高效率

一个segment由一个.log和一个.index文件组成，其中.log文件为数据文件用来存储数据分段数据，.index为索引文件保存对应的.log文件的索引信息

这两个文件的命名规则：partition全局的第一个segment从0开始，后续的每个segment文件名为上一个segment文件的最后一条消息的offset值

通过查找.index文件可以获知每个存储在当前segment中的offset在.log文件中的开始位置

每条日志有固定格式：包括offset编号，日志长度，key的长度，通过这个固定格式的数据可以确定出当前offset的结束位置，从而对数据进行读取

##7.kafka可靠性保障AR ISR OSR

**AR**
kafka分区中，维护了一个AR列表，其中包括了所有的分区的副本编号，AR分为ISR和OSR

**ISR**
同步列表，只有当所有的ISR内的副本都同步了leader中的数据，数据才能被提交，才能被消费者访问

**OSR**
非同步列表，OSR内的副本是否同步了leader的数据，不影响数据的提交，OSR内的follower只是尽力的去同步leader，数据版本可能落后。

最开始所有的副本都在ISR中，在kafka工作的过程中，如果某个副本同步速度慢于replica.lag.time.max.ms指定的阈值，则被踢出ISR 存入OSR，如果后续速度恢复可以回到ISR中

这种方案是介于leader独裁和所有民主方式之间，更加的灵活，相对于zookeeper的过半同意过半存活机制，提供了更好的可用性。牺牲了一部分的可靠性，换来的可用性对于kafka这样的消息队列来说很有意义

##8.Kafka和RabbitMQ的区别

**架构方面不同**

RabbitMQ遵循AMQP协议，RabbitMQ的broker由Exchange,Binding,queue组成，其中exchange和binding组成了消息的路由键；客户端Producer通过连接channel和server进行通信，Consumer从queue获取消息进行消费（长连接，queue有消息会推送到consumer端，consumer循环从输入流读取数据）。rabbitMQ以broker为中心；有消息的确认机制。

kafka遵从一般的MQ结构，producer，broker，consumer，以consumer为中心，消息的消费信息保存的客户端consumer上，consumer根据消费的点，从broker上批量pull数据；无消息确认机制。

**应用场景**

RabbitMQ，循AMQP协议，用于实时的对可靠性要求比较高的消息传递上。

kafka主要用于处理活跃的流式数据,大数据量的数据处理上

**吞吐量**

kafka具有高的吞吐量，内部采用消息的批量处理，zero\-copy机制，数据的存储和获取是本地磁盘顺序批量操作，具有O(1)的复杂度，消息处理的效率很高

rabbitMQ在吞吐量方面稍逊于kafka，他们的出发点不一样，rabbitMQ支持对消息的可靠的传递，支持事务，不支持批量的操作；基于存储的可靠性的要求存储可以采用内存或者硬盘。

**8.Kafka和RabbitMQ的区别**

**架构方面不同**

RabbitMQ遵循AMQP协议，RabbitMQ的broker由Exchange,Binding,queue组成，其中exchange和binding组成了消息的路由键；客户端Producer通过连接channel和server进行通信，Consumer从queue获取消息进行消费（长连接，queue有消息会推送到consumer端，consumer循环从输入流读取数据）。rabbitMQ以broker为中心；有消息的确认机制。

kafka遵从一般的MQ结构，producer，broker，consumer，以consumer为中心，消息的消费信息保存的客户端consumer上，consumer根据消费的点，从broker上批量pull数据；无消息确认机制。

**应用场景**

RabbitMQ，循AMQP协议，用于实时的对可靠性要求比较高的消息传递上。

kafka主要用于处理活跃的流式数据,大数据量的数据处理上

**吞吐量**

kafka具有高的吞吐量，内部采用消息的批量处理，zero\-copy机制，数据的存储和获取是本地磁盘顺序批量操作，具有O(1)的复杂度，消息处理的效率很高

rabbitMQ在吞吐量方面稍逊于kafka，他们的出发点不一样，rabbitMQ支持对消息的可靠的传递，支持事务，不支持批量的操作；基于存储的可靠性的要求存储可以采用内存或者硬盘。

##9.kafka生产者生产数据的可靠性

生产者向leader发送数据时，可以选择需要的可靠性级别

通过request.required.acks参数设置（0：至多一次，1：至少一次，\-1：刚好一次）

**0（至多一次）：**
生产者不停向leader发送数据，而不需要leader反馈成功消息，这种模式效率最高，可靠性最低，可能在发送过程中丢失数据。可能在leader宕机时丢失数据（可能因为网络的不稳定丢失数据。Leader宕机后，宕机期间没有接受到数据，就丢失了）

**1（默认，至少一次）：**
producer在ISR中的leader已成功收到数据并得到确认后才会发送下一条数据，如果等待响应超时，生产者自动重发数据。（不会因为网络不稳定而丢失，但可能在leader宕机而新数据未同步完成时，因新的leader选举后截断未同步数据而造成丢失数据。如果网络不稳定，在重发的过程中，可能会导致多数据）

**\-1（恰好一次）**
producer需要等待ISR中的leader和所有follower都确认接收到数据后才算一次发送完成，才会发送下一条数据，如果等待响应超时，生产者自动重发,数据可靠性最高（效率很低）。

但是这样也不能保证数据完全不丢失，例如当ISR中只有leader时，此时，leader宕机，如果不允许OSR中的follower成为新的leader可以保障写入数据的一致性，但除非原来的leader恢复，否则集群一直无法恢复。或者可以允许OSR列表中的follower成为新的leader，但此时存在写数据不一致的风险。

kafka还提供了min.insync.replicas参数，这个参数要求ISR列表中至少要有指定数量个副本leader才可以接受数据

即使配置request.required.acks=\-1，min.insync.replicas=2，也只能保证第二个层面的可靠性，即不丢数据，但仍可能多数据。如果想要实现恰好一次的语义，则需要在这个基础上进一步的加上去重机制

Kafka提供了GUID机制，能够在客户端根据算法为每条日志增加一个全局唯一标识，重发时会保持GUID一致，从而实现了标识每条数据。

##10.消费者是从leader中拿数据，还是从follow中拿数据?

kafka是由follower周期性或者尝试去pull(拉)过来(其实这个过程与consumer消费过程非常相似)，

写是都往leader上写，但是读并不是任意flower上读都行，读也只在leader上读，flower只是数据的一个备份，

保证leader被挂掉后顶上来，并不往外提供服务。

## 11.说说kafka的ISR机制？

*   kafka 为了保证数据的一致性使用了isr 机制，
*   1\. leader会维护一个与其基本保持同步的Replica列表，该列表称为ISR(in\-sync Replica)，每个Partition都会有一个ISR，而且是由leader动态维护
*   2\. 如果一个flower比一个leader落后太多，或者超过一定时间未发起数据复制请求，则leader将其重ISR中移除
*   3\. 当ISR中所有Replica都向Leader发送ACK时，leader才commit

**12.SparkStreaming连接kafka如何保证数据的不重复不丢失
**sparkStreaming接收kafka数据的方式有两种：
1.利用Receiver接收数据；
2.直接从kafka读取数据（Direct 方式）

**保证数据不丢失**
（1）Receiver方式为确保零数据丢失，必须在Spark Streaming中另外启用预写日志（Write Ahead Logs）。这将同步保存所有收到的Kafka数据到分布式文件系统（例如HDFS）上，以便在发生故障时可以恢复所有数据。
（2）Direct方式依靠checkpoint机制来保证。每次streaming 消费了kafka的数据后，将消费的kafka offsets更新到checkpoint。当你的程序挂掉或者升级的时候，就可以接着上次的读取，实现数据的零丢失。
（Direct需要用户采用checkpoint或者第三方存储来维护offsets，而不像Receiver\-based那样，通过ZooKeeper来维护Offsets，此提高了用户的开发成本）

kafka的acks参数有一个非常重要的作用。如果acks设置为0，表示Producer不会等待Broker的响应，Producer无法确定消息是否发送成功，可能会导致数据丢失，但acks值为0时，会得到最大的系统吞吐量。如果acks设置为1，表示Producer会在leader Partition收到消息并得到Broker的一个确认，这样会有更好的可靠性。如果设置为\-1，Producer会在所有备份的Partition收到消息时得到Broker的确认，这个设置可以得到最高的可靠性保证。

**保证数据不重复**

这里业务场景被区分为两个:

**幂等操作**

**业务代码需要自身添加事物操作**

所谓幂等操作就是重复执行不会产生问题，如果是这种场景下，你不需要额外做任何工作。但如果你的应用场景是不允许数据被重复执行 的，那只能通过业务自身的逻辑代码来解决了。
这个spark给出了官方方案:

	dstream.foreachRDD {(rdd, time) =
	              rdd.foreachPartition { partitionIterator =>
	                val partitionId = TaskContext.get.partitionId()
	                val uniqueId = generateUniqueId(time.milliseconds,partitionId)
	                //use this uniqueId to transationally commit the data in partitionIterator
	                 }
	      }
　　就是说针对每个partition的数据，产生一个uniqueId，只有这个partition的所有数据被完全消费，则算成功，否则算失效，要回滚。下次重复执行这个uniqueId时，如果已经被执行成功，则skip掉。
　　
##13.kafka 是如何清理过期数据的？

kafka的日志实际上是以日志的方式默认保存在/kafka\-logs文件夹中的，默认7天清理机制，

日志的真正清理时间。当删除的条件满足以后，日志将被“删除”，但是这里的删除其实只是将

该日志进行了“delete”标注，文件只是无法被索引到了而已。但是文件本身，仍然是存在的，只有当过了log.segment.delete.delay.ms 这个时间以后，文件才会被真正的从文件系统中删除。

##14.一条message中包含哪些信息？

*   包含 header,body。
*   一个Kafka的Message由一个固定长度的header和一个变长的消息体body组成。
*   header部分由一个字节的magic(文件格式)和四个字节的CRC32(用于判断body消息体是否正常)构成。
*   当magic的值为1的时候，会在magic和crc32之间多一个字节的数据：attributes(保存一些相关属性，比如是否压缩、
压缩格式等等)；*   如果magic的值为0，那么不存在attributes属性body是由N个字节构成的一个消息体，包含了具体的key/value消息

#kafka系统设计开篇
## 引言

MQ（消息队列）是跨进程通信的方式之一，可理解为异步rpc，上游系统对调用结果的态度往往是重要不紧急。使用消息队列有以下好处：业务解耦、流量削峰、灵活扩展。接下来介绍消息中间件Kafka。

## Kafka是什么？

Kafka是一个分布式的消息引擎。具有以下特征

能够发布和订阅消息流（类似于消息队列）
以容错的、持久的方式存储消息流
多分区概念，提高了并行能力

## Kafka架构总览

![Kafka系统架构](https://blog-article-resource.oss-cn-beijing.aliyuncs.com/kafka/kafka%E6%9E%B6%E6%9E%84.png)

## Topic

消息的主题、队列，每一个消息都有它的topic，Kafka通过topic对消息进行归类。Kafka中可以将Topic从物理上划分成一个或多个分区（Partition），每个分区在物理上对应一个文件夹，以”topicName\_partitionIndex”的命名方式命名，该dir包含了这个分区的所有消息(.log)和索引文件(.index)，这使得Kafka的吞吐率可以水平扩展。

## Partition

每个分区都是一个 顺序的、不可变的消息队列， 并且可以持续的添加;分区中的消息都被分了一个序列号，称之为偏移量(offset)，在每个分区中此偏移量都是唯一的。
producer在发布消息的时候，可以为每条消息指定Key，这样消息被发送到broker时，会根据分区算法把消息存储到对应的分区中（一个分区存储多个消息），如果分区规则设置的合理，那么所有的消息将会被均匀的分布到不同的分区中，这样就实现了负载均衡。
![partition_info](https://blog-article-resource.oss-cn-beijing.aliyuncs.com/kafka/partition.jpg)

## Broker

Kafka server，用来存储消息，Kafka集群中的每一个服务器都是一个Broker，消费者将从broker拉取订阅的消息
Producer
向Kafka发送消息，生产者会根据topic分发消息。生产者也负责把消息关联到Topic上的哪一个分区。最简单的方式从分区列表中轮流选择。也可以根据某种算法依照权重选择分区。算法可由开发者定义。

## Cousumer

Consermer实例可以是独立的进程，负责订阅和消费消息。消费者用consumerGroup来标识自己。同一个消费组可以并发地消费多个分区的消息，同一个partition也可以由多个consumerGroup并发消费，但是在consumerGroup中一个partition只能由一个consumer消费

## CousumerGroup

Consumer Group：同一个Consumer Group中的Consumers，Kafka将相应Topic中的每个消息只发送给其中一个Consumer

# Kafka producer 设计原理

## 发送消息的流程

![partition_info](https://blog-article-resource.oss-cn-beijing.aliyuncs.com/kafka/sendMsg.jpg)
**1.序列化消息&&.计算partition**
根据key和value的配置对消息进行序列化,然后计算partition：
ProducerRecord对象中如果指定了partition，就使用这个partition。否则根据key和topic的partition数目取余，如果key也没有的话就随机生成一个counter，使用这个counter来和partition数目取余。这个counter每次使用的时候递增。

**2发送到batch&&唤醒Sender 线程**
根据topic\-partition获取对应的batchs（Dueue ），然后将消息append到batch中.如果有batch满了则唤醒Sender 线程。队列的操作是加锁执行，所以batch内消息时有序的。后续的Sender操作当前方法异步操作。
![send_msg](https://blog-article-resource.oss-cn-beijing.aliyuncs.com/kafka/send2Batch1.png)![send_msg2](https://blog-article-resource.oss-cn-beijing.aliyuncs.com/kafka/send2Batch2.png)

**3.Sender把消息有序发到 broker（tp replia leader）**
**3.1 确定tp relica leader 所在的broker**

Kafka中 每台broker都保存了kafka集群的metadata信息，metadata信息里包括了每个topic的所有partition的信息: leader, leader\_epoch, controller\_epoch, isr, replicas等;Kafka客户端从任一broker都可以获取到需要的metadata信息;sender线程通过metadata信息可以知道tp leader的brokerId
producer也保存了metada信息，同时根据metadata更新策略（定期更新metadata.max.age.ms、失效检测，强制更新：检查到metadata失效以后，调用metadata.requestUpdate()强制更新

```
public class PartitionInfo {
    private final String topic; private final int partition;
    private final Node leader; private final Node[] replicas;
    private final Node[] inSyncReplicas; private final Node[] offlineReplicas;
}
```

**3.2 幂等性发送**

为实现Producer的幂等性，Kafka引入了Producer ID（即PID）和Sequence Number。对于每个PID，该Producer发送消息的每个<Topic, Partition>都对应一个单调递增的Sequence Number。同样，Broker端也会为每个<PID, Topic, Partition>维护一个序号，并且每Commit一条消息时将其对应序号递增。对于接收的每条消息，如果其序号比Broker维护的序号）大一，则Broker会接受它，否则将其丢弃：

如果消息序号比Broker维护的序号差值比一大，说明中间有数据尚未写入，即乱序，此时Broker拒绝该消息，Producer抛出InvalidSequenceNumber
如果消息序号小于等于Broker维护的序号，说明该消息已被保存，即为重复消息，Broker直接丢弃该消息，Producer抛出DuplicateSequenceNumber
Sender发送失败后会重试，这样可以保证每个消息都被发送到broker

**4\. Sender处理broker发来的produce response**
一旦broker处理完Sender的produce请求，就会发送produce response给Sender，此时producer将执行我们为send（）设置的回调函数。至此producer的send执行完毕。

## 吞吐性&&延时：

buffer.memory：buffer设置大了有助于提升吞吐性，但是batch太大会增大延迟，可搭配linger\_ms参数使用
linger\_ms：如果batch太大，或者producer qps不高，batch添加的会很慢，我们可以强制在linger\_ms时间后发送batch数据
ack：producer收到多少broker的答复才算真的发送成功
0表示producer无需等待leader的确认(吞吐最高、数据可靠性最差)
1代表需要leader确认写入它的本地log并立即确认
\-1/all 代表所有的ISR都完成后确认(吞吐最低、数据可靠性最高)

## Sender线程和长连接

每初始化一个producer实例，都会初始化一个Sender实例，新增到broker的长连接。
代码角度：每初始化一次KafkaProducer，都赋一个空的client

```
public KafkaProducer(final Map<String, Object> configs) {
    this(configs, null, null, null, null, null, Time.SYSTEM);
}
```

![Sender_io](https://blog-article-resource.oss-cn-beijing.aliyuncs.com/kafka/SenderIO.jpg)

终端查看TCP连接数：
lsof \-p portNum \-np | grep TCP

# Consumer设计原理

## poll消息

![consumer-pool](https://blog-article-resource.oss-cn-beijing.aliyuncs.com/kafka/consumerPoll.jpg)

*   消费者通过fetch线程拉消息（单线程）
*   消费者通过心跳线程来与broker发送心跳。超时会认为挂掉
*   每个consumer
    group在broker上都有一个coordnator来管理，消费者加入和退出，以及消费消息的位移都由coordnator处理。

## 位移管理

consumer的消息位移代表了当前group对topic\-partition的消费进度，consumer宕机重启后可以继续从该offset开始消费。
在kafka0.8之前，位移信息存放在zookeeper上，由于zookeeper不适合高并发的读写，新版本Kafka把位移信息当成消息，发往\_\_consumers\_offsets 这个topic所在的broker，\_\_consumers\_offsets默认有50个分区。
消息的key 是groupId+topic\_partition,value 是offset.

![consumerOffsetDat](https://blog-article-resource.oss-cn-beijing.aliyuncs.com/kafka/consumerOffsetData.jpg)![consumerOffsetView](https://blog-article-resource.oss-cn-beijing.aliyuncs.com/kafka/consumerOffsetView.jpg)

## Kafka Group 状态

![groupState](https://blog-article-resource.oss-cn-beijing.aliyuncs.com/kafka/groupState.jpg)

*   Empty：初始状态，Group 没有任何成员，如果所有的 offsets 都过期的话就会变成 Dead
*   PreparingRebalance：Group 正在准备进行 Rebalance
*   AwaitingSync：Group 正在等待来 group leader 的 分配方案
*   Stable：稳定的状态（Group is stable）；
*   Dead：Group 内已经没有成员，并且它的 Metadata 已经被移除

## 重平衡reblance

当一些原因导致consumer对partition消费不再均匀时，kafka会自动执行reblance，使得consumer对partition的消费再次平衡。
什么时候发生rebalance？：

*   组订阅topic数变更
*   topic partition数变更
*   consumer成员变更
*   consumer 加入群组或者离开群组的时候
*   consumer被检测为崩溃的时候

## reblance过程

举例1 consumer被检测为崩溃引起的reblance
比如心跳线程在timeout时间内没和broker发送心跳，此时coordnator认为该group应该进行reblance。接下来其他consumer发来fetch请求后，coordnator将回复他们进行reblance通知。当consumer成员收到请求后，只有leader会根据分配策略进行分配，然后把各自的分配结果返回给coordnator。这个时候只有consumer leader返回的是实质数据，其他返回的都为空。收到分配方法后，consumer将会把分配策略同步给各consumer

举例2 consumer加入引起的reblance

使用join协议，表示有consumer 要加入到group中
使用sync 协议，根据分配规则进行分配
![reblance-join](https://blog-article-resource.oss-cn-beijing.aliyuncs.com/kafka/reblance-join.jpg)![reblance-sync](https://blog-article-resource.oss-cn-beijing.aliyuncs.com/kafka/reblance-sync.jpg)

(上图图片摘自网络)

## 引申：以上reblance机制存在的问题

在大型系统中，一个topic可能对应数百个consumer实例。这些consumer陆续加入到一个空消费组将导致多次的rebalance；此外consumer 实例启动的时间不可控，很有可能超出coordinator确定的rebalance timeout(即max.poll.interval.ms)，将会再次触发rebalance，而每次rebalance的代价又相当地大，因为很多状态都需要在rebalance前被持久化，而在rebalance后被重新初始化。

## 新版本改进

**通过延迟进入PreparingRebalance状态减少reblance次数**

![groupStateOfNewVersion](https://blog-article-resource.oss-cn-beijing.aliyuncs.com/kafka/groupStateOfNewVersion.jpg)

新版本新增了group.initial.rebalance.delay.ms参数。空消费组接受到成员加入请求时，不立即转化到PreparingRebalance状态来开启reblance。当时间超过group.initial.rebalance.delay.ms后，再把group状态改为PreparingRebalance（开启reblance）。实现机制是在coordinator底层新增一个group状态：InitialReblance。假设此时有多个consumer陆续启动，那么group状态先转化为InitialReblance，待group.initial.rebalance.delay.ms时间后，再转换为PreparingRebalance（开启reblance）

# Broker设计原理

Broker 是Kafka 集群中的节点。负责处理生产者发送过来的消息，消费者消费的请求。以及集群节点的管理等。由于涉及内容较多，先简单介绍，后续专门抽出一篇文章分享 

## broker zk注册

![brokersInZk](https://blog-article-resource.oss-cn-beijing.aliyuncs.com/kafka/brokersInZk.jpg)

## broker消息存储

Kafka的消息以二进制的方式紧凑地存储，节省了很大空间  
此外消息存在ByteBuffer而不是堆，这样broker进程挂掉时，数据不会丢失，同时避免了gc问题   
通过零拷贝和顺序寻址，让消息存储和读取速度都非常快  
处理fetch请求的时候通过zero\-copy 加快速度 

## broker状态数据

broker设计中，每台机器都保存了相同的状态数据。主要包括以下：

controller所在的broker ID，即保存了当前集群中controller是哪台broker
集群中所有broker的信息：比如每台broker的ID、机架信息以及配置的若干组连接信息
集群中所有节点的信息：严格来说，它和上一个有些重复，不过此项是按照broker ID和\*\*\*类型进行分组的。对于超大集群来说，使用这一项缓存可以快速地定位和查找给定节点信息，而无需遍历上一项中的内容，算是一个优化吧
集群中所有分区的信息：所谓分区信息指的是分区的leader、ISR和AR信息以及当前处于offline状态的副本集合。这部分数据按照topic\-partitionID进行分组，可以快速地查找到每个分区的当前状态。（注：AR表示assigned replicas，即创建topic时为该分区分配的副本集合）

## broker负载均衡

**分区数量负载**：各台broker的partition数量应该均匀
partition Replica分配算法如下：

将所有Broker（假设共n个Broker）和待分配的Partition排序
将第i个Partition分配到第（i mod n）个Broker上
将第i个Partition的第j个Replica分配到第（(i + j) mod n）个Broker上

**容量大小负载：**每台broker的硬盘占用大小应该均匀
在kafka1.1之前，Kafka能够保证各台broker上partition数量均匀，但由于每个partition内的消息数不同，可能存在不同硬盘之间内存占用差异大的情况。在Kafka1.1中增加了副本跨路径迁移功能kafka\-reassign\-partitions.sh，我们可以结合它和监控系统，实现自动化的负载均衡

# Kafka高可用

在介绍kafka高可用之前先介绍几个概念

同步复制：要求所有能工作的Follower都复制完，这条消息才会被认为commit，这种复制方式极大的影响了吞吐率
异步复制：Follower异步的从Leader pull数据，data只要被Leader写入log认为已经commit，这种情况下如果Follower落后于Leader的比较多，如果Leader突然宕机，会丢失数据

## Isr

Kafka结合同步复制和异步复制，使用ISR（与Partition Leader保持同步的Replica列表）的方式在确保数据不丢失和吞吐率之间做了平衡。Producer只需把消息发送到Partition Leader，Leader将消息写入本地Log。Follower则从Leader pull数据。Follower在收到该消息向Leader发送ACK。一旦Leader收到了ISR中所有Replica的ACK，该消息就被认为已经commit了，Leader将增加HW并且向Producer发送ACK。这样如果leader挂了，只要Isr中有一个replica存活，就不会丢数据。

## Isr动态更新

Leader会跟踪ISR，如果ISR中一个Follower宕机，或者落后太多，Leader将把它从ISR中移除。这里所描述的“落后太多”指Follower复制的消息落后于Leader后的条数超过预定值（replica.lag.max.messages）或者Follower超过一定时间（replica.lag.time.max.ms）未向Leader发送fetch请求。

broker Nodes In Zookeeper
/brokers/topics/\[topic\]/partitions/\[partition\]/state 保存了topic\-partition的leader和Isr等信息

![partitionStateInZk](https://blog-article-resource.oss-cn-beijing.aliyuncs.com/kafka/partitionStateInZk.jpg)

## Controller负责broker故障检查&&故障转移（fail/recover）

1.  Controller在Zookeeper上注册Watch，一旦有Broker宕机，其在Zookeeper对应的znode会自动被删除，Zookeeper会触发
    Controller注册的watch，Controller读取最新的Broker信息
2.  Controller确定set\_p，该集合包含了宕机的所有Broker上的所有Partition
3.  对set\_p中的每一个Partition，选举出新的leader、Isr，并更新结果。

3.1 从/brokers/topics/\[topic\]/partitions/\[partition\]/state读取该Partition当前的ISR

3.2 决定该Partition的新Leader和Isr。如果当前ISR中有至少一个Replica还幸存，则选择其中一个作为新Leader，新的ISR则包含当前ISR中所有幸存的Replica。否则选择该Partition中任意一个幸存的Replica作为新的Leader以及ISR（该场景下可能会有潜在的数据丢失）

![electLeader](https://blog-article-resource.oss-cn-beijing.aliyuncs.com/kafka/electLeader.jpg)
3.3 更新Leader、ISR、leader\_epoch、controller\_epoch：写入/brokers/topics/\[topic\]/partitions/\[partition\]/state

1.  直接通过RPC向set\_p相关的Broker发送LeaderAndISRRequest命令。Controller可以在一个RPC操作中发送多个命令从而提高效率。

## Controller挂掉

每个 broker 都会在 zookeeper 的临时节点 "/controller" 注册 watcher，当 controller 宕机时 "/controller" 会消失，触发broker的watch，每个 broker 都尝试创建新的 controller path，只有一个竞选成功并当选为 controller。  

# 使用Kafka如何保证幂等性

不丢消息

首先kafka保证了对已提交消息的at least保证   
Sender有重试机制   
producer业务方在使用producer发送消息时，注册回调函数。在onError方法中重发消息   
consumer 拉取到消息后，处理完毕再commit，保证commit的消息一定被处理完毕   

不重复

consumer拉取到消息先保存，commit成功后删除缓存数据

# Kafka高性能

partition提升了并发  
zero\-copy  
顺序写入   
消息聚集batch  
页缓存  
业务方对 Kafka producer的优化  

增大producer数量  
ack配置  
batch  


*   Kafka的用途有哪些？使用场景如何？  
    Kafka具有吞吐量大 简单的优点，适用于日志收集 大数据实时计算等场景

*   Kafka中的ISR、AR又代表什么？ISR的伸缩又指什么  
    AR：Assigned Replicas 所有副本列表  
    ISR：InSync Replicas 同步副本列表  
    ISR expand ： 有副本恢复同步状态  
    ISR shrink ： 有副本脱离同步状态  

*   Kafka中的HW、LEO、LSO、LW等分别代表什么？  
    HW： High Watermark/高水位。 是已备份消息位置，HW之前的消息均可被consumer消费 leader.HW=min(ISR.LEO) follower.HW=min(follower.LEO,leader.HW)  
    LEO: Log End Offset/日志末端偏移。是**下一条**消息写入位置(LEO=10 有9条消息)  
    LSO:last stable offset/稳定偏移 。 LSO之前的消息状态都已确认（commit/aborted）主要用于事务  
    LW:

*   Kafka中是怎么体现消息顺序性的？  
    kafka每个partition中的消息在写入时都是有序的，消费时，每个partition只能被每一个group中的 **一个**消费者消费，保证了消费时也是有序的。  
    整个topic不保证有序  

*   Kafka中的分区器、序列化器、拦截器是否了解？它们之间的处理顺序是什么？  
    分区器:根据键值确定消息应该处于哪个分区中，默认情况下使用轮询分区，可以自行实现分区器接口自定义分区逻辑  
    序列化器:键序列化器和值序列化器，将键和值都转为二进制流 还有反序列化器 将二进制流转为指定类型数据  
    拦截器:两个方法 doSend()方法会在序列化之前完成 onAcknowledgement()方法在消息确认或失败时调用 可以添加多个拦截器按顺序执行  
    调用顺序: 拦截器doSend() \-> 序列化器 \-> 分区器  

*   Kafka生产者客户端的整体结构是什么样子的？  

*   “消费组中的消费者个数如果超过topic的分区，那么就会有消费者消费不到数据”这句话是否正确？如果不正确，那么有没有什么hack的手段？  
    正确

*   消费者提交消费位移时提交的是当前消费到的最新消息的offset还是offset+1?  
    offset+1  

*   有哪些情形会造成重复消费？  
    消费者消费后没有commit offset(程序崩溃/强行kill/消费耗时/自动提交偏移情况下unscrible)  

*   那些情景下会造成消息漏消费？  
    消费者没有处理完消息 提交offset(自动提交偏移 未处理情况下程序异常结束)  

*   KafkaConsumer是非线程安全的，那么怎么样实现多线程消费？  
    每个线程一个消费者  

*   简述消费者与消费组之间的关系  
    消费者从属与消费组，消费偏移以消费组为单位。每个消费组可以独立消费主题的所有数据，同一消费组内消费者共同消费主题数据，每个分区只能被同一消费组内一个消费者消费。

*   当你使用kafka\-topics.sh创建（删除）了一个topic之后，Kafka背后会执行什么逻辑？  
    创建:在zk上/brokers/topics/下节点 kafkabroker会监听节点变化创建主题  
    删除:调用脚本删除topic会在zk上将topic设置待删除标志，kafka后台有定时的线程会扫描所有需要删除的topic进行删除  

*   topic的分区数可不可以增加？如果可以怎么增加？如果不可以，那又是为什么？  
    可以增加  

*   topic的分区数可不可以减少？如果可以怎么减少？如果不可以，那又是为什么？  
    不能减少 会丢失数据  

*   创建topic时如何选择合适的分区数？    
    根据集群的机器数量和需要的吞吐量来决定适合的分区数  

*   Kafka目前有那些内部topic，它们都有什么特征？各自的作用又是什么？  
    \_\_consumer\_offsets 以双下划线开头，保存消费组的偏移

*   优先副本是什么？它有什么特殊的作用？  
    优先副本 会是默认的leader副本 发生leader变化时重选举会优先选择优先副本作为leader

*   Kafka有哪几处地方有分区分配的概念？简述大致的过程及原理

1.  创建主题时  
    如果不手动指定分配方式 有两种分配方式

2.  消费组内分配

*   简述Kafka的日志目录结构  
    每个partition一个文件夹，包含四类文件.index .log .timeindex leader\-epoch\-checkpoint
    .index .log .timeindex 三个文件成对出现 前缀为上一个segment的最后一个消息的偏移 log文件中保存了所有的消息 index文件中保存了稀疏的相对偏移的索引 timeindex保存的则是时间索引
    leader\-epoch\-checkpoint中保存了每一任leader开始写入消息时的offset 会定时更新
    follower被选为leader时会根据这个确定哪些消息可用
    
**1 什么是kafka**

> Kafka是分布式发布\-订阅消息系统，它最初是由LinkedIn公司开发的，之后成为Apache项目的一部分，Kafka是一个分布式，可划分的，冗余备份的持久性的日志服务，它主要用于处理流式数据。

**2 为什么要使用 kafka，为什么要使用消息队列**

> 缓冲和削峰：上游数据时有突发流量，下游可能扛不住，或者下游没有足够多的机器来保证冗余，kafka在中间可以起到一个缓冲的作用，把消息暂存在kafka中，下游服务就可以按照自己的节奏进行慢慢处理。
>
> 解耦和扩展性：项目开始的时候，并不能确定具体需求。消息队列可以作为一个接口层，解耦重要的业务流程。只需要遵守约定，针对数据编程即可获取扩展能力。
>
> 冗余：可以采用一对多的方式，一个生产者发布消息，可以被多个订阅topic的服务消费到，供多个毫无关联的业务使用。
>
> 健壮性：消息队列可以堆积请求，所以消费端业务即使短时间死掉，也不会影响主要业务的正常进行。
>
> 异步通信：很多时候，用户不想也不需要立即处理消息。消息队列提供了异步处理机制，允许用户把一个消息放入队列，但并不立即处理它。想向队列中放入多少消息就放多少，然后在需要的时候再去处理它们。

**3.Kafka中的ISR、AR又代表什么？ISR的伸缩又指什么**

> ISR:In\-Sync Replicas 副本同步队列
> AR:Assigned Replicas 所有副本
> ISR是由leader维护，follower从leader同步数据有一些延迟（包括延迟时间replica.lag.time.max.ms和延迟条数replica.lag.max.messages两个维度, 当前最新的版本0.10.x中只支持replica.lag.time.max.ms这个维度），任意一个超过阈值都会把follower剔除出ISR, 存入OSR（Outof\-Sync Replicas）列表，新加入的follower也会先存放在OSR中。AR=ISR+OSR。

**4.kafka中的broker 是干什么的**

> broker 是消息的代理，Producers往Brokers里面的指定Topic中写消息，Consumers从Brokers里面拉取指定Topic的消息，然后进行业务处理，broker在中间起到一个代理保存消息的中转站。

**5.kafka中的 zookeeper 起到什么作用，可以不用zookeeper么**

> zookeeper 是一个分布式的协调组件，早期版本的kafka用zk做meta信息存储，consumer的消费状态，group的管理以及 offset的值。考虑到zk本身的一些因素以及整个架构较大概率存在单点问题，新版本中逐渐弱化了zookeeper的作用。新的consumer使用了kafka内部的group coordination协议，也减少了对zookeeper的依赖，
>
> 但是broker依然依赖于ZK，zookeeper 在kafka中还用来选举controller 和 检测broker是否存活等等。

**6.kafka follower如何与leader同步数据**

> Kafka的复制机制既不是完全的同步复制，也不是单纯的异步复制。完全同步复制要求All Alive Follower都复制完，这条消息才会被认为commit，这种复制方式极大的影响了吞吐率。而异步复制方式下，Follower异步的从Leader复制数据，数据只要被Leader写入log就被认为已经commit，这种情况下，如果leader挂掉，会丢失数据，kafka使用ISR的方式很好的均衡了确保数据不丢失以及吞吐率。Follower可以批量的从Leader复制数据，而且Leader充分利用磁盘顺序读以及send file(zero copy)机制，这样极大的提高复制性能，内部批量写磁盘，大幅减少了Follower与Leader的消息量差。

**7.什么情况下一个 broker 会从 isr中踢出去**

> leader会维护一个与其基本保持同步的Replica列表，该列表称为ISR(in\-sync Replica)，每个Partition都会有一个ISR，而且是由leader动态维护 ，如果一个follower比一个leader落后太多，或者超过一定时间未发起数据复制请求，则leader将其重ISR中移除 。

**8.kafka 为什么那么快**

> *   Cache Filesystem Cache PageCache缓存
>
> *   顺序写 由于现代的操作系统提供了预读和写技术，磁盘的顺序写大多数情况下比随机写内存还要快。
>
> *   Zero\-copy 零拷技术减少拷贝次数
>
> *   Batching of Messages 批量量处理。合并小的请求，然后以流的方式进行交互，直顶网络上限。
>
> *   Pull 拉模式 使用拉模式进行消息的获取消费，与消费端处理能力相符。
>

**9.kafka producer如何优化打入速度**

> *   增加线程
>
> *   提高 batch.size
>
> *   增加更多 producer 实例
>
> *   增加 partition 数
>
> *   设置 acks=\-1 时，如果延迟增大：可以增大 num.replica.fetchers（follower 同步数据的线程数）来调解；
>
> *   跨数据中心的传输：增加 socket 缓冲区设置以及 OS tcp 缓冲区设置。
>

**10.kafka producer 打数据，ack  为 0， 1， \-1 的时候代表啥， 设置 \-1 的时候，什么情况下，leader 会认为一条消息 commit了**

> 1.  1（默认）  数据发送到Kafka后，经过leader成功接收消息的的确认，就算是发送成功了。在这种情况下，如果leader宕机了，则会丢失数据。
> 2.  0 生产者将数据发送出去就不管了，不去等待任何返回。这种情况下数据传输效率最高，但是数据可靠性确是最低的。
> 3.  \-1 producer需要等待ISR中的所有follower都确认接收到数据后才算一次发送完成，可靠性最高。当ISR中所有Replica都向Leader发送ACK时，leader才commit，这时候producer才能认为一个请求中的消息都commit了。

**11.kafka  unclean 配置代表啥，会对 spark streaming 消费有什么影响**

> unclean.leader.election.enable 为true的话，意味着非ISR集合的broker 也可以参与选举，这样有可能就会丢数据，spark streaming在消费过程中拿到的 end offset 会突然变小，导致 spark streaming job挂掉。如果unclean.leader.election.enable参数设置为true，就有可能发生数据丢失和数据不一致的情况，Kafka的可靠性就会降低；而如果unclean.leader.election.enable参数设置为false，Kafka的可用性就会降低。

**12.如果leader crash时，ISR为空怎么办**

> kafka在Broker端提供了一个配置参数：unclean.leader.election,这个参数有两个值：
> true（默认）：允许不同步副本成为leader，由于不同步副本的消息较为滞后，此时成为leader，可能会出现消息不一致的情况。
> false：不允许不同步副本成为leader，此时如果发生ISR列表为空，会一直等待旧leader恢复，降低了可用性。

**13.kafka的message格式是什么样的**

> 一个Kafka的Message由一个固定长度的header和一个变长的消息体body组成
>
> header部分由一个字节的magic(文件格式)和四个字节的CRC32(用于判断body消息体是否正常)构成。
>
> 当magic的值为1的时候，会在magic和crc32之间多一个字节的数据：attributes(保存一些相关属性，
>
> 比如是否压缩、压缩格式等等);如果magic的值为0，那么不存在attributes属性
>
> body是由N个字节构成的一个消息体，包含了具体的key/value消息

**14.kafka中consumer group 是什么概念**

> 同样是逻辑上的概念，是Kafka实现单播和广播两种消息模型的手段。同一个topic的数据，会广播给不同的group；同一个group中的worker，只有一个worker能拿到这个数据。换句话说，对于同一个topic，每个group都可以拿到同样的所有数据，但是数据进入group后只能被其中的一个worker消费。group内的worker可以使用多线程或多进程来实现，也可以将进程分散在多台机器上，worker的数量通常不超过partition的数量，且二者最好保持整数倍关系，因为Kafka在设计时假定了一个partition只能被一个worker消费（同一group内）。

**15.Kafka中的消息是否会丢失和重复消费？**

> 要确定Kafka的消息是否丢失或重复，从两个方面分析入手：消息发送和消息消费。
>
> **1、消息发送**
>
>          Kafka消息发送有两种方式：同步（sync）和异步（async），默认是同步方式，可通过producer.type属性进行配置。Kafka通过配置request.required.acks属性来确认消息的生产：
>
> 1.  *0\-\-\-表示不进行消息接收是否成功的确认；*
> 2.  *1\-\-\-表示当Leader接收成功时确认；*
> 3.  *\-1\-\-\-表示Leader和Follower都接收成功时确认；*
>
> 综上所述，有6种消息生产的情况，下面分情况来分析消息丢失的场景：
>
> （1）acks=0，不和Kafka集群进行消息接收确认，则当网络异常、缓冲区满了等情况时，**消息可能丢失**；
>
> （2）acks=1、同步模式下，只有Leader确认接收成功后但挂掉了，副本没有同步，**数据可能丢失**；
>
> **2、消息消费**
>
> Kafka消息消费有两个consumer接口，Low\-level API和High\-level API：
>
> 1.  Low\-level API：消费者自己维护offset等值，可以实现对Kafka的完全控制；
>
> 2.  High\-level API：封装了对parition和offset的管理，使用简单；
>
>
> 如果使用高级接口High\-level API，可能存在一个问题就是当消息消费者从集群中把消息取出来、并提交了新的消息offset值后，还没来得及消费就挂掉了，那么下次再消费时之前没消费成功的消息就“*诡异*”的消失了；
>
> **解决办法**：
>
>         针对消息丢失：同步模式下，确认机制设置为\-1，即让消息写入Leader和Follower之后再确认消息发送成功；异步模式下，为防止缓冲区满，可以在配置文件设置不限制阻塞超时时间，当缓冲区满时让生产者一直处于阻塞状态；
>
>         针对消息重复：将消息的唯一标识保存到外部介质中，每次消费时判断是否处理过即可。
>
> 消息重复消费及解决参考：[https://www.javazhiyin.com/22910.html](https://www.javazhiyin.com/22910.html)

**16.为什么Kafka不支持读写分离？**

> 在 Kafka 中，生产者写入消息、消费者读取消息的操作都是与 leader 副本进行交互的，从 而实现的是一种**主写主读**的生产消费模型。
>
> Kafka 并不支持主写从读，因为主写从读有 2 个很明 显的缺点:
>
> *   (1)**数据一致性问题**。数据从主节点转到从节点必然会有一个延时的时间窗口，这个时间 窗口会导致主从节点之间的数据不一致。某一时刻，在主节点和从节点中 A 数据的值都为 X， 之后将主节点中 A 的值修改为 Y，那么在这个变更通知到从节点之前，应用读取从节点中的 A 数据的值并不为最新的 Y，由此便产生了数据不一致的问题。
>
> *   (2)**延时问题**。类似 Redis 这种组件，数据从写入主节点到同步至从节点中的过程需要经 历网络→主节点内存→网络→从节点内存这几个阶段，整个过程会耗费一定的时间。而在 Kafka 中，主从同步会比 Redis 更加耗时，它需要经历网络→主节点内存→主节点磁盘→网络→从节 点内存→从节点磁盘这几个阶段。对延时敏感的应用而言，主写从读的功能并不太适用。
>

**17.Kafka中是怎么体现消息顺序性的？**

> kafka每个partition中的消息在写入时都是有序的，消费时，每个partition只能被每一个group中的一个消费者消费，保证了消费时也是有序的。
> 整个topic不保证有序。如果为了保证topic整个有序，那么将partition调整为1.

**18.消费者提交消费位移时提交的是当前消费到的最新消息的offset还是offset+1?**

> offset+1

**19.kafka如何实现延迟队列？**

> Kafka并没有使用JDK自带的Timer或者DelayQueue来实现延迟的功能，而是**基于时间轮自定义了一个用于实现延迟功能的定时器（SystemTimer）**。JDK的Timer和DelayQueue插入和删除操作的平均时间复杂度为O(nlog(n))，并不能满足Kafka的高性能要求，而基于时间轮可以将插入和删除操作的时间复杂度都降为**O(1)**。时间轮的应用并非Kafka独有，其应用场景还有很多，在Netty、Akka、Quartz、Zookeeper等组件中都存在时间轮的踪影。
>
> 底层使用数组实现，数组中的每个元素可以存放一个TimerTaskList对象。TimerTaskList是一个环形双向链表，在其中的链表项TimerTaskEntry中封装了真正的定时任务TimerTask.
>
> Kafka中到底是怎么推进时间的呢？Kafka中的定时器借助了JDK中的DelayQueue来协助推进时间轮。具体做法是对于每个使用到的TimerTaskList都会加入到DelayQueue中。**Kafka中的TimingWheel专门用来执行插入和删除TimerTaskEntry的操作，而DelayQueue专门负责时间推进的任务**。再试想一下，DelayQueue中的第一个超时任务列表的expiration为200ms，第二个超时任务为840ms，这里获取DelayQueue的队头只需要O(1)的时间复杂度。如果采用每秒定时推进，那么获取到第一个超时的任务列表时执行的200次推进中有199次属于“空推进”，而获取到第二个超时任务时有需要执行639次“空推进”，这样会无故空耗机器的性能资源，这里采用DelayQueue来辅助以少量空间换时间，从而做到了“精准推进”。Kafka中的定时器真可谓是“知人善用”，用TimingWheel做最擅长的任务添加和删除操作，而用DelayQueue做最擅长的时间推进工作，相辅相成。
>


**20.Kafka中的事务是怎么实现的？**

在说Kafka的事务之前，先要说一下Kafka中幂等的实现。幂等和事务是Kafka 0.11.0.0版本引入的两个特性，以此来实现EOS（exactly once semantics，精确一次处理语义）。

幂等，简单地说就是对接口的多次调用所产生的结果和调用一次是一致的。生产者在进行重试的时候有可能会重复写入消息，而使用Kafka的幂等性功能之后就可以避免这种情况。

开启幂等性功能的方式很简单，只需要显式地将生产者客户端参数enable.idempotence设置为true即可（这个参数的默认值为false）。

Kafka是如何具体实现幂等的呢？Kafka为此引入了producer id（以下简称PID）和序列号（sequence number）这两个概念。每个新的生产者实例在初始化的时候都会被分配一个PID，这个PID对用户而言是完全透明的。

对于每个PID，消息发送到的每一个分区都有对应的序列号，这些序列号从0开始单调递增。生产者每发送一条消息就会将对应的序列号的值加1。

broker端会在内存中为每一对维护一个序列号。对于收到的每一条消息，只有当它的序列号的值（SN\_new）比broker端中维护的对应的序列号的值（SN\_old）大1（即SN\_new = SN\_old + 1）时，broker才会接收它。

如果SN\_new< SN\_old + 1，那么说明消息被重复写入，broker可以直接将其丢弃。如果SN\_new> SN\_old + 1，那么说明中间有数据尚未写入，出现了乱序，暗示可能有消息丢失，这个异常是一个严重的异常。

引入序列号来实现幂等也只是针对每一对而言的，也就是说，Kafka的幂等只能保证单个生产者会话（session）中单分区的幂等。幂等性不能跨多个分区运作，而事务可以弥补这个缺陷。

事务可以保证对多个分区写入操作的原子性。操作的原子性是指多个操作要么全部成功，要么全部失败，不存在部分成功、部分失败的可能。

为了使用事务，应用程序必须提供唯一的transactionalId，这个transactionalId通过客户端参数transactional.id来显式设置。事务要求生产者开启幂等特性，因此通过将transactional.id参数设置为非空从而开启事务特性的同时需要将enable.idempotence设置为true（如果未显式设置，则KafkaProducer默认会将它的值设置为true），如果用户显式地将enable.idempotence设置为false，则会报出ConfigException的异常。

transactionalId与PID一一对应，两者之间所不同的是transactionalId由用户显式设置，而PID是由Kafka内部分配的。

另外，为了保证新的生产者启动后具有相同transactionalId的旧生产者能够立即失效，每个生产者通过transactionalId获取PID的同时，还会获取一个单调递增的producer epoch。如果使用同一个transactionalId开启两个生产者，那么前一个开启的生产者会报错。

从生产者的角度分析，通过事务，Kafka可以保证跨生产者会话的消息幂等发送，以及跨生产者会话的事务恢复。

前者表示具有相同transactionalId的新生产者实例被创建且工作的时候，旧的且拥有相同transactionalId的生产者实例将不再工作。

后者指当某个生产者实例宕机后，新的生产者实例可以保证任何未完成的旧事务要么被提交（Commit），要么被中止（Abort），如此可以使新的生产者实例从一个正常的状态开始工作。

KafkaProducer提供了5个与事务相关的方法，详细如下：

```
void initTransactions();
void beginTransaction() throws ProducerFencedException;
void sendOffsetsToTransaction(Map<TopicPartition, OffsetAndMetadata> offsets,
                              String consumerGroupId)
        throws ProducerFencedException;
void commitTransaction() throws ProducerFencedException;
void abortTransaction() throws ProducerFencedException;

```

initTransactions()方法用来初始化事务；beginTransaction()方法用来开启事务；sendOffsetsToTransaction()方法为消费者提供在事务内的位移提交的操作；commitTransaction()方法用来提交事务；abortTransaction()方法用来中止事务，类似于事务回滚。

在消费端有一个参数isolation.level，与事务有着莫大的关联，这个参数的默认值为“read\_uncommitted”，意思是说消费端应用可以看到（消费到）未提交的事务，当然对于已提交的事务也是可见的。

这个参数还可以设置为“read\_committed”，表示消费端应用不可以看到尚未提交的事务内的消息。

举个例子，如果生产者开启事务并向某个分区值发送3条消息msg1、msg2和msg3，在执行commitTransaction()或abortTransaction()方法前，设置为“read\_committed”的消费端应用是消费不到这些消息的，不过在KafkaConsumer内部会缓存这些消息，直到生产者执行commitTransaction()方法之后它才能将这些消息推送给消费端应用。反之，如果生产者执行了abortTransaction()方法，那么KafkaConsumer会将这些缓存的消息丢弃而不推送给消费端应用。

![在这里插入图片描述](https://img-blog.csdnimg.cn/201904090954550.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9oaWRkZW5wcHMuYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

日志文件中除了普通的消息，还有一种消息专门用来标志一个事务的结束，它就是控制消息（ControlBatch）。控制消息一共有两种类型：COMMIT和ABORT，分别用来表征事务已经成功提交或已经被成功中止。

RecordBatch中attributes字段的第6位用来标识当前消息是否是控制消息。如果是控制消息，那么这一位会置为1，否则会置为0，如上图所示。

attributes字段中的第5位用来标识当前消息是否处于事务中，如果是事务中的消息，那么这一位置为1，否则置为0。由于控制消息也处于事务中，所以attributes字段的第5位和第6位都被置为1。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190409095515133.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9oaWRkZW5wcHMuYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)
KafkaConsumer可以通过这个控制消息来判断对应的事务是被提交了还是被中止了，然后结合参数isolation.level配置的隔离级别来决定是否将相应的消息返回给消费端应用，如上图所示。注意ControlBatch对消费端应用不可见。

我们在上一篇Kafka科普系列中还讲过LSO——《[Kafka科普系列 | 什么是LSO](https://blog.csdn.net/u013256816/article/details/88985769)》，它与Kafka的事务有着密切的联系，看着下图，你回忆起来了嘛。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190409095541760.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9oaWRkZW5wcHMuYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

---

**21**.**Kafka中有那些地方需要选举？这些地方的选举策略又有哪些？**

kafka leader选举机制原理

kafka在所有broker中选出一个controller，所有Partition的Leader选举都由controller决定。controller会将Leader的改变直接通过RPC的方式（比Zookeeper Queue的方式更高效）通知需为此作出响应的Broker。同时controller也负责增删Topic以及Replica的重新分配。
当有broker fari over controller的处理过程如下：

1.Controller在Zookeeper注册Watch，一旦有Broker宕机（这是用宕机代表任何让系统认为其die的情景，包括但不限于机器断电，网络不可用，GC导致的Stop The World，进程crash等），其在Zookeeper对应的znode会自动被删除，Zookeeper会fire Controller注册的watch，Controller读取最新的幸存的Broker

2.Controller决定set_p，该集合包含了宕机的所有Broker上的所有Partition

3.对set_p中的每一个Partition

3.1 从/brokers/topics/[topic]/partitions/[partition]/state读取该Partition当前的ISR

3.2 决定该Partition的新Leader。如果当前ISR中有至少一个Replica还幸存，则选择其中一个作为新Leader，新的ISR则包含当前ISR中所有幸存的Replica（选举算法的实现类似于微软的PacificA）。否则选择该Partition中任意一个幸存的Replica作为新的Leader以及ISR（该场景下可能会有潜在的数据丢失）。如果该Partition的所有Replica都宕机了，则将新的Leader设置为-1。

3.3 将新的Leader，ISR和新的leader_epoch及controller_epoch写入/brokers/topics/[topic]/partitions/[partition]/state。注意，该操作只有其version在3.1至3.3的过程中无变化时才会执行，否则跳转到3.1

4. 直接通过RPC向set_p相关的Broker发送LeaderAndISRRequest命令。Controller可以在一个RPC操作中发送多个命令从而提高效率。



LeaderAndIsrRequest响应过程

1.若请求中controllerEpoch小于当前最新的controllerEpoch，则直接返回ErrorMapping.StaleControllerEpochCode。2.对于请求中partitionStateInfos中的每一个元素，即（(topic, partitionId), partitionStateInfo)：

2.1 若partitionStateInfo中的leader epoch大于当前ReplicManager中存储的(topic, partitionId)对应的partition的leader epoch，则：

2.1.1 若当前brokerid（或者说replica id）在partitionStateInfo中，则将该partition及partitionStateInfo存入一个名为partitionState的HashMap中

2.1.2否则说明该Broker不在该Partition分配的Replica list中，将该信息记录于log中2.2否则将相应的Error code（ErrorMapping.StaleLeaderEpochCode）存入Response中

3.筛选出partitionState中Leader与当前Broker ID相等的所有记录存入partitionsTobeLeader中，其它记录存入partitionsToBeFollower中。

4.若partitionsTobeLeader不为空，则对其执行makeLeaders方。

5.若partitionsToBeFollower不为空，则对其执行makeFollowers方法

6.若highwatermak线程还未启动，则将其启动，并将hwThreadInitialized设为true。

7.关闭所有Idle状态的Fetcher。

LeaderAndIsrRequest处理过程如下图所示

　　对于收到的LeaderAndIsrRequest，Broker主要通过ReplicaManager的becomeLeaderOrFollower处理，流程如下：




如何处理所有Replica都不工作

　　上文提到，在ISR中至少有一个follower时，Kafka可以确保已经commit的数据不丢失，但如果某个Partition的所有Replica都宕机了，就无法保证数据不丢失了。这种情况下有两种可行的方案：

1.等待ISR中的任一个Replica“活”过来，并且选它作为Leader

2.选择第一个“活”过来的Replica（不一定是ISR中的）作为Leader

　　这就需要在可用性和一致性当中作出一个简单的折衷。如果一定要等待ISR中的Replica“活”过来，那不可用的时间就可能会相对较长。而且如果ISR中的所有Replica都无法“活”过来了，或者数据都丢失了，这个Partition将永远不可用。选择第一个“活”过来的Replica作为Leader，而这个Replica不是ISR中的Replica，那即使它并不保证已经包含了所有已commit的消息，它也会成为Leader而作为consumer的数据源（前文有说明，所有读写都由Leader完成）。Kafka0.8.*使用了第二种方式。根据Kafka的文档，在以后的版本中，Kafka支持用户通过配置选择这两种方式中的一种，从而根据不同的使用场景选择高可用性还是强一致性。 unclean.leader.election.enable 参数决定使用哪种方案，默认是true，采用第二种方案　
