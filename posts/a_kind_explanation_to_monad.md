+++
title = "A kind explanation to Monad(Haskell)"
description = ""
tags = [
    "haskell",
    "type",
    "monad",
]
categories = [
    "type theory",
]
date = "2024-08-24"
menu = "main"
+++

几乎是每个接触 Haskell 的朋友的必然，我们会尝试对 `Monad` 给出一个解释，不为别的，只是因为 `Monad` 在 Haskell 中避无可避的重要性，让我们不得不对这么做

<!--more-->

但想要对 `Monad` 进行解释并不是一个容易做到的事情，这个从范畴论当中提取出来的概念并不是那么的好解释，而且 `Monad(Pure Mathematics)` 和 `Monad(Haskell)` 之间亦有差距，不过经过变换后他们仍然保持了相同的本质。

## 简述 Haskell 中的 Monad

我们知道，函数式编程有一个特点，纯函数特别的**纯**，这体现在，对于计算机这个状态机来说，这样的纯函数完全不会影响其余的环境，它就仿佛真的只是代入计算，映射出一个结果，不触碰其余一分一毫。

但这肯定不对啊，要这样的话，你根本不可能实现与操作系统交互这种行为，你总是需要告诉操作系统，告诉硬件去变换某种状态，只有纯函数这显然是不可能做到的

`Monad` 于是横空出世了，在 Haskell，你即使只是一个 hello world 也必须使用到 `IO Monad` 

```haskell
main :: IO ()
-- The type of main: IO () 
main = putStrLn "Hello, World!"
```

在 Monad 内部，它能完成一些并不怎么纯的行为，比如读写文件描述符

## 从范畴到单子

> 无法避开的抽象概念，当你直面它的时候，事情才会变得合理

### 范畴

![alt text](pics/category_defination.png)

### 函子
![alt text](pics/functor_defination.png)

### 自然变换

![alt text](pics/natural_transformations.png)

### 单子
上述都是一些相当简单的概念，我们接下来来看看 Monad 的数学定义：

![alt text](pics/monad_defination.png)

十分简单的构造，有一个自函子，将范畴中的一个成员映射成另一个，另一个还能再次映射。再配以两个自然变换，用以映射与展平

## 单子(Haskell)
让我们来看看 Haskell 的单子定义

```haskell
class Functor m => Monad m where
  return :: a -> m a
  (>>=)  :: m a -> (a -> m b) -> m b
```

注意，Haskell 的单子首先是一个 `functor` 

```haskell
class Functor (f :: * -> *) where
fmap :: (a -> b) -> f a -> f b
```

我们要明确，我们讨论的这个范畴是Haskell所有类型所组成的一个类型范畴，在这种情况下 `m` 就是我们在上面数学定义中提到的那个 endofunctor ，而接下来剩下的 `return` 和 `>>=` 则是两个自然变换，前者称之为 `unit` 后者则称之为 `bind`

但我们可以看到的是，这和数学上说的那两个自然变换并不是相同的，前面还好说，`unit` 即是那个 {{< katex >}} \eta {{< /katex >}}，但是这个 `bind` 和原本的定义有什么关系

这还没完，`Monad` 还有三大定律：

```haskell
return x >>= f = f x
m >>= return = m
(m >>= f) >>= g = m >>= (\x -> f x >>= g)
```

但不管怎么样，看起来好像和范畴论里那个单子是有关联的，事实上他俩确实是同一种定义，`bind` 则是经过一些转换可以得到 {{< katex >}} \mu {{< /katex >}}(`join`)

```haskell
join :: Monad m => m (m a) -> m a
join x = x >>= id
```

> 我认为 `join` 是一个相当重要的概念，但Haskell 迂回的使用了 `bind`，似乎并不总是一个合适的行为，但它的意义会在 do 语句块中体现



## 三大定律
将数学语言翻译一下可以得到4条要求

1. fmap g . return = return . g
2. fmap g . join = join . fmap (fmap g)
3. join . fmap join = join . join
4. join . return = id = join . fmap return

Haskell 三大定律其实完全翻译自数学的定义以 `fmap` 和 `join` 作为原语，我们得到

```
 return x >>= f
= join (fmap f (return x))
= join (return (f x))
= f x
```
```
  a >>= return
= join (fmap return a)
= a
```
```
  (a >>= f) >>= g
= join (fmap g (join (fmap f a)))
= join (join (fmap (fmap g) (fmap f a)))
= join (fmap join (fmap (fmap g) (fmap f a)))
= join (fmap (join . fmap g . f) a)
= a >>= join . fmap g . f
= a >>= \ x -> join (fmap g (f x))
= a >>= \ x -> f x >>= g
```

进一步可证

```
  fmap f (return x)
= return x >>= return . f
= return (f x)
```
```
  fmap f (join a)
= (a >>= id) >>= return . f
= a >>= \ x -> id x >>= return . f
= a >>= \ x -> x >>= return . f
= a >>= fmap f
= a >>= \ x -> id (fmap f x)
= a >>= \ x -> return (fmap f x) >>= id
= (a >>= return . fmap f) >>= id
= join (fmap (fmap f) a)
```
```
  join (join a)
= (a >>= id) >>= id
= a >>= \ x -> x >>= id
= a >>= \ x -> join x
= a >>= \ x -> return (join x) >>= id
= (a >>= return . join) >>= id
= join (fmap join a)
```
```
  join (return a)
= return a >>= id
= id a
= a
```
```
  join (fmap return a)
= (a >>= return . return) >>= id
= a >>= \ x -> return (return x) >>= id
= a >>= \ x -> return x
= a >>= return
= a
```

## `do-notion`
Monad 的作用不是处理副作用，而是以一种间接映射的方式去处理内容。Monad 允许你创建一种可以将一系列行为组合变成一种大的行为，这也是为什么他被成为单子

> 有一个小故事，曾有人问我为什么我笃定单子叫 `dan'zi` (中文拼音)而不是 `shan'zi` ，其实正是因为这个原因

`do-notion` 只是一种语法糖，它直接对应着 Monad 本身的运算

```
do { x }                 -->  x
do { let { y = v }; x }  -->  let y = v in do { x }
do { v <- y; x }         -->  y >>= \v -> do { x }
do { y; x }              -->  y >>= \_ -> do { x }
```

![alt text](pics/do_block_requirements.png)

## 实例
对于常见的许多其他语言，过程式天生就有将大量操作串联起来的能力，因此完全不引入 Monad 对于这些编程范式混合的语言来说并没有什么不合理的地方

举一个人尽皆知的例子，Rust 中的 `std::option` ，其实它就是 Haskell 中的 `Maybe Monad` ，Rust 引入了最功利的东西，让我们可以方便的传递结果与错误，并对其进行处理

单子本身在函数的连续调用，高阶函数等中体现的价值正是它在函数式编程中取得关键地位的原因

---
作者：Github @feipiao594

参考文献：

[1]. 范畴论 维基百科 https://en.wikipedia.org/wiki/Category_theory

[2]. 单子 维基百科 https://en.wikipedia.org/wiki/Monad_(category_theory)

[3]. https://stackoverflow.com/questions/78867164/how-to-understand-the-in-haskells-do-notation-with-an-uncommon-impleme

[4]. https://wiki.haskell.org/Category_theory/Monads#Monads_in_Haskell

[5]. https://en.m.wikibooks.org/wiki/Haskell/Category_theory
