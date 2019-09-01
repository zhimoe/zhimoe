---
title: "Spark Basic"
date: "2018-03-31T00:17:30+08:00"
toc: true
categories:
 - "编程"
tags:
 - code
 - spark
---

## 引言
大数据计算和普通的程序并无本质区别：数据输入=>计算=>输出和结果的持久化。这里的挑战在于计算的效率和容错性。由于数据输入巨大，计算的效率是基本的要求。为了在通用硬件上高效完成大量计算，唯一的途径就是将计算任务拆分分布式计算。这就引出了新的问题：分布式计算资源的管理（Mesos，YARN），分布式计算失败后的恢复（容错性）（Spark RDD），以及分布式的数据输入和保存（分布式文件HDFS）。hadoop生态圈就是为了解决几个问题设计的(YARN,MapR,HDFS)。只不过在计算这一环节Spark做的更加高效取代了MapR。所以先看下hadoop的核心两个组件。

##  HDFS
* HDFS是hadoop的虚拟分布式文件系统。满足大数据问题下要求的：可扩展的，容错的，硬件通用的和高并发的特性。HDFS最重要的特性是不可变性--数据提交到HDFS后即不可更新了，也就是所谓的WORM(write once read many)。
* 文件在HDFS中是以block构成，默认一个block是128M。block是是分布式的，即如果集群中如果有多于1个节点，那么有文件可能会被分布在多个节点上。block是被复制的，这主要是两个目的：1.容错，2.增加数据局部性的概率，有利于访问。block复制在数据节点接收（ingest：消化）block时同时发生。如图所示：

![File ingestion into a multinode cluster](/spark/File-ingestion-into-a-multinode-cluster.png)

* NameNode：不知道怎么翻译，NameNode主要负责管理HDFS的元数据，包括directory,文件对象和相关属性（e.g. ACL)，元数据是常驻内存中的，硬盘上也有备份以及日志保证持久性和崩溃后的一致性（和数据库相似）。还包括block的位置信息--block之间的关系。注意，数据（文件）并不经过NameNode，否则很容易成为性能瓶颈，数据是直接到达DataNode，并上报给NameNode管理。
* 数据节点（DataNode）负责：block复制；管理本节点的存储；向NameNode上报block信息。注意，数据节点不会意识到HDFS的目录（directory）和文件（Files）的概念，这些信息是NameNode管理保存的，客户端只会和NameNode交道。
* hdfs客户端分为：fs shell;hdfs java api;rest proxy接口（HttpFS等）。
* 常见命令：

```bash
# 上传一个文件 -f表示覆盖
hadoop fs -put -f jour.txt /user/dahu/jour/
# 下载
hadoop fs -get /user/dahu/jour/jour.txt
# ls
hadoop fs -ls /user/dahu/
# 删除 -r表示递归，删除目录
hadoop fs -rm /user/dahu/jour/jour.txt
hadoop fs -rm -r /user/dahu/jour
```
## YARN

* YARN:Yet Another Resource Negotiator是hadoop的资源管理器。YARN有个守护进程--ResourceManager,负责全局的资源管理和任务调度，把整个集群当作计算资源池，只关注分配，不管应用，且不负责容错。YARN将application（或者叫job）分发给各个NodeManager,NodeManager是实际的worker或者worker的代理。ResourceManager主要有两个组件：Scheduler 和 ApplicationsManager。 下图是YARN的结构示意图：

![yarn_architecture](/spark/yarn_architecture.gif)

*  上图中ResourceManager负责管理和分配全局的计算资源。而NodeManager看着更复杂一些：1.用户提交一个app给RM（ResourceManager）；2.RM在资源充足的NodeManager上启动一个ApplicationMaster（也就是这个app对应的第一个container）。3.ApplicationMaster负责在所有NodeManagers中协调创建几个task container，也包括ApplicationMaster自己所在的NodeManager（上图中紫色2个和红色的4个分别表示2个app的task container和ApplicationMaster）。4. NodeManager向各个ApplicationMaster汇报task container的进展和状态。5. ApplicationMaster向RM汇报应用的进展和状态。6.RM向用户返回app的进度，状态，结果。用户一般可通过Web UI查看这些。

* 上面的示意图是YARN 的核心概念，Spark程序的运行结构示意图和上面的示意图相同。每个组件都可以近似一样的理解，例如，上面的Client在Spark中叫Driver程序;ResourceManager在Spark中叫Cluster Manager（为了理解方便，认为一样即可，Spark的ClusterManager目前主要有YARN,Mesos和Spark自带的三种）；NodeManager就是Spark中的Worker Node。


## Spark基本概念

