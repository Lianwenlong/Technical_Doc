# Hadoop 总结

## 一.  Hadoop 概述
### 1.1  Hadoop概念
1. Hadoop 是由Apache基金会所开发的**分布式系统基础架构**。
2. 主要解决,**海量数据**的**存储**和海量数据的**分析计算**问题
3. 通常来讲，Hadoop通常是指一个更广泛的概念--Hadoop生态圈

### 1.2  Hadoop 三大发行版本

① Apache

② Cloudera

③ Hortonworks

### 1.3  Hadoop 优势

① 高可靠性： Hadoop底层维护多个数据副本，所以即使Hadoop某个计算元素或存储出现故障，也不会导致数据的丢失。

② 高拓展性：在集群间分配任务数据，可方便的拓展数以千计的节点。

③ 高效性：在Map Reduce的思想下，Hadoop是并行工作的，以加快任务处理速度。

④ 高容错性：能够自动将失败的任务重新分配。

### 1.4  Hadoop 组成

![Hadoop版本](../Hadoop/img/Hadoop版本区别.png)

在Hadoop1.x版本中，Hadoop的Map Reduce同时是处理业务逻辑运算和资源的调度，耦合性较大。

在Hadoop2.x版本中，增加了Yarn。Yarn只负责资源的调度，Map Reduce只负责运算。

Hadoop3.x版本在组成上没有再做变化。

### 1.5  HDFS 架构概念

HDFS是Hadoop Distributed File System，简称HDFS，是一个分布式文件系统。

① NameNode（NN）：存储文件的**元数据**，如文件名，文件目录结构，文件属性（生成时间、副本数、文件权限），以及每个文件的块列表和块所在的DataNode等

② DataNode（DN）：在本地文件系统**存储文件块数据**，以及**块数据的校验和**。

③ Secondary NameNode：每隔一段时间对NameNode元数据备份

### 1.6  Yarn 架构概述

Yet Another Resource Negotiator简称Yarn，另一种资源协调者，是Hadoop的资源管理器。
![yarn](../Hadoop/img/Yarn.png)

① ResouceManager（RM）：整个集群资源（CPU、内存等）的老大

② NodeManager（NM）：单节点服务器资源老大

③ ApplicationMaster（AM）：单个任务运行的老大

④ Container：容器，相当于一台独立的服务器，里面封装了任务运行所需要的资源，**如内存、CPU、磁盘、网络等。**

注意：

客户端可以有多个，集群上可以运行多个ApplicationMaster，每个NodeManager上可以有多个Container

### 1.7  MapReduce 架构概述

MapReduce将计算过程分为两个阶段：Map和Reduce

① Map阶段并行处理输入数据

② Reduce阶段对Map结果进行汇总

![mapreduce](../Hadoop/img/MapReduce.png)



## 二.  Hadoop 环境搭建（todo）

### 2.n  常用端口号说明

|                                  | Hadoop2.x  | Hadoop3.x            |
| -------------------------------- | ---------- | -------------------- |
| NameNode内部通信端口             | 8020、9000 | 8020、9000、**9820** |
| NameNode HTTP UI（对外暴露端口） | **50070**  | **9870**             |
| MapReduce任务执行情况查看端口    | 8088       | 8088                 |
| 历史服务器通信端口               | 19888      | 19888                |


## 三.  Hadoop 运行模式（todo）

## 四.  常见错误及解决方案（todo）

## 五.  HDFS

### 5.1  HDFS概述

#### 5.1.1  产生背景和定义

​	随着数据量越来越大,在一个操作系统存不下所有的数据,那么就分配到更多的操作系统管理的磁盘中,但是不方便管理和维护,迫切**需要一种系统来管理多台计算器上的文件**,这就是分布式文件管理系统。**HDFS只是分布式文件管理系统中的一种**。

​	HDFS（Hadoop Distributed File System），它是一个文件系统，用于存储文件，通过目录树来定位文件：其次，它是分布式的，由很多服务器联合起来实现其功能，集群中的服务器有各自的角色。

​	HDFS的使用场景：**适合一次写入，多次读取得场景**。一个文件经过创建、写入和关闭之后就不需要改变。



#### 5.1.2  优缺点

- **优点：**
  - **高容错性**
    - 数据自动保存多个副本。它通过增加副本形式，提高容错性。
    - 某一个副本丢失以后，它可以自动恢复。
  - **适合处理大数据**
    - 数据规模：能够处理数据规模达到GB、TB、甚至PB级别的数据
    - 文件规模：能够处理**百万**规模以上的**文件数量**。
  - 可**构建在廉价机器上**，通过多副本机制，提高可靠性。
- **缺点：**
  - **不适合低延时的数据访问，比如毫秒级的存储数据，是做不到的。**
  - **无法高效的对大量小文件进行存储。**
    - 存储大量小文件的话，它会占用NameNode大量的内存来存储文件目录和块信息。这样是不可取的，因为NameNode的内存总是有限的。
    - 小文件存储的寻址时间会超过读取时间，它违反了HDFS的设计目标。
  - **不支持并发写入、文件随机修改**
    - 一个文件只能有一个写，不允许多个线程同时写
    - 仅支持数据append（追加），不支持文件的随机修改
  
  

#### 5.1.3  组成架构

![HDFS Architecture](../Hadoop/img/hdfsarchitecture.png)

​	HDFS采用Master/Slave架构，一个HDFS集群是由一个NameNode和一定数量的DataNode组成的。

