### Hadoop yarn详解

#### 一、YARN概述(Yet Another Resource Negotiantor)

时至今日Hadoop已经走过了许多年头，目前已经到达第三代，相比第二代框架得基本架构上没有变化。但是二代相比第一代，却有很大得变化，独立出来了资源管理框架。第一代将资源调度交给mapreduce中的jobtracher。

#### 二、YARN架构

![preview](https://pic1.zhimg.com/v2-5974feb77300f55f623b7f711b85e588_r.jpg)



yarn主要由ResourceManager，ApplicationsMaster，NodeManager，Container。下面我们一一介绍。

##### 1、ResourceManager

ResourceManager（简写RM），主要有处理客户端的请求，监控NodeManager，启动监控ApplicationMaster以及资源的调度和分配，他是全局的资源管理器。

它有两个组件组成Scheduler，以及Applications Manager。Scheduler只负责任务调度，分配资源给正在运行的应用程序。Applications Manager（注意与ApplicationMaster区分）主要负责管理系统中所有的应用程序，包括提交，启停等。

##### 2、NodeManager

NodeManager（简写NM），主要监控单个节点上的资源，处理来自ResourceManager以及ApplicationsMaster的命令。hadoop 2中，每个datanode都会运行一个NodeManager用来执行yarn的命令。

##### 3、ApplicationMaster

提交到yarn上的每一个应用都有一个专门的ApplicationMaster（简写AM）。为应用程序申请资源并分配给内部的任务。以及任务的监控与容错。

##### 4、Container

Container是yarn中的资源抽象，它封装了某个节点上的多维度资源，如内存，cpu，磁盘，网络等。

#### 三、YARN工作机制

1、客户端向ResourceManager提交任务。（以WordCount为例）

2、yarnrunner向RM申请一个Application，RM返回该WC的资源路径。

3、提交WC所需要的资源到HDFS上，申请运行WCAppMaster。

4、RM将WC进行拆分，拆分成Task。然后向NM下发Task任务。NM创建容器Container，并产生WCAppMaster。

5、Container从HDFS上拷贝WC的资源到本地。WCAppMaster向RM申请Task资源。RM将运行。Task任务分配给另外的NodeManager，另两外的NodeManager分别领取任务并创建容器。

6、程序运行完毕后，WC会向RM申请注销自己。

#### 四、YARN资源调度

##### 1、FIFO（先进先出调度器）

![img](https://img2018.cnblogs.com/blog/31857/201908/31857-20190801204346575-976593023.png)

先进先出的调度器，顾名思义，就是按照任务到达时间排序，先到先服务。将YARN上所有的资源全部调集过来，交给当前这个任务来运行，后续的任务由于没有资源，只能等这个任务完成。

它的优点是运行较大任务时可以得到更多的资源，实现和原理都十分简单。但是他的缺点也是显而易见的，任务的优先级就靠时间来排序，并发程度低。

##### 2、Capacity Scheduler （容量调度器）

目前Hadoop默认使用的调度策略。Capacity调度器允许多个组织共享集群的资源，每个组织都有集群的一部分计算能力。每个队列相对独立，每个队列都有一定的集群资源。而且每个队列内部还可以垂直划分，拥有多个用户。注意，在一个队列内，资源调度用的是FIFO策略。

它的优点是，支持了并行的运行多个MR。但是它的缺点是，如果有一个任务运用的资源大，就会占用其他队列的资源。而其他的队列如果有任务，资源不够。就需要去找占了它资源的队列要回资源。如果热奶没有完成，会一直占用资源，这样就造成了阻塞等待的现象。

##### 3、FairScheduler （公平调度器）

CDH版本的Hadoop默认的调度策略。公平调度器的设计目标是所有的应用分配公平的资源。

它是支持多队列的工作。假如有A，B两个队列，最多有一半资源，最少没有资源。这时候A来了一个任务，而B没有任务，那么A将会有全部资源。这时候B有了一个任务，那么A会把B的资源还回来。AB各一般的资源。假如A又来了一个任务。那么A的每个任务就又0.25的资源，B的任务还是0.5的资源。结果资源最终在两个用户之间公平的共享。

它的好处显而易见，动态分配资源，并行执行，也不会造成堵塞。但是相比FIFO调度和容器调度要复杂许多。