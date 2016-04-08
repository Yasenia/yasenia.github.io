---
title: 深入理解 Monad
date: 2016-04-08 12:14:54
tags: [fp, monad, scala]
---


在函数式编程中，Monad 是一个重要的抽象概念。它为我们提供了一个更高的抽象层次来处理多态类型，在编码实践中有着广泛的应用。

然而，Monad 到底是什么，它能做什么，如何使用它？网上相关介绍的文章（尤其是中文）很少，而且其中大部分都是泛泛而谈，少部分又讳莫如深。这使得初次接触这个领域的人面临一个很高的学习门槛。

关于 Monad，有这样一句广为流传的定义：

> *A monad is just a monoid in the category of endofunctors, what's the problem?*

对于 Wadler 这样的大神级人物而言，或许仅仅一句话就能解释 Monad 的本质，但对于大多数人而言，看到这样的定义只会更加的迷惑不解吧。

本文将基于 scala 语言，从范畴论的简单基础谈起，介绍 Monad 等概念的定义，进而讨论 Monad 与范畴论之间的联系。

目录：

1. *代数系统简介——半群、幺半群与群*
2. *Semigroup 与 Monoid 特质*
3. *Functor 特质*
4. *Applicative 特质*
5. *Monad 特质*
6. *Monad 与 Monoid 的联系*

## 1. 代数系统简介——半群、幺半群与群
在正式讨论 Monad 及其相关概念之前，本文将先介绍一些基本的代数学知识进行铺垫。这并不需要你有范畴论、离散数学等相关方面的专业知识，只需要对下面几种概念有个大致的了解即可。

   - 半群(Semigroup)
      
      > 给定代数系统 $V =(S, ⊙)$，其中 $S$ 为非空集合，$⊙$ 为作用在 $S$ 上的二元运算。若对任意的 $a, b, c ∈ S$ ，总有 $a ⊙ b ∈ S$ 且 $(a ⊙ b) ⊙ c = a ⊙ (b ⊙ c)$。则称 $V$ 是一个半群。
      
   - 幺半群(Monoid)
      
      > 给定半群 $M = (S, ⊙)$。若存在 $e ∈ S$，对任意的 $x ∈ S$，总有 $e ⊙ x = x ⊙ e = e$。则称 $M$ 是一个幺半群,也记做 $M=(S, ⊙, e)$。
      
   - 群(Group)
      
      > 给定幺半群 $G = (S, ⊙, e)$。若对任意的 a ∈ S，总存在 $b ∈ S$，使得 $a ⊙ b = e$ 成立。则称 $G$ 是一个群。

下表对比了半群、幺半群与群所拥有的特性：

|特性    |半群(Semigroup)|幺半群(Monoid)|群(Group)|
|:-----:|:------------:|:-----------:|:-------:|
|封闭性  |√             |√            |√        |
|结合律  |√             |√            |√        |
|单位元  |×             |√            |√        |
|逆元    |×             |×            |√        |

## 2. Semigroup 与 Monoid 特质
根据半群与幺半群的定义，我们可以很容易的给出它们对应数据结构特质的实现：

``` scala
trait Semigroup[A] {
  def op(a1: A, a2: A): A
}
trait Monoid[A] extends Semigroup[A] {
  val zero: A
}
```

由于幺半群是一种特殊的半群（具有幺元的半群），因此在这里特质`Monoid`继承自特质`Semigroup`。

需要注意的是，在这里我们只是定义了两个代数系统对应的数据结构，这并不意味着所有实现了`Semigroup`(或`Monoid`)特质的类型就是半群（或幺半群）。以`Monoid `特质为例，我们仅在特质里定义了幺半群中的二元运算`op`与单位元`zero`的存在，但并不保证二元运算满足结合律且单位元满足不变性，即不能保证以下断言可以运行通过：
  
   - 结合律断言 `assert(m.op(m.op(a, b), c) == m.op(a, m.op(b, c)))`
   - 不变性断言 `assert(m.op(m.zero, a) == a && m.op(a, m.zero) == a)`

这两点需要我们在具体实现的代码中书写正确的逻辑来确保。

这里给出几个`Monoid`的简单实现：

