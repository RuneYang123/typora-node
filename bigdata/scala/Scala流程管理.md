### Scala流程管理

#### if分支结构

scala的分支结构类似java的分支结构，主要分为单分支,双分支，多分支，以及嵌套分支。

##### 单分支

if(判断语句){行为语句}，当if后判断语句为true时执行行为语句。

```scala
if (age > 18) {println("成年")}
```

单分支因为只有一个条件，也可以写成

```scala
if (age > 18) println("成年")
```

##### 双分支

if(判断语句){行为语句1}else{行为语句2}，当if后判断语句为true时执行行为语句1，为false是执行行为语句2。

```scala
if (age > 18) println("成年") else println("未成年")
```

看到这种简化后的，可以看出类似java的三元判断。

```java
String san = inputAge > age ? "成年":"未成年";
```

##### 多分支

```scala
if (age > 18) {
      println("成年")
    } else if (age < 6 ) {
      println("童年")
    } else {
      println("未成年")
    }
```

##### 嵌套分支

实际上就是在if的行为语句中再加if判断，也可以直接用多分支。

```
  if (age<18) {
      if (age<6){
        println("儿童")
      }
      else {
        println("未成年")
      }
    }
```

##### if返回值

有时候我们需要返回分支的结果，并不是说直接打印。而if默认返回的之是unit，而且printIn也只是打印，并不会返回值。如下：

```scala
  var ifElse = if (age > 18) {
      println("成年")
    } else if (age < 6 ) {
      println("童年")
    } else {
      println("未成年")
    }
  println("ifElse" + ifElse)
```

这样最后输出的ifElse是(),也就是Unit。那是不是可以将ifElse的格式定为String，直接改发现报错。因为scala会自动判断参数的格式，我们在行动语句中也没有值，也就是返回Unit，所以在行动语句中增加字符。

```scala
   var ifElse : String = if (age > 18) {
      println("成年")
     "成年"
    } else if (age < 6 ) {
      println("童年")
     "童年"
    } else {
      println("未成年")
     "未成年"
    }
    println("ifElse+" + ifElse)
    println(s"今年${age}是一个${ifElse}")
```

注：有值时，再把String改成Unit并不会报错，但是返回是ifElse是空值。

#### for循环

for循环相比java的循环功能更加强大。for(i<-range){循环行动}。其中range表示遍历的范围。

##### for to until

range中，可以自己规定范围。to表示包括边界，until不包括边界。

```scala
 for (i <- 1 to 10) {
      println(i + ".hello scala")
    }
```

```
 for (i <- 1 until 10) {
      println(i + ".hello scala")
    }
```

1 to 10 就是 1到10，包括10。而1 until 10 就是 1到9。

##### for集合

range也可以用集合来表示。（Array，List，Set）

```scala
    val arr = Array(1,2,3,4,5)
    for (i <- arr) {
      println(i + ".hello scala")
    }
```

##### for步幅

以上都是从头到尾遍历，有时候我们只需要奇数偶数或者有规律的遍历。使用by关键字，确定步幅，我们只要奇数。

```scala
    for (i <- 1 to 10 by 2) {
      println(i + ".hello scala")
    }
```

##### for多个遍历

```scala
for (i <- 1 to 3;j <-1 to 4 ) {
  println(s"i=${i},j=${j}")
}
```

会输出两个区间所有的可能。

##### for守卫

在有些时候，循环中我们需要挑选遍历的元素。比如我们不需要3和4。

```
  for (i <- 1.to(6)) {
      if(i != 4 && i != 3) println(i+".hello")
    }
```

##### for返回值

for循环本身是不能够返回值的，我们假如需要for循环返回所有的循环结果，要使用yirId关键字。for循环中的 yield 会把当前的元素记下来，保存在集合中，循环结束后将返回该集合。Scala中for循环是有返回值的。如果被循环的是Map，返回的就是Map，被循环的是List，返回的就是List，以此类推。

```scala
val yie = for (i <- 1 to 5) yield i
```

也可以对i进行计算。

```scala
val yie = for (i <- 1 to 5) yield i*3
```

##### 退出循环

在scala中，也可以在特定情况下跳出循环。

```scala
    val arr = List(1,5,6,3,7)
    Breaks.breakable(for (i <- arr) {
      if (i==3) Breaks.break()
      println(i+"====")
      }
```

我们可以将导入包时，导入Breaks所有的函数，简化成。

```scala
import scala.util.control.Breaks._
breakable(for (i <- arr) {
      if (i == 3) break()
      println(i + "====")
    }
    )
```

注：while循环以及do while循环基本类似与java。