- **NameNode（NN）**：就是Master，它是一个中心服务器。负责：
  - 管理HDFS的名称空间
  - 配置副本策略
  - 管理数据块（Block）映射信息
  - 处理客户端读写请求
- **DataNode（DN）**：就是Slave。NameNode下达命令，DataNode执行实际操作。
  - 存储实际的数据块
  - 执行数据块的读写操作
- **Clent**：客户端
  - 文件切分。文件上传HDFS时，Client将文件切分成一个个块（Block），然后进行上传
  - 与NameNode交互，获取文件的位置信息
  - 与DataNode交互，读写数据
  - Client提供一些命令来管理HDFS，比如NameNode格式化
  - Clent可以通过一些命令来访问HDFS，比如对HDFS增删查改操作
- **Secondary NameNode**：并非NameNode的热备。当NameNode挂掉的时候，它并不能马上替换NameNode并提供服务。
  - 辅组NameNode，分担器工作量，比如定期合并Fsimage和Edits，并推送给NameNode
  - 在紧急情况下，可辅助恢复NameNode



#### 5.1.4 HDFS Block大小<font color=red>（面试重点）</font>

​	HDFS中的文件在物理上时分块（block）存储的，块的大小可以通过参数（dfs.blocksize）来设置，在Hadoop 1.x 默认是64m，在Hadoop 2.x和Hadoop 3.x中默认为128m。<font color=red>**HDFS块的大小设置主要取决于磁盘传输速率**</font>。

​	目前普通的硬盘的传输速率普遍为100m/s，固态硬盘能达到200~300m/s。专家认为当寻址时间（查询到目标块所花费的时间）为传输时间的1%时，则为最佳状态。如果寻址时间约为10ms，即查找到目标block的时间为10ms。那么传输时间为 10ms/0.01=1000ms = 1s。那么如果是普通磁盘的话，那么块大小就设置为128m，如果是固态硬盘，则可以设置为256m。

​	HDFS块的大小不能设置太小也不能设置太大。如果<font color=red>**设置太小，会增加寻址时间**</font>，程序一直在找块的开始位置。如果块<font color=red>**设置太大，从磁盘传输数据的时间会明显大于定位这个块开始位置所需的时间**</font>。而且快太大，也会导致程序在处理这块数据是，会非常慢。



### 5.2  HDFS的Shell相关操作（todo）

### 5.3  HDFS的客户端API（todo）

### 5.4  <font color=red>HDFS的读写流程 (重点)</font>

#### 5.4.1 HDFS写数据流程

![HDFSWrite](../Hadoop/img/HDFSWrite.png)

① HDFS客户端要往集群传送数据，首先需要在客户端创建一个分布式的文件系统客户端，创建完之后，向NameNode发送上传文件请求。

② NameNode需要对要上传的文件进行校验，校验文件权限和文件是否存在等合法性校验,校验完之后给客户端响应是否可以上传

③ 当校验通过后，客户端向NameNode请求获取上传文件块存放的节点列表

④ NameNode经过筛选之后将合适的节点信息返回客户端

⑤ 客户端创建数据管道流与DataNode建立通信，建立通信之后

⑥ 应答成功

⑦ 数据传输，以Packet（chunk512byte + chunksum4byte）为单位（64k）进行数据传输，发送时有一个ack packet队列，存放了传输的packet，当应答成功时，将packet从队列里移除




#### 5.4.2 网络拓扑-节点距离计算

​	在HDFS写数据过程中，NameNode会选择距离待上传数据距离最近的DataNode接收数据。<font color=red>**节点距离：两个节点到达最近的共同祖先的距离总和。**</font>

如下图: 
![节点距离计算](../Hadoop/img/节点距离.png)

Distance(r2-3，r2-3) = 0 (在同一节点上)

Distance（r3-1，r3-2）= 2 （在同一机架的不同节点）

Distance（r1-0，r2-0）= 4 （同一机房或数据中心不同机架上）

Distance（r1-1，r4-0） = 6 （不同机房或数据中心的节点）



#### 5.4.3 机架感知（副本存储节点选择）

源码位置：**BlockPlacementPolicyDefault**类中的**chooseTargetOrder**方法中。

![副本节点选择](../Hadoop/img/副本节点选择.png)

<font color=red>**① 第一个副本在Client所处节点上。**</font>

<font color=red>**② 第二个副本在另一个机架上随机一个节点。(保证数据的可靠性)**</font>

<font color=red>**③ 第三个副本在第二个副本所在机架的随机节点（传输效率高）**</font>



#### 5.4.2 HDFS读数据流程

![HDFS读取文件](../Hadoop/img/HDFS读取文件.png)

① 客户端通过DistributedFileSystem向NameNode请求下载文件

② NameNode通过查询元数据，找到文件块所在的DataNode地址。

③ 选择节点最近的一台DataNode，请求读取数据。

④ DataNode开始传输数据给客户端（从磁盘里面读取数据输入流，以Packet为单位来做校验）

⑤ 如果DataNode1负载达到量级之后，通过负载均衡可以从另一个节点NameNode读取数据。假如读取第二块时，DataNode1已经达到负载，那么从DataNode开始读取数据。

⑥ 客户端以Packet为单位接收，现在本地缓存，然后写入目标文件。（注意：读取数据是串行读取的，第一块读完之后读第二块进行追加）



### 5.5  NameNode和SecondaryNameNode工作机制

​	NameNode的功能是用来存储元数据的，那么元数据存储在哪里？内存还是磁盘？

