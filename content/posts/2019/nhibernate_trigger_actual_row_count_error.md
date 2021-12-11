---
id: "61ad91fcf43445f5c2b9f17ddb3863b5460a055c"
title: "專案有使用 NHibernate 時，為 Table 加上 Trigger 出現錯誤"
description: "專案有使用 NHibernate 時，為 Table 加上 'Trigger 出現 Batch update returned unexpected row count from update; actual row count: 2; expected: 1' 的錯誤"
date: 2019-03-15T23:29:28+08:00
draft: false
images:
    - "/images/og_image.png"
categories:
    - 問題解決紀錄
    - 軟體開發
tags:
    - NHibernate
    - trigger
    - Database
aliases:
    - /post/2019/nhibernate_trigger_actual_row_count_error/
---

最近有踩到這個雷，當使用 NHibernate 去存取資料庫時，你在變更的 Table 加上 Trigger，
而這個 Trigger 有去異動到資料，

那 NHibernate 的類似保護機制的東西就會噴這個錯誤:

```text
Batch update returned unexpected row count from update; actual row count: 2; expected: 1
```

<!--more-->
就算不懂英文拿去翻譯也知道他是說:「我只動了 1 Row，DB 卻說我動了 2 Row，這有問題，RollBack！」

既然這樣，應該有方法要 DB 不去計算 Trigger 所異動的 Row

經 Google 後查到了這個設定:

```sql
SET NOCOUNT ON
```

把它加到 Trigger 內的第一行，就可以解決這個問題了！

