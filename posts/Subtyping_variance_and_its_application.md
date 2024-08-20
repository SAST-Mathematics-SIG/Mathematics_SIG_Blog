+++
title = "Subtyping variance and its application"
description = ""
tags = [
    "rust",
    "type",
]
categories = [
    "Type Theory",
]
date = "2024-6-5"
menu = "main"
+++

子类型型变及其应用

<!--more-->

## 无类型lambda演算

无类型lambda演算含有以下三个组成部分

- term  {{< katex >}} t {{< /katex >}}
- abstraction  {{< katex >}} λx.t {{< /katex >}}
- application  {{< katex >}} t_1\ t_2 {{< /katex >}}

### Example 1

$$
\begin{aligned}
    & (λx.x) ((λx.x) (λz. (λx.x) z)) \\\\
    \rightarrow & (λx.x) ((λx.x) (λz.z)) \\\\
    \rightarrow & (λx.x) (λz.z) \\\\
    \rightarrow & (λz.z) \\\\
    \rightarrow & (λx.x) \\\\
\end{aligned}
$$

## 简单类型lambda演算

简单类型lambda演算({{< katex >}} \lambda_{\rightarrow} {{< /katex >}})含有以下五个组成部分

- term {{< katex >}} t {{< /katex >}}
- abstraction {{< katex >}} λx.t {{< /katex >}}
- application {{< katex >}} t_1\ t_2 {{< /katex >}}
- type of functions {{< katex >}} T_1\rightarrow T_2 {{< /katex >}}
- typing context {{< katex >}} \Gamma {{< /katex >}}

### Example 2

$$
\dfrac{\dfrac{\Gamma, x: T_1 \vdash t_1: T_2}{\Gamma \vdash \lambda x: T_1. t_1: T_1 \to T_2} \quad \quad \genfrac{}{}{0pt}{0}{}{\Gamma \vdash t_2: T_1}}{\Gamma \vdash \left(\lambda x: T_1. t_1\right) \  t_2: T_2}
$$

## 含有子类型的简单类型lambda演算

> {{< katex >}} <: {{< /katex >}} 笑脸可爱捏

相比简单类型lambda演算({{< katex >}} \lambda_{\rightarrow} {{< /katex >}})，唯独**多出**一个子类型({{< katex >}} \lambda_{<:} {{< /katex >}})的运算: 

- subtyping  {{< katex >}} S <: T {{< /katex >}}

如果 S 是 T 的子类型，意思是在任何需要使用 T 类型对象的环境中，都可以安全地使用 S 类型的对象。

### 子类型的性质

- reflexive | 自反性 : {{< katex >}} S <: S {{< /katex >}}
- transitive | 传递性 : {{< katex >}} \dfrac{S <: U \qquad U <: T}{S <: T}\ {{< /katex >}}
- top and bottom | 上界与下界 : {{< katex >}} \text{Bot} <: S <: \text{Top} {{< /katex >}}

目前为止还没有什么证明，要说明这里的第一条和第二条都很好理解，第三条是指上界和下界的存在性

## 子类型型变

这其实也是指的一组性质

- covariant | 协变 $$ \dfrac{S <: T}{\text{list of } S <: \text{list of } T} $$
- contravariant | 逆变  $$ \dfrac{S <: T}{T \to U <: S \to U} $$
- invariant | 不变  $$ \dfrac{S <: T}{S \to S \text{ is neither subtype nor supertype of } T \to T} $$

这里也是，协变和不变的性质很好理解，但逆变的性质就不是很好理解了

>接下来将对逆变给出一个并不严格的证明，或者说是说明

**contravariant | 逆变**：

翻译一下，S 是 T 的子类型，那么 “T到U的函数” 这个类型是 “S到U的函数” 这个类型的子类型

根据我们之前所提到的子类型的含义和逆变性质，我们现在来思考一下是否在任何需要使用 “S到U的函数” 类型对象的环境中，都可以安全地使用 “T到U的函数” 类型的对象。

从逻辑上来说并没有问题，根据多态，我需要用到 “S到U的函数” 的环境其实最后就是为了代入函数求值，但其实能代入 S 的实例化对象求值的场景，你把这个 “S到U的函数” 换成 “T到U的函数” 也能正常跑，完全没有任何问题

## 应用: Rust 类型系统的生命周期

观察如下一小段 Rust 代码
```rust
{
    let foo: &u8 = &1;
    {
        let bar = &foo;
        println!("bar = {}", **bar);
    }
}
```

