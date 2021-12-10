---
id: "78fe26ea1be2030cfe68bbdd2a9d13d322c76d4a"
title: "新增 Bootstrap 4 到用 vue-cli 3.x 建立的專案中"
date: 2018-12-04T22:31:10+08:00
draft: false
description: "新增 Bootstrap 4 到用 vue-cli 3.x 建立的專案中"
images:
    - "/images/og_image.png"
tags: 
    - Front-end
    - Vue
    - Web
    - bootstrap
categories: 
    - 軟體開發
---

在這邊紀錄一下怎麼將 Bootstrap 4 新增進 vue-cli 3.x 所建立的專案中。

<!--more-->

## 1. Install

使用 npm 安裝 Bootstrap 4 以及相依套件。

```bash
npm install bootstrap jquery popper.js

// or

yarn add bootstrap jquery popper.js
```

## 2. Import

在 `main.js` 中匯入 Bootstrap 的 js 與 css

```javascript
import 'bootstrap'; // Import js file
import 'bootstrap/dist/css/bootstrap.min.css'; // Import css file
```

接下來就可以去檢查看看有沒有匯入成功囉！

<img src="/images/2018/vue_cli_bootstrap4_success.png" alt="Image" width="500">

---
## 客製化 Bootstrap

如果想要客製化一些 Bootstrap 的變數或樣式，可以自己開一個 `custom.scss` 的檔案，在裡面寫好要改的變數或樣式後，
再開一個 `main.scss` 的檔案，並把匯入 Bootstrap 的 scss 與剛剛的 `custom.scss`，如下：

```scss
@import "custom";
@import "~bootstrap/scss/bootstrap";
```

之後再把 `main.scss` 匯入 `main.js` 就可以了
```javascript
import './styles/main.scss';
```

