---
title: "解決使用 Laravel-mix 時 Phpstorm 會 lag 的問題"
date: "2017-09-05T22:21:26+08:00"
description: ""
images:
    - "/images/og_image.png"
tags: ["Laravel", "phpstorm"]
categories: ["問題解決"]
---

最近開始在用laravel 5.5開發專案時發現的問題，只要用執行`npm run watch`後，
每次跑完編譯phpstorm都會停頓幾秒。

<!--more-->

經過一番搜尋，終於在laracasts找到的國外網友也有發生類似問題。 
[原文](https://laracasts.com/discuss/channels/javascript/laravel-mix-phpstorm-lag)

原因是因為每次build完後phpstorm都去重建index，所以才照成延遲問題，其解法就是將`public\js`跟`public\css`在phpstorm設定排除就可以了。

<img src="/images/2017/2017_09_05_solve_laravel_mix_phpstorm_lag.png" alt="Image" width="500">
