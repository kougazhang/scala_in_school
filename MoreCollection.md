## 目录

+ 基础知识: 基本集合类型
+ 层次结构: 集合抽象层次
+ 方法
+ 可变集合
+ 与 Java 集合交互

Scala 中提供了一套很好的集合实现(*集合是函数式编程的基础, 所有的函数式编程语法对集合都提供了很好的支持*), 提供了一些集合类型的抽象.(*也就是如下数据类型都可以是集合, 方法是通用的*) 

## 基础知识

### List: 链表

```scala
// 普通方式定义
scala> List(1,2,3)
res9: List[Int] = List(1, 2, 3)
// 函数式方法定义
scala> 1 :: 2 :: 3 :: 4 :: Nil
res10: List[Int] = List(1, 2, 3, 4)

```

### Set: 集

集没有重复, 无序

```scala
>Set(1,2,2)
Set(1,2)
```

### Seq: 序列

序列有一个给定的顺序. Seq 是一个特质. List 是序列的实现. Seq 也是一个工厂单例对象, 可以用来创建列表.

```scala
scala> Seq(1,1,2)
// 返回值是 List, 说明是工厂单例对象.
res11: Seq[Int] = List(1, 1, 2)
```

### 映射: Map

Map 是键值容器.

```scala
Map('a'->1, 'b'->2)
```

## 层次结构

下面介绍的都是特质(trait), 在可变和不可变的(集合)包中都有对应的实现.

### Traversable: 所有集合可遍历

所有集合都可以被遍历。这个特质定义了标准函数组合子。 这些组合子根据 `foreach` 来写，所有集合必须实现。(*trait 就是接口, traversable 是一个用来遍历的接口*)

// todo 

好像这类 trait 已经不存在了, 变成了  TraversableOnce 等等.

### Iterable: 迭代

`iterator` 方法返回一个 Iterator 来迭代元素.

### Seq 序列

有顺序的对象序列.

### Set 集

没有重复的对象集合

### Map 

键值对

## 方法

### Traversable 

下面所有方法在子类中都是可用的. 参数和返回值的类型可能会因为子类的覆盖而不同.

**head: 获取头部元素**

函数签名:

```scala
def head: A
```

例子:

```scala
scala> val a = List(1,2,23,3,4)
a: List[Int] = List(1, 2, 23, 3, 4)
```

**tail: head 剩余部分都是尾巴**

语法模板:

```scala
def tail: Traversable[A]
```

例子:

```scala
scala> a.tail
res15: List[Int] = List(2, 23, 3, 4)
```

**map**

 映射. B 是返回值的类型; f 表示映射关系

```scala
def map [B] (f: (A) => B): CC[B]
```

**foreach**

在集合的每一个元素上执行 f.

foreach 类似于 map, 但是返回值是 Unit, 可能有副作用

```scala
def foreach[U](f: Elem => U): Unit
```

**find: 查找元素**

返回匹配谓词函数的第一个元素.

查找某个元素, 返回值的是容器 Option, 代表可能存在 Some 或 None.

语法签名:

```scala
def find(p: (A) => Boolean): Option[A]
```

例子:

```scala
scala> a.find(_ == 3)
res17: Option[Int] = Some(3)
```

**filter**

`filter` 返回匹配谓词函数的元素集合

语法模板:

```scala
def filter(p: (A) => Boolean): Traversable[A]
```

例子:

```scala
scala> a.filter(_<10)
res20: List[Int] = List(1, 2, 3, 4)
```

**patition: 划分**

语法模板:

```scala
def partition(p: (A) => Boolean): (Traversable[A], Traversable[A])
```

例子:

```scala
scala> a.partition(_<10)
res21: (List[Int], List[Int]) = (List(1, 2, 3, 4),List(23))
```

**groupBy**

`groupBy` 就是 SQL 中的 groupBy, 把集合中的数据按照某个标准进行分类.

语法模板:

```scala
// f 是分类的标准函数
// 返回值是 Map, K 是 f 的返回值, Traversable[A] 是符合这个标准的集合.
def groupBy[K](f: (A) => K): Map[K, Traversable[A]]
```

例子:

```scala
scala> a.groupBy(_ == 10)
res23: scala.collection.immutable.Map[Boolean,List[Int]] = Map(false -> List(1, 2, 23, 3, 4))

scala> a.groupBy(_ % 2)
res24: scala.collection.immutable.Map[Int,List[Int]] = Map(1 -> List(1, 23, 3), 0 -> List(2, 4))

scala> a.groupBy(_ % 3)
res25: scala.collection.immutable.Map[Int,List[Int]] = Map(2 -> List(2, 23), 1 -> List(1, 4), 0 -> List(3))
```

**转换**

你可以转换集合类型.

```scala
def toArray: Array[A]
// 元素改变了数据类型.
def toArray: [B >: A] (implicit arg0: ClassManifest[B]): Array[B]
def toBuffer [B >: A]: IndexedSeq[B]
def toIndexedSeq [B >: A]: IndexedSeq[B]
def toIterable: Iterable[A]
def toIterator: Iterator[A]
def toList: List[A]
def toMap [T, U] (implicit ev: <:<[A, (T,U)]): Map[T,U]
def toSeq: Seq[A]
def toSet [B >: A]: Set[B]
def toStream: Stream[A]
def toString(): String
def toTraversable: Traversable[A]
```

