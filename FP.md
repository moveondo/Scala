
### 3.   FP

#### 3.1.     函数

函数的地位和一般的变量是同等的，可以作为函数的参数，可以作为返回值。

传入函数的任何输入是只读的，比如一个字符串，不会被改变，只会返回一个新的字符串。

Java里面的一个问题就是很多只用到一次的private方法，没有和使用它的方法紧密结合；Scala可以在函数里面定义函数，很好地解决了这个问题。

3.1.1.  函数定义

函数和方法一般用def定义；也可以用val定义匿名函数，或者定义函数别名。

def m0(x:Int) = x*x

val m1 = (x:Int)=> x*x // ()是必须的

val m2 = {x:Int=> x*x} // 不用(), 用{}

m0(10) // 100

m1(10) // 100

m2(10) // 100

不需要返回值的函数，可以使用def f() {...}，永远返回Unit（即使使用了return）,即：

    def f() {...}  等价于 def f():Unit = {...}

例如：

def f() { return "hello world" }

f() // Unit，而不是 "hello world"


需要返回值的函数，用 def f() = {...} 或者 def f = {...}

def f() = { "hello world" }

f() // "hello world"

def f = { "hello world" } // f是匿名函数 {"hello world"}的别名

f // "hello world"


三种定义方式的区别：

```

def f() { return .. }   调用：f, f()皆可   始终返回：Unit

def f() = ...       调用：f, f()皆可         返回Unit或者值

def f = ...       调用：f            返回Unit或者值

```

提示：函数式风格——尽量编写有返回值的函数; 尽量简短（函数体不使用{...}）


3.1.2.  映射式定义

一种特殊的定义：映射式定义（直接相当于数学中的映射关系）;

其实也可以看成是没有参数的函数，返回一个匿名函数；调用的时候是调用这个返回的匿名函数。

 

例子1：

def f:Int=>Double = { // 请看做 def f: (Int=>Double) = {...}

    case 1 => 0.1

    case 2 => 0.2

    case _ => 0.0

}

f(1) // 0.1

f(3) // 0.0

 

例子2：

def m:Option[User]=>User = {

    case Some(x) => x

    case None => null

}

m(o).getOrElse("none...")

 

例子3：(多->1)

def m:(Int,Int)=>Int = _+_

m(2,3) // 5

 

例子4：

def m:Int=>Int = 30+  // 相当于30+_,如果唯一的"_"在最后,可以省略

m(5) // 35

3.1.3.  特殊函数名 + - * /

方法名可以是*：

def *(x:Int, y:Int) = { x*y }

*(10,20) // = 200

1+2, 相当于1.+(2)

 

定义一元操作符（置前）可用unary_。

注：unary ：一元的，单一元素的, 单一构成的 。发音：【`ju: ne ri】


-2，相当于：(2).unary_- // -2

+2，相当于：(2).unary_+ // 2

!true, 相当于：(true).unary_! // false

~0，相当于 (0).unary_~  // -1

3.1.4.  缺省参数、命名参数

注意：从Scala2.8开始支持。

请对比以下两种写法的可读性：

一般函数调用


```
sendEmail(

"jon.pretty@example.com",

List("recipient@example.com"),

"Test email",

b,

Nil, Nil, Nil)
```


命名函数调用

```
sendEmail(

    body = b,

    subject = "Test email",

    to = List("recipient@example.com"),

    from = "jon.pretty@example.com",

    attachments = List(file1, file2) )

``` 

定义和使用：

def join(a:List[String], s:String="-") = { a.mkString(s) }

join(List("a","b","c")) // a-b-c

join(List("a","b","c"), ":") // a:b:c

调整参数调用顺序：

join(s=":",a=List("a","b","c")) // a:b:c

3.1.5.  函数调用

def f(s: String = "default") = { s }

f // "hello world"

f() // "hello world"

 
>  对象的无参数方法的调用，可以省略.和()

"hello world" toUpperCase // "HELLO WORLD"


> 对象的1个参数方法的调用，可以省略.和()

"hello world" indexOf w // 6

"hello world" substring 5 // "world"

Console print 10 // 但不能写 print 10，只能print(10)，省略Console.

1 + 2 // 相当于 (1).+(2)

> 对象的多个参数方法的调用,也可省略.但不能省略()：

"hello world" substring (0, 5) // "hello"


注意：

*  不在class或者object中的函数不能如此调用：

def m(i:Int) = i*i

m 10 // 错误

*  但在class或者object中可以使用this调用：

```
object method {

    def m(i:Int) = i*i

