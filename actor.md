7.   actor

http://www.scala-lang.org/docu/files/actors-api/actors_api_guide.html#


Scala中处理并发，有很多选择：

*  actor消息模型，类似Erlang，首选，Lift和akka也实现了自己的actor模型。

*  Thread、Runnable

*  java.util.concurennt

*  3rd并发框架如Netty，Mina

7.1.     actor模型

71.jpg


7.2.     多核计算

对比如下的算法：


72.jpg

上面是Java的写法，也可以用Scala的actor写法：


Scala写法1：

```

import actors.Actor,actors.Actor._

class A1 extends Actor {

def act { react { case n:Int=>println(perfect(n)) }}}

n to n+10 foreach (i=>{ (new A1).start ! i})

```

Scala写法2：

```

val aa = Array.fill(11)(actor { react { case n:Int=>println(perfect(n)) }})

n to n+10 foreach (i=>aa(i-n) ! i)

或者：

n to n+10 foreach (i=> actor { react { case n:Int=>println(perfect(n)) }} ! i)
```
 

7.3.     Actor用法

Scala会建立一个线程池共所有Actor来使用。receive模型是Actor从池中取一个线程一直使用；react模型是Actor从池中取一个线程用完给其他Actor用


实现方式1：

import scala.actors._

object Actor1 extends Actor { // 或者class

// 实现线程

def act() { react { case _ =>println("ok"); exit} }

}

//发送消息：

Actor1.start ! 1001 // 必须调用start

 

实现方式2：

import scala.actors.Actor._

val a2 = actor { react { case _ =>println("ok") } } // 马上启动

//发送消息：

a2 ! "message" // 不必调用start

 
73.jpg

7.4.     方式1：接受receive

特点：要反复处理消息，receive外层用while(..)
```
    import actors.Actor, actors.Actor._
    
    val a1 = Actor.actor {
    
    var work = true
    
    while(work) {
    
    receive { // 接受消息, 或者用receiveWith(1000)
    
    case msg:String => println("a1: " + msg)
    
    case x:Int => work = false; println("a1 stop: " + x)
    
    }
    
    }
    
    }

```
a1 ! "hello" // "a1: hello"

a1 ! "world" // "a1: world"

a1 ! -1 // "a1 stop: -1"

a1 ! "no response :("

7.5.     方式2：接受react, loop

特点：

*  从不返回

*  要反复执行消息处理，react外层用loop，不能用while(..);

*  通过复用线程，比receive更高效，应尽可能使用react

```

import actors.Actor, Actor._

val a1 = Actor.actor {

react {

        case x:Int => println("a1 stop: " + x)

        case msg:String => println("a1: " + msg); act()

    }

}

a1 ! "hello" // "a1: hello"

a1 ! "world" // "a1: world"

a1 ! -1 // "a1 stop: -1"

a1 ! "no response :("
```
 

如果不用退出的线程，可使用loop改写如下：
```

val a1 = Actor.actor {

  loop {

react {

case x:Int => println("a1 stop: " + x); exit()

        case msg:String => println("a1: " + msg)

    }

  }

}

```

7.6.     REPL接受消息

```
scala> self ! "hello"

scala> self.receive { case x => x }

scala> self.receiveWithin(1000) { case x => x }

```

7.7.     actor最佳实践

7.7.1.  不阻塞actor

actor不应由于处理某条消息而阻塞，可以调用helper-actor处理耗时操作(helper actor虽然是阻塞的，但由于不接受消息所以没问题)，以便actor接着处理下一条消息

```

import actors._, actors.Actor._

val time = 1000

  // （1）原来阻塞的程序

  val mainActor1 = actor {

    loop { react {

        case n: Int => Thread.sleep(time)

                         println(n)

        case s => println(s) } }

  }

  1 to 5 foreach { mainActor1 ! _ } // 5秒钟后打印完数字

 

  // （2）改写由helper actor去阻塞的程序

  val mainActor2: Actor = actor {

    loop { react {

        case n: Int => actor { Thread.sleep(time); mainActor2 ! "wakeup" }

                        println(n)

        case s => println(s) } }

  }

  1 to 5 foreach { mainActor2 ! _ } // 马上打印数字; 1秒钟后打印5个wakeup

```
-----------------------------------------

7.7.2.  actor之间用且仅用消息来通讯

actor模型让我们写多线程程序时只用关注各个独立的单线程程序（actor），他们之间通过消息来通讯。例如，如果BadActor中有一个GoodActor的引用:

class BadActor(a:GoodActor) extends Actor {...}

那在BadActor中即可以通过该引用来直接调用GoodActor的方法，也可以通过“!”来传递消息。选择后者！因为一旦BadActor通过引用读取GoodActor实例的私有数据，而这些数据可能正被其他线程改写值，结果就避免不了“共享数据-锁”模型中的麻烦事：即必须保证BadActor线程读取GoodActor的私有数据时，GoodActor线程在这块成为“共享数据”的操作上加锁。GoodActor只要有了共享数据，就必须来加锁防范竞用冲突和死锁，你又得从actor模型退回到“共享数据-锁”模型（注：actor对消息是顺序处理的，本来不用考虑共享数据）。

