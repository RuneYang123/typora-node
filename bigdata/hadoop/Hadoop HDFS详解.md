### HDFS详解

#### 一、HDFS概述

在Hadoop分布式系统的框架中，首要的存储数据的功能是由HDFS这个分布式文件系统完成的。如果把Hadoop框架比喻成一个工厂，那么HDFS就像是整个工厂的仓库。

#### 二、HDFS优缺点

优点：高容错性，能够将失败的任务重新分配，适合大数据开发，可以构建在廉价机器上。

缺点：不适合低延时时间数据访问，无法对大量小文件进行储存，小文件寻址时间会超过读取时间，不支持并发写入，文件随意修改，仅支持追加。

#### 三、HDFS架构

![preview](https://pic4.zhimg.com/v2-36520d5fe03b5f4e8907625a839874fb_r.jpg)

HDFS是一个分布式的储存组件，是由主从体系结构。主要由namenode（简写NN），datanode（简写DN），secondarynamenode（简写2NN）三部分组成。

##### 1、namenode

负责管理文件系统的元数据，以及每个文件的元数据和映射关系。配置数据的副本策略，处理客户端的读写请求。

##### 2、datanode

DN是实际储存数据的空间，执行客户端的读写请求。根据NN的副本机制来储存。

##### 3、secondarynamenode

用来监控HDFS状态的辅助后台程序，每隔一段时间获取HDFS的元数据快照，辅助namenade管理元数据信息。但是当NN挂掉，不能马上代替NN的服务。

#### 四、HDFS的安全模式

安全模式是hadoop的一种保护机制，用于保证集群中的数据快的安全性。当集群启动之后，会首先进入安全模式。当系统处于安全模式时会检查数据块的完整性。此时只接受读数据请求，不接受删除，修改等变更请求。而且安全模式下，hdfs会自动的核查副本数，多删少复制。当整个系统达到安全标准时，HDFS自动离开安全模式。30s。

```shell
    hdfs  dfsadmin  -safemode  get #查看安全模式状态
    hdfs  dfsadmin  -safemode  enter #进入安全模式
    hdfs  dfsadmin  -safemode  leave #离开安全模式
```

#### 五、HDFS文件块机制

HDFS会将所有的文件全部抽象成为block块来进行储存。不管文件的大小都是以block块统一大小和形式进行储存，方便我们的分布式文件系统对文件的管理。默认块的大小为128M。（可通过 hdfs-site.xml当中文件进行指定）

```
<property>
    <name>dfs.block.size</name>
    <value>块大小 以字节为单位</value> //只写数值就可以
</property>
```

文件寻址时间为传输时间的1%则最佳。

#### 六、HDFS文件写过程

![preview](https://pic1.zhimg.com/v2-1905edb4b2cd63637d7e60ae22cb39a8_r.jpg)

1、Client 发起文件上传请求，通过 RPC 与 NameNode 建立通讯, NameNode 检查目标文件是否已存在，父目录是否存在，返回是否可以上传；

2、Client 请求第一个 block 该传输到哪些 DataNode 服务器上；

3、NameNode 根据配置文件中指定的备份数量及机架感知原理进行文件分配, 返回可用的 DataNode 的地址如：A, B, C；

4、Client 请求 3 台 DataNode 中的一台 A 上传数据（本质上是一个 RPC 调用，建立 pipeline ），A 收到请求会继续调用 B，然后 B 调用 C，将整个 pipeline 建立完成， 后逐级返回 client；

5、Client 请求 3 台 DataNode 中的一台 A 上传数据（本质上是一个 RPC 调用，建立 pipeline ），A 收到请求会继续调用 B，然后 B 调用 C，将整个 pipeline 建立完成， 后逐级返回 client；

6、Client 开始往 A 上传第一个 block（先从磁盘读取数据放到一个本地内存缓存），以 packet 为单位（默认64K），A 收到一个 packet 就会传给 B，B 传给 C。A 每传一个 packet 会放入一个应答队列等待应答；

7、数据被分割成一个个 packet 数据包在 pipeline 上依次传输，在 pipeline 反方向上， 逐个发送 ack（命令正确应答），最终由 pipeline 中第一个 DataNode 节点 A 将 pipelineack 发送给 Client；当一个 block 传输完成之后，Client 再次请求 NameNode 上传第二个 block，重复步骤 5；

#### 七、HDFS读文件过程

![img](https://pic2.zhimg.com/80/v2-218999adb1d95cba9629b1cfba66c401_720w.jpg)

1、Client向NameNode发起RPC请求，来确定请求文件block所在的位置；NameNode会视情况返回文件的部分或者全部block列表，对于每个block，NameNode 都会返回含有该 block 副本的 DataNode 地址； 

2、这些返回的 DN 地址，会按照集群拓扑结构得出 DataNode 与客户端的距离，然后进行排序，排序两个规则：网络拓扑结构中距离 Client 近的排靠前；心跳机制中超时汇报的 DN 状态为 STALE，这样的排靠后；

3、Client 选取排序靠前的 DataNode 来读取 block，如果客户端本身就是DataNode，那么将从本地直接获取数据(短路读取特性)；

4、底层上本质是建立 Socket Stream（FSDataInputStream），重复的调用父类 DataInputStream 的 read 方法，直到这个块上的数据读取完毕；

5、当读完列表的 block 后，若文件读取还没有结束，客户端会继续向NameNode 获取下一批的 block 列表；

6、读取完一个 block 都会进行 checksum 验证，如果读取 DataNode 时出现错误，客户端会通知 NameNode，然后再从下一个拥有该 block 副本的DataNode 继续读。

7、read 方法是并行的读取 block 信息，不是一块一块的读取；NameNode 只是返回Client请求包含块的DataNode地址，并不是返回请求块的数据；

8、最终读取来所有的 block 会合并成一个完整的最终文件。

#### 八、NameNode的工作机制

![img](https://pic4.zhimg.com/80/v2-b73149a783f6c06a93ed67dcef4fd1bf_720w.jpg)

##### 1、namenode工作机制

1、第一次启动namenode格式化后，创建fsimage和edits文件。如果不是第一次启动，直接加载编辑日志和镜像文件到内存。

2、客户端对元数据进行增删改的请求。

3、namenode记录操作日志，更新滚动日志。

4、namenode在内存中对数据进行增删改查。

##### 2、secondary namenode 工作机制

1、secondary namenode询问 namenode 是否需要 checkpoint。直接带回 namenode 是否检查结果。触发条件（定时时间到，Edits日志文件数据满了）。

2、secondary namenode 请求执行 checkpoint。

3、namenode 滚动正在写的edits日志。

4、将滚动前的编辑日志和镜像文件拷贝到 secondary namenode。

5、secondary namenode 加载编辑日志和镜像文件到内存，并合并。

6、生成新的镜像文件 fsimage.chkpoint。

7、拷贝 fsimage.chkpoint 到 namenode。

8、namenode将 fsimage.chkpoint 重新命名成fsimage。

##### 3、FSImage与edits详解

所有的元数据信息都保存在了，FSImage与edits文件中，这两个文件就记录了所有的数据元数据信息。

1、edits：客户端对hdfs进行写文件时，首先会被记录在edits中。edits修改时元数据也会修改。每次hdfs更新时edits先更新后客户端才会看到最新信息。

2、fsimage：是namenode中关于元数据的镜像，一般称为检查点，fsimage内容包含了namenode管理下的所有datanode中文件及文件block及block所在的datanode的元数据信息。随着edits内容增大，就需要在一定时间点和fsimage合并。

#### 九、datanode工作机制以及数据储存

##### 1、datanode工作机制

1、一个数据块在datanode上以文件形式存储在磁盘上，包括两个文件，一个是数据本身，一个是元数据包括数据块的长度，块数据的校验和，以及时间戳。

2、DataNode启动后向namenode注册，通过后，周期性（1小时）的向namenode上报所有的块信息。(dfs.blockreport.intervalMsec)。

3、心跳是每3秒一次，心跳返回结果带有namenode给该datanode的命令如复制块数据到另一台机器，或删除某个数据块。如果超过10分钟没有收到某个datanode的心跳，则认为该节点不可用。

4、集群运行中可以安全加入和退出一些机器。

##### 2、数据完整性

1、当DataNode读取block的时候，它会计算checksum。

2、如果计算后的checksum，与block创建时值不一样，说明block已经损坏。

3、client读取其他DataNode上的block。

4、datanode在其文件创建后周期验证checksum。



注本文属于个人笔记，仅供学习使用。

部分引用原文出处：

作者：五分钟学大数据
链接：https://zhuanlan.zhihu.com/p/424355304