    def main(args: Array[String]) = {

        val ii = this m 15  // 等同于 m(15), this 不能省略

        println(ii)

    }

}
```
 

提示：这种调用方法最大的用途在于操作符重载，以及阅读性强的DSL。

例子：

    System exit 0

    Thread sleep 10

    Console println "hello world" // 省略Console则报错

    2 ** 10

    "-" * 100

 

3.1.6.  匿名函数

形式：((命名参数列表)=>函数实现)(参数列表)

特殊地：

*  无参数： (()=>函数实现)()

*  有一个参数且在最后： (函数实现)(参数)

*  无返回值： ((命名参数列表)=>Unit)(参数列表)

 
有参数的匿名函数（参数名称不能省）：

((i:Int)=> i*i)(3) // 9

((i:Int, j:Int) => i+j)(3, 4) // 7

 

有一个参数且在最后的：

(10*)(2) // 20, 相当于 ((x:Int)=>10*x)(2)

(10+)(2) // 12, 相当于 ((x:Int)=>10+x)(2)

(List("a","b","c") mkString)("=") // a=b=c

 
无参数的匿名函数：

(()=> 10)() // 10


无参数无返回值：

(() => Unit)

( ()=> {println("hello"); 20*10} )()

{ println("hello"); 20*10 }

 
例子1：

def m = (i:Int)=> i*i

m(3) // 9

List(1,2,3,4).map(m) // List(1, 4, 9, 16)


例子2：

def times3(m: ()=> Unit) = { m();m();m() }

times3 ( ()=> println("hello world") )

由于是无参数的匿名函数，可进一步简化：

def times3(m: =>Unit) = { m;m;m } // 参见“lazy参数”

times3 ( println("hello world") )
 
3.1.7.  偏函数

用下划线代替1+个参数的函数叫偏函数（partially applied function），如：

```

def sum(a:Int,b:Int,c:Int) = a+b+c

val p0 = sum _ // 正确

val p1 = sum(10,_,20) // 错误

val p2 = sum(10,_:Int,20) // 正确

val p3 = sum(_:Int,100,_:Int)

p0(1,2,3) // 6

p2(100) // 130

p3(10,1) // 111

或者：

  (sum _)(1 2 3) // 6

  (sum(1,_:Int,3))(2) // 6

  (sum(_:Int,2,_:Int))(1,3) // 6
  
