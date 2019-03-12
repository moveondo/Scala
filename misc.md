### 8.   misc

### 8.1.     xml

### 8.1.1.  生成

Scala原生支持xml，就如同Java支持String一样，这就让生成xml和xhtml很简单优雅：

val name = "james"

val age = 10

val html = <html>name={name}, age="{age}"</html> toString

// <html>name=james, age=&quot;10&quot;</html>

又如：

val html = <html><head><title>{myTitle}</title></head><body>{"hello world"}</boty></html>

更复杂的例子：

val x = <r>{(1 to 5).map(i => <e>{i}</e>)}</r>

// <r><e>1</e><e>2</e><e>3</e><e>4</e><e>5</e></r>

val x0 = <users><user name="qh"/></users>

val <users>{u}</users> = x0  // u: scala.xml.Node = <user name="qh"></user>

### 8.1.2.  xml文件

xml.XML loadString "<p></p>"

xml.XML loadFile "abc.xml"

xml.XML.saveFull("foo.xml", node, "UTF-8", xmlDecl: Boolean, doctype : DocType)

### 8.1.3.  读取：

val x = <r>{(1 to 5).map(i => <e>{i}</e>)}</r>

// <r><e>1</e><e>2</e><e>3</e><e>4</e><e>5</e></r>

(x \ "e") map (_.text.toInt) // List(1, 2, 3, 4, 5)

val x0 = <users>

<user name="qh"><age>20</age></user>

<user name="james"><age>30</age></user>

</users>

(x0 \ "user") // <user name="qh"><age>20</age></user>, <user name="james"><age>30</age></user>)

(x0 \ "user" \ "age") // (<age>20</age>, <age>30</age>)

(x0 \ "age")  // 直接下级: ()

(x0 \\ "age") // 所有下级：(<age>20</age>, <age>30</age>)

(x0 \ "_") 所有

### 8.1.4.  访问属性

val x = <uu><u name="qh" /><u name="james" /><u name="qiu" /></uu>

(x \ "u" \\ "@name") foreach println // "qh\njames\nqiu"

例子：

```
val data =

<shopping>
  <item name="bread" quantity="3" price="2.50"/>
  <item name="milk" quantity="2" price="3.50"/>
</shopping>

val res = for (item <- data \ "item" ; 
                 price = (item \ "@price").text.toDouble ; 
                 qty = (item \ "@quantity").text.toInt)
           yield (price * qty)

printf("$%.2f\n", res.sum)
```

### 8.1.5.  格式化输出

val pp = new xml.PrettyPrinter(80, 4)  // 行宽 80，缩进为 4  

pp formatNodes <b><a/></b>  

结果是字符串 

<b> 

    <a></a> 

</b> 

 

### 8.2.     json

Scala-json

### 8.3.     Configgy

http://www.lag.net/configgy/

简单配置及logging：

----------------------------

log {

  filename = "/var/log/pingd.log"

  roll = "daily"

  level = "debug"

}


hostname = "pingd.example.com"

port = 3000

----------------------------


val conf = net.lag.configgy.Configgy.configure("/etc/pingd.conf").config

val hostname = conf.getString("hostname", "localhost")

val port = conf.getInt("port", 3000)

 

### 8.4.     正则表达式regex

例子1：完全匹配

//将字符串转为scala.util.matching.Regex

val pattern = "^[a-z0-9._%\\-+]+@(?:[a-z0-9\\-]+\\.)+[a-z]{2,4}$"

val emailRegex = pattern.r // 或者 new scala.util.matching.Regex(pattern)

//emailRegex.pattern=>java.util.regex.Pattern 使用java的Pattern

emailRegex.pattern.matcher("tt@16.cn").matches // true


例子2：查找匹配部分

val p = "[0-9]+".r  

p.findAllIn("2 ad 12ab ab21 23").toList // List(2, 12, 21, 23)

p.findFirstMatchIn("abc123xyz").get // scala.util.matching.Regex.Match = 123

 

更多例子如下：

定义：

val r1 = "(\\d+) lines".r  // syntactic sugar

val r2 = """(\d+) lines""".r  // using triple-quotes to preserve backslashes

或者

import scala.util.matching.Regex

val r3 = new Regex("(\\d+) lines")  // standard

val r4 = new Regex("""(\d+) lines""", "lines") // with named groups
 

search和replace（java.lang.String的方法）：

"99 lines" matches "(\\d+) lines" // true

"99 Lines" matches "(\\d+) lines" // false

"99 lines" replace ("99", "98") // "98 lines"

"99 lines lines" replaceAll ("l", "L") // 99 Lines Lines
 

search（regex的方法）：
```

"\\d+".r findFirstIn "99 lines" // Some(99)
"\\w+".r findAllIn "99 lines" // iterator(长度为2)
"\\s+".r findPrefixOf "99 lines" // None
"\\s+".r findPrefixOf "  99 lines" // Some(  )
val r4 = new Regex("""(\d+) lines""", "g1") // with named groups
r4 findFirstMatchIn "99 lines-a" // Some(99 lines)
r4 findPrefixMatchOf "99 lines-a" // Some(99 lines)
val b = (r4 findFirstMatchIn "99 lines").get.group("g1") // "99"
 ```

match（regex的方法）：

```

val r1 = "(\\d+) lines".r
val r4 = new Regex("""(\d+) lines""", "g1")
val Some(b) = r4 findPrefixOf "99 lines" // "99 lines"

for {

  line <- """|99 lines-a

             |99 lines

             |pass

             |98 lines-c""".stripMargin.lines

} line match {

  case r1(n) => println("Has " + n + " Lines.") // "Has 99 Lines."
  case _ =>

}
 
for (matched <- "(\\w+)".r findAllIn "99 lines" matchData)

  println("Matched from " + matched.start + " to " + matched.end)

```
输出：

Matched from 0 to 2

Matched from 3 to 8

 

replace（regex的方法）：

```

val r2 = """(\d+) lines""".r  // using triple-quotes to preserve backslashes

r2 replaceFirstIn ("99 lines-a", "aaa") // "aaa-a"

r2 replaceAllIn ("99 lines-a, 98 lines-b", "bbb") // "bbb-a, bbb-b"
```

其他：使用正则表达式定义变量

```

val regex = "(\\d+)/(\\d+)/(\\d+)".r

val regex(year, month, day) = "2010/1/13"

// year: String = 2010

// month: String = 1

// day: String = 13
```

### 8.5.     GUI

### 8.5.1.  java方式

import javax.swing.JFrame

var jf = new JFrame("Hello!")

jf.setSize(800, 600)

jf.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE)

jf.setVisible(true)
 

### 8.5.2.  scala方式

import swing._, swing.Swing._

object Swing1 extends SimpleGUIApplication { // scala2.7

```

  def top = new MainFrame { // 必须实现top方法

    title = "窗口1"

    preferredSize = (400, 300)

   

    val label = new Label("hello world cn中文")

    contents = new BorderPanel {

      layout(label) = BorderPanel.Position.Center

    }

  }

}
```
 