​	假设元数据存储在NameNode节点的磁盘中，因为经常需要进行随机访问，还有响应客户端请求，效率就非常低。如果元数据存储在内存中，数据可靠性就不能得到保证，一旦服务宕机，元数据就丢失，会导致元数据丢失。<font color=red>**因此，就产生了FsImage，用FsImage在磁盘中备份元数据。**</font>

​	但是，这样又有一个新问题，当在内存中的元数据更新时，如果同时更新FsImage，（如果随机读写）就会导致效率过低，但是如果不更新，就会发生一致性问题，一旦NameNode节点断电，就会产生数据丢失。<font color=red>**因此，引入Edits文件（只进行追加，效率很高）。用来存储元数据操作，每当元数据有更新或者添加元数据时，修改内存中的元数据并最佳到Edits中。**</font>这样，一旦NameNode节点断电，可以通过FsIamge和Edits的合并，合成元数据。

​	但是，如果长时间添加数据到Edits中，会导致该文件数据过大，效率地下，而且一旦断电，恢复元数据需要的时间过长。因此，需要定期进行FsImage和Edits合并，如果这个操作由NameNode节点完成，又会效率过低。<font color=red>**因此，引入一个新得节点SecodaryNameNode，专门用于FsImage和Edits的合并。**</font>



#### 5.5.1  Fsimage和Edits概念

- **FsImage文件：** HDFS文件系统元数据的有一个<font color=red>**永久性的检查点**</font>，其中包含HDFS文件系统的所以有目录和文件inode的序列化信息。
- **Edits文件：**存放HDFS文件系统的所有更新操作和路径，文件系统客户端执行的所以也有写操作首先会被记录到Edits文件中
- **seen_txid：**除了以上两个文件之外，该文件也在$HADOOP_HOME/data/tmp/dfs/name/current目录下，seen_txid保存的是一个数字，就是最后一个edits_对应的数字




#### 5.5.2  工作机制

![NN&2NN工作机制](../Hadoop/img/NN和2NN工作机制.png)

- **第一阶段：NameNode启动**

  ① 第一次启动NameNode格式化后，创建Fsimage和Edits文件。如果不是第一次启动，直接加载镜像文件Fsimage和编辑日志到内存。

  ② 客户端对元数据进行增删改的请求

  ③ NameNode 记录操作日志，更新滚动日志

  ④ NameNode 在内存中对元数据进行增删改

- **第二阶段：SecondaryNameNode工作**

  ① SecondaryNameNode询问NameNode是否需要CheckPoint。直接带回NameNode是否检查结果

  ② SecondaryNameNode请求执行CheckPoint。

  ③ NameNode滚动正在写的Edits文件。

  ④ 将滚动前的编辑日志和镜像文件拷贝到SecondaryNameNode

  ⑤ SecondaryNameNode加载编辑日志和镜像文件到内存，并合并

  ⑥ 生成新的镜像文件fsimage.chkpoint

  ⑦ 拷贝fsimage.chkpoint到NameNode

  ⑧ NameNode将fsimage.chkpoint重命名为fsimage，替换原有的fsimage

  

### 5.6  DataNode工作机制

​	一个数据块（Block）在DataNode上以文件形式存储在磁盘上，包括两个文件，一个是数据本身，一个是元数据包括数据块的长度、块数据、校验和以及时间戳。

![DataNode工作机制](../Hadoop/img/DataNode工作机制.jpg)

① DataNode启动后会向NameNode注册，告诉NameNode，自身节点上有哪些活着的块信息。

② NameNode接收到DataNode的注册信息后，会把对应的信息记录在元数据里面。然后返回注册成功。

③ 注册通过后，周期性（6小时）向NameNode上报所有的块信息。（以确保块信息的状态是否完好等）

​	这里涉及到两个参数：

```xml
<!-- DataNode 向NameNode汇报当前节点信息的时间间隔，默认6小时 -->
<property>
	<name>dfs.blockreport.intervalMsec</name>
    <value>21600000</value>
</property>
<!-- DataNode 扫描自己节点块信息列表的时间,自查块信息是否损坏等，默认6小时 -->
<property>
    <name>dfs.datanode.directoryscan.interval</name>
    <value>21600s</value>
</property>
```

④ DataNode会和nameNode保持心跳，每3秒一次，心跳返回结果带有NameNode给该DataNode的命令（如复制块数据到另一台节点或删除某数据块）。

⑤ 如果超过10分钟+30秒（<font color=red>2\*dfs.namenode.heartbeat.recheck-interval + 10\*dfs.heartbeat.interval</font>）没有收到某个节点的心跳，则任务该节点不可用。




## 六.  MapReduce

### 6.1  MapReduce概述

#### 6.1.1  定义

​	MapReduce是有一个<font color=red>**分布式运算程序**</font>的编程框架，是用户开发“基于Hadoop的数据分析应用“的核心框架

​	MapReduce核心功能是将与用户编写的业务逻辑代码和自带默认组件整合成一个完整的分布式运算程序，并发运行在一个Hadoop集群上。



#### 6.1.2  优缺点

