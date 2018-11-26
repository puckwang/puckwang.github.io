+++
title = "解決 vuex 在 Laravel-mix 中使用展開運算子(...)出現語法錯誤的問題"
date = "2017-09-06T01:09:04+08:00"
description = ""
tags = ["laravel", "Vue"]
categories = ["問題解決"]
+++

在你把vuex的Actions與getters注入到components中時，可能會使用到展開運算子(...)，此時你可能會遇到語法錯誤的問題。

<!--more-->

```text
 ERROR  Failed to compile with 2 errors                                                                                                                                               上午1:06:35

 error  in ./resources/assets/js/components/include/Sidebar.vue

Syntax Error: Unexpected token (25:8)

  23 |     },
  24 |     computed: {
> 25 |         ...mapGetters([
     |         ^
  26 |             'getArticleIndex'
  27 |         ]),
  28 |     },
```

這是因為laravel-mix缺少`stage-2`的關係，把它補裝起來就可以解決這個問題。

1. 安裝stage-2
```bash
npm install --save-dev babel-preset-stage-2
```

2. 建立`.babelrc`到你專案的根目錄中，並新增一下內容
```json
{
  "presets": ["stage-2"]
}
```

3. 執行編譯專案`npm run dev` or `npm run watch`