``` scala
// M(Int, +, 0)
val intAdditionMonoid: Monoid[Int] = new Monoid[Int] {
  override val zero: Int = 0
  override def op(a1: Int, a2: Int): Int = a1 + a2
}
// M(Int, *, 1)
val intProductMonoid: Monoid[Int] = new Monoid[Int] {
  override val zero: Int = 1
  override def op(a1: Int, a2: Int): Int = a1 * a2
}
// M(String, +, "")
val stringConcatMonoid: Monoid[String] = new Monoid[String] {
  override val zero: String = ""
  override def op(a1: String, a2: String): String = a1 + a2
}
```

## 3. Functor 特质
我们见过许多泛型类，如`List[A]`、`Option[A]`等。在大多数 Monad 相关的教程中，人们总是把这样的泛型类比喻成“盒子”，这个比喻当然很精妙，但也不总是那么贴切，不过在这里我们暂时沿用它。

假设我们有一个`F[A]`类型的数据，我们可以知道`F`这个盒子里的数据是`A`类型的（事实上并不一定是这样，准确的说应该是盒子的数据形态依赖于`A`，这也是为什么说“盒子”的比喻不总是贴切的），同时我们有一个函数`f: A => B`，它接收`A`类型的数据并返回`B`类型的数据。如果我们希望通过某种途径，基于`f: A => B`和`F[A]`得到一个新的数据结构`F[B]`，应该怎么做呢？

把大象放进冰箱只需要三步：打开冰箱 → 把大象放进去 → 关闭冰箱。我们的问题看上去复杂一些：打开盒子 → 取出a → 把a变成b → 把b放进盒子里 → 关上盒子。五步走可以解决吗？

事实上，在大多数情况下，我们并不关心变换是如何实现的，因为变换操作与具体的`F`类型有关。好的盒子我们甚至能“隔空取物”去改变它里面的数据，而一个不好的盒子，或许根本就无法实现这样的操作。我们需要关心的是，这个盒子，能否支持这样的操作。

对于一个`F`，如果我们能找到一个函数`map`，它接收`F[A]`和`f: A => B`作为参数并返回`F[B]`，并且当`f = a => a` 时，`map`函数返回值与接收的`F[A]`相同，则称这个`map`为类型`F`的函子(Functor)。

通过上面的定义，我们给出`Functor`特质的实现：

```scala
trait Functor[F[_]] {
  def map[A, B](fa: F[A])(f: A => B): F[B]
}
```

和前文中提到的`Semigroup`与`Monoid`一样，实现了特质`Functor[F]`并不等价于得到了一个`F`的函子，还需要`map`函数满足断言：`assert(map(fa)(a => a) == fa)`。

我们把所有的类型共同构成的集合称作一个范畴$C1$，每一个类型`x`对应的`F[x]`构成了另一个范畴$C2$。在范畴$C1$中，像`A => B`这样的类型转换被称作`A`到`B`类型的*态射*。可以看到，通过`F[_]`这样的类型构造器，我们可以将范畴$C1$中的`x`类型转换成范畴$C2$中的`F[x]`类型，而借助`Functor[F]`，我们可以将范畴$C1$中的`x => y`态射转换成范畴$C2$中的`F[x] => F[y]`态射。我们把这两种范畴$C1$到范畴$C2$的变换成为*函子变换*。`Functor`特质和`F[_]`的类型构造器共同组成了类型`F`上的函子（这也是为什么在部分书籍资料中，`Functor[F]`特质里还会有一个`A => F[A]`函数的原因，这个函数对应了`F[_]`的类型构造器）。

下面给出`Functor[List]`与`Functor[Option]`的实现：

```scala
val listFunctor: Functor[List] = new Functor[List] {
  override def map[A, B](as: List[A])(f: A => B): List[B] = as map f
}
val optionFunctor: Functor[Option] = new Functor[Option] {
  override def map[A, B](oa: Option[A])(f: (A) => B): Option[B] = oa map f
}
```


我们可以在`Functor`特质中定义一些基于`map`函数的衍生函数。这样，凡是具有`Functor[F]`实现的`F`类型数据，我们都可以用这些衍生函数来进行操作。下面给出了一个扩展了`distribute`函数的`Functor`特质：