- **优点：**

  - **易于编程**： 用户只关系业务逻辑。实现框架的接口。

    它简单的实现一些接口，就可以已完成一个分布式程序，这个分布式程序可以分布到大量廉价的PC机器上运行。

  - **良好的扩展性**：可以动态增加服务器，解决计算资源不足问题

  - **高容错性**：任何一台机器挂掉，可以以将任务转移到其他节点

    MapReduce设计的初衷就是使程序能够部署在廉价的PC机器上，这就要求它具有很高的容错性。比如其中一台机器挂了，它可以把上面的计算任务转移到另一个节点上运行，不至于这个任务运行失败，而且这个过程不需要人工参与，而完全是由Hadoop内部完成的。

  - **适合海量数据的离线处理（PB级以上）**：可以实现几千台服务器共同并发工作，提供数据处理能力。

- **缺点：**

  - **不擅长实时计算**：

    MapReduce无法像Mysql一样一样，在毫秒或者秒级别内返回结果。

  - **不擅长流式计算：** 

    流式计算的输入数据是动态的，而MapReduce的输入数据是静态的，不能动态变化。这是也因为MapReduce自身的设计特点决定了数据源必须是静态的。

  - **不擅长DAG（有向无环图）计算：**

    多个应用程序存在依赖关系，后一个应用程序的输入为前一个应用的输出。在这种情况下，MapReduce并不是不能做，而是使用后，<font color=red>**每个MapReduce作业的输出结果会写入到磁盘，会照成大量的磁盘IO，导致性能非常低。**</font>
    
    

#### 6.1.3  MapReduce核心思想

![MapReduce核心思想](../Hadoop/img/MapReduce.png)

​	MapReduce核心思想<font color=red>**分而治之，先分再合。移动计算到数据端**</font>一个Map/Reduce *作业（job）* 通常会把输入的数据集切分为若干独立的数据块，由 *map任务（task）*以完全并行的方式处理它们。框架会对map的输出先进行排序， 然后把结果输入给*reduce任务*。通常作业的输入和输出都会被存储在文件系统中。 整个框架负责任务的调度和监控，以及重新执行已经失败的任务。即将一个大的、复杂的工作或任务，拆分成多个小的任务，并行处理，最终进行合并。适用于大量复杂的、时效性不高的任务处理场景（大规模离线数据处理场景）。

​	① 分布式的运算程序往往需要分成至少两个阶段

​	① 第一阶段Map负责分，将复杂的任务分解成若干小的任务来处理，MapTask并发实例完全并行处理，互不干扰。

​	② 第二阶段Reduce阶段负责合，对Map阶段的结果进行全局汇总。ReduceTask也是并发处理互不干扰，它的数据依赖于上一个阶段的运行结果。

​	③ MapReduce编程模型只能包含一个Map阶段合一个Reduce阶段，如果用户业务逻辑非常复杂，那就只能多个MapReduce串行运行。



#### 6.1.4  MapReduce 进程

​	一个完整的MapReduce程序再分布式运行时有三类实例进程：

​	① **MrAppMaster**：负责整个程序的过程调度及状态协调（ApplicationMaster的子类）

​	② **MapTask**：负责Map阶段的整个数据处理流程（查看进程的时候，叫YarnChild）

​	③ **ReduceTask**：负责Reduce阶段的整个数据处理流程。（查看进程的时候，叫YarnChild）



### 6.2  序列化

### 6.3  核心框架原理（重点）

![MapReduce](../Hadoop/img/mr.png)

#### 6.3.1  InputFormat 数据输入

##### 6.3.1.1  切片与MapTask并行度决定机制

  MapTask的并行度决定Map阶段的任务处理并发度，进而应先到整个Job的处理速度。1G的数据，启动8个MapTask，可以提高集群的并发处理能力。那么1k的数据，也启动8个MapTask，并不会提高集群的性能，开启每个任务的时间都比计算的时间要长，开启map任务需要一些准备工作，比如内存初始化。

![map](../Hadoop/img/map.png)

**MapTask并行度决定机制：**

-  **数据块：**Block是HDFS物理上把数据分成 一块一块。数据块是HDFS存储数据单位。
-  **数据切片：**数据切片只是在逻辑上对输入进行分片，并不会在磁盘上将其切分成片进行存储。<font color=red>**数据切片是MapReduce程序计算输入数据的单位**</font>，一个切片会对应启动一个MapTask

假设切片大小设置为100m时，

![切片与task](../Hadoop/img/切片与task.png)

第一个MapTask的数据的Node1，从0~100m，第二个MapTask从100~200m，第三个MapTask从200~300m，此时数据在两台节点上，因此需要跨服务器通信，效率就比较低了。

假设切片大小设置为128m。

![切片与task](../Hadoop/img/切片与task1.png)

每个切片的数据都在本地，每个任务直接从本地获取数据，效率更高。



<font color=red>**① 一个Job的Map阶段并行度由客户端在提交Job时的切片数决定**</font>

② 每一个split切片分配MapTask并行实例处理

③ 默认情况下，切片大小=blockSize

④ 切片时不考虑数据集整体，而是逐个针对每一个文件单独切片。

![切片与task](../Hadoop/img/切片与task2.png)



##### <font color=red>6.3.1.2  Job 提交流程源码与切片源码详解</font>

**核心提交流程源码：**

