---
id: "c0a19edfd5defc18c35918f118af0d6428d84180"
title: "CSS Animation 入門筆記"
date: 2018-12-03T12:00:00+08:00
draft: false
description: "快速學會怎麼使用 CSS 做出動畫效果"
images:
    - "/images/og_image.png"
tags:
    - CSS3
    - Front-end
    - Web
categories:
    - 軟體開發
---

利用 CSS 動畫，可不必使用 Javascript 或是 Git，就能讓 HTML 的元素有動畫的效果。

這此是為了製作 Loading 的圖示，才會來研究 CSS 動畫。

<!--more-->

# @keyframes

藉由 CSS 動畫可以自動改變樣式，而 `@keyframes` 就是在定義樣式該怎麼變化。

下面這個範例是讓 `background-color` 從紅色變黃色最後再回到紅色，且這個過程會是連續的
，也就是所看到顏色的變化會是紅 -> 橘 -> 黃 -> 紅。
```css
@keyframes example {
    from {background-color: red;}
    to {background-color: yellow;}
}
```
[W3Schools Demo](https://www.w3schools.com/css/tryit.asp?filename=trycss3_animation1)


另一種形式的 `@keyframes`，可以使用百分比來表示。
```css
@keyframes example {
    0%   {background-color: red;}
    25%  {background-color: yellow;}
    50%  {background-color: blue;}
    100% {background-color: green;}
}
```
[W3Schools Demo](https://www.w3schools.com/css/tryit.asp?filename=trycss3_animation2)

> 不是全部的 CSS 屬性都是可動畫化的，可以到這邊查看有哪些可以用。
https://www.w3schools.com/cssref/css_animatable.asp

# animation 語法

定義完 `@keyframes` 後就可以使用 `animation` 套用動畫囉！

語法：
```text
animation: name duration timing-function delay iteration-count direction fill-mode play-state;
```

`animation` 其實是一堆屬性的縮寫，其代表如下：

* **animation-name**: 指定 `@keyframes` 的名稱。
* **animation-duration**: 指定一次循環的週期，可以是秒(s)，或毫秒(ms)。
* **animation-timing-function**: 指定動畫的速度，選項有以下幾個。[W3Schools Demo](https://www.w3schools.com/cssref/playit.asp?filename=playcss_animation-timing-function&preval=linear)
    * linear - 開始至結束速度都一樣
    * ease - 一般 -> 快 -> 慢
    * ease-in - 慢 -> 快
    * ease-out - 快 -> 慢
    * ease-in-out - 慢 -> 快 -> 慢
    * cubic-bezier - 使用貝茲曲線去定義 ([工具](http://cubic-bezier.com/#.09,.87,1,.58))
* **animation-delay**: 指定動畫開始前要延遲的時間，可以是秒(s)，或毫秒(ms)。
* **animation-iteration-count**: 指定動畫的執行次數，可填入數字(預設1)或infinite(無窮)
* **animation-direction**: 指定動畫的方向，選項有以下幾個。[W3Schools Demo](https://www.w3schools.com/cssref/playit.asp?filename=playcss_animation-direction&preval=normal)
    * normal: 預設值，正常方向
    * reverse: 反方向
    * alternate: 正向 -> 反向	
    * alternate-reverse: 反向 -> 正向
* **animation-fill-mode**: 指定動畫開始前與結束後的樣式，選項有以下幾個。
    * none: 預設值，無。
    * forwards: 元素將保留最後一個 `@keyframes` 的樣式，其最後一個取決於 `animation-iteration-count` 與 `animation-direction`
    * backwards: 元素將取得第一個 `@keyframes` 的樣式，延遲期間也會套用，其第一個取決於 `animation-direction`
    * both: 套用上述兩個規則
* **animation-play-state**: 指定動畫當下的播放狀態，可以是 paused 或 running。

推薦工具：http://animista.net/play/basic/scale-up

可以先用這個工具試過各種動畫，在作微調。

# Demo 

用簡單的 CSS 動畫語法，就可以做出不錯的 Loading 圖示。

<style>
.rotate-scale-down {
	animation: rotate-scale-down 1.5s infinite linear both;
	background-color: #19dcea;
	color: #FFF;
    font-size: 50px;
    line-height: 150px;
    text-align: center;
    width: 150px;
    height: 150px;
    border-radius: 15px;
}
@keyframes rotate-scale-down {
    0% {
        transform: scale(1) rotateZ(0);
      }
    25% {
        transform: scale(0.5) rotateZ(80deg);
    }
    50% {
        transform: scale(1) rotateZ(160deg);
    }
    75% {
        transform: scale(1) rotateZ(0);
    }
    100% {
        transform: scale(1) rotateZ(0);
    }
}
</style>

<div class="rotate-scale-down">Puck</div>

```html
<style>
.rotate-scale-down {
	animation: rotate-scale-down 1.5s infinite linear both;
	background-color: #19dcea;
	color: #FFF;
    font-size: 50px;
    line-height: 150px;
    text-align: center;
    width: 150px;
    height: 150px;
    border-radius: 15px;
}
@keyframes rotate-scale-down {
    0% {
        transform: scale(1) rotateZ(0);
      }
    25% {
        transform: scale(0.5) rotateZ(80deg);
    }
    50% {
        transform: scale(1) rotateZ(160deg);
    }
    75% {
        transform: scale(1) rotateZ(0);
    }
    100% {
        transform: scale(1) rotateZ(0);
    }
}
</style>

<div class="rotate-scale-down">Puck</div>
```


參考文件：https://www.w3schools.com/cssref/css3_pr_animation.asp
