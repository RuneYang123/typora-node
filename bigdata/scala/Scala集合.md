# Scala数据集合

## 数据集合类型

scala的数据集合类型主要有数组(Array)、链表(List)、集合(Set)、映射(Map)、元组等。都分为可变以及不可变的集合，可变集合可以更新和拓展。不可变集合也能够更新和拓展，不过不可变集合是通过创建一个新的集合从而达到改变的效果。

### 不可变数组

数组是有序的，可重复的，可存放不同数据类型的数据集合。

创建数组的几种方式：

```scala
val array1:Array[Int] = new Array[Int](5)//基本格式，确定数组长度
val array2 = Array(1,2,3,4,5)//快速创建
val array3 = Array.ofDim[Int](3, 3)//使用ofDim创建多维3*3数组
```

遍历数组的几种方式

```scala
for (i <- array) {println(s"${i}.hello")}//for直接遍历数组
for (i <- 0 until array.length) {println(s"${i}.hello")}//利用长度遍历
//在Idea中会黄色显示，可以用下面索引遍历
for (i <- array.indices) {println(s"${i}.hello")}//for用indices遍历，索引遍历
```

数组增改查

```scala
array(2)//查找数组第3个
array(2)=8//赋值
val array1 = 1+: array :+2//在数组第一个加1，最后追加2，并传递给array1。因为不可变数组，所以必须再传递，数组本身没有发生变化
```

不可变数组不能直接打印

```scala
val array2 = Array(1,2,3,4,5)
println(array2)//这样打印的是数组的内存地址
```



### 可变数组

首先导入可变数组的包，创建可变数组：

```scala
import scala.collection.mutable.ArrayBuffer
val array:ArrayBuffer[Int] = ArrayBuffer[Int](5)
```

数组增删改查

```scala
array.+= (2)//在末尾追加2
array.append(4)//在末尾追加4
array.insert(1,2,3)//在1的下标位置插入2，3
array(1)=2//在1的下标位置更新为2
array.remove(3)//将第四个元素删除
```

可变数组可以直接打印

```scala
val array = ArrayBuffer(1,2,3,4,5)
println(array)//输出ArrayBuffer(1, 2, 3, 4, 5)
```

注：最好不要将两个可变数组赋值，这样会使后面对其中一个数组进行改变是，另一个 也会改变。可变数据的名称实际上指向的是数组的存储空间，两个相等，使他们指向的空间一样，值也会一样。

在使用中可变数组推荐使用关键字（调用函数）对数组进行更改，不可变数组推荐使用标识符（:+2）进行更改。

### 不可变列表List

不可变列表是数据有顺序，但是没有序列，可存储重复数据，可存储不同数据的的一种集合。

创建列表

```scala
val list1:List[Int] =  List[Int](5)//创建长度为5的空列表
val list2 = List(1,2,3,4,5)
```

列表操作

```scala
val list2 = 1 :: 2 :: 3 :: 4 :: Nil//在空列表List中追加数据
val list3 = list2.+:(5)//在list2中末尾追加一个5
val list3 = list2 ::: list1//将list2，1 合并
list3.foreach(println)//遍历输出
println(list3(2))//输出list3的第三个，list这个方法底层是遍历输出第三个。没有序列的表现是不能够直接赋值修改
```

### 可变列表List

创建列表

```scala
    val list0:ListBuffer[Int] = ListBuffer[Int](5)//创建长度为5的空列表
    val list = ListBuffer(1,2,3,4)
```

列表操作

```scala
    list.append(5)//追加5
    list.insert(1,2)//在1位置插入2
    list(1)=4//修改直接赋值
    list.update(1,5)//修改
    list.remove(0)//删除数据
```

### 不可变Set

数据是无序的，而且数据不可重复，如果创建是有重复数据，最后输出只有一个。

```scala
    val set0:Set[Int] = Set[Int](5)//创建长度为5的空Set
    val set = Set(1,2,3,4,5)
    for (i <- set)(
      println(i)
    )//遍历输出，无序的输出
```

### 可变Set

```scala
    val set = mutable.Set(1,2,3,4,5,3)//创建Set
    set += 7//增加元素
    set -= 5//删除元素
    val set2 =set.+(9)//赋值并增加元素
    for (i <- set2)(
      println(i)
    )//创建Set
  }
```

### 不可变Map

是一个散列表，储存的内容是键值对（key-value），

```scala
    val map = Map("a" -> 12,"b" -> 42,"c" -> 52)//创建Map
    println(map)
    map.foreach(println)//遍历打印
    println(map.get("a"))//返回当前value
    println(map.getOrElse("d" , 0))//如果没有K=d，V=0
    println(map("a"))//输出K=a的v
```

### 可变Map

```scala
    val map1 = mutable.Map("a" -> 13,"b" -> 43,"c" -> 53)
    val map2 = mutable.Map("q" -> 13,"w" -> 43,"e" -> 53)
    map1.put("d" ,45)//添加元素
    map1.update("d" ,34)//将k=d的v改为34
    map1.remove("c")//删除元素
    map1 ++= map2//无则添加，有则修改值
```

### 元组

元组是一个可以存放不同数据类型的集合，是将多个无关的数据封装为一个整体。

```scala
    val tuple1 = (1,"qwe",23)//创建元组
    println(tuple1)//输出元组
    println(tuple1._1)//输出元组第一个值
    for (i <- tuple1.productIterator) {
      println(i)
    }//遍历元组
```

## 数据集合常用函数

### 通用函数

```scala
    val list = List(1,2,3,4,5,6)
    println(list.length)//集合长度
    println(list.size)//集合大小
    for (i <- list) println(i)
    list.foreach(println)//遍历集合
    println(list.mkString(","))//生成字符串
    println(list.contains(3))//是否包含
```

### 衍生集合

```scala
    //单个集合
    println(list1.head)//数据第一个数据，有序集合
    println(list1.tail)//数据尾，除了第一个数据，其他都是尾，返回list
    println(list1.init)//数据头，除了最后一个数据，其他都是尾，返回list
    println(list1.last)//数据最后一个元素
    println(list1.reverse)//反转
    println(list1.take(3))//前n个元素
    println(list1.takeRight(3))//后n个元素
    println(list1.drop(2))//去前n个元素
    println(list1.dropRight(3))//去后n个元素
```

```scala
    val list1 = List(1,2,3,4,5,6)
    val list2 = List(1,12,3,14,15,16)
    println(list1.union(list2))//并集
    println(list1.intersect(list2))//交集
    println(list1.diff(list2))//差集
    println(list2.diff(list1))//差集
    println(list1.zip(list2))//拉链,将两个数据一一连，形成2元组
    for (i <- list1.sliding(3)) println(i)//滑窗
    for (i <- list1.sliding(3,2)) println(i)//滑窗  设定步长
    //滑窗
```

### 计算函数

```scala
    println(list1.sum)//求和
    println(list1.product)//求积
    println(list1.max)//求最大值
    println(list1.min)//求最小值
    println(list1.sortWith(_ > _))
    println(list1.sorted)//排序
```