```java
// 入口
waitForCompletion();
// 提交
	submit();
		// 确保状态正确与新旧api适配 (非重点)
		ensureState(JobState.DEFINE);
		setUseNewApi();

        // 1. 建立连接
        connect();
			// 创建提交Job的代理
			new Cluster(getConfiguration());
				// 1) 判断时本地运行环境还是yarn集群运行环境
				initialize(jobTrackAddre,conf);

	    // 2. 提交Job
	    // 检查job输出路径 (非重点)
	    checkSpecs(job);
	    // 提交过程 (重点！！！！)
	    submitter.submitJobInternal(Job.this,cluster);
		    // 1) 创建给集群提交数据的Stag路径(临时路径) 例： /tmp/hadoop/mapred/staging/o12xxx/.staging
		    Path jobStagingArea = JobSubmissionFiles.getStagingDir(cluster,conf);
		    // 2) 获取jobid，并创建job路径 /tmp/hadoop/mapred/staging/o12xxx/.staging + /job_jobid
		    JobId jobid = submitClient.getNewJobId();
		    job.setJobId(jobId);
		    Path submitJobDir = new Path(jobStagingArea,jobId.toString());
		    // 3) 拷贝jar包到集群 (本地模式不拷贝jar包)
		    copyAndConfigureFiles(job,submitJobDir);
		    rUploader.uploadResources(job,jobSubmitDir);
		    // 4) 计算分片，生成切片规划文件,并设置maps个数
		    int keyLen = writeSplits(job,submitJobDir);
			    maps = writeNewSplits(job,submitJobDir);
			    input.getSplits(job);
            conf.setInt("mapreduce.job.maps", keyLen);
		    // 5) 向Stag路径写入xml配置文件
			writeConf(conf,submitJobFile);
				conf.writeXml(out);
		    // 6) 提交job，返回状态信息
		    status = submitClient.submitJob(jobId,submitJobDir.toString(),job.getCredentials());
```

![提交流程源码](../Hadoop/img/提交流程源码图解.jpg)



**FileInputFormat 切片源码解析（input.getSplits(job))**

```java
writeSplits(job, submitJobDir);
	writeNewSplits(job,jobSubmitDir);
		input.getSplits(job);
			// (1和mapreduce.input.fileinputformat.split.minsize的最小值
	        long minSize = Math.max(this.getFormatMinSplitSize(), getMinSplitSize(job));
			// mapreduce.input.fileinputformat.split.maxsize。没设置默认long数值最大值
	        long maxSize = getMaxSplitSize(job);

            // 1) 先找到数据存储的目录
            this.listStatus(job);
            Iterator i$ = files.iterator();
            // 2) 遍历处理(规划切片)目录下的每一个文件
            while(i$.hasNext()) {
                // a. 获取文件大小
                long length = file.getLen();
                // b. 计算切片大小，默认情况下切片大小为blocksize
                this.computeSplitSize(blockSize, minSize, maxSize);
                	Math.max(minSize, Math.min(maxSize, blockSize));
                // c. 开始切片，每次切片都要判断剩下的部分是否大于block的1.1倍，不大于1.1倍就划到一块切片
                for(bytesRemaining = length; (double)bytesRemaining / (double)splitSize > 1.1D; bytesRemaining -= splitSize) {
                                // d. 将切片信息写到一个切片规划文件中
                                // e. 整个切片的核心过程都在getSplit()方法中完成
                                // g. InputSplit只记录了切片的元数据信息，比如起始位置、长度以及所在节点列表等
                                blkIndex = this.getBlockIndex(blkLocations, length - bytesRemaining);
                                splits.add(this.makeSplit(path, length - bytesRemaining, splitSize, blkLocations[blkIndex].getHosts(), blkLocations[blkIndex].getCachedHosts()));
                            }
            }
			// 3.提交切片规划文件到Yarn上，yarn上的MAppMaster就可以根据切片规划文件计算开启MapTask个数。
```

<font color=red>**强调：**</font>

<font color=red>	**① 默认情况下，切片大小=blockSize**</font>

<font color=red>	**② 每次切片时，都要判断切完剩下的部分是否大于块的1.1倍。不大于时，直接划分为一块。**</font>

<font color=red>	**③ InputSplit只记录切片的元数据信息，比如其实位置、长度及所在节点等。**</font>

<font color=red>	**④ 提交切片规划文件到Yarn上，Yarn上的MrAppMaster根据切片规划文件计算开启MapTask任务。几个切片就几个任务。**</font>



**FileInputFormat切片大小的参数配置：**

```java
Math.max(minSize,Math.min(maxSize,blockSize));

// minSize -> mapreduce.input.fileinputformat.split.minsize = 1 默认值1
// maxSize -> mapreduce.input.fileinputformat.split.maxsize = Long.MAXValue 默认值Long.MAXValue
// 默认情况下，切片为blockSize

// 调小切片：设置maxSize比blockSize小
// 调大切片：设置minSize比blockSize大
```



**获取切片信息API**

// 获取切片的文件名称

String name = inputSplit.getPath().getName();

// 根据文件类型获取切片信息

FileSplit inputSplit = (FileSplit) context.getInputSplit();



##### 6.3.1.3  TextInputFormat 

##### 6.3.1.4  CombineTextInputFormat切片机制（小文件切片处理）

​	框架默认的TextInputFormat切片机制是对任务按文件规划切片，<font color=red>**不管文件多小，都会是一个单独的切片，都会交给一个MapTask，如果有大量小文件，就会产生大量的MapTask，处理效率极其低下。**</font>

① 应用场景

​	combineTextInputFormat用于小文件过多的场景，它可以将多个小文件从逻辑上规划到一个切片中，这样，多个小文件就可以交给一个MapTask处理。

② 虚拟存储切片最大值设置

​	CombineTextInputFormat.setMaxInputSplitSize(job，419430)；// 4m

​	注意：虚拟存储切片最大值设置最好根据实际的小文件大小情况来设置具体的值。

