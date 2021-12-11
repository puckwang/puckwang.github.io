---
id: "783737326a6a298c206876b9faa15bcb9245f061"
title: "Vue Router Lazy Loading"
description: "SPA 的網站會因為功能變多造成 build 出來的產物越來越大，雖然 Webpack 可以切 Chunk，但也會讓載入的時間變長，此時可以搭配 Vue 的 Components Dynamic Async 的功能，讓 Component 只在使用到的時候才去載入，不會一次就全部載完。"
date: 2019-04-14T12:23:35+08:00
draft: false
images:
    - "/images/og_image.png"
categories:
    - 軟體開發
    
tags:
    - Front-End
    - Vue
    - Web
    - dynamic import
    - Lazy
    - Vue Router
aliases:
    - /post/2019/vue_router_lazy_loading/
---

SPA 的網站會因為功能變多造成 build 出來的產物越來越大，雖然 Webpack 可以切 Chunk，但也會讓載入的時間變長，此時可以搭配 Vue 的
`Components Dynamic Async` 的功能，讓 Component 只在使用到的時候才去載入，不會一次就全部載完。
<!--more-->

## 安裝
安裝 `@babel/plugin-syntax-dynamic-import` 與 `babel-plugin-component`
```bash
yarn add @babel/plugin-syntax-dynamic-import
yarn add babel-plugin-component
```

## 修改 Router
修改 Vue 的 Router，改使用 `() => import(xxx)` 的方式匯入，如果要使用命名 Chunk，可以加上 `/* webpackChunkName: "Home" */` 的設定 (需要 Webpack > 2.4)。

```js
 routes: [
    {
        path: '/',
        name: 'home',
        component: () => import(/* webpackChunkName: "Home" */ './views/Home.vue'),
    },
]
```

## 修改 babel.config.js

請將以下設定複製到 `babel.config.js` 裡面的 `plugins` 陣列中，如果沒有自己就新增。
```js
[
    "component",
    {
        "libraryName": "element-ui",
        "styleLibraryName": "theme-chalk"
    },
    "syntax-dynamic-import"
]
```

新增之後會像這樣
```js
module.exports = {
    presets: [
        '@vue/app',
    ],
    plugins: [
        [
            "component",
            {
                "libraryName": "element-ui",
                "styleLibraryName": "theme-chalk"
            },
            "syntax-dynamic-import"
        ]
    ]
}

```


## Check
如果以上設定都正確，應該可以看到在 Build 時會將 js 及 css 會被依照 Component 切開並以剛剛的設定命名，那如果沒有命名的話，則為 `chunk-<HASH>` 。

```
File                                      Size             Gzipped

dist/js/chunk-vendors.581c5419.js         1544.14 KiB      532.24 KiB
dist/js/LinearEquationsSystem.dfe60d31    553.85 KiB       157.23 KiB
.js
dist/js/HashGenerator.8fff6a26.js         299.59 KiB       100.21 KiB
dist/js/ConsoleTextColor.646655f1.js      173.49 KiB       57.01 KiB
dist/js/QrCode.5b2cd9c0.js                50.63 KiB        14.80 KiB
dist/js/Codecs.96c54721.js                34.11 KiB        11.10 KiB
dist/js/Timer.3deaa96b.js                 32.53 KiB        6.87 KiB
dist/js/LuckyDraw.6be6aa36.js             31.49 KiB        5.88 KiB
dist/js/app.a4a82af8.js                   24.30 KiB        9.91 KiB
dist/js/EatSomething.c64031b3.js          8.28 KiB         2.66 KiB
dist/js/PasswordGenerator.b0d3641e.js     6.69 KiB         1.98 KiB
dist/js/Home.97b62c6e.js                  2.60 KiB         1.27 KiB
dist/js/Disclaimer.38d74c8d.js            1.94 KiB         0.96 KiB
dist/js/NotFound.9965bb20.js              1.13 KiB         0.56 KiB
dist/js/About.aa93a27f.js                 0.40 KiB         0.28 KiB
dist/css/app.6e24b52c.css                 171.19 KiB       24.15 KiB
dist/css/ConsoleTextColor.1bad9303.css    6.66 KiB         1.79 KiB
dist/css/EatSomething.9fc153d6.css        0.43 KiB         0.20 KiB
```

## 相關連結

[Vue Router lazy loading](https://router.vuejs.org/zh/guide/advanced/lazy-loading.html#%E6%8A%8A%E7%BB%84%E4%BB%B6%E6%8C%89%E7%BB%84%E5%88%86%E5%9D%97)
