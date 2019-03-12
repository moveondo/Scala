### 4.   OOP

### 4.1.     类class

### 4.1.1.  定义

例子1：

class User {

var name = "anonymous"

var age:Int = _

val country = "china"

def email = name + "@mail"

}

使用：

val u = new User

// var定义的属性可读可写

u.name = "qh"; u.age = 30

println(u.name + ", " + u.age) // "qh, 30"

// val 定义的属性只读不可写

u.country = "usa" // 报错

println(u.country) // "china"

// def 定义的是方法, 每次调用时重新计算

u.email // "qh@mail"

 
例子2：

// 定义

class Person(ln : String, fn : String, s : Person = null) {

  def lastName = ln;   // 用def定义后才是属性，ln，fn，s不可见

  def firstName = fn;

  def spouse = s;

  def introduction() : String =

    return ("Hi, " + firstName + " " + lastName) +

    (if (spouse != null) " and spouse, " + spouse.firstName + " " + spouse.lastName + "."

     else ".");

}

// 调用

new Person("aa","bb", new Person("cc","dd")).introduction();

### 4.1.2.  构造方法

class c1(x:String) // 等同于：class c1(private var x:String)

val o1 = new c1("aaa")

o1.x // 报错，因为是private的，定义成 class c1(var x:String) 才能这样用


例子1：

object construct1 {

class c1(name:String, age:Int) { // (1)直接在类定义处

    def this() { this("anonymous", 20) } // (2)用this定义

        def m1() = {  printf("%s=%d\n", name, age) }

    }

    def main(args:Array[String]) = {

new c1().m1()

new c1("qh", 30).m1()

}

}

编译：fsc construct1.scala

运行：java construct1

 

例子2：继承中的构造方法：

class c2(name:String, age:Int, female:Boolean=false)

    extends c1(name,age) {

    override def toString = { name + "," +  age + "," + female }

}


### 4.1.3.  override

不同于Java的使用 @Override，或者直接使用相同名字覆盖父类方法。

    override def toString = { name + "," +  age + "," + female }

如果是覆盖抽象方法，可以不用overriade关键字。

### 4.1.4.  object单例对象

如：

Java

```
public class User {

  private String name;

  private User(String name) { this.name=name; }

  public static User instance(String name) {

return new User(name)

  }

}
```

Scala

```
object User {    

  var name:String = _

  def apply(name:String){this.name=name; this}

  override def toString = "name: " + name

}

调用：

val u = User("qh") // "name: qh"

```


### 4.1.5.  静态方法

Scala没有静态方法，类似静态方法的函数定义在object中：

object Stringx {

    def left(s0:String, s:String) = ...

}

直接调用Stringx.left(s0, s)，或者 Stringx left (s0, s)

 

定义在object中的implicit方法也能被直接调用：

例如：

--------- ImportSub.scala

object ImportSub {

  def fac(n: Int) = 1 to n reduceLeft (_ * _)

  implicit def foo(n: Int) = new { def ! = fac(n) }

}

--------- ImportMain.scala

import ImportSub._

object ImportMain {

  def main(args : Array[String]) : Unit = {

      println(5!) // 调用ImportSub中定义的implicit函数

  }

}

### 4.1.6.  case class(条件类)

例如：

case class Person(name:String, age:Int)   

特殊之处：

*  新建类实例不用new Person(..)，直接用Person("qh",20)

*  自动定义好getXX方法，Person("qh",20).name // "qh"

*  提供默认的toString(), Person("qh",20) // "Person(qh,20)"

*  结合类继承可以通过模式匹配进行分解

例子1：

abstract class Person

case class Man(power:Int) extends Person

case class Woman(beauty:Int, from:String) extends Person


val w1 = Woman(100,"china")

val w2 = w1.copy(from="usa") // Woman(100,"usa")

 
def f(t:Person) = t match {

case Man(x) => "man's power:" + x

case Woman(x,y) => y + " beauty:" + x

}

f(Man(100)) // man's power:100

f(Woman(90, "china")) // china beauty:90


注：基本类型直接可以用math case


例子2：可变的类状态

case class C1(var s: String, var ops: Int) { 

  def >>= (f: (String=>String)) = {

    s = f(s)  // s改变

    ops += 1  // ops改变

    this // 返回自身，可以连续调用

  } 

}  

val C1(res, ops) = C1("ab", 0) >>= (_ * 3) >>= (_ drop 3)

// res="ab"->"ababab"->"bab", ops=0-> 0+1+1->2


例子3：用case class代替tuple

val p = ("qh",20) // p._1 = "qh", p._2 = 20；好处是简洁，但无意义

case class person(name:String, age:Int)

val p = person("qh",20) // p.name = "qh", p.age = 20; 好处是有名字，自说明，可读性强


例子4：用case class来描述元数据

xml的版本：

<todo name = "housework">

<item priority = "high">Clean the hose</item>