③ 切片机制

​	生成切片过程包括：虚拟存储过程和切片过程两部分。

（1）虚拟存储过程：

将输入目录下所有文件大小，依次和设置的setMaxInputSplitSize值比较，如果不大于设置的最大值，逻辑上划分一个块。如果输入文件大于设置的最大值且大于两倍，那么以最大值切割一块；当剩余数据大小超过设置的最大值且不大于最大值2倍，此时将文件均分成2个虚拟存储块（防止出现太小切片）。

例如setMaxInputSplitSize值为4M，输入文件大小为8.02M，则先逻辑上分成一个4M。剩余的大小为4.02M，如果按照4M逻辑划分，就会出现0.02M的小的虚拟存储文件，所以将剩余的4.02M文件切分成（2.01M和2.01M）两个文件。

（2）切片过程：

（a）判断虚拟存储的文件大小是否大于setMaxInputSplitSize值，大于等于则单独形成一个切片。

（b）如果不大于则跟下一个虚拟存储文件进行合并，共同形成一个切片。

（c）**测试举例：有4个小文件大小分别为1.7M、5.1M、3.4M以及6.8M这四个小文件，则虚拟存储之后形成6个文件块，大小分别为：**

**1.7M，（2.55M、2.55M），3.4M以及（3.4M、3.4M）**

**最终会形成3个切片，大小分别为：**

**（1.7+2.55）M，（2.55+3.4）M，（3.4+3.4）M**



#### 6.3.2  MapReduce工作流程图解

![mr工作流程](../Hadoop/img/mapreduce工作流程1.png)

#### 6.3.3  Shuffle 机制

​	Map方法之后，Reduce方法之前的数据处理过程称之为Shuffle。

![shuffle机制](../Hadoop/img/shuffle机制.png)

① 数据先进入环形缓冲区（准确来讲先进入getPartition方法为数据打上分区标志），环形缓冲区默认100m，左侧存索引，右侧存数据，当存储到80%时，进行反向存储（从后往前在缓冲区写数据）。留时间给数据溢写到文件，从而达到高效运行。

② 在数据溢写之前，会对数据进行排序，排序手段是快排。<font color=red>对key的索引按照字典排序。</font>

③ 溢写回产生两个文件，一个是索引文件，一个是真正落地数据文件。溢写是可选对数据进行聚合combiner。

④ 对溢写的文件进行归并和排序，然后还可以对数据进行压缩，最终数据写入磁盘，等待reduce拉去数据

⑤ reduce拉去到数据后先尝试放到内存里，如果内存不够，溢写到磁盘

⑥ 对数据进行归并、排序、分组，之后进入reduce方法。



#### 6.3.4  OutputFormat 数据输出（todo）

#### 6.3.5  MapReduce内核源码解析

##### 6.3.5.1  MapTask工作机制

​	MapTask主要分为五个阶段

![shuffle机制](../Hadoop/img/shuffle机制.png)

![map阶段工作机制](../Hadoop/img/Map阶段工作机制.png)

​	① Read阶段：MapTask通过InputFormat获得的RecordReader，从输入InputSplit中解析出一个个 K/V。

​	② Map阶段：该阶段主要是讲解析出来的K/V交给用户编写的map()函数处理，并产生一系列新的K/V。

​	③ Collect收集阶段：在用户编写map()函数中，当数据处理完成后，一般会调用OutputCollector。collect()输出结果。在该函数内部，它会将生成的K/V<font color=red>**分区**</font>，并写入一个环形内存缓冲区中。

​	④ 溢写阶段：当环形缓冲区满后，MapReduce会将数据写到本地磁盘上，生成一个临时文件。需要注意的是，将数据写入本地磁盘之前，先要对数据进行一次本地排序，并在必要时对数据进行合并、压缩操作。

​		1）利用快速排序算法对缓冲区内的数据进行排序，排序方式是，先按照分区编号进行排序，然后按照Key进行排序。这样，经过排序后，数据以分区为单位聚集在一起，且同一分区内所有的数据按照key有序。

​		2）按照分区编号由小到大依次将每个分区中的数据写入任务工作目录下的临时文件output/spillN.out（N表示当前溢写次数）中。如果用户设置了combiner，则写入文件之前，对每个分区中的数据进行一次聚集操作。

​		3）将分区数据的元信息写到内存索引数据结构SpillRecord中，其中每个分区的元信息包括在临时文件中的偏移量、压缩前数据大小和压缩后的数据大小。如果当前内存索引大小超过1m，则将内存索引写到文件output/spillN.out.index中。

​	⑤ merge阶段：当所有数据处理完成后，MapTask对所有临时文件进行一次合并，生成一个大文件，并保存到文件output/file.out中，同时生成相应的索引文件output/file.out.index。在文件合并过程中，MapTask以分区为单位进行合并。对于某个分区，它将采用多轮递归合并的方式。每轮合并mapreduce.task.io.sort.factor(默认10)个文件，并将产生的文件重新加入带合并列表中，对文件排序后，重复以上过程，直到最终只有一个大文件。让每个MapTask最终只生成一个数据文件，可避免同时打开大量文件和同时读取大量小文件产生的随机读取带来的开销。



##### 6.3.5.2  ReduceTask工作机制

![reduce工作机制](../Hadoop/img/reduce阶段工作机制.png)

​	①  Copy阶段：ReduceTask从各个MapTask上拷贝一片数据，并针对某一片数据，如果其大小超过一定阈值，则写到磁盘上，否则直接放到内存中。

