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

其他回调方法:

+ map
+ flatMap
+ foreach
+ filter
+ `foldLeft/foldRight`
+ andThen

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





## 