## Runable/Callable

`Runable` 接口只有一个没有返回值的方法.

```scala
trait Runnable {
  def run(): Unit
}
```

`Callable` 与之类似, 除了它有一个返回值:

```scala
trait Callable[V] {
  def call(): V
}
```

## 线程

Scala 并发是建立在 Java 并发模型基础上的.

一个线程需要一个 Runable. 必须调用线程的 `start` 方法来运行 `Runnable`

```scala
scala> val hello = new Thread(new Runnable {
     | def run() {
     | println("hello world")
     | }
     | })
hello: Thread = Thread[Thread-2,5,main]

scala> hello.start
hello world
```

### 单线程代码

pass

### Executors

你可以通过 `Executors` 对象的静态方法得到一个 `ExecutorService` 对象。这些方法为你提供了可以通过各种政策配置的 `ExecutorService` ，如线程池。

```scala
import java.net.{Socket, ServerSocket}
import java.util.concurrent.{Executors, ExecutorService}
import java.util.Date

class NetworkService(port: Int, poolSize: Int) extends Runnable {
  val serverSocket = new ServerSocket(port)
  val pool: ExecutorService = Executors.newFixedThreadPool(poolSize)

  def run() {
    try {
      while (true) {
        // This will block until a connection comes in.
        val socket = serverSocket.accept()
        pool.execute(new Handler(socket))
      }
    } finally {
      pool.shutdown()
    }
  }
}

class Handler(socket: Socket) extends Runnable {
  def message = (Thread.currentThread.getName() + "\n").getBytes

  def run() {
    socket.getOutputStream.write(message)
    socket.getOutputStream.close()
  }
}

(new NetworkService(2020, 2)).run
```

### Futures

`Future` 代表异步计算. 你可以把你的计算包装在 Future 中, 当你需要计算结果的时候, 你需要调用一个阻塞的 `get()` 方法就可以了.

一个 `FutureTask` 是一个Runnable实现，就是被设计为由 `Executor` 运行的.

```scala
val future = new FutureTask[String](new Callable[String]() {
  def call(): String = {
    searcher.search(target);
}})
executor.execute(future)
现在我需要结果，所以阻塞直到其完成。

val blockingResult = Await.result(future)
```

## Futures

(*scala 并发这部分介绍的太简要了, 所以换个教程学习这部分*)

### Future

**Future 定义**

A Future is an object holding a value which may become available at some point.

Future 是一个容器类型, 表示异步 IO 操作的结果.

**Future 的状态**

1. 未完成: 如果在某个时间点计算(*一般是 io 操作吧*)没有完成,则 Future 是未完成的状态.
2. 已完成: io 操作完成了呗. 完成了有 2 种可能:
   1. 成功.
   2. 失败

**创建 Future**

```scala
// Future 需要隐式的 ExecutionContext
import scala.concurrent.ExecutionContext.Implicits.global

Future {
  timeCostWork()
}
```

**Future 为什么需要隐式的 ExecutionContext?**

ExecutionContext 的定义:

An ExecutionContext is similar to an Executor: it is free to execute computations in new thread, in a pooled thread or in the current thread.

定义中提到的 `Executor` 是 `java.util.concurrent.Executor`,  它是 Java 中线程管理的抽象. Java 这么做是为了把线程的管理和线程的执行分开, 让计算代码只考虑计算的业务问题而不理会线程的管理细节. 

ExecutionContext 是 java 的 Executor 在 Scala 的定义.(*哦, 也就是用来管理线程的呗*) 我们可以把 `ExecutionContext` 想象成一个自动管理的线程池, 把任务交给它就会运行, 我们只需要管理好业务逻辑即可. 

Scala 的 Future 底层是基于 JDK 的 ForkJoinPool, 它也是一种 Executor 的实现.

**Future 的 Callback 机制**

Java 的 Future 需要 get 来阻塞线程以获得结果, 相比之下 scala 的则不需要, 为什么呢? 因为回调机制.

Scala 允许开发者在一个 Future 上设置回调, 在任务完成后自动执行回调里的代码. 常见的回调方法: `OnComplete`

```scala
def onComplete[U](f: scala.Function1[scala.util.Try[T], U])(implicit executor: scala.concurrent.ExecutionContext): scala.Unit
```