```scala
trait Functor[F[_]] {
  def map[A, B](fa: F[A])(f: A => B): F[B] 
  // 将 F[(A, B)] 类型数据分解成 F[A] 与 F[B]
  def distribute[A, B](fab: F[(A, B)]): (F[A], F[B]) = (map(fab)(_._1), map(fab)(_._2))
}

```

现在，我们可以使用`distribute`函数来处理`List`了（当然`Option`也一样）：

```scala
scala> val pairList = List((1, "one"), (2, "two"), (3, "three"))
pairList: List[(Int, String)] = List((1,one), (2,two), (3,three))

scala> val (intList, stringList) = listFunctor.distribute(pairList)
intList: List[Int] = List(1, 2, 3)
stringList: List[String] = List(one, two, three)
```

## 4. Applicative 特质
函子(Functor)为我们处理抽象的泛型数据提供了一个有力的工具。通过定义基于`map`的衍生函数，让我们得到更高层次抽象出来的组合子(combinator)。然而，函子只要求实现了一个`map`函数，基于它我们虽然能衍生出许多实用的函数，但仍然有许多操作我们难以实现。

我们重新审视上一节中基于`map`得到的衍生函数`distribute`，这个函数接收了一个`F[(A, B)]`类型的参数并将其分解，返回`(F[A], F[B])`类型的数据。那么，我们能否编写一个新函数`combine`，将两个`F[A]`、`F[B]`类型的数据重新组合成`F[(A, B)]`类型的数据呢？我们先看看`combine`函数的签名：

```scala
def combine[A, B](fa: F[A], fb: F[B]): F[(A, B)]
```

事实上，如果我们手上只有`Functor`特质中提供的`map`函数作为基础，是无法实现`combine`这样的函数的。`map`只提供了`F[A] => (A => B) => F[B]`这样一对一的映射函数，而解决我们的问题，则需要`(F[A], F[B]) => ((A, B) => C) => F[C]`这样二对一的映射函数。我们称它为`map2`，签名如下：

```scala
def map2[A, B, C](fa: F[A], fb: F[B])(f: (A, B) => C): F[C]
```

基于`map2`，我们可以轻易的实现`combine`函数：

```scala
def combine[A, B](fa: F[A], fb: F[B]): F[(A, B)] = map2(fa, fb)((_, _))
```

现在，我们手上拥有了`map`和`map2`两样“原料”，它们之间有什么区别和联系呢？事实上，`map2`函数比`map`函数更为强大，因为我们可以基于`map2`函数来实现`map`，而反过来却不可以！

```scala
def map[A, B](fa: F[A])(f: A => B): F[B] = map2(fa, fa)((a, _) => f(a))
```

这里，我们给出一个继承自`Functor`特质的`PowerfulFunctor`，在这个新特质中，我们定义了`map2`函数，并用它重写了`map`函数：

```scala
trait PowerfulFunctor[F[_]] extends Functor[F] {
  override def map[A, B](fa: F[A])(f: A => B): F[B] = map2(fa, fa)((a, _) => f(a))
  def map2[A, B, C](fa: F[A], fb: F[B])(f: (A, B) => C): F[C]
}
```

要实现`PowerfulFunctor`，只需要实现`map2`函数即可，对于具有`PowerfulFunctor`的类型`F`，我们除了可以进行`Functor[F]`所提供的操作外，还可以基于`map2`函数实现更多的功能（比如`combine`函数等）。

可以看到，`PowerfulFunctor`特质比`Functor`特质要强大。但在实际中，仍然有许多操作，仅仅通过`map2`函数是无法实现的。

重新回到我们把`F`比作盒子的假说，考虑一件最简单的事情：怎样把数据放到这个盒子里去？对`F`进行操作，要通过一个`A`类型的数据来得到一个`F[A]`类型的数据似乎是一件再普遍不过的操作了。然而就是这个简单的操作，无论是`Functor`或是更强大的`PowerfulFunctor`都无法办到。这里我们需要一个简单的打包函数，签名如下：

```scala
def unit[A](a: A): F[A]
```

现在，我们重新定义一个名为`Applicative`的特质，该特质定义了`unit`与`map2`函数，我们知道，`Applicative`是一个更为强大的`Functor`，可以看做是`Functor`的一个拓展，因此，我们让`Applicative`继承自`Functor`，并在其中实现了一个基于`unit`与`map2`函数衍生出来的函数`sequence`:

