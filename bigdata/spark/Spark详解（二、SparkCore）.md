### Spark详解（二、SparkCore）

SparkCore是Spark计算引擎的基础，后面的sparksql以及sparkstreaming等，都是基于SparkCore的。这里笔者就开始详细的介绍SparkCore。如果要介绍SparkCore，必须详细介绍一下RDD。

#### 一、RDD编程

RDD（Resilient Distributed Dataset）叫做分布式数据集，是Spark中最基本的数据抽象，它代表一个不可变、可分区、里面的元素可并行计算的集合。在 Spark 中，我们如果要对数据进行操作，不外乎就是创建RDD对数据进行操作。

##### 1、不可变（只读）

RDD数据集是不可变的，就是说，RDD一旦生成，里面的数据集就不会改变。那我们怎么对数据集进行操作？我们可以在原来的基础上，在生成新的RDD

##### 2、分区

Spark会生成很多个抽象意义上分区，这些分区是Spark的最小计算单位。

##### 3、lazy特性

Spark是lazy执行的，在使用转换算子的时候是不会运算的，只有在使用行动算子的时候才会执行。

#### 二、创建RDD

在内存中创建RDD，使用parallelize，和makeRDD直接将内存中的集合创建成RDD。

```scala
    val rdd1 = sc.parallelize(
      List(1, 2, 3, 4)
    )
    val rdd2 = sc.makeRDD(List(1, 2, 3, 4))
```

从底层来说makeRDD就是parallelize方法。这里笔者觉得makeRDD比parallelize更加直观才有这样的更新。

从外部储存系统中的集合创建RDD。使用textFile。

```scala
val rdd3 = sc.textFile("input/user_visit_action.txt")
```

#### 三、RDD转换算子

这里举几个比较常用的RDD转换算子。

##### 1、map

map可以说是rdd中最常用的一个算子了。他的作用是，将处理的数据逐条进行映射转换，这里的转换可以是类型以及值的转化。

```scala
val value = rdd1.map(_ + 1)//所有的加一
    val rdd1 = sc.parallelize(
      List((1,2), (3,4), (5,6), (7,8))
    )
    val value = rdd1.map(_._1)//只有每个第一个数
```

##### 2、flatMap

将所有的数据进行扁平化处理，map会把每一行分开。而flatMap会把所有的数据成为一行。

```scala
val value = rdd1.flatMap(_.split(" "))//split通过符号分隔
```



##### 3、mapPartitions

将待处理的数据以分区为单位发送到计算节点进行处理，这里的处理是指可以进行任意的处理，哪怕是过滤数据。就是在分区内操作。

```scala
    val value1 = value.mapPartitions(
      _.filter(_ == 2)
    )
```



##### 4、groupBy、groupByKey

将数据根据指定的规则进行分组, 分区默认不变，但是数据会被打乱重新组合，我们将这样的操作称之为 shuffle。groupBy可以根据自定义的规则分区，groupByKey就是通过Key来分区。

##### 5、reduceByKey

可以将数据按照相同的 Key 对 Value 进行聚合。

##### 6、filter

将数据根据指定的规则进行筛选过滤，符合规则的数据保留，不符合规则的数据丢弃。当数据进行筛选过滤后，分区不变，但是分区内的数据可能不均衡，生产环境下，可能会出现数据倾斜。

##### 7、distinct

去重，将数据集中重复的数据去重。

##### WordCount案例

```scala
val word = sc.textFile("input/Word.txt").flatMap(_.split(" ")).map((_, 1)).reduceByKey(_ + _)
```

#### 四、行动算子

##### 1、collect

在驱动程序中，以数组 Array 的形式返回数据集的所有元素

```scala
rdd.collect().foreach(println)
```

##### 2、foreach

分布式遍历 RDD 中的每一个元素，调用指定函数

```scala
rdd.collect().foreach(println)
```

##### 3、**save** 相关算子

将数据保存到不同格式的文件中

```scala
// 保存成 Text 文件
rdd.saveAsTextFile("output")
// 序列化成对象保存到文件
rdd.saveAsObjectFile("output1")
// 保存成 Sequencefile 文件
rdd.map((_,1)).saveAsSequenceFile("output2")
```

##### 4、count、countByKey

统计个数，count是统计元素的个数，countByKey是统计Key的个数。