把映射转为一个数组, 会得到一个元素成员为 Tuple2 的数组.

```scala
scala> Map("A"->1,"B"->2).toArray
res31: Array[(String, Int)] = Array((A,1), (B,2))
```

### Iterable: 迭代器

语法签名: `def iterator: Iterator[A]`

迭代器支持的方法:

```scala
def hasNext(): Boolean
def next(): A
```

### Set: 集合

```scala
// 查看是否包含某元素
def contains(key: A): Boolean
// 并集
def +(elem: A): Set[A]
// 补集
def -(elem: A): Set[A]
```

### Map

**初始化**

```scala
// 将键值对列表传入, 使用语法糖 -> 
scala> Map("a" -> 1, "b" -> 2)
res37: scala.collection.immutable.Map[String,Int] = Map(a -> 1, b -> 2)
//
scala> Map(("a",1),("b", 2))
res38: scala.collection.immutable.Map[String,Int] = Map(a -> 1, b -> 2)
// 使用 ++ 操作符
scala> Map.empty ++ List(("a",1), "b"-> 2)
res39: scala.collection.immutable.Map[String,Int] = Map(a -> 1, b -> 2)
```

### 常用子类

HashSet 和 HashMap 都支持快速查找.

TreeMap 是 SortedMap 的一个子类, 支持有序访问. (即 Map 是有顺序的)

Vector 快速随机选择和快速更新.

```scala
scala> IndexedSeq(1,2,3,4)
res40: IndexedSeq[Int] = Vector(1, 2, 3, 4)
```

Range 等间隔的 Int 有序序列. 经常和 `for` 循环搭配使用.

```scala
// range 左右都是闭区间, 比 Python 设计的好, Python 是左闭右开.
scala> for ( i <- 1 to 10) { println(i) }
1
2
3
4
5
6
7
8
9
10
```

### 默认实现

使用特质的 `apply` 方法会给你默认实现的实例, 例如, Iterable(1, 2) 会返回一个列表作为其默认实现.

```scala
scala> Iterable(1,2)
res42: Iterable[Int] = List(1, 2)

scala> Seq(1,2)
res43: Seq[Int] = List(1, 2)
```

### 一些描述性的特质

IndexedSeq 快速随机访问元素和一个快速的长度操作.

LinearSeq 通过 head 快速访问第一个元素, 也有一个快速的 tail 操作.

**可变 vs 不可变**

+ 优点: 在多线程中不会改变;
+ 缺点: 一点也不能改变.

所以, 在 Scala 中鼓励的开发风格是, 优先使用不可变, 必须时再使用可变.

### 可变集合

#### HashMap

**添加元素: `+=`**

```scala
scala> import scala.collection.mutable.HashMap
import scala.collection.mutable.HashMap

scala> val h = HashMap((1,2))
h: scala.collection.mutable.HashMap[Int,Int] = Map(1 -> 2)

scala> h += 2->3
res45: h.type = Map(2 -> 3, 1 -> 2)
```

**获取, 如果不存在则更新**

```scala
scala> h.getOrElseUpdate(2, 6)
res48: Int = 3
```

#### ListBuffer

ListBuffer, ArrayBuffer 是可变集合

LiknedList, DoubleLinkedList, 链表和双向链表

PriorityQueue, 具有优先级的队列

Stack, ArrayStack, 栈

StringBuilder, 字符串

## 与 Java 互相转化

可以通过 `JavaConveryers Package` 轻松地在 Java 和 Scala 的集合类型之间转换. 它用 `asScala` 装饰常用的 Java 集合以和用 asJava 方法装饰 Scala 集合.

```scala
scala> import scala.collection.JavaConverters._
import scala.collection.JavaConverters._

scala> val sl = new scala.collection.mutable.ListBuffer[Int]
sl: scala.collection.mutable.ListBuffer[Int] = ListBuffer()

// 把 scala 集合转为 java 集合
scala> val j1:java.util.List[Int] = sl.asJava
j1: java.util.List[Int] = []

// 把 java 集合转为 scala 集合
scala> val sl2: scala.collection.mutable.Buffer[Int] = j1.asScala
sl2: scala.collection.mutable.Buffer[Int] = ListBuffer()
```

双向转换:

```scala
scala.collection.Iterable <=> java.lang.Iterable
scala.collection.Iterable <=> java.util.Collection
scala.collection.Iterator <=> java.util.{ Iterator, Enumeration }
scala.collection.mutable.Buffer <=> java.util.List
scala.collection.mutable.Set <=> java.util.Set
scala.collection.mutable.Map <=> java.util.{Map, Dictionary }
scala.collection.mutable.ConcurrentMap <=> java.util.concurrentMap
```

单向转换:

```scala
scala.collection.Seq => java.util.List
scala.collection.mutable.Seq => java.util.List
scala.collection.Set => java.util.Set
scala.collection.Map => java.util.Map
```

