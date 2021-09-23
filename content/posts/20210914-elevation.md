---
title: "Elevation in Material Design"
date: 2021-09-23T11:49:35+08:00
tags: ["UI/UX"]
categories:
- "Design"
---

## 前言

最近跟公司同事開了一個 Material Design 的讀書會，由我導讀 Elevation 的部分，在此紀錄一些當初讀這篇的筆記。

原文：{{< newtabref  href="https://material.io/design/environment/elevation.html" title="Material Design - Elevation" >}}


## 計算 Elevation

> Elevation is the relative distance between two surfaces along the z-axis.

- Elevation 是兩個平面在 z 軸上的相對距離，也就是高度差（或高度，若以 0dp 為基準）。
- 用 {{< newtabref  href="https://material.io/design/layout/pixel-density.html#pixel-density" title="dp" >}} 這個單位測量，並通常（但不總是）會以陰影來繪製這段高度差。

![https://material.io/design/environment/elevation.html#elevation-in-material-design](https://i.imgur.com/JMY2w6b.png)

在上圖中 A 和 B 兩張卡片都在 8dp 高的地方，但因為 B 後面有卡片 C，所以他的陰影只有 B 和 C 的高度差 4dp，而 A 則是 8dp 的陰影。

當我們使用 CSS Library 在繪製陰影時，指定某元件有 2dp 的 Elevation 指的是他的陰影有 2dp 的厚度，與他當下的所在的高度無關。

例如下圖是根據上圖所給予卡片的 Elevation，B 和 C 兩張卡片的 Elevation 都是 4dp。

![example of depicting elevation](/20210914/elevation-1.png)


## Elevation 的功能簡介

- 使得物件擁有不同的高度：前後關係（上下層之分）。
- 有不同的高度能提示使用者他們是不同群體。
- 高度特別突出能吸引使用者的注意，反之則降低重要程度。

高度的繪製不僅有陰影，還可以透過填色或透明度來表現。

使用陰影：Google News 的 Header  
使用填色：Youtube Mobile APP 的底部 Tab  
使用透明度：Youtube Desktop Website 的 Header  

後面有更詳細關於繪製的守則。


### 靜止高度 Resting Elevation

手機上的靜止高度通常比桌面還要高，因為手機只有 Elevation 可以提示使用者你現在可以與這個元件互動，電腦有其他線索，例如 mouse hover。


### 改變高度

動態偏移高度：在某個事件發生時元件改變的高度多寡。

注意一致性，同樣的元件在整個應用中應有相同的靜止高度跟動態偏移高度。


### 避免碰撞

根據 Material serface proterty，我們要避免穿透。

![](https://i.imgur.com/LmCYFkh.png)

於是在考量高度變化時，如果可能會有穿透，應讓被碰撞的物件消失或移出畫面。

或是，覺得處理碰撞很麻煩的話可以在設計物件的位置時就先避免。

![](https://i.imgur.com/mOCkaZl.png)


## 繪製高度

### Surface edges

凸顯出他的邊界，以告訴使用者他們是不同的物件。


### Surface overlap

凸顯他們正在重疊，也就是正所在不同高度。

顏色對比、透明度可以展現出邊界跟重疊，但沒有展現高度差，不過也是表達他們前後關係的方式之一。


### 距離感

使用 scrim（遮幕）遮著後方的內容可以提升遮幕上物件的高度，Dialog 最常使用，scrim 也有人稱 overlay, backdrop。

陰影是能最細緻繪製高度的方法，它不必遮住其他內容，而且他可以表達出高度的量，兩平面間的距離是 5dp 高還是 8dp 高。

![](https://i.imgur.com/ikGjqux.png)


## Elevation hierarchy

高度可以提示使用者：
- 資訊擺得愈高，他更重要 -> 把次要的東西擺在較低的位置。
- 吸引你的注意。
- 他主宰/控制較低高度的物件，如 Nav Bar, Header。

