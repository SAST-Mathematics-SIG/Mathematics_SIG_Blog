---
title: "本站扩展语法"
---

# 本站扩展语法——shortcodes

本站使用hugo与hugo-book搭建，支持一些称之为shortcodes的markdown扩展语法，接下来我们就详细介绍一下

## 按钮

按钮其实是一个装饰后的超链接，它可以导向外部连接或者本地

### 例子

```tpl
{{</* button relref="/" [class="..."] */>}}回到首页{{</* /button */>}}
{{</* button href="https://github.com/SAST-Mathematics-SIG" */>}}前往SIG的GITHUB组织{{</* /button */>}}
```

{{< button relref="/" >}}回到首页{{< /button >}}
{{< button href="https://github.com/SAST-Mathematics-SIG" >}}前往SIG的GITHUB组织{{< /button >}}

## 细节展示

细节展示(Details)是一个展开的栏目，可以隐藏一些信息

### 例子
```tpl
{{</* details "Title" [open] */>}}
### 内容
细节展示(Details)是...
{{</* /details */>}}
```
```tpl
{{</* details title="Title" open=true */>}}
### 内容
细节展示(Details)是...
{{</* /details */>}}
```

{{< details "Title" open >}}
### 内容
细节展示(Details)是...
{{< /details >}}

## 分列

分列(Columns)有助于组织简短的文字，使之排版更加清晰可读



```html
{{</* columns */>}} <!-- begin columns block -->
# Left Content
Lorem markdownum insigne...

<---> <!-- magic separator, between columns -->

# Mid Content
Lorem markdownum insigne...

<---> <!-- magic separator, between columns -->

# Right Content
Lorem markdownum insigne...
{{</* /columns */>}}
```
### 例子

{{< columns >}}
# 左段落
左段落内容...
```cpp
#include <iostream>
int main(){
    std::cout <<"hello world";
}
```

<--->

# 中段落
中段落内容...

WARNING: 注意在分栏中不能使用扩展语法

<--->

# 右段落
右段落内容...

> SAST.Mathematics SIG

{{< /columns >}}

# 高亮

高亮是一种不同颜色的引用块，
有三种颜色可以被选择: `info`, `warning` and `danger`.

```tpl
{{</* hint [info|warning|danger] */>}}
**内容**  
这里是*[info|warning|danger]*喵
{{</* /hint */>}}
```

## 例子

{{< hint info >}}
**内容**  
这里是 *Info* 喵
{{< /hint >}}

{{< hint warning >}}
**内容**  
这里是 *warning* 喵
{{< /hint >}}

{{< hint danger >}}
**内容**  
这里是 *danger* 喵
{{< /hint >}}

## KaTeX

KaTeX 可以让你在Markdown中插入Latex公式，在本站相当重要。详情可见 [KaTeX](https://katex.org/)

```latex
{{</* katex >}}\pi(x){{< /katex */>}}
//这是行内公式

{{</* katex display=true class="optional" >}}
f(x) = \int_{-\infty}^\infty\hat f(\xi)\,e^{2 \pi i \xi x}\,d\xi
{{< /katex */>}}
// 这是行外公式
```

**注意** 对于行外公式，你也可以使用markdown自带的写法，甚至更加推荐这种写法

```tpl
$$a^2+b^2=c^2$$
```

$$a^2+b^2=c^2$$

### 显示模式/例子


这里是行内公式: {{< katex >}}\pi(x){{< /katex >}}
下面是行外公式, 具有`display: block`属性

{{< katex display=true >}}
f(x) = \int_{-\infty}^\infty\hat f(\xi)\,e^{2 \pi i \xi x}\,d\xi
{{< /katex >}}

文本可以在这里继续

# tabs

tabs 有助于你组织不同情况下不同的文本

```tpl
{{</* tabs "uniqueid" */>}}
{{</* tab "MacOS" */>}} # MacOS Content {{</* /tab */>}}
{{</* tab "Linux" */>}} # Linux Content {{</* /tab */>}}
{{</* tab "Windows" */>}} # Windows Content {{</* /tab */>}}
{{</* /tabs */>}}
```

## Example

{{< tabs "uniqueid" >}}
{{< tab "MacOS" >}}
# MacOS

这是 **MacOS** 的tab内容

SAST.Mathematics SIG
{{< /tab >}}
{{< tab "Linux" >}}

# Linux

这是 **Linux** 的tab内容

SAST.Mathematics SIG
{{< /tab >}}
{{< tab "Windows" >}}

# Windows

这是 **Windows** 的tab内容

SAST.Mathematics SIG
{{< /tab >}}
{{< /tabs >}}

# Mermaid 图表

[MermaidJS](https://mermaid-js.github.io/) 是用于从文本生成 svg 图表和图表的库

{{< hint info >}}
**覆盖Mermaid的初始化配置**

要覆盖 Mermaid 的[初始化配置](https://mermaid-js.github.io/mermaid/#/Setup)，请在 assets 文件夹中创建一个 mermaid.json 文件！
{{< /hint >}}

## 例子


<div class="book-columns flex flex-wrap">
  <div class="flex-even markdown-inner">

```tpl
{{</* mermaid class="optional" >}}
stateDiagram-v2
    State1: The state with a note
    note right of State1
        Important information! You can write
        notes.
    end note
    State1 --> State2
    note left of State2 : This is the note to the left.
{{< /mermaid */>}}
```

{{< mermaid class="optional" >}}
stateDiagram-v2
    State1: The state with a note
    note right of State1
        Important information! You can write
        notes.
    end note
    State1 --> State2
    note left of State2 : This is the note to the left.
{{< /mermaid >}}