<item priority = "medium">Wash the dishes</item>

<item priority = "medium">Buy more soap</item>

</todo>

Lisp的版本：

(todo "housework"

 (item (priority high) "Clean the house")

 (item (priority medium) "Wash the dishes")

 (item (priority medium) "Buy more soap"))

Scala的版本：

    case class item(priority:String, s:String)

    case class todo(name:String, items:List[item])

    todo (name="housework",

items=item("high","Clean the house")::

        item("medium","Wash the dishes")::
        
             item("medium","Buy more soap")::Nil)

 

### 4.1.7.  case object(条件单例对象)

比如定义一个标识类（而不是字符串）：

case object Start

case object Stop

 

### 4.1.8.  枚举

Java中：

   enum fruits { apple, banana, cherry }
   
在Scala中，则是：

sealed abstract class Fruits // sealed类似于java的final

case object Apple extends Fruits

case object Banana extends Fruits

case object Cherry extends Fruits

也可以是 case class


### 4.1.9.  属性和Bean

例子1（直接定义和使用属性）：

class c {

var name = "anonymous" // var定义的是r/w的属性

val age = 20  // val定义的是只r属性

}

val o = new c

o.name = "qh"

o.name // "qh"

o.age = 10 // 错误

o.age // 20


例子2（定义get/set方法）：

class c2 {

@reflect.BeanProperty var name = "anonymous"

}

val o2 = new c2

o2.name = "qh" // 也可以直接存取

o2.name // "qh"

o2.setName("james") // 增加了set/get方法

o2.getName() // "james"

### 4.1.10.      反射

Scala没有太特别的反射机制，使用java的即可，不过Scala在match..case中可以匹配类型：

    case o:FooClass1 => ...

相关还有isInstanceOf[T], asInstanceOf[T]


例1（利用java的reflect）：

"hello".getClass.getMethods.map(_.getName).toList.sortWith(_<_).mkString(", ")

 

例子2：

classOf[String] // 相当于java中的String.class

"aaa".isInstanceOf[String] // true

"aaa".asInstanceOf[String]

### 4.2.  trait超级接口

注：trait  [treit] n.特征，特点，特性

和Java的Interface类似，但可以定义实体方法，而非仅仅方法定义

trait可以看作有方法实现和字段的interface；代表一类事物的特性；

比如

Tom，可能是Engine 和Son两个trait的混合；

Sunny可能Sales、Son和Father三个trait的混合；

当在运行时往Son里面增加方法或者字段的时候，Tom和Sunny都得到增加的特性。

4.2.1.  trait使用

trait Runnable {

  def run(): Unit;

}


只是用一个接口，就用extends:

class c1 extends Runnable {...}


2个接口（或一个继承一个接口），用with而不是implements如下：

class c1 extends c0 with Runnable {

    def run(): Unit = {...}

}


一个类可以组合多个trait：

class c1 extends t1 with t2 with t3 {...}

 

4.2.2.  mixin

class Human

class Child

 

trait Dad {

    private var children:List[Child] = Nil

    def add(child:Child) = child :: children

}

 

class Man1(name:String) extends Human with Dad // 静态mixin

class Man2(name:String) extends Human // 先不具备Dad trait

 

val m1 = new Man1("qh")

m1.add(new Child)

     

val m2 = new Man2("小孩")

//    m2.add(new Child) // 报错   

val m2$ = new Man2("james") with Dad // 动态mixin

m2$.add(new Child)

 
### 4.3.     协变和逆变(co-|contra-)variance

### 4.3.1.  概念

使用“+”“-”差异标记

Function[A, B]和Function[-A, +B]的区别图示：

![](https://github.com/moveondo/Scala/blob/master/image/12.jpg)

 

-A是A的子集，叫逆变

+B是B的超集，叫协变


### 4.3.2.  类型上下界
 
![](https://github.com/moveondo/Scala/blob/master/image/13.jpg)

### 4.3.3.  协变、逆变结合上下界

例子1：
![](https://github.com/moveondo/Scala/blob/master/image/14.jpg)

 
例子2：

// 非变

case class T1[T](e:T)

val v1:T1[java.lang.Integer] = new T1(100)

val v2:T1[java.lang.Integer] = v1

v2.e // 100

val v3:T1[java.lang.Number] = v1 // 报错

 

// 协变

case class T1[+T](e:T)

val v1:T1[java.lang.Integer] = new T1(100)

val v2:T1[java.lang.Integer] = v1

v2.e // 100

val v3:T1[java.lang.Number] = v1 // 合法

v3.e // 100

val v4:T1[java.lang.Integer] = v3 //非法

 

// 逆变

class T1[-T](e:T)

val v1:T1[java.lang.Number] = new T1(100)

val v2:T1[java.lang.Number] = v1

val v3:T1[java.lang.Integer] = v1 // 合法

val v4:T1[java.lang.Number] = v3 // 非法

