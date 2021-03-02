## Effective Scala

### 格式化

**命名**

+ 对作用域较短的变量使用短名字. 

  + is, js 和 ks 可以出现在循环中.

+ 对作用域较长的变量使用长名字:

  + 外部 APIs 的名称应该长到不加注释也能理解其含义

+ 使用通用的缩写

+ 避免使用 ` 声明保留字.

  + 用 typ 代替 `type`

+ 用主动语态 (active) 来命名具有副作用的操作:

  + User.activate() 而非 user.setActive()

+ 对**有返回值的方法** 使用具有描述性的名字:

  + src.isDefined 而非 src.defined. *(区分了有返回值和没有返回值的方法, 这点很赞)*

+ Getters 不使用前缀 get:

  + site.count 而非 site.getCount

+ 不必重复已经被 package 或 object 封装过的名称:

  使用:

  ```scala
  object User{
    def get(id: Int): Option[User]
  }
  ```

  弃用:

  ```scala
  object User {
    def getUser(id: Int): Option[User]
  }
  ```

  **花括号**

  花括号用于创建复合表达式.

  **模式匹配**

  尽可能直接在函数定义的地方使用模式匹配:

  赞成:

  ```scala
  list map {
    case Some(x) => x
    case None => default
  }
  ```

  反对:

  ```scala
  list map { item =>
    item match {
      case Some(x) => x
      case None => default
    }
  }
  ```

  **类型别名**

  在提供便捷的命名或阐述意图时可以使用.

  ### 集合

  + 优先使用不可变集合.
    + 不可变集合默认线程安全, 易于并发

  **风格**

  集合支持链式操作, `a.b.c.d` , 这样会降低可读性, 办法增加中间变量:

  赞成:

  ```scala
   val votesByLang = votes groupBy { case (lang, _) => lang }
   val sumByLang = votesByLang map { case (lang, counts) =>
     val countsOnly = counts map { case (_, count) => count }
     (lang, countsOnly.sum)
   }
   val orderedVotes = sumByLang.toSeq
     .sortBy { case (_, count) => count }
     .reverse
  ```

  反对:

  ```scala
   val votes = Seq(("scala", 1), ("java", 4), ("scala", 10), ("scala", 1), ("python", 10))
   val orderedVotes = votes
     .groupBy(_._1)
     .map { case (which, counts) =>
       (which, counts.foldLeft(0)(_ + _._2))
     }.toSeq
     .sortBy(_._2)
     .reverse
  ```

  **性能**

  高阶集合库性能确实低, 等到发现问题在高级集合库时再换到低阶.

  高阶集合库（通常也伴随高阶构造）使推理性能更加困难：你越偏离直接指示计算机——即命令式风格——就越难准确预测一段代码的性能影响。然而推理正确性通常很容易；可读性也是加强的。在Java运行时使用Scala使得情况更加复杂，Scala对你隐藏了装箱(boxing)/拆箱(unboxing)操作，可能引发严重的性能或内存空间问题。(*推理性和性能是相悖的, 首先保证功能和可读性*)

  ### 并发

  **模块化的目标和资源管理相悖**

  现代服务是高度并发的—— 服务器通常是在10–100秒内并列上千个同时的操作——处理隐含的复杂性是创作健壮系统软件的中心主题。

  *线程* 提供了一种表达并发的方式：它们给你独立的，堆共享的(heap-sharing)由操作系统调度的执行上下文。然而，在Java里线程的创建是昂贵的，是一种必须托管的资源，通常借助于线程池。这对程序员创造了额外的复杂，也造成高度的耦合：很难从所使用的基础资源中分离应用逻辑。

  当创建高度分散(fan-out)的服务时这种复杂度尤其明显： 每个输入请求导致一大批对另一层系统的请求。在这些系统中，线程池必须被托管以便根据每一层请求的比例来平衡：一个线程池的管理不善会导致另一个线程池也出现问题。

  一个健壮系统必须考虑超时和取消，两者都需要引入更多“控制”线程，使问题更加复杂。注意若线程很廉价这些问题也将会被削弱：不再需要一个线程池，超时的线程将被丢弃，不再需要额外的资源管理。因此，资源管理危害了模块化。

  (*线程需要考虑线程池, 超时, 取消等问题, 这些问题是技术细节, 是阻碍我们模块化, 抽象业务逻辑的*)

  **Future**

  *因为线程的资源管理和模块化的需求相悖, 所以需要 Future 等高阶模块.*

  使用Future管理并发。它们将并发操作从资源管理里解耦出来：例如，Finagle（译注：twitter的一个RFC框架）以有效的方式在少量线程上实现并发操作的复用。Scala有一个轻量级的闭包字面语法(literal syntax)，所以Futures引入了很少的语法开销，它们成为很多程序员的第二本能。

  Futures允许程序员用一种可扩充的，有处理失败原则的声明风格，来表达并发计算。这些特性使我们相信它们尤其适合在函数式编程中用，这也是鼓励使用的风格。

  ### 控制结构

  为什么函数式编程不鼓励使用控制结构?

  函数式的风格倾向于使用更少的控制结构(*少用语句*), 并且使用声明式风格写的程序更易读.

  想使用声明式的风格, 需要打破你的逻辑(*因为总是会不自觉的使用 for 这类的来思考*), 要把这样的逻辑拆分到若干个小的方法或函数, 用匹配表达式 (match expression) 把他们粘到一起. 

  函数式风格也倾向于面向表达式: 条件分支是同一类型的值计算, `for (...) yield` 表达式, 以及 递归.

  ### 函数式编程

  **case 类模拟数据类型**

  把 case 类当做 Golang 里的 struct 使用.

  **Options**

  使用 Option 代替 null.

  **解构赋值**

  使用 `()` 进行解构赋值.

  但是要当心, 当成员数目不对时可能会异常. 所以文档是这么说的:

  解构绑定与模式匹配有关。它们用了相同的机制，但解构绑定可应用在当匹配只有一种选项的时候 (以免你接受异常的可能)。解构绑定特别适用于元组(tuple)和样本类(case class).

  ```scala
   val tuple = ('a', 1)
   val (char, digit) = tuple
  
   val tweet = Tweet("just tweeting", Time.now)
   val Tweet(text, timestamp) = tweet
  ```

  **flatmap**

  flatmap 的结果有时难以琢磨. *(我还以为是我没学会)*

### 面向对象编程

**trait**

trait 就是接口, 接口需要简短并且正交.