​	② Sort阶段：在远程拷贝数据的同时，ReduceTask启动了两个后台线程对内存和磁盘上的文件进行合并，以防止内存使用过多或磁盘上文件过多。按照MapReduce语义，用户编写reduce()函数输入数据是按key进行聚集的一组数据。为了将key相同的数据聚在一起，Hadoop采用了基于排序的策略。由各个MapTask已经实现对自己的处理结果进行了局部排序，因此，ReduceTask只需对所有数据进行一次归并排序即可。

​	③ Reduce阶段：reduce()函数将计算结果写到HDFS上。



##### 6.3.5.3 ReuceTask并行度决定机制

​	MapTask并行度由切片个数决定，切片个数由输入文件和切片规则决定。ReduceTask的并行度同样影响整个Job的执行并发度和执行效率，但与MapTask的并发数由切片数决定不同，ReduceTask数量的决定是可以直接手动设置。

```java
// 默认值是1，手动设置为4
job.setNumReduceTasks(4);
```

<font color=red>**注意：**</font>

​	1）ReduceTask=0，表示没有Reduce阶段，输出文件个数和Map个数一致

​	2）ReduceTask默认为1，所以输出文件个数为1个。

​	3）如果数据分布不均匀，就有可能在Reduce阶段产生数据倾斜

​	4）具体多少个ReduceTask，需要根据集群性能而定

​	5）如果分区数不是1，但是ReduceTask为1，不执行分区过程。因为在MapTask的源码中，执行分区的前提是先判断ReduceNum个数是否大于1。不大于1，不执行分区。



##### 6.3.5.4 MapTask源码解析

```java
// 自定义的map方法的写入
context.write(k,NullWriable.get());
	output.write(key,value);
		// maptask 727行收集方法，进入两次
		collector.collect(key,value,partitioner.getPartition(key,value,partitions));
			// 默认分区器
			HashPartitioner();
		collect();
		// MapTask map端所有的kv全部写出后会走下面的close方法
			close;
				// 溢出刷盘
				collector.flush();
					// 溢写排序
					sortAndSpill();
						// QuickSort 快排
						sorter.sort();
					// 合并文件
					mergeParts();
				// 收集器关闭，即将进入reduceTask
				collector.close();
```



##### 6.3.5.5 ReduceTask源码解析

```java
if(isMapOrReduce())
initialize(job,getJobId(),reporter,useNewApi);
shuffleConsumerPlugin.init(shuffleContext);
	totalMaps = job.getNumMapTasks();
	// 合并方法
	merge = createMergeManager(context);
		// 内存合并
		this.inMemoryMerger = createInMemoryMerger();
		// 磁盘合并
		this.onDiskMerger = new OnDiskMerger(this);
rIter = shuffleConsumerPlugin.run();
	// 开始抓取数据
	eventFecher.start();
	// 抓取结束
	eventFetcher.shutDown();
	// copy阶段完成
	copyPhase.complete();
	// 开始排序阶段
	taskStatus.setPhase(TaskStatus.Phase.SORT)
// 排序阶段完成
sortPhase.complete();
// 自定义的reduce
reduce();
	// reduce完成之前会最后调用一次cleanup方法
	cleanup(context);
```




#### 6.3.6  Join 应用

#### 6.3.7  数据清洗ETL

#### 6.3.8  开发总结

### 6.4  数据压缩

#### 6.4.1  概述

1）压缩的好处和坏处

​	优点：以减少磁盘IO、减少磁盘存储空间

​	缺点：增加CPU开销

2）压缩原则

​	① 运算密集型的Job，少用压缩

​	② IO密集型的Job，多用压缩



## 七.  Yarn

### 7.1  Yarn资源调度器

​	Yarn是一个资源调度平台，<font color=red>负责为运算程序提供服务器运算资源，相当于一个分布式的操作系统平台</font>，而<font color=red>MapReduce</font>等运算程序则相当于运行于<font color=red>操作系统之上的应用程序。</font>

#### 7.1.1  Yarn基础架构

​	Yarn主要由ResourceManager、NodeManager、ApplicationMaster和Container等组件构成。

![yarn基础架构](../Hadoop/img/Yarn基础架构.png)

**1）ResourceManager（RM）：**

​	① 处理客户端请求

​	② 监控NodeManager

​	③ 启动或监控ApplicationMaster

​	④ 资源的分配和调度

**2）Node Manager（NM）：**

​	① 管理单节点上的资源

​	② 处理来自ResourceManager的命令

​	③ 处理来自ApplicationMaster的命令

**3）ApplicationMaster（AM）：**

​	① 为应用程序申请资源分配给内部的任务

​	② 任务的监控与容错

**4）Container：**

​	Container是Yarn中的资源抽象，它封装了某个节点上多维度资源，<font color=red>如内存、CPU、磁盘、网络等。</font>



#### <font color=red>7.1.2  Yarn工作机制（重点）</font>

![yarn工作机制](../Hadoop/img/yarn工作机制.png)

​	1）MR程序提交到客户端所在节点，YarnRunner向ResourceManager申请一个Application

​	2）ResourceManager将该应用程序的资源路径返回给YarnRunnner

​	3）该程序将运行所需要的资源提交到HDFS上

​	4）资源提交完成后，向Resource Manager申请运行mrAppMaster

​	5）ResourceManager将用户请求初始化成一个Task，放到缓存队列中

