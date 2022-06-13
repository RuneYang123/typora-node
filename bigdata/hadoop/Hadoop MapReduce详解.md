### Hadoop MapReduce详解

#### 一、MapReduce简介

之前我们我们讲解了Hadoop的分布式文件储存系统HDFS，曾把它比作一个工厂的仓库。而今天我们要介绍的MapReduce（简称MR）分布式计算框架，就可以把他看作一个工厂的流水线。

##### 1、MR的编程思想

MR的核心的思想就是分而治之，通俗的来说，就是将复杂的事情分割成很多小的事情，一一去完成，最终合并结果。那么我们可以明白MR的过程实际就是输入，分，处理，合并，输出。

MR的过程分为两个大的阶段Map以及Reduce阶段。既然是分开治理，Map阶段包含许多的mapTask，他们完全并行互不相干。同理Reduce阶段包含很多ReduceTask，他们也完全并行互不相干，但是ReduceTask需要注意的是，他们的数据依赖于他的上一个节点mapTask的输出。

但是我们需要注意的是MapReduce编程模型只能包含一个map阶段和一个reduce计算，如果用户的业务逻辑非常复杂，那只能多个MapReduce程序，串行运行。

##### 2、MR的优缺点

优点：1、易于编程，MR框架有大量默认的代码，用户只需要关注业务逻辑。2、有良好的拓展性，可以动态的增加服务器。3、高容错性任何一台机器挂了，可转移给其他的节点。4、适合海量数据计算

缺点：1、不擅长实时计算。2、不擅长流式计算。3、不擅长DAG有向无环图。

#### 二、WordCount案例

![img](https://upload-images.jianshu.io/upload_images/3402387-3367b798a9c52923.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

上面就是学习大数据最经典的WordCount案例的MR的流程图。

##### 1、MR编程规范

MR的编程不仅仅是Map阶段和Reduce阶段，还需要diver阶段，相当于yarn客户端，用于提交我们整个程序到yarn集群，提交封装的mr程序相关运行参数的job对象。

##### 2、代码依赖

```xml
    <dependencies>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>3.1.3</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.30</version>
        </dependency>
    </dependencies>
```

##### 3、map阶段

```java
public class WCMapper extends Mapper<LongWritable, Text , Text , IntWritable> {
    private final Text outK = new Text();
    private final IntWritable outV = new IntWritable(1);
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        //获取一行
        String Line = value.toString();
        //split
        String[] Words = Line.split(" ");
        //循环写出
        for (String word : Words) {
            outK.set(word);
            context.write(outK, outV);
        }
    }
}

```



##### 4、reduce阶段

```java
public class WCReducer extends Reducer<Text, IntWritable,Text , IntWritable> {
    private final IntWritable outV = new IntWritable();
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        int sum = 0;

        for (IntWritable value : values) {
            sum += value.get();
        }
        outV.set(sum);
        context.write(key,outV);
    }
}
```

##### 5、diver阶段

```java
public class WCDriver {
    public static void main(String[] args) throws IOException, InterruptedException, ClassNotFoundException {
        //获取job
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);
        //jar包路径
        job.setJarByClass(WCDriver.class);
        //关联mapper和reducer
        job.setMapperClass(WCMapper.class);
        job.setReducerClass(WCReducer.class);
        //设置map输出的KV类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);
        //设置reducer输出的KV类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);
        //设置输入输出路径
        FileInputFormat.setInputPaths(job,new Path("E:\\input.txt"));
        FileOutputFormat.setOutputPath(job,new Path("E:\\output"));
        //提交job
        boolean result = job.waitForCompletion(true);
        System.exit(result ? 0 : 1);
    }
}
```

#### 
