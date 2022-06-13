### Scala函数式编程

#### 函数基础

def 函数名（传入参数名：参数类型）：函数返回值 {函数体}

```
 def add(a:Int,b:Int):Int = {
      a+b
    }
```

##### 函数参数

函数参数有默认参数（无赋值就用默认值，有则覆盖）、可变参数以及带名参数（改变参数输入顺序）。

默认参数、可变参数一般放在参数列表的后面。

```scala
    def func(s:String*):Unit={
      println(s)
    }
    func("qwe","asd")
    func()//返回WrappedArray(qwe, asd)，List()

    def function(age:Int=18): Unit ={
      println(age+"岁")
    }
    function()
    function(34)

    def function1(age:Int=18,name:String): Unit ={
      println(age+"岁")
    }
    function1(name="qwe")
    function1(12,"qwe")
```



##### 函数多种用法

在Scala中，函数属于一等公民，作用非常强大。函数可以作为值，参数，返回值在函数中使用。

函数作为一个值传递。如果参数的值类型，返回值类型不能确定就在函数名后加一个下划线。能确定参数类型就可以加函数类型的变化。

```scala
    def function(a:Int,b:Int):Int={
      a+b
    }
    val fun = function _ //未知的函数参数类型
    val fun2:(Int,Int)=>Int = function
    println(fun)
    println(fun(2,3))
```

函数作为参数进行传递。经典的二元计算的例子，我们在函数中固定两个数字，而这两个数字该做什么操作，用其他函数作为参数。

```scala
    def function(f:(Int,Int)=>Int):Int ={
      f(1,2)
    }
    def add(a:Int,b:Int):Int = {
      a+b
    }
    def chen(a:Int,b:Int):Int = {
      a*b
    }
    println(function(add))
    println(function(chen _))
  }
```

函数作为函数返回值返回。

```scala
    def function() ={
      def fun(): Unit ={  
      }
      fun _
    }
  }
```



#### 函数与方法

函数是完成某一功能的集合，方法是Java中类中的函数。函数没有重载重写的概念。

#### 函数柯里化以及闭包

##### 闭包

如果一个函数访问到了外部的参数，这个函数的环境叫做闭包。

##### 柯里化

把一个参数列表的多个参数，变成多个参数列表。 柯里化其实就是将复杂的参数逻辑变得简单化，就一定会有闭包。我个人觉得，柯里化也会加大函数的复用性。

```scala
    def function(a:Int)(b:Int):Int = {
      a+b
    }
    var function4 = function(4) _
    println(function4(5))
  }
```

这个例子本体是两数相加，将参数分离。我们就可以改成4加某个数了。

#### 递归

一个函数的方法在函数的方法体内又调用了本身，我们称之为递归调用。但是需要注意，必须要规定一个跳出的操作，不然函数会自己不断的执行。

```scala
   def function(n:Int):Int = {
     if(n==0) return 0//跳出操作
     else function(n-1)+n
   }
    println(function(5))
  }
```

#### 尾递归优化

在计算机中，递归非常的耗费资源。所以我们使用尾递归优化，在递归中释放函数所占内存，将数值记录下来。可以在函数加@tailrec判断是不是尾递归优化。

```scala
    def function1(n:Int):Int = {
      @tailrec
      def fun(n:Int,sun:Int):Int = {
        if(n==0) return sun
        fun(n-1,sun+n-1)
      }
      fun(n,n)
    }
    println(function1(5))
```

#### 控制抽象

让参数调用一个代码快，其中调用几次数值，那么代码块执行几次。例如下代码块，其中function2中输出两次，那么说明调用a两次，输出的是a调用 n=12 a调用 n=12.

```
    def a(n:Int):Int = {
      println("a调用")
      n
    }
    def function2(n: =>Int):Unit = {
      println(s"n=${n}")
      println(s"n=${n}")
    }
    function2(a(12))
```

#### 惰性加载

当函数返回值被声明为 lazy 时，函数的执行将被推迟，直到我们首次对此取值，该函数才会执行。这种函数我们称之为惰性函数。只要不调用，他本身就不会执行。

```scala
lazy def a= function2(a(12))
```