```scala
trait Applicative[F[_]] extends Functor[F] {
  override def map[A, B](fa: F[A])(f: A => B): F[B] = map2(fa, unit(()))((a, _) => f(a))
  def unit[A](a: A): F[A]
  def map2[A, B, C](fa: F[A], fb: F[B])(f: (A, B) => C): F[C]
  // 将 List[F[A]] 类型数据重组成 F[List[A]]
  def sequence[A](fas: List[F[A]]): F[List[A]] = fas.foldRight(unit(List.empty[A]))(map2(_, _)(_ :: _))
}
```

以`Option`为例，我们给出`Applicative[Option]`的实现：

```scala
val optionApplicative: Applicative[Option] = new Applicative[Option] {
  override def unit[A](a: A): Option[A] = Some(a)
  override def map2[A, B, C](fa: Option[A], fb: Option[B])(f: (A, B) => C): Option[C] = for {
    a <- fa
    b <- fb
  } yield f(a, b)
}
```

让我们在 REPL 中测试一下`Applicative`为我们提供的`sequence`函数：

```scala
scala> val optionList1 = List(Some("Functor"), Some("Applicative"), Some("Monad"))
optionList1: List[Some[String]] = List(Some(Functor), Some(Applicative), Some(Monad))

scala> val optionList2 = List(Some("Semigroup"), Some("Monoid"), None)
optionList2: List[Option[String]] = List(Some(Semigroup), Some(Monoid), None)

scala> optionApplicative.sequence(optionList1)
res0: Option[List[String]] = Some(List(Functor, Applicative, Monad))

scala> optionApplicative.sequence(optionList2)
res1: Option[List[String]] = None
```

## 5. Monad 特质
`Applicative`特质无疑比`Functor`特质为我们实现更多更强大的函数提供了更好的基础，然而它依旧有那么一点缺陷。

假设我们有一个被`F`类型包装的数据`fa: F[A]`，一个函数`f: A => F[B]`，如果执行`map(fa)(f)`操作，我们将得到一个`F[F[A]]`类型的对象。在很多情况下，这个结果或许并不是我们想要的，比如一个`Option[Option[A]]`。我们不想一层层的去打开这个盒子然后看看里面到底装了什么，只想要一个单层的`Option[A]`。

在这种场景下，我们需要这样的一个函数`flatMap`，它的签名如下：

```scala
def flatMap[A, B](fa: F[A])(f: A => F[B]): F[B]
```

我们已经在`Applicative`特质定义了两个函数`unit`和`map2`，现在是不是意味着我们需要再往里面添砖加瓦，新增一个`flatMap`呢？其实我们可以看到，有了`flatMap`之后，`map2`函数已经是多余的了，我们可以用`unit`函数与`flatMap`函数来实现`map2`:

```scala
def map2[A, B, C](fa: F[A], fb: F[B])(f: (A, B) => C): F[C] = flatMap(fa)(a => flatMap(fb)(b => unit(f(a, b))))
```

现在，我们可以重新定义一个特质，它继承自`Applicative`特质，定义了一个新的`flatMap`函数，并且实现了`Applicative`特质中的`map2`函数。这个特质，就是`Monad`，下面给出了`Monad`特质的实现，并在其中添加了一个基于`flatMap`衍生的函数`compose`：

```scala
trait Monad[F[_]] extends Applicative[F] {
  override def unit[A](a: A): F[A]
  override def map2[A, B, C](fa: F[A], fb: F[B])(f: (A, B) => C): F[C] = flatMap(fa)(a => flatMap(fb)(b => unit(f(a, b))))
  def flatMap[A, B](fa: F[A])(f: A => F[B]): F[B]
  def compose[A, B, C](f: A => F[B], g: B => F[C]): A => F[C] = a => flatMap(f(a))(g)
}
```

同样的，我们也可以实现一个`Monad[List]`

```scala
val listMonad: Monad[List] = new Monad[List] {
  override def unit[A](a: A): List[A] = List(a)
  override def flatMap[A, B](as: List[A])(f: (A) => List[B]): List[B] = as flatMap f
}
```
在 REPL 中测试`Monad`提供的`compose`函数：

