2.   lang

2.1.     和Java的异同

2.1.1.  语法

![](https://github.com/moveondo/Scala/blob/master/image/1.jpg)
 

2.1.2.  库

以下功能通过库的形式提供：

l  assert

l  enum

l  property

l  event

l  actor

l  resource control（自动释放）

def using[T <: { def close() }] (res:T)(block:T=>Unit) = {

  try { block(res) } finally { if(res!=null) res.close }}

using (new BufferedReader(new FileReader(path))) { f=> println(f.readLine) }

不用每次使用Java中的finally

val f = new BufferedReader(new FileReader(path)

try { println(f.readLine) } finally { if (f!=null) f.close }

l  query

2.2.     变量

2.2.1.  保留字

abstract   case       catch      class      def

do         else       extends    false      final

finally    for        if         implicit   import

match      new        null       object        override

package    private    protected requires   return

sealed        super      this       throw      trait

try        true       type       val        var

while      with       yield

_      :      =      =>     <-     <:     <%     >:     #      @

 

Scala调用Java的方法时，会碰到有Scala的保留字，如Thread.yield()

这在Scala中是非法的，专门有个解决办法，写成： Thread.`yield`()

 

注意：没有break和continue

2.2.2.  变量标识

这些标识在Java中是非法的，在Scala中是合法的，可以当作函数名使用，使接口更加DSL：

val empty_? = true

val + = "hello"

val `yield` = 10

val ** = "power"

2.2.3.  变量定义

2.2.3.1     val, var

var 可变，可重新赋值，赋值为"_"表示缺省值(0, false, null)，例如：

var d:Double = _ // d = 0.0

var i:Int = _ // i = 0

var s:String = _ // s = null

var t:T = _  // 泛型T对应的默认值


val 不可变，相当于const/final，但如果val为数组或者List，val的元素可以赋值；

val pi = 3. // 相当于3.0d

val pi = 3.f // 相当于3.0f

提示：向函数式风格推进的一个方式，就是尝试不用任何var来定义变量。

2.2.3.2     花样定义

和Python一样方便的赋值方式：

val x,y = 0 // 赋同一初始值

val (x,y) = (10, "hello") // 同时定义多个变量，注意：val x,y=10,"hello" 是错误的

更花：

val x::y = List(1,2,3,4)  // x = 1, y = List(2,3,4)

val List(a,b,c) = List(1,2,3) // a = 1, b = 2, c = 3

进一步花样：

val Array(a, b, _, _, c @ _*) = Array(1, 2, 3, 4, 5, 6, 7)  // 也可以用List，Seq

a // 1

b // 2

c // Array(5, 6, 7), _*匹配0个到多个

 

使用正则表达式定义：

val regex = "(\\d+)/(\\d+)/(\\d+)".r

val regex(year, month, day) = "2010/1/13"

// year: String = 2010

// month: String = 1

// day: String = 13

2.2.3.3     lazy, val, def的区别

val

定义时就一次求值完成，保持不变

val f = 10+20 // 30 

lazy

定义时不求值，第一次使用时完成求值，保持不变

lazy f = 10+20 // <lazy>

f // 30

def

定义时不求值，每次使用时都重新求值

def f = 10+20 // 30

def t = System. currentTimeMillis // 每次不一样



scala> val f1 = System.currentTimeMillis

f1: Long = 1279682740376    // 马上求值

scala> f1

res94: Long = 1279682740376 // 之后保持不变

scala> lazy val f2 = System.currentTimeMillis

f2: Long = <lazy>  // 定义时不求值

scala> System.currentTimeMillis

res95: Long = 1279682764297

scala> f2

res96: Long = 1279682766545 // 第一次使用时求值，注意：6545 > 4297

scala> f2

res97: Long = 1279682766545 // 之后保持不变

scala> def f3 = System.currentTimeMillis

f3: Long

scala> f3

res98: Long = 1279682784478 // 每次求值

scala> f3

res99: Long = 1279682785352 // 每次求值

2.3.     基本类型

尽量使用大写形式： Int, Long, Double, Byte, Short, Char, Float, Double, Boolean

编译时Scala自动对应到Java原始类型，提高运行效率。Unit对应java的void

 

用 asInstanseOf[T]方法来强制转换类型：

def i = 10.asInstanceOf[Double] // i: Double = 10.0

List('A','B','C').map(c=>(c+32).asInstanceOf[Char]) // List('a','b','c')

 
用isInstanceOf[T]方法来判断类型：

val b = 10.isInstanceOf[Int] // true

而在match ... case 中可以直接判断而不用此方法。


用Any统一了原生类型和引用类型。

![](https://github.com/moveondo/Scala/blob/master/image/2.jpg)


 

2.3.1.  Int

-3 abs // 3

-3 max -2 // -2

-3 min -2 // -3

1.4 round // 1 四舍五入

1.6 round // 2 四舍五入

1.1 ceil // 2.0 天花板

1.1 floor // 1.0 地板


无++，--操作，但可以+=, -=, 如下：

var i = 0

i++  // 报错，无此操作

i+=1 // 1

i--  // 报错，无此操作

i-=1 // 0

 
def even(n:Int) = 0==(n & 1)

def odd(n:Int) = !even(n)

2.3.2.  Char

String可以转化为List[Char]

在String上做循环，其实就是对String中的每一个Char做操作，如：

"12345" map (toInt) // (49,50,51,52,53)

"jamesqiu" max // 'u'

"jamesqiu" min // 'a'

('a' to 'f') map (_.toString*3) // (aaa, bbb, ccc, ddd, eee, fff)

2.4.     BigInt

可以表示很大的整数：

BigInt(10000000000000000000000000) // 报错

BigInt("10000000000000000000000000") // scala.math.BigInt = 10000000000000000000000000

 

例如：

def  fac(n:Int):BigInt = if (n==0) 1 else fac(n-1)*n

fac(1000)

或者写成：

def fac2(n:Int) = ((1:BigInt) to n).product

// res1: BigInt = 9332621544394415268169923885626670049071596826438......000000000000000000

2.5.     字符串

"..." 或者 """...""""

println("""|Welcome to Ultamix 3000.  

           |Type "HELP" for help.""".stripMargin)

输出：

Welcome to Ultamix 3000.  

Type "HELP" for help.

 

scala中，字符串除了可以+，也可以*

"abc" * 3 // "abcabcabc"

"abc" * 0 // ""

 
例子：

"google".reverse // "elgoog"

"abc".reverse.reverse=="abc" // true


例子：

"Hello" map (_.toUpper) // 相当于 "Hello".toUpperCase

2.5.1.  类型转换

"101".toInt // 101，无需 Integer.parseInt("101");

"3.14".toFloat // 3.14f

101.toString

3.14.toString

转换整个列表：

List("1","2","3") map (_.toInt) // List(1,2,3)

或者

List("1","2","3") map Integer.parseInt // List(1,2,3)


2.5.2.  StringBuilder

val sb = new StringBuilder

sb += 'H'

sb ++= "ello"

sb.toString // "Hello"

sb clear // StringBuilder()

2.5.3.  文本格式化

使用java.text.MessageFormat.format:

val msg = java.text.MessageFormat.format(
    "At {1,time} on {1,date}, there was {2} on planet {0}.",
    "Hoth", new java.util.Date(), "a disturbance in the Force")
    
输出

At 17:50:34 on 2010-7-20, there was a disturbance in the Force on planet Hoth.

 

方法2：

"my name is %s, age is %d." format ("james", 30) // my name is james, age is 30.

 

注意：format还可以这么用

"%s-%d：%1$s is %2$d." format ("james", 30) // james-30：james is 30.

    "%2$d age's man %1$s: %2$d" format ("james", 30) // 30 age's man james: 30

 

2.6.     Null, None, Nil, Nothing

Null

Trait，其唯一实例为null，是AnyRef的子类，*不是* AnyVal的子类

Nothing

Trait，所有类型（包括AnyRef和AnyVal）的子类，没有实例

None

Option的两个子类之一，另一个是Some，用于安全的函数返回值

Unit

无返回值的函数的类型，和java的void对应

Nil

长度为0的List

 

2.7.     ==和eq

Scala的==很智能，他知道对于数值类型要调用Java中的==，ref类型要调用Java的equals()

"hello"=="Hello".toLowerCase()

在java中为false，在scala中为true

 

Scala的==总是内容对比

基本类型Int，Double，比值

其他类型 相当于A.equals(B)

eq才是引用对比


例如：

val s1,s2 = "hello"

val s3 = new String("hello")

s1==s2 // true

s1 eq s2 // true

s1==s3 // true 值相同

s1 eq s3 // false 不是同一个引用

2.8.     Option[T]

2.8.1.  概念

l  Option[T]可以是任意类型或者空，但一旦声明类型就不能改变；

l  Option[T]可完美替代Java中的null，可以是Some[T]或者None；

l  Option实现了map, flatMap, and filter 接口，允许在 'for'循环里使用它；

![](https://github.com/moveondo/Scala/blob/master/image/3.jpg)

 

2.8.2.  使用

Some(3).getOrElse(4) // 3

None.getOrElse(4) // 4

例如打印key=3的value：

写法1：

def p(map:Map[Int,Int]) = println(map(3))

p(Map(1->100,2->200)) // 抛异常

写法2：

def p(map:Map[Int,Int]) = println(map get 3 getOrElse "...")

p(Map(1->100,2->200)) // ...

p(Map(1->100,3->300)) // 300

 

2.8.3.  例子

例子1：

  def m(k:Int) = {

    Map((1,100),(2,200),(3,300)) get(k) match {

      case Some(v) =>

        k + ": " + v

      case None =>

        "not found"

    }

  }

 

  def main(args : Array[String]) : Unit = {

    println(m(1)) // 100

    println(m(2)) // 200

    println(m(3)) // 300

    println(m(4)) // "not found"

    println(m(-1)) // "not found"

  }

 

例子2：

val l = List(Some(100), None, Some(200), Some(120), None)

for (Some(s) <- l) yield s // List(100, 200, 120)

或

l flatMap (x=>x) // List(100, 200, 120)

 

例子3： Option结合flatMap

def toint(s:String) =

try { Some(Integer.parseInt(s)) } catch { case e:Exception => None }

List("123", "12a", "45") flatMap toint // List(123, 45)

List("123", "12a", "45") map toint // List(Some(123), None, Some(45))

 

2.9.     区分<-,=>,->

![](https://github.com/moveondo/Scala/blob/master/image/4.jpg)

 

2.10.    match..case(switch)

2.10.1.      和switch..case的区别

Java里面的写法：

switch(n) {

    case(1): ...; break;

    case(2): ...; break;

    default: ...;

}

变成Scala写法：

def m(n:String) =

n match {

    case "a" | "b" => ... // 这个比较好

    case "c" => ...

    case _ => ...

}

每个case..=>结束不用写break了，_相当于default

 

2.10.2.      匹配数据类型

match 可以很简单地匹配数据类型（不需要isInstanceOf[T]）：

def f(v:Any) = v match {

    case null => "null"

    case i:Int => i*100

    case s:String => s

    case _ => "others"

}

注意：上面case中的i、s都叫模式变量

f(null) // "null"

f(5) // 500

f("hello") // "hello"

f(3.14) // "others"

 

注意：自定义类型如果也要匹配，需要用case class

2.10.3.      命令行参数解析例子

/** Basic command line parsing. */

```
object Main {

  var verbose = false  // 记录标识，以便能同时对-h和-v做出响应
 
  def main(args: Array[String]) {
    for (a <- args) a match {
      case "-h" | "-help"    =>
        println("Usage: scala Main [-help|-verbose]")
      case "-v" | "-verbose" =>
        verbose = true
      case x => // 这里x是临时变量
        println("Unknown option: '" + x + "'")
    }
    if (verbose) println("How are you today?")
  }
}
```

2.10.4.      使用case的递归函数

写法1：

def fac(n:Int):Int = n match {

  case 0=>1

  case _=>n*fac(n-1)

}

 

写法2（使用映射式函数）：

def fac: Int=>Int = {

case 0=> 1

case n=> n*fac(n-1)

}

 

写法3（使用尾递归）：

  def fac: (Int,Int)=>Int = {

    case (0,y) => y

    case (x,y) => fac(x-1, x*y)

  }
  
fac(5,1) // 120
 
写法4（reduceLeft+!）：

def fac(n:Int) = 1 to n reduceLeft(_*_)

implicit def foo(n:Int) = new { def ! = fac(n) }

5! // 120
 
写法5：(最简洁高效)

  def fac(n:Int) = (1:BigInt) to n product
  
  fac(5) // 120
  
2.10.5.      变量匹配

常量匹配很简单，即case后跟的都是常量；

变量匹配需要注意，case后跟的是match里面的临时变量，而不是其他变量名：

3 match {

case i => println("i=" + i) // 这里i是模式变量（临时变量），就是3

}

val a = 10

20 match { case a => 1 } // 1， a是模式变量，不是10

为了使用变量a，必须用`a`:

        20 match { case `a` => 1; case b => -1 } // -1，`a`是变量10
        
或者用大写的变量：

        val A = 10
        
        20 match { case A => 1; case b => -1 } // -1，大写A是变量10
 
2.10.6.      case..if条件匹配

写法1：

```

(1 to 20) foreach {                          

    case x if (x % 15 == 0) => printf("%2d:15n\n",x)

    case x if (x % 3 == 0)  => printf("%2d:3n\n",x)

    case x if (x % 5 == 0)  => printf("%2d:5n\n",x)

    case x => printf("%2d\n",x)                          

}
```

写法2：

```
(1 to 20) map (x=> (x%3,x%5) match {
  case (0,0) => printf("%2d:15n\n",x)
  case (0,_) => printf("%2d:3n\n",x)
  case (_,0) => printf("%2d:5n\n",x)
  case (_,_) => printf("%2d\n",x)
})
```

2.11.    try..catch..finally

var f = openFile()

try {

  f = new FileReader("input.txt")

} catch {

  case ex: FileNotFoundException => // Handle missing file

  case ex: IOException => // Handle other I/O error

} finally {

  f.close()

}

2.12.    require

def f(n:Int) = { require(n!=0); 1.0/n }

def f(n:Int) = { require(n!=0, "n can't be zero"); 1.0/n }

f(0)

// java.lang.IllegalArgumentException: requirement failed: n can't be zero

 

2.13.    main方法

Scala的main方法(包括所有类似java的static方法)必须定义在一个object内：

object Test1 {

    def main(args: Array[String]) {

       println("hello world")

    }

}

编译：

fsc Test1.scala // 常驻内存编译服务器，第一次转载之后比scalac快

运行：

scala Test1.scala // 方式1，没有输出

scala -cp e:\scala\lib\scala-library.jar Test1 // 方式2，慢

java -cp e:\scala\lib\scala-library.jar Test1 // 方式3，快

 

如果文件不是utf8编码，执行需要使用.scala文件的编码，如：

scala -encoding gbk test.scala

2.13.1.      Application

不带命令行参数的简化main方法：

object app1 extends Application {

    println("hello world")

}

2.14.    package, import

2.14.1.      import

Scala的import可以只在局部作用域内生效；

可以格式 “import javax.swing.{JFrame=>jf}”来声明类型的别名。

jf.show()

l  import javax.swing._

l  import java.util.{List, Map}

l  import java.util._, java.io._

Scala 缺省导入如下包：

l  java.lang.*

l  scala.*

l  scala.Predef

 

由于Scala的package可以是相对路径下定义，有可能命名冲突，可以用：

import _root_.java.lang.Long

2.14.2.      package

package com.wr3 { // C# 和Ruby的方式，也可以改用Java的方式

    // import java.nio._ // "*" 是scala的正常函数名，所以用_

    class c1 {

       def m1() { println("c1.m1()") }

    }

 

    object o1 {

       def main(args: Array[String]) {

          println("o1.main()")

          new c1().m1()

        }

    }

}

编译：

fsc package.scala

运行：

java com.wr3.o1 // 方式1

scala com.wr3.o1 // 方式2

 

2.14.3.      包对象

Scala2.8+支持包对象（package object），除了和2.8之前一样可以有下级的object和class，还可以直接有下级变量和函数，例如：

-------------------------------- foo.scala

package p0

package object p1 {

    val a = 10

    def b = "hello " + a

    def main(args:Array[String]):Unit = printf("%s", p0.p1.b)

}

--------------------------------

p1就是一个包对象，a和b就是包p1直属的常量和函数,

$fsc foo.scala 命令产生如下class：

./p0/p1/package.class

调用：

scala p0.p1.package

2.15.    if..else

没有java的：

b = (x>y) ? 100 : -1

就用：

if (x>y) 100 else -1

2.16.    循环操作

![](https://github.com/moveondo/Scala/blob/master/image/5.jpg)

 

2.16.1.      for

循环中的变量不用定义，如：

for(i<-1 to 10; j=i*i) println(j)

for (s <- ss) foo(s)

for (i <- 0 to n) foo(i) // 包含n，即Range(0,1,2,...,n,n+1)

for (i <- 0 until n) foo(i)  // 不包含n，即Range(0,1,2,3,...,n)

例如：

for (n<-List(1,2,3,4) if n%2==1) yield n*n  // List(1, 9)

for (n<-Array(1,2,3,4) if n%2==1) yield n*n  // Array(1, 9)

注意：如果if后面不止一条语句，要用{..}包裹。

var s = 0; for (i <- 0 until 100) { s += i } // s = 4950

 

等价于不用for的写法：

List(1,2,3,4).filter(_%2==1).map(n => n*n)

 

如果for条件是多行，不能用(),要用{}

for(i<-0 to 5; j<-0 to 2) yield i+j

// Vector(0, 1, 2, 1, 2, 3, 2, 3, 4, 3, 4, 5, 4, 5 , 6, 5, 6, 7)

for{i<-0 to 5

j<-0 to 2} yield i+j

 

例子1：

// 边长21以内所有符合勾股弦的三角形：

def triangle(n: Int) = for {

  x <- 1 to 21

  y <- x to 21

  z <- y to 21

  if x * x + y * y == z * z

} yield (x, y, z)

结果：

// Vector((3,4,5), (5,12,13), (6,8,10), (8,15,17), (9,12,15), (12,16,20))

 

2.16.1.    for .. yield

把每次循环的结果“移”进一个集合（类型和循环内的一致）

for {子句} yield {循环体}


正确：

for (e<-List(1,2,3)) yield (e*e) // List(1,4,9)

for {e<-List(1,2,3)} yield { e*e } // List(1,4,9)

for {e<-List(1,2,3)} yield e*e // List(1,4,9)


错误：

for (e<-List(1,2,3)) { yield e*e } // 语法错误,yield不能在任何括号内

 

2.16.2.      foreach

List(1,2,3).foreach(println)

1

2

3

也可以写成：

(1 to 3).foreach(println)

或者

(1 until 4) foreach println

或者

Range(1,3) foreach println


注意：

l  to包含，until不包含（最后的数）

l  都可以写步长，如：

1 to (11,2) // 1,3,5,7,9,11 步长为2

    1 to 11 by 2

1 until (11,2) // 1,3,5,7,9

1 until 11 by 2

val r = (1 to 10 by 4) // (1,5,9), r.start=r.first=1; r.end=10, r.last=9

l  也可以是BigInt

(1:BigInt) to 3

2.16.3.      forall

"所有都符合"——相当于 A1 && A2 && A3 && ... && Ai && ... && An

(1 to 3) forall (0<) // true

(-1 to 3) forall (0<) // false

 

又如：

def isPrime(n:Int) = 2 until n forall (n%_!=0)

for (i<-1 to 100 if isPrime(i)) println(i)

(2 to 20) partition (isPrime _) // (2,3,5,7,11,13,17,19), (4,6,8,9,10,12,14,15,16,18,20)

也可直接调用BigInt的内部方法：

(2 to 20) partition (BigInt(_) isProbablePrime(10))

// 注：isProbablePrime(c)中c越大，是质数的概率越高，10对应概率：1 - 1/(2**10) = 0.999

 

2.16.4.      reduceLeft

reduceLeft 方法首先应用于前两个元素，然后再应用于第一次应用的结果和接下去的一个元素，等等，直至整个列表。例如

计算阶乘：

def fac(n: Int) = 1 to n reduceLeft(_*_)

fac(5) // 5*4*3*2 = 120

相当于：

    ((((1*2)*3)*4)*5)

计算sum：

    List(2,4,6).reduceLeft(_+_) // 12

相当于：

    ((2+4)+6)

取max：

List(1,4,9,6,7).reduceLeft( (x,y)=> if (x>y) x else y ) // 9

       或者简化为：

List(1,4,9,6,7).reduceLeft(_ max _) // 9

相当于：

    ((((1 max 4) max 9) max 6) max 7)

2.16.5.      foldLeft scanLeft
 

累加或累乘

def sum(L: List[Int]): Int = {

        var result = 0

        for (item <- L)

            result += item

        result

}

更scalable的写法：

def sum(L: Seq[Int]) = L.foldLeft(0)((a, b) => a + b)

def sum(L: Seq[Int]) = L.foldLeft(0)(_ + _)

def sum(L: List[Int]) = (0/:L){_ + _}

调用：

sum(List(1,3,5,7)) // 16

 

乘法：

def multiply(L: Seq[Int]) = L.foldLeft(1)(_ * _)

multiply(Seq(1,2,3,4,5)) // 120

multiply(1 until 5+1) // 120

 

2.16.6.      scanLeft

List(1,2,3,4,5).scanLeft(0)(_+_) // (0,1,3,6,10,15)

相当于：

(0,(0+1),(0+1+2),(0+1+2+3),(0+1+2+3+4),(0+1+2+3+4+5))

 

List(1,2,3,4,5).scanLeft(1)(_*_) // (1,2,6,24,120)

相当于

(1, 1*1, 1*1*2, 1*1*2*3, 1*1*2*3*4, 1*1*2*3*4*5)

 

注：

l  (z /: List(a, b, c))(op) 相当于 op(op(op(z, a), b), c)

简单来说：加法用0，乘法用1

l  (List(a, b, c) :\ z) (op) equals op(a, op(b, op(c, z)))

 

2.16.7.      take drop splitAt

1 to 10 by 2 take 3 // Range(1, 3, 5)

1 to 10 by 2 drop 3 // Range(7, 9)

1 to 10 by 2 splitAt 2 // (Range(1, 3),Range(5, 7, 9))

 

例子：前10个质数

def prime(n:Int) = (! ((2 to math.sqrt(n).toInt) exists (i=> n%i==0)))

2 to 100 filter prime take 10

2.16.8.      takeWhile, dropWhile, span

while语句的缩写，

takeWhile (...)

等价于：while (...) { take }

dropWhile (...)

等价于：while (...) { drop }

span (...)

等价于：while (...) { take; drop }

 

1 to 10 takeWhile (_<5) // (1,2,3,4)

1 to 10 takeWhile (_>5) // ()

10 to (1,-1) takeWhile(_>6) // (10,9,8,7)

1 to 10 takeWhile (n=>n*n<25) // (1, 2, 3, 4)

如果不想直接用集合元素做条件，可以定义var变量来判断：

例如，从1 to 10取前几个数字，要求累加不超过30：

var sum=0;

val rt = (1 to 10).takeWhile(e=> {sum=sum+e;sum<30}) // Range(1, 2, 3, 4, 5, 6, 7)

注意：takeWhile中的函数要返回Boolean，sum<30要放在最后；

 

1 to 10 dropWhile (_<5) // (5,6,7,8,9,10)

1 to 10 dropWhile (n=>n*n<25) // (5,6,7,8,9,10)

 

1 to 10 span (_<5) // ((1,2,3,4),(5,6,7,8)

List(1,0,1,0) span (_>0) // ((1), (0,1,0))

注意，partition是和span完全不同的操作

List(1,0,1,0) partition (_>0) // ((1,1),(0,0))

2.16.9.      break、continue

Scala中没有break和continue语法！需要break得加辅助boolean变量，或者用库（continue没有）.

 

例子1：打印'a' to 'z'的前10个

var i=0; val rt = for(e<-('a' to 'z') if {i=i+1;i<=10}) printf("%d:%s\n",i,e)

或者：

('a' to 'z').slice(0,10).foreach(println)

 

例子2：1 to 100 和小于1000的数

var (n,sum)=(0,0); for(i<-0 to 100 if (sum+i<1000)) { n=i; sum+=i }

// n = 44, sum = 990

 

例子3：使用库来实现break

import scala.util.control.Breaks._

for(e<-1 to 10) { val e2 = e*e; if (e2>10) break; println(e) }

 

2.17.    操作符重载

注意：其实Scala没有操作符，更谈不上操作符重载；+-/*都是方法名，如1+2其实是(1).+(2)

object operator {

    class complex(val i:Int, val j:Int) { // val 是必须的

        def + (c2: complex) = {

            new complex (i+c2.i, j+c2.j)

        }

        override def toString() = { "(" + i + "," + j + ")" }

    }

    def main(args:Array[String]) = {

        val c1 = new complex(3, 10)

        val c2 = new complex(5, 70)

        printf("%s + %s = %s", c1, c2, c1+c2)

    }

}

编译：fsc operator.scala

运行：java operator // (3,10) + (5,70) = (8,80)

 

2.18.    系统定义scala._

![](https://github.com/moveondo/Scala/blob/master/image/6.jpg)

 

2.19.    implicit隐式转换

用途：

l  把一种object类型安全地自动转换成另一种object类型;

l  不改动已有class设计即可添加新的方法;

2.19.1.      类型转换

implicit def foo(s:String):Int = Integer.parseInt(s) // 需要时把String->Int

def add(a:Int, b:Int) = a+b

add("100",8) // 108, 先把"100"隐式转换为100

2.19.2.      例子：阶乘n!

第一步：写函数

def factorial(x: Int) = 1 to n reduceLeft(_*_)

第二步：定义 "!" 函数

    class m1(n: Int) {

        def ! = factorial(n)

    }

implicit def m2(n:Int) = new m1(n) // 隐式转换，即在需要时把n转换为new m1(n)

 

注意：上面可以用匿名类简化为：

    implicit def m2(n:Int) = new { def ! = factorial(n) }

 

第三步：使用

    val n = 100

printf("%d! = %s\n", n, (n!))  // n! 相当于 new m1(n).!()

println(10!)

 

2.19.3.      例子：cout

```
import java.io._

class C1(p:PrintStream) {

def << (a:Any) = {

p.print(a)

p.flush

p

}

}

implicit def foo(p:PrintStream) = new C1(p)

val endl = '\n'

System.out<<"hello"<<" world"<<endl

```

2.19.4.      例子：定义?:


```
implicit def elvisOperator[T](alt: T) = new {
  def ?:[A >: T](pred: A) = if (pred == null) alt else pred
}                  
 

null ?: "" // ""

"abc" ?: "" // "abc"

10 ?: 0 // 10

(null ?: 0).asInstanceOf[Int] // 0


```

2.19.5.      已有Object加新方法


```
object NewMethod {

  // 定义新方法join()

  implicit def foo1[T](list: List[T]) = new {

    def join(s:String) = list.mkString(s)

  }

  // 测试

  def main(args : Array[String]) : Unit = {

    Console println List(1,2,3,4,5).join(" - ") // " 1 - 2 - 3 - 4 – 5"

  }

}

```

解释：

编译器发现List没有join(String)方法，就发查找代码中有没有定义在implicit def xx(List)内的 join(String)方法，如果有就调用这个。

 

为Int增加乘方操作：

```

def pow(n:Int, m:Int):Int = if (m==0) 1 else n*pow(n,m-1)

implicit def foo(n:Int) = new {

  def **(m:Int) = pow(n,m)

}

2**10 // 1024

```
 

例子2：定义如ruby的10.next

implicit def foo(n:Int) = new { def next = n+1 }

10.next // 11

 

2.20.    type做alias

相当于C语言的类型定义typedef，建立新的数据类型名（别名）；在一个函数中用到同名类时可以起不同的别名

例如：

type JDate = java.util.Date       

type SDate = java.sql.Date

val d1 = new JDate()  // 相当于 val d = new java.util.Date()

val d2 = new SDate()  // 相当于 val d = new java.sql.Date()

 

注意：type也可以做泛型


2.21.    泛型

2.21.1.      函数中的泛型：

def foo[T](a:T) = println("value is " + a)

foo(10) // "value is 10"

foo(3.14) // "value is 3.14"

foo("hello") // "value is hello"

 

2.21.2.      类中的泛型：

class C1[T] {

private var v:T = _   // 初始值为T类型的缺省值

def set(v1:T) = { v = v1 }

def get = v

}

new C1[Int].set(10).get // 10

new C1[String].set("hello").get // "hello"

2.21.3.      泛型qsort例子

def qsort[T <% Ordered[T]](a:List[T]): List[T] = if (a.size<=1) a else {

val m = a(a.size/2)

qsort(a.filter(m>)) ++ a.filter(m==) ++ qsort(a.filter(m<))

}

调用：

        val a = List(1, 3, 6, 2, 0, 9, 8, 7, 2)

        qsort[Int](a) // 注意2 a

        val b = List(1.0, 3.3, 6.2, 6.3, 0, 9.5, 8.7, 7.3, 2.2)

        qsort[Double](b) // 注意2 b

 

或者使用自带库：

List(1, 3, 6, 2, 0, 9, 8, 7, 2) sortWith(_<_)

List(1.0, 3.3, 6.2, 6.3, 0, 9.5, 8.7, 7.3, 2.2).sortWith(_<_)

 

2.21.4.      泛型add例子

// 采用implicit parameters

def add[T](x:T, y:T)(implicit n:Numeric[T]) = {

n.plus(x,y) // 或者用 import n._; x + y

}

add(1,2) // 3

add(1,2.14) // 3.14

 

再如，求max：

def max2[T](x:T, y:T)(implicit n:Numeric[T]) = {

  import n._

  if (x > y) x else y

}

max2(2, 3.14) // 3.14

2.21.5.      泛型定义type


```
abstract class C1 {

    type T

    val e:T

}

 

abstract class C2 {

    type T

    val list:List[T]

    def len = list.length

}

 

def m1(e1:Int) = new C1 {

    type T = Int

    val e = e1

}

 

def m2(e1:List[Int]) = new C2 {

    type T = Int

    val list = e1

}

Console println m1(10) // 10

Console println m2(List(1,2,3,4,5)).len // 5

 
```


注意：type也可以做数据类型的alias，类似C语言中的typedef

 

2.21.6.      @specialized

def test[@specialized(Int,Double) T](x:T)  = x match { case i:Int => "int"; case _ => "other" }

没有@specialized之前，是编译成Object的代码；用了@specialized，变成IntTest(),DoubleTest(),...

编译后的文件尺寸扩充了，但性能也强了，不用box，unbox了

2.22.    枚举Enum

Scala没有在语言层面定义Enumeration，而是在库中实现：

例子1：

```

object Color extends Enumeration {

  type Color = Value

  val RED, GREEN, BLUE, WHITE, BLACK = Value

}

 

Color.RED // Color.Value = RED

import Color._

val colorful = Color.values filterNot (e=> e==WHITE || e==BLACK)

colorful foreach println // RED\nGREEN\nBLUE

```
 

例子2：

```

object Color extends Enumeration {

  val RED = Value("红色")

  val GREEN = Value("绿色")

  val BLUE = Value("蓝色")

  val WHITE = Value("黑")

  val BLASK = Value("白")

}

Color.RED // Color.Value = 红色

import Color._

val colorful = Color.values filterNot (List("黑","白") contains _.toString)

colorful foreach println //红色\n绿色\n蓝色

```
 

