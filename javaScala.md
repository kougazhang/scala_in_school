## 目录

+ Javap
+ 类
+ 异常
+ 特质
+ 单例对象
+ 闭包和函数
+ 变化性

## Javap

+ `javap <fileName>.scala`, 把 scala 代码编译成 java 代码.
+ `javap -c <ClassName>\$class`, 查看生成的字节码.

## 类

在 Java 中使用 Scala 类时要考虑 4 个要点:

+ 类参数
+ 类常量
+ 类变量
+ 异常

例子:

```scala
import java.io.IOException
import scala.throws
// 教程里的 scala 版本比较老, 需要改为: scala.beans
import scala.beans.{BeanProperty, BooleanBeanProperty}

// BeanProperty 是注解, 转为 java 时会自动生成 getter setter 方法
class SimpleClass(name: String, val acc: String, @BeanProperty var mutable: String) {
  val foo = "foo"
  var bar = "bar"
  @BeanProperty
  val fooBean = "foobean"
  @BeanProperty
  var barBean = "barbean"
  @BooleanBeanProperty
  var awesome = true

  def dangerFoo() = {
    throw new IOException("SURPRISE!")
  }

  @throws(classOf[IOException])
  def dangerBar() = {
    throw new IOException("NO SURPRISE!")
  }
}
```

### 类参数

+ 默认情况下, 类参数都是 Java 构造函数的参数. 这意味着都是私有成员, 无法从类外部访问.
+ 声明一个类参数为 `val/var` 这段代码是相同的

```scala
// 带下划线的方法, 表示同一个含义的变量, 这是 haskell 的风格.
class SimpleClass(acc_: String) {
  val acc = acc_
}
```

这样操作, acc 在 Java 代码中就可以像其他常量一样被访问.

### 类常量

+ 常量(val) 在 Java 中定义一个获取方法. 你可以通过方法 "foo()" 访问 "val foo" 的值

### 类变量

+ 变量(var) 会生成一个 `_$eq` 方法, 你可以这样调用它, `foo$_eq("newfoo")`

### BeanProperty

你可以通过 `@BeanProperty` 注解 val 和 var 定义. 这会按照 POJO 定义生成 `getter/setter` 方法. 如果你想生成 `isFoo` 方法, 使用 `BooleanBeanProperty` 注解, 丑陋的 `foo$_eq` 将变成:

```scala
setFoo("newfoo");
getFoo();
```

## 异常

scala 也要加对应的注解, Java 中才能捕获对应的异常.

## 特质

先定义一个特质:

```scala
trait MyTrait {
  def traitName:String
  def upperTraitName = traitName.toUpperCase
}
```

 这个特质有一个抽象方法（traitName）和一个实现的方法（upperTraitName）。Scala为我们生成了什么呢？一个名为MyTrait的的接口，和一个名为MyTrait$class的实现类。

```scala
[local ~/projects/interop/target/scala_2.8.1/classes/com/twitter/interop]$ javap MyTrait
Compiled from "Scalaisms.scala"
public interface com.twitter.interop.MyTrait extends scala.ScalaObject{
    public abstract java.lang.String traitName();
    public abstract java.lang.String upperTraitName();
}
```

MyTrait$class更有趣

```scala
[local ~/projects/interop/target/scala_2.8.1/classes/com/twitter/interop]$ javap MyTrait\$class
Compiled from "Scalaisms.scala"
public abstract class com.twitter.interop.MyTrait$class extends java.lang.Object{
    public static java.lang.String upperTraitName(com.twitter.interop.MyTrait);
    public static void $init$(com.twitter.interop.MyTrait);
}
```

MyTrait$class只有以MyTrait实例为参数的**静态方法**。这给了我们一个如何在Java中来扩展一个特质的提示。

```scala
public class JTraitImpl implements MyTrait {
    private String name = null;

    public JTraitImpl(String name) {
        this.name = name;
    }

    public String traitName() {
        return name;
    }
}
```

我们会得到以下错误

```scala
[info] Compiling main sources...
[error] /Users/mmcbride/projects/interop/src/main/java/com/twitter/interop/JTraitImpl.java:3: com.twitter.interop.JTraitImpl is not abstract and does not override abstract method upperTraitName() in com.twitter.interop.MyTrait
[error] public class JTraitImpl implements MyTrait {
[error]   
```

也就是说找不到 `upperTraitName`

我们可以自己实现这个方法:

```scala
package com.twitter.interop;

    public String upperTraitName() {
        return MyTrait$class.upperTraitName(this);
    }
```

## 单例对象

单例对象是Scala实现静态方法/单例模式的方式。

一个Scala单例对象会被编译成由“$”结尾的类。让我们创建一个类和一个伴生对象

```scala
class TraitImpl(name: String) extends MyTrait {
  def traitName = name
}

// object TraitImpl 是一个单例对象
object TraitImpl {
  def apply = new TraitImpl("foo")
  def apply(name: String) = new TraitImpl(name)
}
```

我们可以这样在 Java 中访问:

```scala
MyTrait foo = TraitImpl$.MODULE$.apply("foo");
```

Java 实现:

```scala
local ~/projects/interop/target/scala_2.8.1/classes/com/twitter/interop]$ javap TraitImpl\$
Compiled from "Scalaisms.scala"
public final class com.twitter.interop.TraitImpl$ extends java.lang.Object implements scala.ScalaObject{
    public static final com.twitter.interop.TraitImpl$ MODULE$;
    public static {};
    public com.twitter.interop.TraitImpl apply();
    public com.twitter.interop.TraitImpl apply(java.lang.String);
}
```

其实它里面没有任何静态方法。取而代之的是一个名为MODULE$的静态成员。方法实现被委托给该成员。这使得访问代码很难看，但却是可行的。

## 闭包函数

```scala
class ClosureClass {
  def printResult[T](f: => T) = {
    println(f)
  }

  // f 是函数, 但是可以作为函数参数, 说明函数在 scala 中是 first-class
  def printResult[T](f: String => T) = {
    println(f("HI THERE"))
  }
}
```

生成的 java 代码.

```scala
[local ~/projects/interop/target/scala_2.8.1/classes/com/twitter/interop]$ javap ClosureClass
Compiled from "Scalaisms.scala"
public class com.twitter.interop.ClosureClass extends java.lang.Object implements scala.ScalaObject{
    public void printResult(scala.Function0);
    public void printResult(scala.Function1);
    public com.twitter.interop.ClosureClass();
}
```

这也不是那么恐怖。 “f: => T”被转义成“Function0”，“f: String => T”被转义成“Function1”。Scala实际上从Function0定义到Function22，最多支持22个参数。这真的应该足够了。*这就是 scala 支持 22 个参数的原因吧*

```scala
    @Test public void closureTest() {
        ClosureClass c = new ClosureClass();
        c.printResult(new AbstractFunction0() {
                public String apply() {
                    return "foo";
                }
            });
        c.printResult(new AbstractFunction1<String, String>() {
                public String apply(String arg) {
                    return arg + "foo";
                }
            });
    }
```