```scala
scala> val f: String => List[Int] = s => s.toList.map(_.toString.toInt)
f: String => List[Int] = <function1>

scala> val g: Int => List[Int] = n => List.fill(n)(n)
g: Int => List[Int] = <function1>

scala> val h = listMonad.compose(f, g)
h: String => List[Int] = <function1>

scala> h("123")
res1: List[Int] = List(1, 2, 2, 3, 3, 3)
```

## 6. Monad 与 Monoid
早在第1、2节中，我们就介绍过幺半群的概念并给出了一个`Monoid`特质。相信每一位FP的初学者都会心存疑问：Monad 和 Monoid，这两者之间到底有什么联系？它们的名字如此相似，难道只是巧合吗？

通过前面的介绍，我们知道，对于`F`类型而言，Monad 是由定义在`F`上的两个基本函数`unit`与`flatMap`共同组成的。

上一节中，我们在`Monad`特质里基于`flatMap`实现了一个`compose`函数。仔细研究一下`compose`就会发现，我们也可以通过`compose`函数来实现`flatMap`函数：

```scala
def flatMap[A, B](fa: F[A])(f: A => F[B]): F[B] = compose((_ : Unit) => fa, f)(())
```

这意味着，`compose`函数与`flatMap`函数是等价的。我们完全可以用`compose`函数来取代`faltMap`函数，并基于`unit`函数与`compose`函数衍生出`flatMap`、`map2`、`map`以及更多其它的函数。我们得到一个 Monad 的等价定义，这里给出`MonadEq`特质的实现：

```scala
trait MonadEq[F[_]] extends Applicative[F] {
  override def unit[A](a: A): F[A]
  override def map2[A, B, C](fa: F[A], fb: F[B])(f: (A, B) => C): F[C] = flatMap(fa)(a => flatMap(fb)(b => unit(f(a, b))))
  def compose[A, B, C](f: A => F[B], g: B => F[C]): A => F[C]
  def flatMap[A, B](fa: F[A])(f: A => F[B]): F[B] = compose((_ : Unit) => fa, f)(())
}
```

可以看到，`MonadEq`与`Monad`特质具有同样强大的功能。所有我们通过`Monad`能完成的操作，都可以通过`MonadEq`来实现。

我们把`MonadEq`特质中多余的衍生函数去掉，不再令其继承自`Applicative`特质，并将`unit`函数与`compose`函数稍加变形，得到一个新的特质`Mon`如下：

```scala
trait Mon[F[_]] {
  val unit[A]: A => F[A]
  def compose[A, B, C](f: A => F[B], g: B => F[C]): A => F[C]
}
```

是不是感觉到很眼熟？再看看我们的`Monoid`特质（为了方便比较，这里不再让`Monoid`特质继承自`Semigroup`特质）：

```scala
trait Monoid[A] {
  val zero: A
  def op(a1: A, a2: A): A
}
```

可以看到，`Mon`中的`unit`和`compose`函数极好的吻合了`Monoid`中的`zero`与`op`！

现在我们再重新理解 Wadler 说过的那句关于 Monad 的定义：
> “Monad 不过是一个自函子范畴上的幺半群而已，有什么问题吗？”

`A => F[B]`正是一个从范畴$C1$到范畴$C2$的函子变换。当我们把这样的函子变换构成的集合看做一个新的范畴$C$时，可以非常明显的看出来，`Monad[F]`就是范畴$C$上的幺半群！

把 `Monad[F]` 看成以一个幺半群$M = (S, ⊙, e)$，在这个幺半群中，$S$为所有形如`A => F[B]`的函数构成的集合；二元运算$⊙$为`compose`函数；单位元$e$就是`zero`。
此时，幺半群的二元运算结合律与单位元的不变性表现为以下断言：

```scala
assert(m.compose(m.compose(f, g), h) == m.compose(f, m.compose(g, h)))
assert(m.compose(f, m.unit) == f)
assert(m.compose(m.unit, f) == f)
```

理解了这些，关于函子(functor)与单子(monad)的概念，我们是不是也能有自信的说出一句：“What's the problem?”