​	6）其中一个NodeManager领取到Task任务。

​	7）该NodeManager创建容器Container，并生成MRAppMaster

​	8）Container从HDFS上拷贝资源到本地。

​	9）MRAppMaster向ResourceManager申请运行MapTask资源

​	10）ResourceManager将运行MapTask分配给其他的NodeManager，NodeManager领取任务并创建容器。

​	11）MRAppMaster向接收到任务的NodeManager发送程序启动脚本，NodeManager启动MapTask处理数据

​	12）MRAppMaster等待MapTask任务执行完成后，向ResourceManager申请容器，运行ReduceTask

​	13）ReduceTask向MapTask获取相应分区的数据

​	14）程序运行完成后，MRAppMaster会向ResourceManager申请注销资源



#### 7.1.3  Job提交全过程（todo）

#### 7.1.4  Yarn调度器和调度算法

​	目前，Hadoop作业调度器主要有三种：FIFO、容量（CapacityScheduler）和公平（FairScheduler）。Apache Hadoop3.1.3默认的资源调度器是CapacityScheduler。CDH框架默认调度器是Fair Scheduler。

##### 7.1.4.1  先进先出调度器（FIFO）

​	FIFO调度器（First In First Out）：单队列，根据提交作业的先后顺序，先来先服务。

![FIFO](../Haddoop/../Hadoop/img/FIFO调度.png)

​	优点：简单易懂

​	缺点：不支持多队列，生产环境很少使用



##### 7.1.4.2  容量调度器

​	容量调度器是Yahoo开发的多用户调度器。

![容量调度器](../Haddoop/../Hadoop/img/容量调度器.png)

**特点：**

​	1）多队列：每个队列可配置一定的资源量，如上队列A占20%，队列B占50%，队列C占30%。队列C中两个用户ss和cl各占队列C50%，每个队列采用FIFO调度策略。优先满足先进来的任务，如果队列A资源有10G，那么job11占2个G，如果下一个任务占6G，那么第二个任务也可以同时执行，在一个队列中可同时启动多个任务。

​	2）容量保证：管理员可为每个队列设置资源最低保证和资源使用上线。

​	3）灵活性：如果一个队列中的资源有剩余，可以暂时共享给那些需要资源的队列，而一旦该队列有新的应用程序提交，则其他队列借调的资源会归还给该队列。

​	4）多租户：支持多用户共享集群和多应用程序同时运行。为了防止同一个用户的作业独占队列中的资源，该调度器会对<font color=red>**同一用户提交的作业所占资源量进行限定。**</font>



**调度算法：**

![调度算法](../Hadoop/img/容量调度算法.png)

**1）队列资源分配**

​	从root开始，使用深度优先算法，<font color=red>**优先选择资源占用率最低**</font>的队列分配资源

**2）作业资源分配**

​	默认按照提交作业的<font color=red>**优先级和提交时间**</font>顺序分配资源。

**3）容器资源分配**

​	按照容器的<font color=red>**优先级**</font>分配资源；

​	如果优先级相同，按照<font color=red>**数据本地行原则（节点距离最近）：**</font>

​	① 任务和数据在同一节点

​	② 任务和数据在同一机架

​	③ 任务和数据不在同一节点也不在同一机架




##### 7.1.4.3  公平调度器

​	Fair Scheduler是FaceBook开发的多用户调度器。

![公平调度器](../Haddoop/../Hadoop/img/公平调度器.png)

**1）与容量调度器相同点**

​	① 多队列：支持多队列多作业

​	② 容量保证：管理员可为每个队列设置资源最低保证和资源使用上限

​	③ 灵活性：如果一个队列中的资源有剩余，可以暂时共享给那些需要资源的队列，而一旦该队列有新的应用程序提交，则其他队列借调的资源会归还给该队列。

​	④ 多租户：支持多用户共享集群和多应用程序同时运行该；为了防止同一个用户的作业独占队列中的资源，该调度器会对同一用户提交的作业所占资源量进行限定。

**2）与容量调度器不同点**

​	① 核心调度策略不同

​		容量调度器：优先选择<font color=red>**资源利用率**</font>低的队列

​		公平调度器：优先选择对资源的<font color=red>**缺额**</font>比较大的

​		公平调度器设计目标是：在时间尺度0度器会优先为缺额大的作业分配资源

  ![缺额](../Haddoop/../Hadoop/img/缺额.png)

​	② 每个队列可以单独设置资源分配方式

​		容量调度器：FIFO、DRF

​		公平调度器：FIFO、DRF、FAIR

​	

<font color=red>**公平调度器队列资源分配方式：**</font>

1）FIFO策略

​	公平调度器每个队列资源分配策略如果选择FIFO的话，此时公平调度器相当于上面的容量调度器。

2）Fair 策略

​	Fair策略（默认）是一种基于最大最小公平算法实现的资源多路复用方式，默认情况下，每个队列内部采用该方式分配资源。这意味着，如果一个队列中有两个应用程序同时运行，则每个应用程序可得到1/2资源。如果三个应用程序同时运行，则每个应用程序可得到1/3的资源。

具体资源分配流程和容量调度器一致：① 选择队列，② 选择作业，③ 选择容器，这三步，每一步都是按照公平策略分配资源。

![资源分配](../Hadoop/img/公平调度资源分配.png)

![资源分配算法](../Hadoop/img/公平调度资源分配算法.png)

#### 7.1.5  Yarn常用命令

#### 7.1.6  Yarn生产环境核心参数
