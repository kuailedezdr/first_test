# 第一周

1、spark的任务提交流程：

yarn-client模式提交流程

```
1、客户端提交任务,并且在当前客户端启动driver进程,构建sparkcontext(Dagscheduler和taskscheduler)

2、driver发送请求到resourcemanager

3、resourcemanager（rs）收到客户端的发来的请求，在合适的nodemanager上启动applicationmaster

4、appmaster启动后向rm请求资源，用于启动executor

5、rm收到appmaster的资源申请后会分配container,appmaster在资源分配指定的nodemanager上启动executor

6、executor启动后向driver反向注册.driver向executor发送task，并且监控执行情况和回收结果
```

yarn-cluster模式
```
1、客户端提交任务，向ResourceManager通讯申请启动ApplicationMaster,随后ResourceManager分配container，在合适的 NodeManager上启动ApplicationMaster,此时的appmaster就是driver

2、driver向ResourceManager申请Executor 内存

3、resourcemanager（rs）收到客户端的发来的请求，在合适的nodemanager上启动applicationmaster

4、appmaster启动后向rm请求资源，用于启动executor

5、rm收到appmaster的资源申请后会分配container,appmaster在资源分配指定的nodemanager上启动executor

6、executor启动后向driver反向注册.driver向executor发送task，并且监控执行情况和回收结果
```




2. 写出下列代码的打印结果。（5分）

def joinRdd(sc:SparkContext) {

​	val name= Array((1,"spark"),(2,"flink"),(3,"hadoop"),(4,”java”))

​	val score= Array((1,100),(2,90),(3,80),(5,90))

​	val namerdd=sc.parallelize(name);

​	val scorerdd=sc.parallelize(score);

​	val result = namerdd.join(scorerdd);

​	result.collect.foreach(println);

}

答案：



```
(2,(flink,90))
(1,(spark,100))
(3,(hadoop,80))
```



3.写出触发shuffle的算子（至少五个）(5分)

```scala
//去重
def distinct()
def distinct(numPartitions: Int)

//聚合
def reduceByKey(func: (V, V) => V, numPartitions: Int): RDD[(K, V)]
def reduceByKey(partitioner: Partitioner, func: (V, V) => V): RDD[(K, V)]
def groupBy[K](f: T => K, p: Partitioner):RDD[(K, Iterable[V])]
def groupByKey(partitioner: Partitioner):RDD[(K, Iterable[V])]
def aggregateByKey[U: ClassTag](zeroValue: U, partitioner: Partitioner): RDD[(K, U)]
def aggregateByKey[U: ClassTag](zeroValue: U, numPartitions: Int): RDD[(K, U)]
def combineByKey[C](createCombiner: V => C, mergeValue: (C, V) => C, mergeCombiners: (C, C) => C): RDD[(K, C)]
def combineByKey[C](createCombiner: V => C, mergeValue: (C, V) => C, mergeCombiners: (C, C) => C, numPartitions: Int): RDD[(K, C)]
def combineByKey[C](createCombiner: V => C, mergeValue: (C, V) => C, mergeCombiners: (C, C) =>, partitioner: Partitioner, mapSideCombine: Boolean = true, serializer: Serializer = null): RDD[(K, C)]

//排序
def sortByKey(ascending: Boolean = true, numPartitions: Int = self.partitions.length): RDD[(K, V)]
def sortBy[K](f: (T) => K, ascending: Boolean = true, numPartitions: Int = this.partitions.length)(implicit ord: Ordering[K], ctag: ClassTag[K]): RDD[T]

//重分区
def coalesce(numPartitions: Int, shuffle: Boolean = false, partitionCoalescer: Option[PartitionCoalescer] = Option.empty)
def repartition(numPartitions: Int)(implicit ord: Ordering[T] = null)

//集合或者表操作
def intersection(other: RDD[T]): RDD[T]
def intersection(other: RDD[T], partitioner: Partitioner)(implicit ord: Ordering[T] = null): RDD[T]
def intersection(other: RDD[T], numPartitions: Int): RDD[T]
def subtract(other: RDD[T], numPartitions: Int): RDD[T]
def subtract(other: RDD[T], p: Partitioner)(implicit ord: Ordering[T] = null): RDD[T]
def subtractByKey[W: ClassTag](other: RDD[(K, W)]): RDD[(K, V)]
def subtractByKey[W: ClassTag](other: RDD[(K, W)], numPartitions: Int): RDD[(K, V)]
def subtractByKey[W: ClassTag](other: RDD[(K, W)], p: Partitioner): RDD[(K, V)]
def join[W](other: RDD[(K, W)], partitioner: Partitioner): RDD[(K, (V, W))]
def join[W](other: RDD[(K, W)]): RDD[(K, (V, W))]
def join[W](other: RDD[(K, W)], numPartitions: Int): RDD[(K, (V, W))]
def leftOuterJoin[W](other: RDD[(K, W)]): RDD[(K, (V, Option[W]))]
```



4.RDD的概念和特性（10分）

```scala
//概念
RDD（Resilient Distributed Dataset）叫做弹性分布式数据集，是 Spark 中最基本的数据
处理模型。代码中是一个抽象类，它代表一个弹性的、不可变、可分区、里面的元素可并行
计算的集合。
//特性
弹性:
1.存储的弹性：内存与磁盘的自动切换；
2.容错的弹性：数据丢失可以自动恢复；
3.计算的弹性：计算出错重试机制；
4.分片的弹性：可根据需要重新分片。
分布式：数据存储在大数据集群不同节点上
数据集：RDD 封装了计算逻辑，并不保存数据
数据抽象：RDD 是一个抽象类，需要子类具体实现
不可变：RDD 封装了计算逻辑，是不可以改变的，想要改变，只能产生新的 RDD，在
新的 RDD 里面封装计算逻辑
可分区、并行计算
```



5.Spark的这些参数什么意思 ？（5分）

```
--master 指定主节点

--class 主类名，应用程序的主类

--executor-memory  每个executor的内存

--total-executor-cores 所有executor的总核数

--num-executor 启动executor的数量
```



6. Transformtion算子和Action算子的区别（举例说明）(5分)

```
Transformtion算子会生成一个新的RDD   从数据源中生成一个新的RDD或者从一个RDD中生成一个RDD

Action算子在RDD上运行计算,并返回结果给Driver或写入文件系统 

所有的transformation都是采用的懒策略，就是如果只是将transformation提交是不会执行计算的，计算只有在action被提交的时候才被触发。
```



7. 什么是持久化，为什么要用持久化？（5分）

```
1.狭义的理解: “持久化”仅仅指把域对象永久保存到数据库中；广义的理解,“持久化”包括和数据库相关的各种操作

2.如果一个有持久化数据的节点发生故 障，Spark 会在需要用到缓存的数据时重算丢失的数据分区。如果 希望节点故障的情况不会拖累我们的 执行速度，也可以把数据备份到多个节点上。
```



8. 统计下面语句中，Spark的出现次数，哪个单词出现的次数最多。（10分）

