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

## Todo

scala 并发这部分介绍的太简要了. 

