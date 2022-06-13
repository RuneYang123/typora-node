### Spark详解（三、SparkSQL）

#### 一、SparkSQL简介

SparkSession 是 SparkSQL的查询起始点，包括 DataFrame 和 DataSet 模型。用于方便对于结构化数据的分析操作，在实际生产环境中大部分情况下都是使用SparkSQL进行计算的。SparkSession实际上封装了SparkCore，所以实际的计算是通过SparkCore完成的。在spark-shell中，我们可以通过spark调用SparkSQL的。在idea中则需要新建对象。

```scala
val spark = SparkSession.builder().config(sparkConf).getOrCreate()
```

#### 二、DataFrame

DataFrame 是一种以 RDD 为基础的分布式数据集，类似于传统数据库中的二维表格。DataFrame所表示的二维表数据集的每一列都带有名称和类型。在 IDEA 中开发程序时，如果需要 RDD 与 DF 或者 DS 之间互相操作，那么需要引入**import spark.implicits._**

##### 1、创建DataFrame

```scala
val dataFrame = spark.read.json("input/user.json")//还支持table text textFile csv format jdbc json等
```

##### 2、SQL语法

SQL 语法风格是指我们查询数据的时候使用 SQL 语句来查询，这种风格的查询，必须要有临时视图或者全局视图来辅助。

```scala
dataFrame.createOrReplaceTempView("user")//创建临时表
val sqlDf = spark.sql("select * from user")
```

```scala
dataFrame.createGlobalTempView("user")//创建全局表
val sqlDf = spark.sql("select * from global_temp.user")
```

**普通临时表是 Session 范围内的，如果想应用范围内有效，可以使用全局临时表。使用全局临时表时需要全路径访问，global_temp.user。**

##### 3、DSL语法

这是DataFrame提供的一个特定的管理使用DataFrame的一个语法，不需要创建临时视图或者全局视图，就可以对数据进行操作。

```scala
dataFrame.select("username","age").show()// 直接查看数据
```

```scala
dataFrame.select($"username",$"age"+10).show()//对数据进行操作需要添加$符，或者
dataFrame.select('username,'age+10).show()
dataFrame.fiter(age > 30).show()
dataFrame.groupBy("age").count.show
```

需要注意的是，如果是在IDEA中，需要引入import spark.implicits._。

##### 4、RDD转换为DataFrame

实际开发中，我们会使用样例类将RDD转换为DataFrame。

```scala
case class user(
                 name:String,
                 age:Int
                 )
sc.makeRDD(List(("zhangsan",30), ("lisi",40))).map(t => user(t._1,t._2)).toDF.show
```

```scala
    var rdd = toDF.rdd
    rdd.collect().foreach(println)//DataFrame转化为RDD
```

DataFrame其实是对RDD的封装，所以可以直接获取内部的RDD。但是直接转化的RDD的储存的类型是ROW。

#### 三、DataSet

DataSet 是分布式数据集合。DataSet 是 Spark 1.6 中添加的一个新抽象，是 DataFrame的一个扩展。它提供了 RDD 的优势（强类型，使用强大的 lambda 函数的能力）以及 Spark SQL 优化执行引擎的优点。DataSet 也可以使用功能性的转换（操作 map，flatMap，filter等等）。所以说DataFrame可以做的大部分操作DataSet也是可以的。

##### 1、创建DataSet

```scala
DS = Seq(Person("zhangsan",2)).toDS()
```

在实际使用的时候，很少用到把序列转换成DataSet，更多的是通过RDD来得到DataSet。

```scala
    val toDs = sc.makeRDD(List(("zhangsan", 30), ("lisi", 40))).map(t => person(t._1,
      t._2)).toDS()
```

SparkSQL 能够自动将包含有 case 类的 RDD 转换成 DataSet，case 类定义了 table 的结构，case 类属性通过反射变成了表的列名。Case 类可以包含诸如 Seq 或者 Array 等复杂的结构。

##### 2、DataFrame **和** DataSet 转换

DataFrame 其实是 DataSet 的特例，所以它们之间是可以互相转换的。

所以从DataSet转化为DataFrame可以直接转化。

```scala
val frame = toDs.toDF()
```

DataFrame转换为DataSet，则需要带上样例类。

```scala
val ds = df.as[User]
```