熟悉 Rust 的朋友有可能在学习使用生命周期的时候接触过这段代码，但为了给更多人讲解，先让我们给它标记上更明显的生命周期标签

> **WARNING**: 下面这一段代码是一段伪代码，Rust 编译器将不能编译它

```rust
'b: {
    let foo: &'b u8 = &1;
    'a: {
            let bar: &'a &'b u8 = &foo;
            println!("bar = {}", **bar);
    }
}
```

此时此刻可以看到我们给外部作用域起一个生命周期的标签为 `'b` ，当然这里的标签是不能写在作用域前的，实际上被写在了第二行的 `foo` 变量的初始化中，这个标记意味着 `foo` 这个引用的生命周期是 `'b` ，而 `bar` 变量也是一样的，只是这边要强调一个点：`&'a &'b u8` 这个是**引用的引用**，**隐式的要求 `'a` 要满足最多生命周期和 `'b` 一样长** 

那这和我们上面的子类型有什么关系呢，我们看第二行：`let foo: &'b u8 = &1;` ，其中 `&1` 的生命周期在 Rust 中被规定为 `'static` 所以 `&1` 的类型其实是 `&'static u8` ，而 `&'static u8` 可以赋给 `&'b u8` ，是为 `&'static u8` 为 `&'b u8` 的子类型

总结以下，在这里
`Bot = &'static T <: &'b T <: &'a T`

接下来我们要提 Rust 中利用生命周期造就的经典的类型分析器的漏洞，在本文发布的时间点(2024.8.19)还未被修复

**Oops!**
```rust
static UNIT: &'static &'static () = &&();

fn foo<'a, 'b, T>(_: &'a &'b (), v: &'b T) -> &'a T { v }

fn bad<'a, T>(x: &'a T) -> &'static T {
    let f: fn(_, &'a T) -> &'static T = foo;
    f(UNIT, x)
}
```

可以看到 `bad` 函数很神奇的传入了一个 `&'a T` 类型的 `x` 却返回了 `&'static T` 的 `x`，这可了不得，这意味着你可以将一个引用的生命周期无端的延长，想象一下，这将不可避免的造成引用空悬，内存安全也就不存在了。众所周知，Rust 的一大亮点就是安全，但这段神奇的代码居然能“安全”的过编译

让我们来看看它是怎么工作的

```rust
fn foo<'a, 'b, T>(_: &'a &'b (), v: &'b T) -> &'a T

fn foo<'a, 'b, T>(_: &'a &'static (), v: &'b T) -> &'a T

fn foo<'b, T>(_: &'static &'static (), v: &'b T) -> &'static T
```

上面这3行揭示了编译器在进行转换时发生的几个步骤：第一行是我们原来 `foo` 的标签，编译器首先将他转化为第二行，`&'a &'b ()` 转化为 `&'a &'static ()` ，因为 `&'a &'static ()` 是 `&'a &'b ()` 的子类型，那么根据逆变规则，这里的第一行的签名就是第二行的子类型，那么在任何需要使用第二行类型对象的环境中，都可以安全地使用第一行类型的对象。那么编译器便试图创建第二行的环境(因为你写的是 `_` ，要编译器去推导)，以适应你的 `foo` ，但是此时此刻，你告诉编译器 `'a` 是 `'static` ，编译器直接懵了，那就替换吧，于是借了编译器之手成就了天才的**Lifetime_Expansion**

那么如何规避这个问题呢，其实关键就是这当中丢失了一个关键因素：第一行到第二行的时候丢失了一个 `&'a &'b ()` 包含的隐式的信息：`'a` 应该生命周期短于 `'b` ，而转化到第二步恰恰丢失了这一步信息，`'a` 和 `'b` 完全是两个无关的生命周期标签了

所以只要我们再给编译器一点提示，不至于让他绕进去就好了:

```rust
//Manual solution
fn foo<'a, 'b, T>(_: &'a &'b (), v: &'b T) -> &'a T
    where 'b: 'a
```

当我们告诉编译器 `'b` 与 `'a` 之间的偏序关系，即表示 `'b` 至少要活得跟 `'a` 一样久。

> 值得注意的是，这只是一个类型分析器的 bug ，当你试图将 `_` 省略的内容写清楚，也会使得这个 bug 消除

[cve-rs](https://github.com/Speykious/cve-rs) 正是利用了这点，在完全不含 unsafe 块的 Rust 代码中引入了 segmentfault 等内存错误，~~这太有乐子了~~。

---

作者：Github @NKID00

增改：Github @feipiao594

参考文献：

[1]. Github repo: cve-rs <https://github.com/Speykious/cve-rs>