Get Spark from the [downloads page](http://spark.apache.org/downloads.html) of the project website This documentation is for,Spark,version,Spark,uses,Hadoop,s,client,libraries,for,HDFS,and,YARN.Downloads are pre packaged.for a handful of popular.Hadoop versions Users can also download a Hadoop free binary and run Spark with any Hadoop version [by augmenting Spark s classpath](http://spark.apache.org/docs/latest/hadoop-provided.html) Scala and Java users can include.Spark in their projects using its Maven coordinates and in the future Python users can also install Spark from PyPI.

```scala
import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}

object SparkCoreWordCount {
  def main(args: Array[String]): Unit = {
    val conf: SparkConf = new SparkConf().setMaster("local[*]")
      .setAppName(this.getClass.getSimpleName.stripSuffix("$"))
    val sc = new SparkContext(conf)
    sc.setLogLevel("WARN")
    val inputRDD: RDD[String] = sc.textFile("file:///C:\\Users\\ljh\\Desktop\\wordCount.txt", 2)
    val resultRDD: RDD[(String, Int)] = inputRDD.filter(line => null != line && line.trim.length > 0)
      .flatMap(_.split("\\s+"))
      .mapPartitions(iter => iter.map(_ -> 1))
      .reduceByKey(_ + _).sortBy(_._2)

    resultRDD.coalesce(1)
      .foreachPartition(
        iter=>iter.foreach(println)
      )
    sc.stop()
  }
}
```

```
(spark,7)
(hadoop,4)
(and,4)
```



9. 有如下数据

val list =

List((“jack”,100),(“jack”,80),(“tom”,88),(“jack”,56),(“tom”,66),(“tom”,78),(“tom”,99))

分组取Top2？(10分)

```scala
val list =List((“jack”,100),(“jack”,80),(“tom”,88),(“jack”,56),(“tom”,66),(“tom”,78),(“tom”,99))分组取Top2？

object top2 {
 def main(args: Array[String]): Unit = {
  val list = List(("jack",100),("jack",80),("tom",88),("jack",56),("tom",66),("tom",78),("tom",99))

   val maps = list.groupBy(_._1)
   val top = maps.map( t => {
     val key = t._1
     val values = t._2.toList.sorted.reverse.take(2)
     (key,values)
   })
    top.foreach(println)
 }
}
```

>(tom,List(99, 88))
>(jack,List(100, 80))



10. 谈谈你对广播变量的理解和它的使用场景（10分）	



```
**理解**：将driver端的变量发送给每一个任务executor，以便于executor的执行任务时使用。若有些task可能使用同一份变量（全局变量），这就会导致发送多次变量，，但是如果发送的次数多，或者list很大，会导致内存溢出，为了防止上面的情况，就使用了广播，将所有的变量一起发送给executor，task需要就从executor上的变量区去拿。但是也可能保存一些用不到的变量，浪费内存。

**使用场景**：executor执行任务时，多次shuffle操作，但是用到同一个变量。就是多次task用到了同一个变量，避免重复的接受变量保存浪费内存。

广播变量用来高效分发较大的对象。向所有工作节点发送一个 较大的只读值，以供一个或多个 Spark
操作使用。比如，如果你的应用需要向所有节点发 送一个较大的只读查询表
默认情况下，task 中的算子中如果使用了外部的变量，每个 task 都会获取一份变量的复
本，这就造成了内存的极大消耗。一方面，如果后续对 RDD 进行持久化，可能就无法将 RDD
数据存入内存，只能写入磁盘，磁盘 IO 将会严重消耗性能；另一方面，task 在创建对象的
时候，也许会发现堆内存无法存放新创建的对象，这就会导致频繁的 GC，GC 会导致工作
线程停止，进而导致 Spark 暂停工作一段时间，严重影响 Spark 性能。
假设当前任务配置了 20 个 Executor，指定 500 个 task，有一个 20M 的变量被所有 task
共用，此时会在 500 个 task 中产生 500 个副本，耗费集群 10G 的内存，如果使用了广播变
量， 那么每个 Executor 保存一个副本，一共消耗 400M 内存，内存消耗减少了 5 倍。
广播变量在每个 Executor 保存一个副本，此 Executor 的所有 task 共用此广播变量，这让变
量产生的副本数量大大减少。
在初始阶段，广播变量只在 Driver 中有一份副本。task 在运行的时候，想要使用广播变
量中的数据，此时首先会在自己本地的 Executor 对应的 BlockManager 中尝试获取变量，如
果本地没有，BlockManager 就会从 Driver 或者其他节点的 BlockManager 上远程拉取变量的
复本，并由本地的 BlockManager 进行管理；之后此 Executor 的所有 task 都会直接从本地的
BlockManager 中获取变量
```



11. 检查点如何设置？使用检查点的好处？（5分）

```
检查点chekpoint的作用就是将DAG中比较重要的数据做一个检查点,将结果存储到一个高可用的地方   rdd.checkPoint()

如果之后有节点出现问题而丢失分区，从做检查点的RDD开始重做 Lineage，就会减少开销.
```



12. 累加器的作用？

```
累加器用来对信息进行聚合，通常在向 Spark 传递函数时，比如使用 map() 函数或者用 filter() 传条件
时，可以使用驱 动器程序中定义的变量，但是集群中运行的每个任务都会得到这些变量的一份新的副
本， 更新这些副本的值也不会影响驱动器中的对应变量。 如果我们想实现所有分片处理时更新共享变
量的功能，那么累加器可以实现我们想要的效果。
1. Spark提供了一个默认的累加器,只能用于求和没啥用
2. 如何使用:
2.1.通过SparkContext对象.accumulator(0) var sum = sc.accumulator(0)
 通过accumulator声明一个累加器,0为初始化的值
 2.2.通过转换或者行动操作,通过sum +=n 来使用
 2.3.如何取值? 在Driver程序中,通过 sum .value来获取值
 3.累加器是懒执行,需要行动触发
```



13. Task分为几种？说说Task的原理（10分）

两种：resultTask、shuffleMapTask

接收到RDD，将数据类型序列化，写入不同的分区，不断地进行数据的trasnformation操作，这就是shuffleMapTask过程，一直到需要将数据shuffle，则最后一次就是resultTask；



14. Spark和Hadoop的MR区别？（5分）

1、效率：spark是基于内存运算的，mr是磁盘并加以不断地持久化导致效率比spark的效率低一些

2、稳定性：但是spark使用的内存的稳定性不强，可能使得数据丢失或者宕机，mr的运算就要稳定了很多

3、运行的场所不同：mr是运行在hdfs上的，spark直接在本地计算



# 第二周

### 1.Spark中Worker的主要作用是什么？

> 在spark的standalone模式部署下,不需要依赖其他的资源调度框架,自身就实现了资源调度的功能,在该模式下有master和worker两个核心组件.
> 1、通过RegisterWorker注册到Master;
>
> 2、定时发送心跳给Master;
>
> 3、根据Master发送的Application配置进程环境，并启动ExecutorBackend(执行Task所需要的临时进程)

### 2.有一个test.json文件，里面的数据如下。利用sparkSQL方式读取文件，然后使用DataFrame进行显示，使两种方式创建DataFrame。（手写代码实现）

> {"age" : ”23” , "name":"xiaoming" }

> **方式一:RDD转换**

```
val ss: SparkSession = SparkSession.builder()
  .appName("SparkSqlTest1").master("local").getOrCreate()
val rdd: RDD[String] = ss.sparkContext.textFile("data/test.json")
val dataFrame: DataFrame = ss.read.json(rdd)
dataFrame.foreach(println(_))
```

> **方式二:读取Json文件直接创建DataFrame**

```
val ss: SparkSession = SparkSession.builder()
	.appName("SparkSqlTest1").master("local").getOrCreate()
val dataFrame: DataFrame = ss.read.format("json").load("data/test.json")
dataFrame.foreach(println(_))
```



### 3.Driver的功能是什么？

>
>
>Driver就是我们写的spark程序,打成jar包后通过spark-submit来提交.
>如果你使用spark-shell去提交job的话Driver会是运行在master上的，如果你使用spark-submit或者IDEA开发工具方式运行，那么它是运行在Client上的。
>standalone模式下:
>	driver进程启动后,首先会构建sparkcontext，sparkcontext主要包含两部分：DAGScheduler和 TaskScheduler,然后TaskScheduler会寻找集群资源管理器(Master/Worker)的Master节点，Master节点接收到ApplicationMaster的注册请求后，通过资源调度算法，在自己的集群的worker上启动Executor进程,启动的executor也会反向注册到TaskScheduler(SchedulerBackend)上.Driver等待资源满足，执行main函数，Spark的查询为懒执行，当执行到 action 算子时才开始真正执行，开始反向推算,一个Action算子触发一个job，并交给DAGScheduler来处理.	
>		DAGScheduler主要做两个部分的事情：
>				①将1个job切分成多个stage.DAGScheduler会根据RDD的血缘关系构成的DAG从后往前，由最终的RDD不断通过依赖回溯判断父依赖是否是宽依赖，遇到一个shuffle就划分一个Stage，将一个Job划分为若干Stages.无shuffle的称为窄依赖，窄依赖之间的RDD被划分到同一个Stage中。划分的Stages分两类，一类叫做ResultStage，为DAG最下游的Stage，由Action方法决定，另一类叫做ShuffleMapStage，为下游Stage准备数据。
>				②将stage打包成Taskset提交.一个Stage如果没有父Stage，那么从该Stage开始提交，父Stage执行完毕才能提交子Stage。Stage提交时会将Task信息（分区信息以及方法等 一个Partition对应一个Task）序列化并被打包成TaskSet交给TaskScheduler，，另一方面TaskScheduler会监控Stage的运行状态，只有Executor丢失或者Task由于Fetch失败才需要重新提交失败的Stage以调度运行失败的任务，其他类型的Task失败会在TaskScheduler的调度过程中重试。
>		TaskScheduler.TaskScheduler将接收的TaskSet封装为TaskSetManager(一对一)加入到调度队列中。一个TaskSet含有n多个task信息，这些task都是同一个stage的。TaskScheduler初始化后会启动SchedulerBackend，它负责跟外界打交道，接收Executor的注册信息，并维护Executor的状态.SchedulerBackend监控到有资源后，会询问TaskScheduler有没有任务要运行，TaskScheduler会从调度队列中按照指定的调度策略选择TaskSetManager去调度运行。TaskSetManager按照一定的调度规则一个个取出task给TaskScheduler，TaskScheduler再交给SchedulerBackend去发到Executor上执行。Task被提交到Executor启动执行.Executor进程内部会维护一个线程池，Executor每接收到一个task，都会用TaskRunner封装task，然后从线程池中取出一个线程去执行taskTaskRunner主要包含两种task：ShuffleMapTask和ResultTask，除了最后一个stage是ResultTask外，其他的stage都是ShuffleMapTask.Executor会将执行状态上报给SchedulerBackend，SchedulerBackend则告诉TaskScheduler，TaskScheduler找到该Task对应的TaskSetManager，并通知到该TaskSetManager.这样TaskSetManager就知道Task的运行状态.对于运行失败的Task，TaskSetManager会记录它失败的次数，如果失败次数还没有超过最大重试次数，那么就把它放回待调度的Task池子中等待重新执行，当重试次数过允许的最大次数，整个Application失败。在记录Task失败次数过程中，TaskSetManager还会记录它上一次失败所在的ExecutorId和Host，这样下次再调度这个Task时，会使用黑名单机制，避免它被调度到上一次失败的节点上，起到一定的容错作用。
>	所有task完成以后，SparkContext向Master注销并释放资源
>
>yarn-client模式下:
>	在YARNClient模式下，Driver在任务提交的本地机器上运行，Driver会向ResourceManager申请启动ApplicationMaster，随后ResourceManager分配container，在合适的NodeManager上启动ApplicationMaster，此时的ApplicationMaster的功能相当于一个ExecutorLaucher，只负责向ResourceManager申请Executor内存。ResourceManager接到ApplicationMaster的资源申请后会分配container，然后ApplicationMaster在资源分配指定的NodeManager上启动Executor进程，Executor进程启动后会向Driver反向注册。另外一条线，Driver自身资源满足的情况下，Driver开始执行main函数，之后执行Action算子时，触发一个job，并根据宽依赖开始划分stage，每个stage生成对应的taskSet，Executor注册完成后，Driver将task分发到各个Executor上执行。(具体细节见上)
>
>yarn-cluster模式下:
>	在 YARN Cluster 模式下，任务提交后会和 ResourceManager 通讯申请启动ApplicationMaster，随后 ResourceManager 分配 container，在合适的 NodeManager上启动 ApplicationMaster，此时的ApplicationMaster 就是 Driver。Driver 启动后向 ResourceManager 申请 Executor 内存，ResourceManager会分配container，然后在合适的 NodeManager 上启Executor 进程，Executor 进程启动后会向 Driver 反向注册。另外一条线，Driver自身资源满足的情况下，开始执行main函数，之后执行Action算子时，触发一个job，并根据宽依赖开始划分stage，每个stage生成对应的taskSet，Executor注册完成后，Driver将task分发到各个Executor上执行。

### 4.解释一下什么是窗口函数？使用具体的例子说明如何应用（需要数据，需求说明以及sql说明）？

> 每当窗口在输入数据流上滑动一次，在这个窗口内的源RDDs 就会被聚合和操作然后产生 基于窗口流的RDDs。在这个例子中，过去三个时间单元的数据会被操作一次，然后每次滑动两个时间单元。这就是说 任何窗口操作都需要指定两个参数：
>
> - 窗口长度：窗口持续时间
> - 滑动间隔：每个窗口操作的时间间隔
>
> 这两个参数必须是输入源数据流间隔时间的倍数
>  让我们来用例子演示一下。比方说，你想要扩展一下之前的例子，要求能够每隔10s中，计算出过去30s的单词的统计值。为了做到这一点，我们需要在过去30s的键值对（word,1）的数据流（DataStream）留上使用 reduceByKey 这个操作.使用reduceByKeyAndWindow这个可以实现这个功能。

### 5.谈谈你知道的spark有哪些性能优化（并解释如何优化）

> **1.资源参数调优**
>
> spark参数调优主要就是对spark运行过程中各个使用资源的地方，通过调节各种参数，来优化资源使用的效率，从而提升spark作业的执行性能。
>
> --executor-cores
>
> **参数说明**：该参数用于设置每个Executor进程的CPU core数量。这个参数决定了每个Executor进程并行执行task线程的能力。因为每个CPU core同一时间只能执行一个task线程，因此每个Executor进程的CPU core数量越多，越能够快速地执行完分配给自己的所有task线程。
>
> **调优建议**：Executor的CPU core数量设置为2~4个较为合适。得根据不同部门的资源队列来定，可以看看自己的资源队列的最大CPU core限制是多少，再依据设置的Executor数量，来决定每个Executor进程可以分配到几个CPU core。同样建议，如果是跟他人共享这个队列，那么num-executors * executor-cores不要超过队列总CPU core的1/3~1/2左右比较合适，避免影响其他同事的作业运行。
>
> --executor-memory
>
> **参数说明**：该参数用于设置每一个Executor进程的内存。Executor内存的大小，很多时候直接决定了spark作业的性能，而且跟常见的JVM OOM异常，也有直接关联。
>
> **调优建议**：每一个Executor进程的内存设置为4G~8G较为合适，但是这也是一个参考值，具体的设置还是得根据不同部门的资源队列来定。可以看看自己团队的资源队列的最大内存限制是多少。num-executor乘以executor-memory，就代表了Spark作业申请到的总内存量（也就是Executor进程的内存总和），这个量是不能超过队列的最大内存量的。此外，如果你是跟团队里其他人共享这个资源队列，那么申请的总内存量最好不要超过资源队列最大总内存的1/3~1/2，避免你自己的Saprk作业占用了队列所有的资源，导致别的同事的作业无法正常运行。
>
> --total-executor-cores
>
> --driver-cores
>
> --driver-memory
>
> **参数说明**：该参数用于设置Driver进程的内存。
>
> **调优建议**：Driver的内存通常来说不设置，或者设置1G左右应该就够了。唯一需要注意的一点是，如果需要使用collect算子将RDD的数据全部拉取到Driver上进行处理，那么必须确保Driver的内存足够大，否则会出现OOM内存溢出的问题。
>
> 
>
> **2.并行度调节**
>
> （1）sc.textFile(xx,minnumpartition) java/scala
>
> （2）sc.parallelize(xx.num) --java/scala
>
> （3）sc.makeRDD(xx,num) --scala
>
> （4）sc.parallelizePairs(xx,num) --java
>
> **参数说明：**以上四个都是设置分区数
>
> （5）rdd.repartitiion(num) /rdd.coalesce(num)
>
> **参数说明**：重分区repartition方法就是调用了coalesce方法,shuffle为true的情况，coalesce没有shuffle
>
> （6）rdd.reduceByKey(xx,num) / groupByKey(xx,num) /join(xx,num)....
>
> **参数说明：**调节聚合后的RDD的并行度
>
> （7）spark.default.parallelism
>
> **参数说明：**该参数用于设置每个stage的默认task数量。这个参数极为重要，如果不设置可能会直接影响你的Spark作业性能。
>
> **参数调优说明：**Spark作业的默认task数量为500~1000个较为合适。很多同学常犯的一个错误就是不去设置这个参数，那么此时就会导致Spark自己根据底层HDFS的block数量来设置task的数量，默认是一个HDFS block对应一个task。通常来说，Spark默认设置的数量是偏少的（比如就几十个task），如果task数量偏少的话，就会导致你前面设置好的Executor的参数都前功尽弃。试想一下，无论你的Executor进程有多少个，内存和CPU有多大，但是task只有1个或者10个，那么90%的Executor进程可能根本就没有task执行，也就是白白浪费了资源！因此Spark官网建议的设置原则是，设置该参数为num-executors * executor-cores的2~3倍较为合适，比如Executor的总CPU core数量为300个，那么设置1000个task是可以的，此时可以充分地利用Spark集群的资源。
>
> （8）自定义分区器partitioner
>
> （9）spark.sql.shuffle.partitions = 200
>
> **参数说明：**在使用Spark SQL时，设置shuffle的分区数，默认是200.
>
> (10)SparkStreaming:
>
> Direct:topic中partiton个数一致，增大topic的分区数|读取dstream 进行重新分区

### 6.Spark任务运行原理？

> ##### Yarn-Cluster模式
>
> （1）ResourceManager接到请求后在集群中选择一个NodeManager分配Container，并在Container中启动ApplicationMaster进程；
> （2）在ApplicationMaster进程中初始化sparkContext；
> （3）ApplicationMaster向ResourceManager申请到Container后，通知NodeManager在获得的Container中启动excutor进程；
> （4）sparkContext分配Task给excutor，excutor发送运行状态给ApplicationMaster。
>
> 
>
> ##### Yarn-Client模式
>
> （1）ResourceManager接到请求后在集群中选择一个NodeManager分配Container，并在Container中启动ApplicationMaster进程；
> （2）driver进程运行在client中，并初始化sparkContext；
> （3）sparkContext初始化完后与ApplicationMaster通讯，通过ApplicationMaster向ResourceManager申请Container，ApplicationMaster通知NodeManager在获得的Container中启动excutor进程；
> （4）sparkContext分配Task给excutor，excutor发送运行状态给driver。

### 7.谈谈spark中的宽窄依赖

> 宽依赖：指的是多个子RDD的Partition会依赖同一个父RDD的Partition，关系是一对多，父RDD的一个分区的数据去到子RDD的不同分区里面，会有shuffle的产生
>
> 窄依赖：指的是每一个父RDD的Partition最多被子RDD的一个partition使用，是一对一的，也就是父RDD的一个分区去到了子RDD的一个分区中，这个过程没有shuffle产生
>
> 
>
> 区分的标准就是看父RDD的一个分区的数据的流向，要是流向一个partition的话就是窄依赖，否则就是宽依赖

### 8.谈谈你对kafka的理解，是否存在主从之分，为什么使用，说明原因，zookeeper在kafka中起到什么作用，详细说明。

> **kafka是否存在主从之分**
>
> 存在,kafka创建副本的单位是topic的分区，每个分区都有一个leader和零或多个followers.所有的读写操作都由leader处理，一般分区的数量都比broker的数量多的多，各分区的leader均匀的分布在brokers中。所有的followers都复制leader的日志，日志中的消息和顺序都和leader中的一致。flowers向普通的consumer那样从leader那里拉取消息并保存在自己的日志文件中。
>
> 
>
> **zookeeper在kafka中的作用**
>
> Zookeeper是一个开放源码的、高性能的协调服务，它用于Kafka的分布式应用。作用：协调Kafka Broker，存储原数据：consumer的offset+broker信息+topic信息+partition个信息。
>
> 一旦Zookeeper停止工作，它就不能服务客户端请求。
>
> Zookeeper主要用于在集群中不同节点之间进行通信
>
> 在Kafka中，它被用于提交偏移量，因此如果节点在任何情况下都失败了，它都可以从之前提交的偏移量中获取
>
> 除此之外，它还执行其他活动，如: leader检测、分布式同步、配置管理、识别新节点何时离开或连接、集群、节点实时状态等等。

### 9.Kafka分区和消费者的关系？

> Kafka提供的两种分配策略： range和roundrobin，由参数partition.assignment.strategy指定，默认是range策略。
>
> 当以下事件发生时，Kafka 将会进行一次分区分配：
>
> - 同一个 Consumer Group 内新增消费者
> - 消费者离开当前所属的Consumer Group，包括shuts down 或 crashes
> - 订阅的主题新增分区

### 10.请说一下kafka为什么需要副本，原始分区和副本是如何设置的，当消费者进行消费数据的时候，怎么才可以精准的消费到数据?

> Kafka 可以保证单个分区里的事件是有序的，分区可以在线（可用），也可以离线（不可用）。在众多的分区副本里面有一个副本是 Leader，其余的副本是 follower，所有的读写操作都是经过 Leader 进行的，同时 follower 会定期地去 leader 上的复制数据。当 Leader 挂了的时候，其中一个 follower 会重新成为新的 Leader。通过分区副本，引入了数据冗余，同时也提供了 Kafka 的数据可靠性。
>
> Kafka 的分区多副本架构是 Kafka 可靠性保证的核心，把消息写入多个副本可以使 Kafka 在发生崩溃时仍能保证消息的持久性。

11.已知学生数据如下：请用spark core完成下列需求

| 1     | 班级  | 学号    | 性别 | 姓名   | 出生年月   | 血型 | 家庭住址        | 身高 | 手机号       |
| ----- | ----- | ------- | ---- | ------ | ---------- | ---- | --------------- | ---- | ------------ |
| **2** | RB171 | RB17101 | 男   | 张祥德 | 1997-02-10 | AB   | 河南省郑州市1号 | 172  | 111222233333 |
| **3** | RB171 | RB17102 | 女   | 冯成刚 | 1996-10-01 | A    | 河南省洛阳市2号 | 175  | 188371101154 |
| **4** | RB171 | RB17103 | 男   | 卢伟兴 | 1998-08-02 | B    | 河南省开封市3号 | 165  | 199992288225 |
| **5** | RB171 | RB17104 | 男   | 杨飞龙 | 1996-08-09 | AB   | 河南省安阳市4号 | 168  | 133225544556 |
| **6** | RB172 | RB17201 | 女   | 姜松林 | 1997-01-03 | A    | 河南省鹤壁市1号 | 170  | 136885522447 |
| **7** | RB172 | RB17202 | 男   | 高飞   | 1996-08-27 | B    | 河南省新乡市2号 | 171  | 135221144558 |
| **8** | RB172 | RB17203 | 女   | 何桦   | 1997-12-20 | B    | 河南省焦作市3号 | 168  | 135669988559 |

> **1.按照身高排序**

> **2.求平均年龄**

> **3.求学生中出现的所有姓氏**

> **4.返回出生在每月最大天数的3人**

> **5.索引出相同生日下同学的姓名链表**

```
package com.qf

import org.apache.spark.rdd.RDD
import org.apache.spark.sql._

object Demo {

  //班级 学号 性别 姓名 出生年月 血型 家庭住址 身高 手机号
  case class Student(clazz: String, sno: String, sex: String, name: String, birth: String, bloodType: String, adress: String, height: Int, phone: String)

  def main(args: Array[String]): Unit = {
    val spark: SparkSession = SparkSession.builder()
      .appName("test1").master("local").getOrCreate()

    val rdd: RDD[String] = spark.sparkContext.textFile("data/student.txt")
    val originRdd: RDD[Student] = rdd.mapPartitions(
      _.map(
        it => {
          val infos: Array[String] = it.split("\t")
          Student(infos(0), infos(1), infos(2), infos(3), infos(4), infos(5), infos(6), infos(7).toInt, infos(8))
        }
      )
    )

    println("1. 按照身高排序")
    originRdd.keyBy(_.height).sortByKey(false).values.foreach(println(_))

    println("2. 求平均年龄")
    originRdd.keyBy(
      it => {
        2020 - it.birth.split("-")(0).toInt
      }
    ).mapPartitions(
      _.map(
        it => {
          (1, (it._1, 1))
        }
      )
    ).reduceByKey(
      (it1, it2) => {
        (it1._1 + it2._1, it1._2 + it2._2)
      }
    ).map(
      it => {
        (it._2._1 / it._2._2)
      }
    ).foreach(println(_))

    println("3. 求学生中出现的所有姓氏")
    originRdd.keyBy(_.name.split("")(0)).reduceByKey(
      (it1, it2) => {
        it1
      }
    ).keys.foreach(println(_))

    println("4. 返回出生在每月最大天数的3人")
    originRdd.keyBy(
      _.birth.split("-")(2).toInt
    ).sortByKey(false).values.take(3).foreach(println(_))

    println("5. 索引出相同生日下同学的姓名链表")
    originRdd.map(
      it => {
        (it.birth.substring(5, 10).replace("-", ""), it.name)
      }
    ).groupByKey().map(
      it => {
        (it._1, it._2.toList)
      }
    ).foreach(println(_))

  }
  
}
```

### 12.手动实现Kafka的API操作

> **1.编写一个kafka的生产者API，要求手动输入值就能够推送到kafka**

```
public class MyProducer {
    public static void main(String[] args) {
 
        Properties props = new Properties();
        props.put("bootstrap.servers", "hadoop01:9092,hadoop02:9092");
        // 针对保存数据的确认
        props.put("acks", "all");
        // 尝试重试的次数
        props.put("retries", 0);
        // 按照batch发送数据
        props.put("batch.size", 16384);
        // 消息发送的延时时间
        props.put("linger.ms", 1);
        // 消息异步发送的存储消息的缓存
        props.put("buffer.memory", 33554432);
        // 指定key和value的序列化
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
 
        // 创建生产者对象
        KafkaProducer<String, String> producer = new KafkaProducer<String, String>(props);
        
        Scanner scanner = new Scanner();
        while(scanner.hasNext()){
            String record = scanner.nextLine();
            producer.send(record);
        }
        producer.close();
    }
```

> **2.编写一个kafka的消费者API，要求循环接受消费消息**

```
public class MyConsumer {
 
    public static void main(String[] args) {
 
        Properties props = new Properties();
        //broker节点
        props.put("bootstrap.servers", "hadoop01:9092,hadoop02:9092");
        //消费者组
        props.put("group.id", "itcast");
        //是否自动提交offset偏移量
        props.put("enable.auto.commit", "true");
        //500ms
        //每次地洞提交偏移量的时间
        props.put("auto.commit.interval.ms", "1000");
        //将数据进行反序列化
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        //创建消费者对象
        KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(props);
        //订阅主题
        consumer.subscribe(Arrays.asList("test"));
 
        while (true){
            //获取到消费的数据集合
            ConsumerRecords<String, String> records = consumer.poll(3000);
            for (ConsumerRecord<String, String> record : records) {
                //获取每一条记录
                String topic = record.topic();
                long offset = record.offset();
                String value = record.value();
                int partition = record.partition();
                System.out.println("topic:"+topic+",partition:"+partition+",offset:"+offset+",value:"+value);
            }
 
        }
 
    }
}
```

> **3.编写一个kafak的API，能够创建删除修改主题**

```
public class ConsoleApi {
    @Autowired
    private  KafkaConsumerConfig consumerConfig;
    private  ZkUtils zkUtils = null;
    private ZooKeeper zooKeeper = null;

    /**
     * -获取集群信息（与getAllBrokersInCluster（）只是返回类型不一致）
     */
    public  Cluster getCluster(){
        zkUtils = ZkUtils.apply(consumerConfig.getZookeeperConn(), 30000, 30000, JaasUtils.isZkSecurityEnabled());
        return zkUtils.getCluster();
    }
      public  void getLeaderAndIsrForPartition(String topicName,int patition){
          zkUtils = ZkUtils.apply(consumerConfig.getZookeeperConn(), 30000, 30000, JaasUtils.isZkSecurityEnabled());
          System.out.println("打印："+zkUtils.getLeaderAndIsrForPartition(topicName,patition));
    }

    public boolean createConsumer(String groupId,String topic) {
        try {
            Properties properties = new Properties();
            properties.put("zookeeper.connect", consumerConfig.getZookeeperConn());//声明zk
            properties.put("group.id", groupId);
            ConsumerConnector consumer = new ZookeeperConsumerConnector(new ConsumerConfig(properties),true);
            java.util.Map<String, Integer> topicCountMap = new HashMap<String, Integer>();
            if (topic!=null &&!"".equals(topic)){
                topicCountMap.put(topic, 1); // 一次从主题中获取一个数据
            }else {
                topicCountMap.put("topic", 1); // 一次从主题中获取一个数据
            }
            java.util.Map<String, List<KafkaStream<byte[], byte[]>>> messageStreams = consumer.createMessageStreams(topicCountMap);
            return true;
        }catch (RuntimeException e){
            return false;
        }
    }
    public boolean createConsumer(String groupId) {
        return createConsumer(groupId,null);
    }
    public boolean deleteUselessConsumer(String group) {
        return deleteUselessConsumer("-1", group);
    }
    /**
     * -删除topic路径
     * @return
     */
    public  String deleteTopicsPath(){
        zkUtils = ZkUtils.apply(consumerConfig.getZookeeperConn(), 30000, 30000, JaasUtils.isZkSecurityEnabled());
        return zkUtils.DeleteTopicsPath();
    }
    
    /**
     * 删除多个topic
     * @param topicNames
     * @return
     */
    public  String[] deleteTopics(final String...topicNames) {
        if(topicNames==null || topicNames.length==0) return new String[0];
         java.util.Set<String> deleted = new LinkedHashSet<String>();
        for(String topicName: topicNames) {
            if(topicName!=null || !topicName.trim().isEmpty()) {
                    deleteTopic(topicName);
                    deleted.add(topicName.trim());
            }
        }
        return deleted.toArray(new String[deleted.size()]);
    }

    /**
     * 判断某个topic是否存在
     * @param topicName
     * @return
     */
    public  boolean topicExists(String topicName) {
        zkUtils = ZkUtils.apply(consumerConfig.getZookeeperConn(), 30000, 30000, JaasUtils.isZkSecurityEnabled());
        boolean exists = AdminUtils.topicExists(zkUtils,topicName);
        return exists;
    }
  
 /**
     * 获取删除主题的路径
     * @param topicName
     * @return
     */
    public  String getDeleteTopicPath(String topicName) {
        zkUtils = ZkUtils.apply(consumerConfig.getZookeeperConn(), 30000, 30000, JaasUtils.isZkSecurityEnabled());
        String stringSeq =  zkUtils.getDeleteTopicPath(topicName);
        return stringSeq;
    }


    /**
     * 删除topic信息（前提是server.properties中要配置delete.topic.enable=true）
     * @param topicName
     */
    public  void deleteTopic(String topicName){
        zkUtils = ZkUtils.apply(consumerConfig.getZookeeperConn(), 30000, 30000, JaasUtils.isZkSecurityEnabled());
        // 删除topic 't1'
        AdminUtils.deleteTopic(zkUtils, topicName);
        System.out.println("删除成功！");
    }

    /**
     *  删除topic的某个分区
     * @param brokerId
     * @param topicName
     */
  public  void deletePartition(int brokerId,String topicName){
        // 删除topic 't1'
      zkUtils = ZkUtils.apply(consumerConfig.getZookeeperConn(), 30000, 30000, JaasUtils.isZkSecurityEnabled());
      zkUtils.deletePartition(brokerId,topicName);
        System.out.println("删除成功！");
    }
    /**
     * 改变topic的配置
     * @param topicName
     */
    public  void updateTopic(String topicName){
        zkUtils = ZkUtils.apply(consumerConfig.getZookeeperConn(), 30000, 30000, JaasUtils.isZkSecurityEnabled());

        Properties props = AdminUtils.fetchEntityConfig(zkUtils, ConfigType.Topic(), "test");
    // 增加topic级别属性
        props.put("min.cleanable.dirty.ratio", "0.3");
    // 删除topic级别属性
        props.remove("max.message.bytes");
        props.put("retention.ms","1000");
    // 修改topic 'test'的属性
        AdminUtils.changeTopicConfig(zkUtils, "test", props);
        System.out.println("修改成功");
        zkUtils.close();
    }
    private boolean deleteUselessConsumer(String topic, String group) {
        //if (topic.endsWith("-1")) {
        StringBuilder sb = new StringBuilder().append("/consumers/")
                .append(group);
        return recursivelyDeleteData(sb.toString());
    }

    private boolean recursivelyDeleteData(String path) {
        List<String> childList = getChildrenList(path);
        if (childList == null) {
            return false;
        } else if (childList.isEmpty()) {
            deleteData(path);
        } else {
            for (String childName : childList) {
                String childPath = path + "/" + childName;
                List<String> grandChildList = getChildrenList(childPath);
                if (grandChildList == null) {
                    return false;
                } else if (grandChildList.isEmpty()) {
                    deleteData(childPath);
                } else {
                    recursivelyDeleteData(childPath);
                }
            }
            deleteData(path);
        }
        return true;
    }

    private boolean deleteData(String path) {
        try {
            zooKeeper.delete(path, -1);
        } catch (InterruptedException e) {
            //log.error("delete error,InterruptedException:" + path, e);
            return false;
        } catch (KeeperException e) {
            //log.error("delete error,KeeperException:" + path, e);
            return false;
        }
        return true;
    }

    private List<String> getChildrenList(String path) {

        try {
            zooKeeper = new ZooKeeper(consumerConfig.getZookeeperConn(), 6000, null);
            return zooKeeper.getChildren(path, false, null);
        } catch (KeeperException e) {
            return null;
        } catch (InterruptedException e) {
            return null;
        } catch (IOException e) {
            e.printStackTrace();
        }
        return new ArrayList<String>(Collections.singleton(path));
    }

    /**
     * get all subscribing consumer group names for a given topic
     * @param brokerListUrl localhost:9092 for instance
     * @param topic         topic name
     * @return
     */
    public static java.util.Set<String> getAllGroupsForTopic(String brokerListUrl, String topic) {
        AdminClient client = AdminClient.createSimplePlaintext(brokerListUrl);
        try {
            List<GroupOverview> allGroups = scala.collection.JavaConversions.seqAsJavaList(client.listAllGroupsFlattened().toSeq());
            java.util.Set<String> groups =  new HashSet<String>();
            for (GroupOverview overview: allGroups) {
                String groupID = overview.groupId();
                java.util.Map<TopicPartition, Object> offsets = JavaConversions.mapAsJavaMap(client.listGroupOffsets(groupID));
                        groups.add(groupID);
            }
            return groups;
        } finally {
            client.close();
        }
    }

}
```



# 第三周

### 1.Redis支持哪几种数据结构？

> 一 string（字符串）
>
> 　　string是最简单的类型，你可以理解成与Memcached一模一样的类型，一个key对应一个value，其上支持的操作与Memcached的操作类似。但它的功能更丰富。
>
> 二 list(双向链表)
>
> 　　list是一个链表结构，主要功能是push、pop、获取一个范围的所有值等等。之所以说它是双向的，因为它可以在链表左，右两边分别操作
>
> 三 dict(hash表)
>
> 　　set是集合，和我们数学中的集合概念相似，对集合的操作有添加删除元素，有对多个集合求交并差等操作。操作中key理解为集合的名字
>
> 四 zset(排序set)
>
> 　　zset是set的一个升级版本，他在set的基础上增加了一个顺序属性，这一属性在添加修改元素的时候可以指定，每次指定后，zset会自动重新按新的值调整顺序。 可以对指定键的值进行排序权重的设定，它应用排名模块比较多
>
> 五 Hash类型
>
> Redis能够存储key对多个属性的数据（比如user1.uname user1.passwd），当然，你完成可以把这些属性以json格式进行存储，直接把它当作string类型进行操作，但这样性能上是对影响的，所以redis提出的Hash类型。

### 2.Redis的持久化如何实现？

> ## 1. RDB(snapshotting快照)
>
> 也是默认方式.(把数据做一个备份，将数据存储到文件)
>
> 快照是默认的持久化方式，这种方式是将内存中数据以快照的方式写到二进制文件中，默认的文件名称为dump.rdb.可以通过配置设置自动做快照持久化的方式。我们可以配置redis在n秒内如果超过m个key键修改就自动做快照.
>
> 
>
> 数据快照的原理是将整个Redis内存中的所有的数据遍历一遍存储到一个扩展名为rdb的数据文件中，通过save命令
>
> 可以调用这个过程。数据快照配置如下:
>
> Save 900 1
>
> Save 300 10
>
> Save 60 10000
>
> 以上在redis.conf中的配置指出在多长时间内，有多少次更新操作，就将数据同步到数据文件中，这个可以多个条件进行配合，上面的含义是900秒后有一个key发生改变就执行save,300秒后有10个key发生改变就执行save,60秒有10000个key发生改变就执行save.
>
> ### RDB持久化机制的工作流程
>
> （1）redis根据配置自己尝试去生成rdb快照文件
>
> （2）fork一个子进程出来
>
> （3）子进程尝试将数据dump到临时的rdb快照文件中
>
> （4）完成rdb快照文件的生成之后，就替换之前的旧的快照文件
>
> dump.rdb，每次生成一个新的快照，都会覆盖之前的老快照
>
> ##  2. Append-onlyfile(缩写aof)的方式    
>
> aof方式:由于快照方式是在一定间隔时间做一次的，所以如果redis意外down掉的话，就会丢失最后一次快照后的所有修改。aof比快照方式有更好的持久化性，是由于在使用aof时，redis会将每一个收到的写命令都通过write函数追加到文件中，当redis重启时会通过重新执行文件中保存的写命令来在内存中重建整个数据库的内容。   
>
> 当然由于os会在内核中缓存write做的修改，所以可能不是立即写到磁盘上。这样aof方式的持久化也还是有可能会丢失部分修改。可以通过配置文件告诉redis我们想要通过fsync函数强制os写入到磁盘的时机。
>
> Appendfsync no/always/everysec
>
> no:表示等操作系统进行数据缓存同步到磁盘。性能最好，持久化没有保障。
>
> Always:表示每次更新操作后手动调用fsync()将数据写到磁盘.每次收到写命令就立即强制写入磁盘，最慢的，但是保障完全的持久化。
>
> Everysec:表示每秒同步一次.每秒钟强制写入磁盘一次，在性能和持久化方面做了很好的折中。
>
> ### AOF rewrite
>
> redis中的数据其实是有限的，很多数据可能会自动过期，可能会被用户删除，可能会被redis用缓存清除的算法清理掉
>
> redis中的数据会不断淘汰掉旧的，就一部分常用的数据会被自动保留在redis内存中
>
> 所以可能很多之前的已经被清理掉的数据，对应的写日志还停留在AOF中，AOF日志文件就一个，会不断的膨胀，到很大很大
>
> 所以AOF会自动在后台每隔一定时间做rewrite操作，比如日志里已经存放了针对100w数据的写日志了，redis内存只剩下10万，基于内存中当前的10万数据构建一套最新的日志，到AOF中，覆盖之前的老日志，确保AOF日志文件不会过大，保持与redis内存数据量一致
>
> redis 2.4之前，还需要手动通过crontab开发一些脚本，通过BGREWRITEAOF命令去执行AOF rewrite，但是redis 2.4之后，会自动进行rewrite操作
>
> 在redis.conf中，可以配置rewrite策略
>
> auto-aof-rewrite-percentage 100
>
> auto-aof-rewrite-min-size 64mb
>
> 
>
> 比如说上一次AOF rewrite之后，是128mb
>
> 
>
> 然后就会接着128mb继续写AOF的日志，如果发现增长的比例，超过了之前的100%，256mb，就可能会去触发一次rewrite
>
> 
>
> 但是此时还要去跟min-size，64mb去比较，256mb > 64mb，才会去触发rewrite
>
> 具体过程
>
> （1）redis fork一个子进程
>
> （2）子进程基于当前内存中的数据，构建日志，开始往一个新的临时的AOF文件中写入日志
>
> （3）redis主进程，接收到client新的写操作之后，在内存中写入日志，同时新的日志也继续写入旧的AOF文件
>
> （4）子进程写完新的日志文件之后，redis主进程将内存中的新日志再次追加到新的AOF文件中
>
> （5）用新的日志文件替换掉旧的日志文件
>
> 
>
> # RDB的问题
>
> Redis有两种存储方式，默认是snapshot方式，实现方法是定时将内存的快照(snapshot)持久化到硬盘，这种方法缺点是持久化之后如果出现crash则会丢失一段数据。
>
> # AOF的问题
>
> aof即append only mode，在写入内存数据的同时将操作命令保存到日志文件，在一个并发更改上万的系统中，命令日志是一个非常庞大的数据，管理维护成本非常高，恢复重建时间会非常长，这样导致失去aof高可用性本意。另外更重要的是Redis是一个内存数据结构模型，所有的优势都是建立在对内存复杂数据结构高效的原子操作上，这样就看出aof是一个非常不协调的部分。其实aof目的主要是数据可靠性及高可用性.
>
> 
>
> 如果同时使用RDB和AOF两种持久化机制，那么在redis重启的时候，会使用AOF来重新构建数据，因为AOF中的数据更加完整
>
> # RDB和AOF到底该如何选择
>
> （1）不要仅仅使用RDB，因为那样会导致你丢失很多数据
>
> （2）也不要仅仅使用AOF，因为那样有两个问题，第一，你通过AOF做冷备，没有RDB做冷备 恢复的速度快; 第二，RDB每次简单粗暴生成数据快照，更加健壮，可以避免AOF这种复杂的备份和恢复机制的bug
>
> （3）综合使用AOF和RDB两种持久化机制，用AOF来保证数据不丢失，作为数据恢复的第一选择; 用RDB来做不同程度的冷备，在AOF文件都丢失或损坏不可用的时候，还可以使用RDB来进行快速的数据恢复

### 3.Spark Streaming的UpdateStateByKey算子和mapWithState算子的区别？

> - updateStateByKey
>
>   ：
>
>   - UpdateStateBykey会统计全局的key的状态，不管有没有数据输入，它会在每一个批次间隔返回之前的key的状态。updateStateBykey会对已存在的key进行state的状态更新，同时还会对每个新出现的key执行相同的更新函数操作。如果通过更新函数对state更新后返回来为none，此时刻key对应的state状态会删除（state可以是任意类型的数据结构）
>   - updataeStateByKey返回在指定的批次间隔内返回之前的全部历史数据
>
> - mapWithState
>
>   ：
>
>   - mapWithState也是用于对于全局统计key的状态，但是它如果没有数据输入，便不会返回之前的key的状态，类型于增量的感觉。
>   - mapWithState只返回变化后的key的值
>
> **总结**
>
> - 统计历史数据使用updateStateByKey
> - 统计实时数据尽量使用mapWithState，`mapWithState`的性能要优于`updateStateByKey`

### 4.Kafka的文件存储机制？

> - Kafka把topic中一个parition大文件分成多个小文件段，通过多个小文件段，就容易定期清除或删除已经消费完文件，减少磁盘占用。
> - 通过索引信息可以快速定位message和确定response的最大大小。
> - 通过index元数据全部映射到memory，可以避免segment file的IO磁盘操作。
> - 通过索引文件稀疏存储，可以大幅降低index文件元数据占用空间大小。

### 5.Spark提交任务运行流程原理

> ##### Yarn-Cluster模式
>
> （1）ResourceManager接到请求后在集群中选择一个NodeManager分配Container，并在Container中启动ApplicationMaster进程；
> （2）在ApplicationMaster进程中初始化sparkContext；
> （3）ApplicationMaster向ResourceManager申请到Container后，通知NodeManager在获得的Container中启动excutor进程；
> （4）sparkContext分配Task给excutor，excutor发送运行状态给ApplicationMaster。
>
> 
>
> ##### Yarn-Client模式
>
> （1）ResourceManager接到请求后在集群中选择一个NodeManager分配Container，并在Container中启动ApplicationMaster进程；
> （2）driver进程运行在client中，并初始化sparkContext；
> （3）sparkContext初始化完后与ApplicationMaster通讯，通过ApplicationMaster向ResourceManager申请Container，ApplicationMaster通知NodeManager在获得的Container中启动excutor进程；
> （4）sparkContext分配Task给excutor，excutor发送运行状态给driver。

### 6.用编码的方式实现SparkSQL查询，要求：

> 1）获取HDFS的数据
>
> 2）用StructType的方式生成Schema
>
> 3）生成DataFrame
>
> 4）生成临时表
>
> 5）结果输出到HDFS

```
object test {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setMaster("local").setAppName("")

    val spark = SparkSession.builder().config(conf).getOrCreate()

    import spark.implicits._

    val input = "file:///D:/data/employee.json"

    val ql = spark.read.json(input)

    ql.createOrReplaceTempView("user")

    val sql1 = spark.sql("select max(salary) max_s,gender, region_code from user group by grouping sets ((region_code,gender),(region_code))")
   
    sql1.createOrReplaceTempView("user1")
    val sql2 = spark.sql("select max(max_s) s from user1 group by region_code,gender  with rollup ")
      .orderBy(desc("max_s")).limit(2).show()

//    ql.show()
//    sql1.show()
    //sql2.show()
  }
}
```

### 7.GC的原理？

> Java GC（Garbage Collection）垃圾回收机制，Java VM中，存在自动内存管理和垃圾清理机制。GC机制对JVM（Java Virtual Machine）中的内存进行标记，并确定哪些内存需要回收，根据一定的回收策略，自动的回收内存，永不停息（Nerver Stop）的保证JVM中的内存空间，防止出现内存泄露和溢出问题。Java中不能显式分配和注销内存。有些开发者把对象设置为null或者调用System.gc()显式清理内存。设置为null至少没什么坏处，但是调用System.gc()会一定程度上影响系统性能。Java开发人员通常无须直接在程序代码中清理内存，而是由垃圾回收器自动寻找不必要的垃圾对象，并且清理掉它们。

### 8.谈谈foreachRDD和foreachPartition和foreach的理解和区别？

> foreachRDD、foreachPartition和foreach的不同之处主要在于它们的作用范围不同，foreachRDD作用于DStream中每一个时间间隔的RDD，foreachPartition作用于每一个时间间隔的RDD中的每一个partition，foreach作用于每一个时间间隔的RDD中的每一个元素。
>
> Foreach与ForeachPartition都是在每个partition中对iterator进行操作,不同的是,foreach是直接在每个partition中直接对iterator执行foreach操作,而传入的function只是在foreach内部使用,而foreachPartition是在每个partition中把iterator给传入的function,让function自己对iterator进行处理（可以避免内存溢出）

### 9.数据形式：aa 11;bb 11;cc 34;aa 22   需求：

>  **1、对上述数据按key值进行分组**
>  **2、对分组后的值进行排序**
>  **3、分组后取top 3 以key-value形式返回结果**

```
object Test {
 def main(args: Array[String]): Unit = {
  val list = List(("aa",11),("bb",11),("cc",34),("aa",22)

   val maps = list.groupBy(_._1)
   val top = maps.map( t => {
     val key = t._1
     val values = t._2.toList.sorted.reverse.take(3)
     (key,values)
   })
    top.foreach(println)
 }
}
```

### 10.Spark On Yarn运行原理（Cluster模式）？

> - Spark Yarn Client向YARN中提交应用程序，包括ApplicationMaster程序、启动ApplicationMaster的命令、需要在Executor中运行的程序等
> - ResourceManager收到请求后，在集群中选择一个NodeManager，为该应用程序分配第一个Container，要求它在这个Container中启动应用程序的ApplicationMaster，其中ApplicationMaster进行SparkContext等的初始化
> - ApplicationMaster向ResourceManager注册，这样用户可以直接通过ResourceManage查看应用程序的运行状态，然后它将采用轮询的方式通过RPC协议为各个任务申请资源，并监控它们的运行状态直到运行结束
> - 一旦ApplicationMaster申请到资源（也就是Container）后，便与对应的NodeManager通信，要求它在获得的Container中启动CoarseGrainedExecutorBackend，而Executor对象的创建及维护是由CoarseGrainedExecutorBackend负责的，CoarseGrainedExecutorBackend启动后会向ApplicationMaster中的SparkContext注册并申请Task。这一点和Standalone模式一样，只不过SparkContext在Spark Application中初始化时，使用CoarseGrainedSchedulerBackend配合YarnClusterScheduler进行任务的调度，其中YarnClusterScheduler只是对TaskSchedulerImpl的一个简单包装，增加了对Executor的等待逻辑等
> - ApplicationMaster中的SparkContext分配Task给CoarseGrainedExecutorBackend执行，CoarseGrainedExecutorBackend运行Task并向ApplicationMaster汇报运行的状态和进度，以让ApplicationMaster随时掌握各个任务的运行状态，从而可以在任务失败时重新启动任务
> - 应用程序运行完成后，ApplicationMaster向ResourceManager申请注销并关闭自己

### 11.Spark Streaming的Receiver方式和直连方式的区别？

> Spark-Streaming获取kafka数据的两种方式-Receiver与Direct的方式，可以从代码中简单理解成Receiver方式是通过zookeeper来连接kafka队列，Direct方式是直接连接到kafka的节点上获取数据了。

### 12.你如何理解scala中的trait的？

>  **一.java中的接口不是面向对象的只是对多继承的一种补充 而scala是纯面向对象的所以使用trait(特征)相当于java中interface+abstract class**
>  **二.scala的没有implements关键字,它使用extends关键字实现trait**
>  **三.scala沿用也java的库所以scala中java的所有库可以当做trait来使用**
>
>  **四.scala也是单继承的使用trait满足了多继承的要求**
>
>  **五.trait 可以拥有抽象方法也可以拥有实现的方法(我们将scala的class文件反编译成java文件了解底层原理)**

### 13.你如何理解scala中的伴生对象和伴生类的?

> scala中class创建的是伴生类，object创建的是伴生对象
>
> 伴生类中可以声明无参构造器和有参构造器，辅助构造器，属性，方法
>
> 伴生对象属于单例对象，也可以声明属性，方法，但是不可以声明构造器
>
> scala创建对象可以通过new的方式也可以通过伴生对象的方式进行创建。但是如果想用伴生对象的方式进行创建就必须定义apply方法，在apply方法中通过new的方式创建对象。
>