```

从上面可以看出，partial函数是一个正常函数中抽出来的一部分（参数不全），故称为partial函数。

3.1.8.  “_”匿名参数

_：匿名函数中的匿名参数，同Groovy的it一样，groovy： 3.times { print it }；但也有差别：

![](https://github.com/moveondo/Scala/blob/master/image/7.jpg)
 

Java：

int add(int i, int j) { return i+j; }

add(3,4)

返回值、函数名和return关键字可以省略，变为：


scala 写法1：

((i:Int, j:Int) => i+j)(3, 4) // 7

i,j变量名可以省略，变为：

 

scala写法2：

    ((_:Int) + (_:Int))(3,4) // 7

    注意：括号(_:Int)括号是必须的

 

其他例子：

"Hello".exists(_.isUpper) // true

例子2：

def sum(x:Int,y:Int,z:Int) = x+y+z

val sum1 = sum _

val sum2 = sum(1000,_:Int,100)

sum1(1,2,3) // 6

sum2(5) // 1105


如果_在最后，还可以省略

1 to 5 foreach println

1 to 5 map (10*)

1 to 5 map (*10) // 错误！_不在最后不能省

3.1.9.  变长参数 *

变长参数只能放在最后一个，否则就歧义了，

def sum(ns:Int*, s:String) = ... // 错误

例子1：

![](https://github.com/moveondo/Scala/blob/master/image/8.jpg)


例子4：自定义loop

def loop2 (cond: =>Boolean)(body: =>Unit): Unit =

  if (cond) { body; loop2(cond)(body) }

调用：

var i=0; loop2 (i<3) (i+=1) // i=3

或者

var i=0; loop2 {i<3} {i+=1} // i=3

或者我们更习惯的方式:

var i=0; loop2 (i<3) {i+=1}

 
例子5：自定义loop..until

def loop3 (body: =>Unit): foo = new foo(body)

class foo(body: =>Unit) {

  def until(cond: =>Boolean) { body; if(!cond) until(cond) }

}

//调用：

var i = 0

loop3 { i+=1; println(i) } until (i>3)

 

3.2.     函数式编程

![](https://github.com/moveondo/Scala/blob/master/image/9.jpg)


3.2.2.  函数作为参数

采用映射式函数定义：

例子1：

def f(x:Int, y:Int, m:(Int, Int)=>Int) = m(x,y)

f(3,4, (x,y)=>x+y) // 7

f(3,4, (x,y)=>scala.math.sqrt(x*x+y*y)) // 5

例子2：

def f(x:Int, y:Int, m: =>Unit) = { println(x*y); m } // 参见“lazy参数”

f(3,4, println("end"))

// 12

// end

例子3：

  def f(f2:Int=>Int) = f2(5)

  f(100+) // 105, 100+是匿名函数 ((x:Int)=>100+x)的简写

3.2.3.  函数作为返回值

def f(s:String):(Int => String) = { n:Int=> s * n }

f("*") // res32: (Int) => String = <function1>

res32(10) // "**********"

f("*")(10) // 相当于先得到f("*")的返回值函数，再用该返回值函数调用一个参数

3.2.4.  Curry化

例如：

def sum(a:Int, b:Int) = { a + b } // sum(1, 2) = 3

Curry化后：

def sum(a:Int)(b:Int) = { a + b } // sum(1)(2) = 3

或者：

def sum(a:Int) = { (b:Int)=> a + b } // sum(1)(2) = 3

// 调用方式二：val t1 = sum(10);  val t2 = t1(20)

 

3.2.5.  递归

使用递归的原因：

其一是很多数学关系、逻辑关系本身就是递归描述（Hanoi、Fib）的；

其二是函数式编程不鼓励用变量循环，而是用递归。

递归包含递归出口和递归体：

递归出口:即递归的终止条件,比如0!=1, fib(1)=fib(2)=1, index==1000

递归体:即和前面或后面的值之间的关系,比如n! = n*(n-1)!, fib(n)=fib(n-2)+fib(n-1)


例子1：幂运算

如计算2^100, scala.math.pow(2,2000)越界，可用递归

def pow(n:BigInt, m:BigInt):BigInt = if (m==0) 1 else pow(n,m-1)*n


例子2：Hanoi汉诺塔

从a挪到c

![](https://github.com/moveondo/Scala/blob/master/image/10.jpg)


def move(n:Int, a:Int, b:Int, c:Int) = {

  if (n==1) println("%s to %s" format (a, c)) else {

move(n-1, a,c,b); move(1,a,b,c); move(n-1,b,a,c) }}

例子3：最大公约数

12  20

    20  12 (=12%20)

12  8 (=20%12)

        8  4 (=12%8)

           4  0 (=8%4)

def gcd(n:Int, m:Int):Int = if(m==0) n else gcd(m, n%m)

 
例子4：上台阶

上N级台阶，即可每次1步也可每次2步走，共有多少种不同走法

解法：迈出第一步有两种方法，第一步后就是N-1和N-2的走法了

def step(n:Int):Int = if (n<=2) n else step(n-1)+step(n-2)

推广：如果每次可走1步或2步或3步，则

def step(n:Int):Int = n match { case 1=>1; case 2=>2; case 3=>4;

case _ =>step(n-1)+step(n-2)+step(n-3) }

3.2.6.  尾-递归

定义：函数尾（最后一条语句）是递归调用的函数。

tail-recursive会被优化成循环，所以没有堆栈溢出的问题。


线性递归的阶乘：

def nn1(n:Int):BigInt = if (n==0) 1 else nn1(n-1)*n

println(nn1(1000)) // 4023...000

println(nn1(10000)) // 崩溃：（

 
尾递归的阶乘：

def nn2(n:Int, rt:BigInt):BigInt = if (n==0) rt else nn2(n-1, rt*n)

println(nn2(1000,1)) // 40...8896

println(nn2(10000,1)) // 2846...000


def nn3(n:Int):BigInt = nn2(n,1)


对比：

![](https://github.com/moveondo/Scala/blob/master/image/11.jpg)


例子：查找第10001个质数：

def prime(n:Int) = 2 to math.sqrt(n).toInt forall (n%_!=0)

def next(p:Int) = p+1 to 2*p find prime get

def f(p:Int, i:Int):Int = if (i==10001) p else f(next(p), i+1)

f(2,1) // 104743

或者使用Stream：

def f2(p:Int):Stream[Int] = p #:: f2(next(p))

f2(2).take(10001).last // 104743

 

例子：fib数列(1 1 2 3 5 8 13 21)

def fib2(n1:BigInt, n2:BigInt, i:Int, n:Int):BigInt =

  if (i==n) n1 else fib2(n2, n1+n2, i+1, n)

def fib(n:Int) = fib2(1,1,0,n)

0 to 10 map fib

fib(100-1) // 第100个fib数=354224848179261915075
}