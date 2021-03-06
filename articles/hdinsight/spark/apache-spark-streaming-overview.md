---
title: Azure HDInsight 中的 Spark 流式处理
description: 如何在 HDInsight Spark 群集上使用 Apache Spark 流式处理应用程序。
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive,seoapr2020
ms.date: 04/23/2020
ms.openlocfilehash: 0ccb87017f962650f099d506e1d200ab408316d6
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2020
ms.locfileid: "82195139"
---
# <a name="overview-of-apache-spark-streaming"></a>Apache Spark 流式处理概述

[Apache Spark](https://spark.apache.org/)流式处理在 HDInsight Spark 群集上提供数据流处理。 使用可保证任何输入事件只处理一次，即使发生节点故障也是如此。 Spark 流是长时间运行的作业，可接收来自各种来源（包括 Azure 事件中心）的输入数据。 另请参阅 Azure IoT 中心、Apache Kafka、Apache Flume、Twitter `ZeroMQ`、、原始 TCP 套接字或监视 Apache Hadoop YARN 文件系统。 与仅事件驱动的过程不同，Spark 流将数据成批地输入到时间窗口中。 例如2秒的切片，然后使用映射、减少、联接和提取操作来转换每批数据。 然后，Spark 流将转换后的数据写入文件系统、数据库、仪表板和控制台。

![使用 HDInsight 和 Spark 流式处理的流处理](./media/apache-spark-streaming-overview/hdinsight-spark-streaming.png)

Spark 流式处理应用程序必须等待一小部分的时间来`micro-batch`收集每个事件，然后再发送该批处理进行处理。 与此相反，事件驱动应用程序会立即处理每个事件。 Spark 流式处理延迟一般为几秒钟。 微批处理方法的优点是数据处理效率更高和聚合计算更简单。

## <a name="introducing-the-dstream"></a>引入 DStream

Spark 流式处理使用称为 DStream 的离散流表示传入数据的连续流**。 可以通过输入源（如事件中心或 Kafka）创建 DStream。 或通过对其他 DStream 应用转换。

DStream 可提供基于原始事件数据的抽象层。

从单一事件开始，例如已连接调温器的温度读数。 当此事件到达 Spark 流式处理应用程序时，将以可靠的方式存储事件，在该事件中，会将其复制到多个节点上。 这种容错可确保任何单个节点的故障不会导致事件丢失。 Spark 核心使用在群集中跨多个节点分发数据的数据结构。 其中每个节点通常在内存中维护自己的数据，以获得最佳性能。 此数据结构称为弹性分布式数据集 (RDD)**。

每个 RDD 表示在用户定义的时间范围（称为*批处理间隔*）内收集的事件。 每个批处理间隔后，将生成新的 RDD，其中包含该间隔的所有数据。 将连续的 Rdd 集收集到 DStream 中。 例如，如果批处理间隔为 1 秒，则 DStream 将每秒发出一个批处理，其中包含一个 RDD（包含该秒期间引入的所有数据）。 处理 DStream 时，温度事件将出现在其中一个批处理中。 Spark 流式处理应用程序处理包含事件的批处理并最终作用于每个 RDD 中存储的数据。

![温度事件的示例 DStream](./media/apache-spark-streaming-overview/hdinsight-spark-streaming-example.png)

## <a name="structure-of-a-spark-streaming-application"></a>Spark 流式处理应用程序的结构

Spark 流式处理应用程序是一个长时间运行的应用程序，可从引入源接收数据。 应用转换来处理数据，然后将数据推送到一个或多个目标。 Spark 流式处理应用程序的结构包含静态部分和动态部分。 静态部分定义数据的来源、对数据进行的处理。 结果应出现的位置。 动态部分无限期运行应用程序，等待停止信号。

例如，下面的简单应用程序通过 TCP 套接字接收一行文本，并对每个单词出现的次数进行计数。

### <a name="define-the-application"></a>定义应用程序

应用程序逻辑定义有四个步骤：

1. 创建 StreamingContext。
2. 从 StreamingContext 创建 DStream。
3. 将转换应用于 DStream。
4. 输出结果。

此定义是静态的，且在运行该应用程序前没有处理任何数据。

#### <a name="create-a-streamingcontext"></a>创建 StreamingContext

从指向群集的 SparkContext 创建 StreamingContext。 创建 StreamingContext 时，指定批处理的大小（以秒为单位），例如：  

```
import org.apache.spark._
import org.apache.spark.streaming._

val ssc = new StreamingContext(sc, Seconds(1))
```

#### <a name="create-a-dstream"></a>创建 DStream

使用 StreamingContext 实例，为输入源创建输入 DStream。 在这种情况下，应用程序会监视默认附加存储中的新文件的外观。

```
val lines = ssc.textFileStream("/uploads/Test/")
```

#### <a name="apply-transformations"></a>应用转换

通过对 DStream 应用转换来实现处理。 此应用程序每次从文件接收一行文本，并将每行拆分为多个单词。 然后使用地图缩减模式来计算每个单词出现的次数。

```
val words = lines.flatMap(_.split(" "))
val pairs = words.map(word => (word, 1))
val wordCounts = pairs.reduceByKey(_ + _)
```

#### <a name="output-results"></a>输出结果

通过应用输出操作，将转换结果推送到目标系统。 在本例中，整个计算中的每个运行的结果都将打印到控制台输出中。

```
wordCounts.print()
```

### <a name="run-the-application"></a>运行应用程序

启动流式处理应用程序并运行，直到收到终止信号。

```
ssc.start()
ssc.awaitTermination()
```

有关 Spark 流 API 的详细信息，请参阅[Apache Spark 流式处理编程指南](https://people.apache.org/~pwendell/spark-releases/latest/streaming-programming-guide.html)。

下面的示例应用程序是自包含的，因此可以在 [Jupyter Notebook](apache-spark-jupyter-notebook-kernels.md) 内运行它。 此示例将在类 DummySource 中创建模拟数据源，该源每五秒输出计数器的值以及当前时间（以毫秒为单位）。 新 StreamingContext 对象的批处理间隔为 30 秒。 每次创建批时，流式处理应用程序都将检查生成的 RDD。 然后，将 RDD 转换为 Spark 数据帧，并通过数据帧创建一个临时表。

```
class DummySource extends org.apache.spark.streaming.receiver.Receiver[(Int, Long)](org.apache.spark.storage.StorageLevel.MEMORY_AND_DISK_2) {

    /** Start the thread that simulates receiving data */
    def onStart() {
        new Thread("Dummy Source") { override def run() { receive() } }.start()
    }

    def onStop() {  }

    /** Periodically generate a random number from 0 to 9, and the timestamp */
    private def receive() {
        var counter = 0  
        while(!isStopped()) {
            store(Iterator((counter, System.currentTimeMillis)))
            counter += 1
            Thread.sleep(5000)
        }
    }
}

// A batch is created every 30 seconds
val ssc = new org.apache.spark.streaming.StreamingContext(spark.sparkContext, org.apache.spark.streaming.Seconds(30))

// Set the active SQLContext so that we can access it statically within the foreachRDD
org.apache.spark.sql.SQLContext.setActive(spark.sqlContext)

// Create the stream
val stream = ssc.receiverStream(new DummySource())

// Process RDDs in the batch
stream.foreachRDD { rdd =>

    // Access the SQLContext and create a table called demo_numbers we can query
    val _sqlContext = org.apache.spark.sql.SQLContext.getOrCreate(rdd.sparkContext)
    _sqlContext.createDataFrame(rdd).toDF("value", "time")
        .registerTempTable("demo_numbers")
}

// Start the stream processing
ssc.start()
```

在启动上面的应用程序以后，等待约 30 秒钟。  然后，你可以定期查询 DataFrame 来查看批处理中存在的当前值集，例如，使用以下 SQL 查询：

```sql
%%sql
SELECT * FROM demo_numbers
```

生成的输出类似于以下输出：

| value | time |
| --- | --- |
|10 | 1497314465256 |
|11 | 1497314470272 |
|12 | 1497314475289 |
|13 | 1497314480310 |
|14 | 1497314485327 |
|15 | 1497314490346 |

存在六个值，因为 DummySource 每隔 5 秒创建一个值，且应用程序每隔 30 秒发出一个批处理。

## <a name="sliding-windows"></a>滑动窗口

若要在一段时间内对 DStream 执行聚合计算，例如，获取过去两秒钟内的平均温度，请使用 Spark `sliding window`流式处理中包含的操作。 滑动窗口具有一个持续时间（窗口长度）和在期间计算窗口内容的时间间隔（即滑动间隔）。

滑动窗口可以重叠，例如，可以定义长度为两秒的窗口，每一秒滑动一次。 此操作意味着每次执行聚合计算时，窗口将包括上一个窗口最后一秒的数据。 和下一秒的任何新数据。

![温度事件的示例初始窗口](./media/apache-spark-streaming-overview/hdinsight-spark-streaming-window-01.png)

![滑动后的温度事件的示例窗口](./media/apache-spark-streaming-overview/hdinsight-spark-streaming-window-02.png)

下面的示例将更新使用 DummySource 的代码，将批处理收集到持续时间为一分钟且滑动为一分钟的窗口中。

```
class DummySource extends org.apache.spark.streaming.receiver.Receiver[(Int, Long)](org.apache.spark.storage.StorageLevel.MEMORY_AND_DISK_2) {

    /** Start the thread that simulates receiving data */
    def onStart() {
        new Thread("Dummy Source") { override def run() { receive() } }.start()
    }

    def onStop() {  }

    /** Periodically generate a random number from 0 to 9, and the timestamp */
    private def receive() {
        var counter = 0  
        while(!isStopped()) {
            store(Iterator((counter, System.currentTimeMillis)))
            counter += 1
            Thread.sleep(5000)
        }
    }
}

// A batch is created every 30 seconds
val ssc = new org.apache.spark.streaming.StreamingContext(spark.sparkContext, org.apache.spark.streaming.Seconds(30))

// Set the active SQLContext so that we can access it statically within the foreachRDD
org.apache.spark.sql.SQLContext.setActive(spark.sqlContext)

// Create the stream
val stream = ssc.receiverStream(new DummySource())

// Process batches in 1 minute windows
stream.window(org.apache.spark.streaming.Minutes(1)).foreachRDD { rdd =>

    // Access the SQLContext and create a table called demo_numbers we can query
    val _sqlContext = org.apache.spark.sql.SQLContext.getOrCreate(rdd.sparkContext)
    _sqlContext.createDataFrame(rdd).toDF("value", "time")
    .registerTempTable("demo_numbers")
}

// Start the stream processing
ssc.start()
```

第一分钟后，会产生 12 个条目 - 窗口中收集到的两个批处理中各有 6 个条目。

| value | time |
| --- | --- |
| 1 | 1497316294139 |
| 2 | 1497316299158
| 3 | 1497316304178
| 4 | 1497316309204
| 5 | 1497316314224
| 6 | 1497316319243
| 7 | 1497316324260
| 8 | 1497316329278
| 9 | 1497316334293
| 10 | 1497316339314
| 11 | 1497316344339
| 12 | 1497316349361

Spark 流式传输 API 中可用的滑动窗口函数包括 window、countByWindow、reduceByWindow 和 countByValueAndWindow。 有关这些函数的详细信息，请参阅 [Transformations on DStreams](https://people.apache.org/~pwendell/spark-releases/latest/streaming-programming-guide.html#transformations-on-dstreams)（DStreams 的转换）。

## <a name="checkpointing"></a>检查点

为提供复原和容错功能，Spark 流式处理依赖检查点，确保即使出现节点故障，流处理仍可以不间断地继续。 Spark 创建持久存储（Azure 存储或 Data Lake Storage）的检查点。 这些检查点存储流式处理应用程序元数据（如配置）和应用程序定义的操作。 此外，任何已排队但尚未处理的批处理。 有时，检查点还包括将数据保存在 Rdd 中，以便更快地从 Spark 管理的 Rdd 中存在的内容重新生成数据状态。

## <a name="deploying-spark-streaming-applications"></a>部署 Spark 流式处理应用程序

通常将 Spark 流式处理应用程序以本地方式生成到 JAR 文件中。 然后，将 JAR 文件复制到默认连接的存储，将其部署到 HDInsight 上的 Spark。 可以使用 POST 操作，通过群集上可用的 LIVY REST API 启动该应用程序。 POST 的正文包括提供 JAR 路径的 JSON 文档。 和类的名称，其 main 方法定义并运行流式处理应用程序，还可以选择作业的资源要求（如执行器、内存和内核的数目）。 此外，应用程序代码所需的任何配置设置。

![部署 Spark 流式处理应用程序](./media/apache-spark-streaming-overview/hdinsight-spark-streaming-livy.png)

此外，可以使用 GET 请求针对 LIVY 终结点检查所有应用程序的状态。 最后，可以通过针对 LIVY 终结点发出 DELETE 请求来结束正在运行的应用程序。 有关 LIVY API 的详细信息，请参阅[使用 Apache LIVY 执行远程作业](apache-spark-livy-rest-interface.md)

## <a name="next-steps"></a>后续步骤

* [在 HDInsight 中创建 Apache Spark 群集](../hdinsight-hadoop-create-linux-clusters-portal.md)
* [Apache Spark Streaming Programming Guide](https://people.apache.org/~pwendell/spark-releases/latest/streaming-programming-guide.html)（Apache Spark 流式处理编程指南）
* [Apache Spark 结构化流的概述](apache-spark-structured-streaming-overview.md)