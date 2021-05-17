---
title: "使用 d3-brush 後文字不見？來了解 svg 的顏色設定"
date: 2021-05-16T20:53:54+08:00
draft: false
tags: ["d3.js", "svg"]
categories:
- "Front-end"
---

前陣子，在修資料視覺化的同學寫作業遇到了一個問題，幫他解決後發現裡面有不少東西可以記錄下來。

這一篇會分成三個部分
- 解決同學遇到的 bug：[使用 d3-brush 後文字消失](#使用-d3-brush-後文字消失)
- [了解 d3-brush 的行為](#d3-brush-的行為)
- [d3-axis 顏色的有趣發現](#d3-axis-的顏色設定)

## 使用 d3-brush 後文字消失

我們要看的是下面這段程式，也可以到我的 {{< newtabref  href="https://observablehq.com/@uier/d3-brush-fill-none?ui=classic" title="observable" >}} 看 render 的結果。

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Demo</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/d3/5.0.0/d3.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/d3-brush/1.0.4/d3-brush.min.js"></script>
</head>
<body>
  <svg width="100" height="100"></svg>

  <script>
    const brush = d3.brush().extent([[0, 0], [100, 100]]);

    const svg = d3.select("svg");
    const g = svg.append("g");

    g.append("text")
      .attr("transform", "translate(0, 50)")
      .attr("font-size", "15px")
      .text("Hello, world");

    // try to comment the code below, and the text will reveal
    g.call(brush);
  </script>
</body>
</html>
```

問題是這樣子，在`<svg>`內有一個群組元素`<g>`，裡面又有一個顯示`Hello, world`的文字元素`<text>`，當套用 d3-brush 後文字就消失不見了，若把`g.call(brush)`註解掉則文字就可以出現。

打開瀏覽器的開發者工具，在 DOM 裡面還是可以找到`<text>`，但我們卻看不到他。

![inspect-element](/d3-brush-demo.png)

不過眼尖就會發現上一層的`<g>`有 attribute`fill="none"`，因為我們沒有為`<text>`指定 fill 的顏色，而`<g>`這個元素會讓他的 children 繼承他的 attribute，所以造成`<text>`繼承了`fill="none"`。關於`<g>`元素的說明可以看 {{< newtabref  href="https://developer.mozilla.org/en-US/docs/Web/SVG/Element/g" title="MDN 的文件" >}}。

結論是，記得也要在`<text>`上設定 fill 屬性，就能順利顯示文字：
```js
g.append("text")
  .attr("transform", "translate(0, 50)")
  .attr("font-size", "15px")
  .attr("fill", "black")
  .text("Hello, world");
```

## d3-brush 的行為

那麼為何上層的`<g>`會是`fill="none"`呢？

當沒有套用 d3-brush 時，會看到`<g>`的 fill 是採用預設值黑色，並沒有被設定成`fill="none"`，`<text>`則是因為繼承`<g>`而也顯示黑色。

所以我們可以懷疑 d3-brush 把我們的`<g>`加上了`fill="none"`。

關於 d3-brush 的行為，{{< newtabref  href="https://github.com/d3/d3-brush#api-reference" title="文件" >}}裡頭有詳細的說明，主要是下列這段：

> The brush also creates the SVG elements necessary to display the brush selection and to receive input events for interaction. You can add, remove or modify these elements as desired to change the brush appearance; you can also apply stylesheets to modify the brush appearance. The structure of a two-dimensional brush is as follows:
```html
<g class="brush" fill="none" pointer-events="all" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);">
  <rect class="overlay" pointer-events="all" cursor="crosshair" x="0" y="0" width="960" height="500"></rect>
  <rect class="selection" cursor="move" fill="#777" fill-opacity="0.3" stroke="#fff" shape-rendering="crispEdges" x="112" y="194" width="182" height="83"></rect>
  <rect class="handle handle--n" cursor="ns-resize" x="107" y="189" width="192" height="10"></rect>
  <rect class="handle handle--e" cursor="ew-resize" x="289" y="189" width="10" height="93"></rect>
  <rect class="handle handle--s" cursor="ns-resize" x="107" y="272" width="192" height="10"></rect>
  <rect class="handle handle--w" cursor="ew-resize" x="107" y="189" width="10" height="93"></rect>
  <rect class="handle handle--nw" cursor="nwse-resize" x="107" y="189" width="10" height="10"></rect>
  <rect class="handle handle--ne" cursor="nesw-resize" x="289" y="189" width="10" height="10"></rect>
  <rect class="handle handle--se" cursor="nwse-resize" x="289" y="272" width="10" height="10"></rect>
  <rect class="handle handle--sw" cursor="nesw-resize" x="107" y="272" width="10" height="10"></rect>
</g>
```

套用後會改變`<g>`的屬性，另外總共會產生 10 個`<rect>`：
- `<rect class="overlay" ... />`會覆蓋整個`<g>`。
- `<rect class="selection" ... />`是顯示 brush 後框選到的範圍，可以看到他有設定 fill 為灰色。
- 後面八個則是用來調整框選範圍大小的控制器，包含了四個邊跟四個角落。

也因此 d3-brush 設定了`fill="none"`來隱藏這些`<rect>`，你也可以打開開發者工具手動把 fill 改成其他顏色看看會如何。在下面這張圖，我把`<g>`更改成`fill="red"`。

![modify-element](/d3-brush-demo2.png)

其實在幫同學看這個問題時有看到他嘗試寫了`<text color="black">`，然而 color 在 SVG 中並不是他所想的那樣，下一個部分會再提到 color 這個屬性。

回頭去想，他之所以會需要在`<g>`裡面加`<text>`，是因為想在 x 軸以及 y 軸旁加上文字標籤，這讓我想到我們在 d3 建立 x 軸跟 y 軸時，也都是包含在`<g>`元素裡的，軸線用的是`<path>`、tick 跟 label 用的是`<line>`跟`<text>`，於是我好奇 d3-axis 是如何設定他們的 fill 屬性。

## d3-axis 的顏色設定

在翻 source code 時讓我發現一件有趣的事情，在 d3 v4（d3-axis <= v1.0.8）時，軸線跟 tick label 被寫死成`fill="#000"`，可以看{{< newtabref  href="https://github.com/d3/d3-axis/blob/v1.0.8/src/axis.js#L63-L76" title="這段 code" >}}，我同時也附在下圖。

```js
path = path.merge(path.enter().insert("path", ".tick")
    .attr("class", "domain")
    .attr("stroke", "#000"));

line = line.merge(tickEnter.append("line")
    .attr("stroke", "#000")
    .attr(x + "2", k * tickSizeInner));

text = text.merge(tickEnter.append("text")
    .attr("fill", "#000")
    .attr(x, k * spacing)
    .attr("dy", orient === top ? "0em" : orient === bottom ? "0.71em" : "0.32em"));
```

而後來，作者自己開了這個 {{< newtabref  href="https://github.com/d3/d3-axis/issues/49" title="issue" >}} 並更改成`fill="currentcolor"`，如下圖。

![change-to-fill=currentcolor](/d3-brush-demo3.png)

### currentcolor 是什麼？

fill, stroke 這些設定顏色的屬性，除了可以設為一個顏色的 rgb 值、hex 值等，也可以設定為 currentcolor，如`fill="currentcolor"`。

而究竟他會設定成什麼顏色呢，答案是 color 這個屬性的顏色，若沒有設定 color 則會繼承 parent 的 color。

currentcolor 讓我們在設定顏色時，可以透過上層的 color 來統一環境的顏色，不必單獨管理每個子元素的顏色。

以上述提到 d3-axis 的 issue 來舉例，作者的考量是希望 axis 的顏色可以彈性的更改。若寫死成黑色，當背景色是深色的情況下，就需要靠開發者自己去重新設定顏色。

我根據這個情境寫了一個{{< newtabref  href="https://observablehq.com/@uier/d3-axis-in-dark-mode-between-v4-v5" title="範例" >}}，讓大家可以對比這個改變的威力。
