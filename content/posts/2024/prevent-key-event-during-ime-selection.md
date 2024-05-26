---
id: "a39eb1e871e9e320b9ff4b2f49b1e814f0a116df"
title: "JavaScript 避免輸入法選字觸發 Enter 事件的方法"
description: "在寫前端時，有時候會遇到需要在 Input 上偵測 Enter 事件來提供特殊功能，像是搜尋的 Input 可以增加 Enter 執行搜尋。在使用 `event.key` 去判斷是否為 `enter` 時，應該會遇到輸入法選字也會誤判的問題。"
date: 2024-05-26T22:52:26+08:00
draft: false
featureImage: "/images/hero-main.jpg"
images:
  - "/images/og_image.png"
categories:
  - 問題解決
tags:
  - NodeJs
  - JavaScript
  - TypeScript
  - HTML
---

在寫前端時，有時候會遇到需要在 Input 上偵測 Enter 事件來提供特殊功能，像是搜尋的 Input 可以增加 Enter 執行搜尋。

在使用 `event.key` 去判斷是否為 `enter` 時，應該會遇到輸入法選字也會誤判的問題。

<!--more-->

本文記錄一下解法：

提供測試的 Input: <input id="test-input"/>
> 可以開啟 Browser 的 Debug Tool，在 Console 中觀察。

## 舊版解法

除了判斷 `key` 以外再增加判斷 `keyCode`，如果是在輸入法的操作，那 `keyCode` 就會是 229。
但新版的 JS 已將 `keyCode` 棄用，雖然大部分瀏覽器都還會提供這個屬性，不過我還是推薦直接使用[新版解法](#新版解法)比較保險。

**Vanilla JS**

```js
document.getElementById('test-input').addEventListener('keydown', event => {
    if (event.key === 'Enter' && event.keyCode !== 229) {
        event.preventDefault();
        console.log("按下了 Enter");
    }
});
```

## 新版解法

除了判斷 `key` 以外再增加判斷 `isComposing`，如果是在輸入法的操作，那 `isComposing` 就會是 `true`。

**Vanilla JS**

```js
document.getElementById('test-input').addEventListener('keyDown', e => {
    if (event.key === 'Enter' && !event.isComposing) {
        event.preventDefault(); // 防止換行
        // Do some...
    }
})
```

**React**

```tsx
const handleKeyDown = (event: React.KeyboardEvent<HTMLTextAreaElement>) => {
	// 限制只在單獨按下 Enter 且非中文選字時才執行送出
	if (event.key === 'Enter' && !event.nativeEvent.isComposing) {
		event.preventDefault(); // 防止換行
		// Do some...
	}
};
```

<script>
document.getElementById('test-input').addEventListener('keydown', event => {
    console.log(`key = ${event.key}; keyCode = ${event.keyCode}; isComposing = ${event.isComposing}`);

    if (event.key === 'Enter' && !event.isComposing) {
        event.preventDefault(); 
        console.log("按下了 Enter (新版解法)");
    }

    if (event.key === 'Enter' && event.keyCode !== 229) {
        event.preventDefault(); 
        console.log("按下了 Enter (舊版解法)");
    }
});
</script>

## 參考資料

- https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/isComposing
- 