这个方法接收一个 `Try` 参数, 返回一个 U 类型, 如果 Future 成功, 参数是 Success 否则是 Failure.

**访问 future 的结果**

+ `onSuccess`, 传入一个偏函数, 用模式匹配来处理你想要的结果
+ `onFailure`, 传入一个偏函数, 用模式匹配来处理异常.
+ `onComplete`, 传入一个 `Try[T]` 类型的函数

```scala
import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent.Future
import scala.util.{Success, Failure}

def computation(): Int = { 25 + 50 }
val theFuture = Future { computation() }

theFuture.onComplete {
  case Success(result) => println(result)
  case Failure(t) => println(s"Error: %{t.getMessage}")
}
```

其他回调方法:

+ map

```scala
object FuturesMap extends App {
  import scala.concurrent._
  import ExecutionContext.Implicits.global
  import scala.io.Source
  import scala.util.Success

  val buildFile: Future[Iterator[String]] = Future { Source.fromFile("build.sbt").getLines }
  // 通过map，Future[Iterator[String]] => Future[String] 的转变，找出最长的一行
  val longestBuildLine: Future[String]    = buildFile.map(lines => lines.maxBy(_.length))
  
  val gitignoreFile = Future { Source.fromFile(".gitignore-SAMPLE").getLines }
  val longestGitignoreLine = for (lines <- gitignoreFile) yield lines.maxBy(_.length)

  longestBuildLine onComplete {
    case Success(line) => log(s"the longest line is '$line'")
  }

  longestGitignoreLine.failed foreach {
    case t => log(s"no longest line, because ${t.getMessage}")
  }
}
```



+ flatMap
+ foreach
+ filter
+ `foldLeft/foldRight`
+ andThen

使用 for 组合多个 Future