7.7.3.  采用不可变消息

Scala的actor模型让每个actor的act方法内部接近于单线程环境，你不用当心act方法里面的操作是否线程安全。在act方法中你可以尽情使用非同步、可变对象，因为每个act方法被有效限制在单个线程中，这也是actor模型被称为“share-nothing” 模型（零共享模型）的原因，其数据的作用范围被限制在单个线程中。不过一旦对象内的数据被用于多个actor之间进行消息传递。这时你就必须考虑消息对象是否线程安全。

保证消息对象线程安全的最好方法就是保证只使用不可变对象作为消息对象。消息类中只定义val字段，且只能指向不可变对象。定义这种不可变消息类的简单方法就是使用case class， 并保证其所有的val字段都是不可变的。Scala API中提供了很多不可变对象可用，例如基本类型、String、Tuple、List，不可变Set、不可变Map等。

如果你发现确实需要把一个可变对象obj1发送给其他actor，也因该是发送一份拷贝对象obj1.clone过去，而不是把obj1直接发过去。例如，数据对象Array是可变且未做同步的，所以Array只应该由一个actor同时存取，如果需要发送数组arr，就发送arr.clone（arr中的元素也应该是不可变对象），或者直接发送一个不可变对象arr.toList更好。

总结：大部分时候使用不可变对象很方便，不可变对象是并行系统的曙光，它们是易使用、低风险的线程安全对象。当你将来要设计一个和并行相关的程序时，无论是否使用actor，都应该尽量使用不可变的数据结构。

7.7.4.  让消息自说明

对每一种消息创建一个对应的case class，而不是使用上面的tuple数据结构。虽然这种包装在很多情况下并非必须，但该做法能使actor程序易于理解，例如：

// 不易理解，因为传递的是个一般的字符串，很难指出那个actor来响应这个消息

lookerUpper ! ("www.scala-lang.org", self)

// 改为如下，则指出只有react能处理LoopupIP的actor来处理：

case class LookupIP(hostname: String, requester: Actor)

lookerUpper ! LookupIP("www.scala-lang.org", self)


7.8.     不同jvm间的消息访问

服务器端：

```

object ActorServer extends Application {

    import actors.Actor, actors.Actor._, actors.remote.RemoteActor

    Actor.actor { // 创建并启动一个 actor

      // 当前 actor 监听的端口： 3000

      RemoteActor.alive(3000)

 

      // 在 3000 端口注册本 actor，取名为 server1。

      // 第一个参数为 actor 的标识，它以单引号开头，是 Scala 中的 Symbol 量，

      // Symbol 量和字符串相似，但 Symbol 相等是基于字符串比较的。

      // self 指代当前 actor （注意此处不能用 this）

      RemoteActor.register('server1, Actor.self)

 

      // 收到消息后的响应

      loop {

        Actor.react {case msg =>

          println("server1 get: " + msg)

        }

      }

    }

}

```

客户端：

  
```
object ActorClient extends Application { 

    import actors.Actor, actors.remote.Node, actors.remote.RemoteActor 

    Actor.actor { 

      // 取得一个节点（ip:port 唯一标识一个节点） 

      // Node 是个 case class，所以不需要 new 

      val node = Node("127.0.0.1", 3000) 

 

      // 取得节点对应的 actor 代理对象 

      val remoteActor = RemoteActor.select(node, 'server1) 

 

      // 现在 remoteActor 就和普通的 actor 一样，可以向它发送消息了！ 

      println("-- begin to send message")

      remoteActor ! "ActorClient的消息" 

      println("-- end")

    } 

}
```

7.9.     STM

http://nbronson.github.com/scala-stm/

a lightweight Software Transactional Memory for Scala, inspired by the STMs in Haskell and Clojure.

```
Cheat-Sheet:

import scala.concurrent.stm._
 
val x = Ref(0) // allocate a Ref[Int]
val y = Ref.make[String]() // type-specific default
val z = x.single // Ref.View[Int]
 
atomic { implicit txn =>
  val i = x() // read
  y() = "x was " + i // write
  val eq = atomic { implicit txn => // nested atomic
    x() == z() // both Ref and Ref.View can be used inside atomic
  }
  assert(eq)
  y.set(y.get + ", long-form access")
}
 
// only Ref.View can be used outside atomic
println("y was '" + y.single() + "'")
println("z was " + z())
 
atomic { implicit txn =>
  y() = y() + ", first alternative"
  if (x getWith { _ > 0 }) // read via a function
retry // try alternatives or block 
} orAtomic { implicit txn =>
  y() = y() + ", second alternative"
}
 
val prev = z.swap(10) // atomic swap
val success = z.compareAndSet(10, 11) // atomic compare-and-set
z.transform { _ max 20 } // atomic transformation
val pre = y.single.getAndTransform { _.toUpperCase }
val post = y.single.transformAndGet { _.filterNot { _ == ' ' } }
 
```
 

