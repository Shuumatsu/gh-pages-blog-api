---
title: Responsive image and optimization
categories: [html]
tags: [optimization, responsive, html] 
date: 2016/10/2 20:46:25
---

图片的响应式的纯 html 实现可以借助 `srcset` 和 `sizes` 属性以及 `<picture>` 元素来实现。
而且观察Chrome开发者工具的 Network项可以发现，任何时候当屏幕变化的时候，如果匹配到新的规则需要的图片改变了的话，浏览器会去加载需要的图片。

<!-- more -->

[demo](http://googlechrome.github.io/samples/picture-element/)

## `srcset` 和 `sizes` 属性

`srcset` 属性接受一个或多个 `image candidate string`，每个值之间用逗号隔开。
`image candidate string` 由图片地址和 零个或多个 `width descriptor`（非零数字后跟一个小写的w） 和 `pixel density descriptor`（非零数字后更一个小写的x） 组成。具体构成见以下引用：

> An image candidate string consists of the following components, in
> order, with the further restrictions described below this list:
> 
> 1. Zero or more space characters.
> 
> 2. A valid non-empty URL that does not start or end with a U+002C COMMA character (,), referencing a non-interactive, optionally
> animated, image resource that is neither paged nor scripted.
> 
> 3. Zero or more space characters.
> 
> 4. Zero or one of the following:
> 
> - A width descriptor, consisting of: a space character, a valid non-negative integer giving a number greater than zero representing
> the width descriptor value, and a U+0077 LATIN SMALL LETTER W
> character.
> 
> - A pixel density descriptor, consisting of: a space character, a valid floating-point number giving a number greater than zero
> representing the pixel density descriptor value, and a U+0078 LATIN
> SMALL LETTER X character.
> 
> 5. Zero or more space characters.

---

- 一个元素的两个 `image candidate string` 不能有相同的宽度描述符
- 一个元素的两个 `image candidate string` 不能有相同的像素密度描述符，为这个要求起见，没有描述符的 `image candidate string` 默认指定一个 `1x` 描述符
- 一个元素的 `image candidate string` 指定了宽度描述符，则它所有其他的 `image candidate string` 也必须指定宽度描述符
- 有 `sizes` 属性的元素，它所有的 `image candidate string` 都必须有宽度描述符

还有个这什么玩意儿：

> The specified width in an image candidate string's width descriptor must match the intrinsic width in the resource given by the image candidate string's URL, if it has an intrinsic width.

### `srcset`
使用 `srcset`，浏览器自动选择哪一个图片是最适合的。
Mat Marquis 给出了一个例子：
你有一个宽度为 320px 像素密度为 1x 的设备，选择有3张图片，分别为 small.jpg (500px wide), medium.jpg (1000px wide), and large.jpg (2000px wide)。
浏览器会计算：
```
500 / 320 = 1.5625
1000 / 320 = 3.125
2000 / 320 = 6.25
```
然后选择和你的设备的像素密度最近的图片，本例中为 small.jpg

### `sizes`
举例说明：`sizes="100vw"` 意味着我假定这个图片将会占据整个 viewport。.
也可以配合媒体查询使用 例如 `sizes="(min-width: 800px) 50vw, 100vw"`，意味着如果浏览器窗口宽度大于 800px 那么这个图片可能会占据一半的宽度，如果小于 800px的话，可能会占据全部宽度。

---

不建议使用：
>  That can get a little complicated and honestly it might be a little dangerous because you're putting CSS stuff in markup and you know how that goes. [Eric Portis](http://ericportis.com/posts/2014/separated/) just wrote about this. Ideally it can be automated or injected server-side.

## picture 元素
一个用来为包含在其中的特定的 `<img>` 元素指定多个 `<source>` 元素的容器。浏览器会根据当前的页面布局（显示图片的 box 的限制）以及将被显示在的设备来选择最适合的来源。
最后的 `<img>` 标签是必须的，而且作为 "fallback"

1. 通过指定 `media` 属性
```
<picture>
 <source srcset="mdn-logo-wide.png" media="(min-width: 600px)">
 <img src="mdn-logo-narrow.png" alt="MDN">
</picture>
```
如果媒体查询评估失败，该 `<source>` 被跳过

2. 根据 `type` 属性
```
<picture>
 <source srcset="mdn-logo.svg" type="image/svg+xml">
 <img src="mdn-logo.png" alt="MDN">
</picture>
```
浏览器会跳过不支持的类型

## webp 图片格式

Filename extension: **.webp**
Internet media type: **image/webp**

---

### 兼容性

#### 检测浏览器是否支持 webp 
http://stackoverflow.com/questions/5573096/detecting-webp-support



参考：
 > https://html.spec.whatwg.org/multipage/embedded-content.html#attr-img-srcset
 > https://css-tricks.com/responsive-images-youre-just-changing-resolutions-use-srcset/
 > http://www.html5rocks.com/en/tutorials/responsive/picture-element/
 > http://stackoverflow.com/questions/5573096/detecting-webp-support
 > http://webpjs.appspot.com/without-webpjs-support.html