参考: [链接](https://github.com/gongbp/scala-in-practice/blob/master/doc/13%20Scala%E4%B8%AD%E7%9A%84%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B%E4%B9%8B%20Future.markdown)

```scala
/**
 * Prefer for-comprehensions to using flatMap directly to make 
 * programs more concise and understandable.
 */
object FuturesFlatMap extends App {
  import scala.concurrent._
  import ExecutionContext.Implicits.global
  import scala.io.Source
  
  val netiquette = Future { Source.fromURL("http://www.ietf.org/rfc/rfc1855.txt").mkString }
  val urlSpec = Future {
    Source.fromURL("http://www.w3.org/Addressing/URL/url-spec.txt").mkString 
  }
  
  // 可读性好
  val answer = for {
    nettext <- netiquette
    urltext <- urlSpec
  } yield {
    "First of all, read this: " + nettext + " Once you're done, try this: " + urltext
  }

  answer foreach {
    case contents => log(contents)
  }

}
```

**异常处理**

```scala
object FuturesRecover extends App {
  import scala.concurrent._
  import ExecutionContext.Implicits.global
  import scala.io.Source

  val netiquetteUrl = "http://www.ietf.org/rfc/rfc1855.doc"
  val netiquette = Future { Source.fromURL(netiquetteUrl).mkString } recover {
    case f: java.io.FileNotFoundException =>
      "Dear boss, thank you for your e-mail." +
      "You might be interested to know that ftp links " +
      "can also point to regular files we keep on our servers."
  }

  netiquette foreach {
    case contents => println(contents)
  }

}
```



### Promise

**Promise 的定义**

Promise are objects that can be assigned  a value or an exception only once. This is only promises are sometimes also called single-assignment variables.

Promise 的关键特性是, 只可以赋值一次.

**Promise 的作用**

todo

### Future 和 Promise 的实现原理

**二者的关系**

A promise and a future represent two aspect of a single-assignment variable, the promise allows you to assign a value to the future object, whereas the future allows you to read the value .

简而言之, 你可以把 Promise 和 Future 想象成数据流的两端, 数据从 Promise 进, 从 Future 读. 因为 Promise 和 Future 都是只能赋值一次, 所以不存在脏读的问题. Scala 通过这两个工具实现了一个基本的并发编程.

**总结**

Scala 的 Future 本质是 Promise, Promise 是更高层次的抽象. 开发者通常接触的 Future , 如果要更深了解才是 Promise.

### async await

异步使用

```scala
object FuturesFlatMap extends App {
  import scala.concurrent._
  import ExecutionContext.Implicits.global
  import scala.io.Source
  import scala.async.Async.{async, await}
  
  val netiquette = Future { Source.fromURL("http://www.ietf.org/rfc/rfc1855.txt").mkString }
  val urlSpec = Future {
    Source.fromURL("http://www.w3.org/Addressing/URL/url-spec.txt").mkString 
  }
  
  // 异步方式
  val answer = async {
    "First of all, read this: " + await(netiquette) + 
    " Once you're done, try this: " + await(urlSpec)
  }

  answer foreach {
    case contents => log(contents)
  }

}
```

同步使用:

```scala
object FuturesFlatMap extends App {
  import scala.concurrent._
  import ExecutionContext.Implicits.global
  import scala.io.Source
  import scala.async.Async.{async, await}
  
  // 同步方式
  val answer = async {
    "First of all, read this: " + 
    await(Future { Source.fromURL("http://www.ietf.org/rfc/rfc1855.txt").mkString }) + 
    " Once you're done, try this: " + 
    await(Future {
      Source.fromURL("http://www.w3.org/Addressing/URL/url-spec.txt").mkString 
    })
  }

  answer foreach {
    case contents => log(contents)
  }

}
```

### 开发实例

```scala
/**
 * 一个List中的Future，有的是成功的，有的是异常的。
 * 普通的做法就是把 List[Future[A]] ==> Future[List[A]]，
 * 但是在这个过程中，如果有一个异常的Future，成功的Future全部被忽略，
 * 最早确定失败的Future最为最后结果返回。
 *
 * 这显然不是我们需要的。我们在实际的开发中，需要的是把List中成功的Future进行计算，
 * 把失败的所有的Future进行异常输出
 */
object FuturesWithException extends App {

  import scala.concurrent._
  import ExecutionContext.Implicits.global
  import scala.util.{Try, Success, Failure}

  log("this log is printed by main thread: begin")

  /**
   * List[Future[Int]] => List[Future[Try[Int]]]
   * @param f
   * @return
   */
  def future2FutureTry(f: Future[Int]): Future[Try[Int]] = 
    f.map(Success(_)).recover { case e => Failure(e) }

  // 把异常的Future的异常信息输出，正常的Future进行结果的计算。
  // 那么如何做呢？

  // 第一种方式
  val list = List(1, 2, 3, 4, 5)

  // 首先生成一个 List[Future[Int]]
  val listFutures: List[Future[Int]] = list map { num =>
    if (num % 2 == 0) Future(throw new RuntimeException(s"hello$num"))
    else Future(num * num)
  }

  // 然后把 List[Future[Int]] => List[Future[Try[Int]]]
  val listFutureTry = listFutures.map(future2FutureTry)

  // 再把 List[Future[Try[Int]]] => Future[List[Try[Int]]]
  val futureListTry = Future.sequence(listFutureTry)

  // 最后分别处理成功的Future和异常的Future
  futureListTry.map(_.collect { case Success(num) => log(s"num in future is $num") })
  futureListTry.map(_.collect { case Failure(e) => log(s"exception in future is $e") })

  log("this log is printed by main thread: end")
  Thread.sleep(4000)
}
```

第二种方法

```scala
object FuturesWithException2 extends App {

  import scala.concurrent._
  import ExecutionContext.Implicits.global
  import scala.util.{Try, Success, Failure}

  log("this log is printed by main thread: begin")

  // 把异常的Future的异常信息输出，正常的Future进行结果的计算。
  // 那么如何做呢？

  // 第二种方式，直接一步到位，生成Future[List[Try[Int]]]
  val list = List(1, 2, 3, 4, 5)

  val futureListTry: Future[List[Try[Int]]] = Future.traverse(list){num =>
   val future = if (num % 2 == 0)
      Future(throw new RuntimeException(s"hello$num"))
    else Future(num * num)

    future.map(Success(_)).recover{case e => Failure(e)}
  }

  // 最后分别处理成功的Future和异常的Future
  futureListTry.map(_.collect { case Success(num) => log(s"num in future is $num") })
  futureListTry.map(_.collect { case Failure(e) => log(s"exception in future is $e") })

  log("this log is printed by main thread: end")
  Thread.sleep(4000)
}
```





## 