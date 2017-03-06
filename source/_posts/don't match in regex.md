---
title: don't match in regex
date: 2017-03-06 15:53:34
tags: [regex]
categories: [regex]
---

问题来自 [Stack Overflow](https://stackoverflow.com/questions/406230/regular-expression-to-match-a-line-that-doesnt-contain-a-word).

Regular expression to match a line that doesn't contain a word?

<!-- more -->

### 答案

`/^((?!hede).)*$/` 会匹配任何一个不包含 `'hede'` 子串的单行字符串，如果要让他支持多行字符串，将 `*` 替换为 `(\n|.)` 即可。

`/^(?!hede).*$/` 是不以 `'hede'` 开头的版本。

### 解释


#### 正则表达式原理

来自 [Wikipedia](https://zh.wikipedia.org/wiki/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F):

正则表达式可以用形式化语言理论的方式来表达。正则表达式由常量和算子组成，它们分别指示字符串的集合和在这些集合上的运算。给定有限字母表Σ定义了下列常量：

- （“空集”）∅指示集合∅

- （“空串”）ε指示集合{ε}

- （“文字字符”）在Σ中的a指示集合{a}

定义了下列运算：

- （“串接”）RS指示集合{ αβ | α ∈ R，β ∈ S }。例如：{"ab","c"}{"d","ef"} = {"abd", "abef", "cd", "cef"}。

- （“选择”）R|S指示R和S的并集。例如：{"ab", "c"}|{"ab", "d", "ef"}= {"ab", "c", "d", "ef"}。

- （“Kleene星号”）R\* 指示包含ε并且闭合在字符串串接下的R的最小超集。这是可以通过R中的零或多个字符串的串接得到所有字符串的集合。例如，{"ab", "c"}\* = {ε, "ab", "c", "abab", "abc", "cab", "cc", "ababab", ... }。

上述常量和算子形成了克莱尼代数。

选择运算拥有最低的优先级，例如 `/gray|grey/` 等同于 `/gr(a|e)y/`。

#### 字符串分析

一个字符串相当于是一串字符。没个字符的前后可看作有一个空字符串。这样来看的话，一个长度为 n 的字符串会有 n+1 个空字符串。

```

    ┌──┬───┬──┬───┬──┬───┬──┬───┬──┬───┬──┬───┬──┬───┬──┬───┬──┐
S = │e1│ A │e2│ B │e3│ h │e4│ e │e5│ d │e6│ e │e7│ C │e8│ D │e9│
    └──┴───┴──┴───┴──┴───┴──┴───┴──┴───┴──┴───┴──┴───┴──┴───┴──┘

index    0      1      2      3      4      5      6      7
```

#### 正向否定预查

`(?!pattern)` 是正向否定预查。来自 [Wikipedia](https://zh.wikipedia.org/wiki/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F) 的解释是：

>正向否定预查，在任何不匹配pattern的字符串开始处匹配查找字符串。这是一个非获取匹配，也就是说，该匹配不需要获取供以后使用。例如“Windows(?!95|98|NT|2000)”能匹配“Windows3.1”中的“Windows”，但不能匹配“Windows2000”中的“Windows”。预查不消耗字符，也就是说，在一个匹配发生后，在最后一次匹配之后立即开始下一次匹配的搜索，而不是从包含预查的字符之后开始

所以会有以下这些结果：

```
// true
/Windows(?!95|98|NT|2000)/.test('Windows')

// true
/Windows(?!95|98|NT|2000)/.test('Windows10')

// false
/Windows(?!95|98|NT|2000)/.test('Windows95')

// ["Windows10", "10"]
/Windows(?!95|98|NT|2000)(\d+)/.exec('Windows10')
```

#### /^((?!hede).)*$/

再回到我们的 `/^((?!hede).)*$/`，表达式的 `(?!hede)` 部分会向右看是否有 `'hede'` 子串，如果没有的话 `.`(dot)会匹配任何除了换行符外的其他字符。预查（look-arounds，是叫这个意思吧）也叫做 zero-width-assertions，因为不会消耗任何字符，仅仅是去验证。

所以在这个例子中，每一个空字符串都会首先去验证它左边是否有一个 `'hede'` 子串。这样重复下去，直到没一个字符都被 `.`(dot) 消耗。

As you can see, the input "ABhedeCD" will fail because on e3, the regex (?!hede) fails (there is "hede" up ahead!).