* 上图中的client程序在Spark中即Driver程序。Driver就是我们编写Spark程序app的主要部分，包括`SparkContext`的创建和关闭以及计算任务（Task）的计划（Planning,包括数据数据，转换，输出，持久化等)。`SparkContext`负责和Cluster Manager通信，进行资源申请，任务的分配和监控。一般认为`SparkContext`代表Driver。
* ClusterManager：就是上面说的三种-Standalone,YARN,Mesos。
* ＷorkerNode: 集群中运行app代码的节点，也就是上图中YARN的NodeManager节点。一个节点运行一个/多个executor.
* Executor：app运行在worker节点的一个进程，进程负责执行task的planning。Spark On YARN 中这个进程叫CoarseGrainedExecutorBackend。每个进程能并行执行的task数量取决于分配给它的CPU个数了。下图是一个Spark程序集群概览图，和上图很相似。

![cluster-overview](/spark/cluster-overview.png)

* 仔细对比上面两个示意图，在YARN的结构示意图中,ResourceManager为程序在某个NodeManager上创建的第一个container叫ApplicationMaster，ApplicationMaster负责只是其他的task container。在Spark On YARN有两种运行模式：client和cluster模式。在cluster模式下，用户编写的driver程序运行在YARN的ApplicationMaster的内部。
*RDD:Spark的核心数据结构。后面详细介绍，可以简单的理解为一个Spark程序所有需要处理的数据在Spark中被抽象成一个RDD，数据需要被拆分分发到各个worker去计算，所以RDD有一个分区（Partation）概念。一般我们的数据是放在分布式文件系统上的(e.g. HDFS)，可以简单理解为一个RDD包含一或多个Partation，每个Partation对应的就是HDFS的一个block。当然，Partation不是和HDFS的block绑定的，你也可以手动的对数据进行分区，即使他们只是待处理的一个本地文件或者一个小数组。 一个Partation包含一到多个Record，Record可以理解为文本中的一行，excel的一条记录或者是kafka的一条消息。
* Task：RDD的一个Patation对应一个Task,Task是单个分区上最小的处理单元。

## RDD
pass
## SparkStreaming
pass
## SparkStreaming+Kafka

```scala
import org.apache.kafka.clients.consumer.ConsumerRecord
import org.apache.kafka.common.serialization.StringDeserializer
import org.apache.spark.streaming.kafka010._
import org.apache.spark.streaming.kafka010.LocationStrategies.PreferConsistent
import org.apache.spark.streaming.kafka010.ConsumerStrategies.Subscribe

val kafkaParams = Map[String, Object](
  "bootstrap.servers" -> "localhost:9092,anotherhost:9092",
  "key.deserializer" -> classOf[StringDeserializer],
  "value.deserializer" -> classOf[StringDeserializer],
  "group.id" -> "use_a_separate_group_id_for_each_stream",
  "auto.offset.reset" -> "latest",
  "enable.auto.commit" -> (false: java.lang.Boolean)
)

val topics = Array("topicA", "topicB")
val stream = KafkaUtils.createDirectStream[String, String](
  streamingContext,
  PreferConsistent,
  Subscribe[String, String](topics, kafkaParams)
)

stream.map(record => (record.key, record.value))
```

DStream的elements:record is ConsumerRecord<K,V>: A key/value pair to be received from Kafka. This consists of a topic name and a partition number, from which the record is being received and an offset that points to the record in a Kafka partition.包含key(),offset(),partation()方法等。

* 当一个StreamingContext中有多个input stream时，记得保证给程序分配了足够的资源（特别是core的数量，必须大于输入源的数量）。
* 本地执行程序时，不要使用“local” or “local[1]” as the master URL，streaming程序至少需要两个thread，一个接受数据，一个处理数据。直接使用local[n],n>输入源个数。
* DStream 和RDD一样支持各种trans和action
* DStream is batches of RDDs.

## 常见错误

```scala
dstream.foreachRDD { rdd =>
  val connection = createNewConnection()  // executed at the driver
  rdd.foreach { record =>
    connection.send(record) // executed at the worker
  }
}
// 上面的写法会导致connection 不可序列化的错误。因为connection需要被发送到worker上，所以必须可以序列化；
// 但是这样的连接对象其实非常少的（第三方库一般都不支持）；
dstream.foreachRDD { rdd =>
  rdd.foreach { record =>
    val connection = createNewConnection()
    connection.send(record)
    connection.close()
  }
}
// 上面是给每个record处理时新建一个连接，会导致严重的性能问题。
// 更好的方式是给每个partation新建一个连接

dstream.foreachRDD { rdd =>
  rdd.foreachPartition { partitionOfRecords =>
    val connection = createNewConnection()
    partitionOfRecords.foreach(record => connection.send(record))
    connection.close()
  }
}
// 最好的方法是维护一个静态线程池：
dstream.foreachRDD { rdd =>
  rdd.foreachPartition { partitionOfRecords =>
    // ConnectionPool is a static, lazily initialized pool of connections
    val connection = ConnectionPool.getConnection()
    partitionOfRecords.foreach(record => connection.send(record))
    ConnectionPool.returnConnection(connection)  // return to the pool for future reuse
  }
}
Note that the connections in the pool should be lazily created on demand and timed out if not used for a while. This achieves the most efficient sending of data to external systems.
```

* DStream的RDD分区数是由topic分区数相同的。

## 最佳实践

