### MR流程详解

MR的流程可以分为输入，分片，map，shuffle，reduce，以及输出。这几个大的部分，下面我们一一讲解。

#### 一、整体流程

1、输入文件之后，分片之后的每个文件分配给每个MapTask。

2、MapTask处理文件，输出kv对，放入内存缓冲区。从内存缓冲区（默认100M）中不断溢出到本地磁盘文件（到达80%内存缓冲区就开始溢出）。

3、可能溢出多个文件，最终会合并称一个文件。在溢出以及合并溢出文件时，都要调用Partitioner进行分区和针对key进行排序。

4、ReduceTask会根据自己的分区去每个MapTask上去相应的结果。会抓取到一个分区上来自不同MapTask上的文件。ReduceTask再将这些文件进行合并（合并时进行归并排序）。最后输出处理好的文件。

#### 二、输入以及分片

输入有许多的常见类的接口，TextInputFormat、KeyValueTextInputFormat、NLineInputFormat、CombineTextInputFormat等等的实现接口类。只要是为了针对不同的数据类型。不过主要还是使用TextInputFormat，他也是输入的一个默认的实现类，按照行来读取文件。

默认的切片是按照文件块来划分，默认一个HDFS一个切片。不考虑数据集整体，针对每个文件单独切片，每个切片对应一个mapTask。

针对小文件的问题，我们有CombineTextInputFormat的机制。何为小文件问题呢？假如我们要处理一些文件，HDFS里文件时块储存的，无论多小的文件，都是一个单独的数据块。但是Hadoop上默认的是，无论多小都分配给一个MapTask。如果有很多的小文件，就会产生大量的MapTask。

CombineTextInputFormat的机制就是将小文件从逻辑上放入一个切片中，多个小文件就可以交给MapTask。

#### 三、Shuffle机制

Map过程之后，Reduce过程之前的数据处理称之为Shuffle。

MapTask会输出KV键值对，会进入内存缓冲区，在内存缓冲区将数据分区并进行排序。这时候溢出的文件在分区内都是有序的。

在将所有溢出文件合并成一个大的溢出文件时，应为之前已经是分区内有序的了，就用归并排序将整个溢出文件的分区内有序。

这时候每个MapTask经过Shuffle输出的文件就是分区，且分区内都有序的文件。注意这个有序是根据Key来排序的。

#### 四、partition分区和combiner合并

##### 1、partition分区

每个ReduceTask都会得到所有MapTask属于自己分区的数据。在Shuffle中进行分区以及排序。

默认的分区是根据Key的hashCode对ReduceTask取模得到的，用户是不能控制key储存到哪个分区。如果需要自定义类继承Partition，需要重写getPartition（）方法。

##### 2、Combiner合并

Combiner也是MR中的一个组件，但是他的父类就是Reducer。所以说，其实Combiner也是一种Reducer。

但是和Reducer的区别是运行的位置，Combiner是在每个MapTask所在的节点运行，Reduce是接受全局所有的MapTask输出结果。其实就是Combiner是对每个Map的输出进行局部的汇总，减少网络传播量。

注意，Combiner能够使用的前提是不能影响最终的业务逻辑。这是什么意思呢？比如说我们的业务是求和，那每个map的分区先求和，在reduce求和不受影响，那么我们能够使用Combiner。但如果是求平均值，就不行。例如mapA输出2，4，mapB输出6。应该的平均值是4。假如先Combiner，mapA输出3，mapB输出6，那求出平均值就是4.5。

另外Combiner输出的kv对要和reduce的输入kv对要对应起来。

