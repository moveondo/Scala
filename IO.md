### 6.IO

### 6.1.     文件I/O

### 6.1.1.  读文件

scala特有的是scala.io.Source，例如：

import scala.io._

Source.fromFile("cn.scala","utf8").mkString


逐行读文件内容：

Source.fromFile(new java.io.File("cn.scala")).getLines().foreach(println)


### 6.1.2.  写文件

直接调用java的io：

import java.io._, java.nio.channels._, java.nio._

// 写文件

val f = new FileOutputStream("o.txt").getChannel

f write ByteBuffer.wrap("a little bit long ...".getBytes)

f close


或者：

var out = new java.io.FileWriter("./out.txt") // FileWriter("./out.txt", true) 为追加模式

out.write("hello\n")

out close

 

### 6.1.3.  复制文件

直接调用java的io：

val in  = new FileInputStream("in").getChannel

val out = new FileOutputStream("out").getChannel

in transferTo (0, in.size, out)


### 6.1.4.  全目录扫描

递归使用listFiles：

  import java.io.File

  class Dir(file:File) {

    // 目录则返回所有1级子文件；文件则返回empty

def child = new Iterable[File] {

  // Iterable接口必须实现elements方法

      def elements = if (file.isDirectory) file.listFiles.elements else Iterator.empty

    }   

    // 递归扫描，组成列表

    def scan:Iterable[File] = {

      Seq.singleton(file) ++ child.flatMap(i=> new Dir(i).scan)

    }

  }

  // 定义一个隐式类型转换

  implicit def foo(f:File) = new Dir(f)

  val file = new File(".")

  Console println new File(".").getCanonicalPath

  val list = file.scan // File对象隐形转换成Dir对象

  for (f<-list) println(f)


### 6.2.     网络I/O

import java.net.{URL, URLEncoder}

import scala.io.Source.fromURL

fromURL(new URL("http://qh.appspot.com")).mkString

或者指定编码：

fromURL(new URL("http://qh.appspot.com"))(io.Codec.UTF8).mkString
 

