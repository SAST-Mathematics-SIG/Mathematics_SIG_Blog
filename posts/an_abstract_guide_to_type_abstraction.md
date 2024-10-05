+++
title = "一份抽象的类型抽象指北"
description = ""
tags = [
    "rust",
    "type",
]
categories = [
    "Type Theory",
]
date = "2024-08-17"
menu = "main"
+++

> 内存本非数，类型亦无形。编译不报错，运行未必行。

程序设计常常涉及类型的抽象。你是否在一些库的文档中看过各种类型，却觉得它们过于抽象？动辄几十几百个成员的枚举类型，总是报错的类型约束，永远设计不出的 trait，以及程序运行时无尽的 panic……看完今天的介绍，不说你是醍醐灌顶，至少也算得上是毫无收获。

<!--more-->

## 引子

### 除法

以下是一个常见的除法定义，采用 Rust 语言，以 `u32` 类型为例：

```rust
fn div(a: u32, b: u32) -> u32 {
    a / b
}

fn main() {
    div(6, 3);  // 2
    div(5, 1);  // 5
    div(4, 0);  // panic!
}
```

函数明明定义为 `fn(u32, u32) -> u32`，但却不一定总能返回 `u32` 类型的值。这是一种意外行为（虽然你大概也许知道 0 不能作为除数，所以你可能或许毫不意外）。

虽然 Rust 标准库文档说明了除法可能 panic，~~但是正经人谁看文档啊，下贱~~，但是文档里面的备注容易被忽略，而这种把潜在错误留到运行时的行为很容易导致意外崩溃。

![impl Div for u32](pics/impl_div_for_u32.png)
![macro div_impl_integer](pics/div_impl_integer.png)

对此，我们可以用两种方法在类型层面解决：

1. 限制除数类型为非零整数，如采用 `NonZero<u32>` 类型：
   ![impl Div<NonZero<u32>> for u32](pics/impl_div_non_zero_u32_for_u32.png)
   ![impl Div<NonZero<$Int>> for $Int source](pics/impl_div_non_zero_u32_for_u32_src.png)
   相当于定义：

   ```rust
   fn div(a: u32, b: NonZero<u32>) -> u32 {
       a / b.get()
   }
   ```

2. 运行时检查可能的错误，返回可能失败的结果，如 `checked_div` 函数：

![method checked_div](pics/checked_div_u32.png)

### 颜色

如何定义 `Color` 类型？

- `String`？（`"black"`、`"white"`、`"red"`、`"rgb(0 0 255)"`、`"#00FF00"`……）
- RGB `(u8, u8, u8)`（真彩色）？
- RGBA `(u8, u8, u8, u8)`？
- `f32`？`f64`？
- 16 色？256 色（`u8`）？

  - `0bRRRGGGBB`？
  - ANSI？

    ANSI 256 色:

    ![ANSI 256 色](pics/ansi-256-color.png)

- `any`？

人眼所能看见的颜色范围很广，但是计算机对颜色的表示（颜色空间）只能覆盖人眼可见颜色的一部分。以下是常见的颜色空间对比：

![颜色空间与 CIE1931xy 对比](pics/CIE1931xy_gamut_comparison.png)

你永远无法定义一个完美的 `Color` 类型！适合使用场景的类型才是最好的类型。

以浏览器渲染流程为例，一个可能的类型工作流程是：

- 在处理原始 CSS 颜色字符串时，使用 `String`
- CSS 解析后可以转为自定义的 `enum` 类型，类似于：

  ```rust
  /// Number range is extracted from
  /// <https://drafts.csswg.org/css-color/#color-type>
  #[non_exhaustive]
  enum Color {
      Transparent,
      Named(NamedColor),
      Hex(u32),                 // 0xRRGGBBAA
      Rgb(f32, f32, f32),       // 0.0..=255.0
      Rgba(f32, f32, f32, f32), // R/G/B: 0.0..=255.0, A: 0.0..=1.0
      Hsl(f32, f32, f32),       // H: 0.0..360.0, S/L: 0.0..100.0
      Hsla(f32, f32, f32, f32),
      // Hwb, Lab, Lch, etc.
  }
  ```

- 内部可以表示为 RGBA 或其他颜色空间
- 处理完透明度混合后，最终渲染到像素点时采用 RGB

### 错误类型

在一个项目（crate）中，如何定义错误类型？

- `String`？（简单明了，但不够规范，难以本地化处理）
- `Box<dyn Error>`？`anyhow::Error`？（与 `String` 相比，规范了输出格式，但仍不够结构化，适合 bin crate）
- `enum` + `#[derive(thiserror::Error)]`？（结构化程度更高，但需要额外设计错误类型，适合 lib crate）
- `any`？

## 好的类型抽象有哪些特点？

~~好的类型抽象应该抽象~~

类型本质是一种集合。好的类型应该和良好定义的集合具有相似的性质。

高中的数学课介绍了集合的三大性质：

- 确定性：对于任意一个元素，它要么属于这个集合，要么不属于这个集合。
- 互异性：集合中的元素是互不相同的。
- 无序性：集合中的元素之间没有顺序关系。

其中无序性主要用于集合相等性的判定，不在这里讨论。

对于确定性的要求，好的抽象应该满足：

- 任何合法的概念都能以这个类型的值来表示。
- 任何非法的概念不能以这个类型的值来表示。

对于互异性的要求，好的抽象应该满足：

- 在该类型的框架下，一个合法的概念只能有一种表示方式。

或者说，好的类型抽象和被抽象的概念**同构**（一一对应）。

同时，好的类型抽象应该尽可能简单，不应该包含多余的信息。

## 实战：任务调度

了解 typestate 设计模式。具体代码可见 [type-abstraction](https://github.com/SAST-Mathematics-SIG/type-abstraction/) 仓库。

---

作者: Github: [@Jisu-Woniu](https://github.com/Jisu-Woniu)
