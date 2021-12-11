---
id: "1e909c304bcd9e9a1868d875dc44e556f0148a77"
title: "使用 Git Bisect 快速找到第一個有問題的 Commit"
description: "現在專案很常使用 Git 作為版本控制系統，所以在遇到 Bug 找不到是哪裡出錯時，可以藉由找出第一次出錯的 Commit 來找到問題原因。在大型專案中，全部 Commit 可能達上千筆，如果遇到很久沒發現的 Bug，就可能會發比較久的時間去找是哪個 Commit 出問題。 常見的可能會看 Commit 訊息來反推可能有問題的 Commit，或是用最笨的方法一個個往回找，這樣效率都不太好。 利用 Git 內建的 Git Bisect 來使用二元搜尋的方式來找有問題的 Commit，就可以大大提升效率。"
date: 2021-04-19T03:34:36+08:00
draft: false
featureImage: "/images/hero-main.jpg"
images:
- "/images/og_image.png"
categories:
- 版本控制
- Debug
tags:
- Git
aliases:
  - /post/2021/use-git-bisect-debug/
---

{{< img src="/images/2021/git-log.png" width="600" caption="Git log" >}}

現在專案很常使用 Git 作為版本控制系統，所以在遇到 Bug 找不到是哪裡出錯時，可以藉由找出第一次出錯的 Commit 來找到問題原因。

但在大型專案中，全部 Commit 可能達上千筆，如果遇到很久沒發現的 Bug，就可能會發比較久的時間去找是哪個 Commit 出問題。

常見的可能會看 Commit 訊息來反推可能有問題的 Commit，或是用最笨的方法一個個往回找，這樣效率都不太好。

利用 Git 內建的 **Git Bisect** 來使用二元搜尋的方式來找有問題的 Commit，就可以大大提升效率。

<!--more-->

**目錄**

- [TL;DR](#tldr)
- [Git Bisect](#git-bisect)
- [基本指令](#基本指令)
- [範例](#範例)
- [重播操作](#重播操作)
- [使用腳本自動化執行](#使用腳本自動化執行)
- [總結](#總結)
- [參考資料](#參考資料)

## TL;DR

不想看內文？執行步驟如下：

```shell
# 開始搜尋並標記好與壞的 Commit
git bisect start <壞的 Commit> <好的 Commit> 

# 開始後將自動 Checkout 到要檢查的 Commit。
# 請執行以下指令標記正常或錯誤
git bisect good
git bisect bad

# 看到以下文字，代表找到第一個有問題的 Commit。
0675162a1b00bf8d2cdd535d4fc80e3a31196a8e is the first bad commit
```

## Git Bisect

Git Bisect 是一個內建於 Git 的指令，它是利用二元搜尋的方式協助使用者找到錯誤的 Commit。

首先要標記一個有問題的 Commit 及一個沒有問題的 Commit，接下來就照的它的指示標記正常或錯誤，就可以在 O(logn) 的時間複雜度下找到有問題的 Commit。

{{< img src="/images/2021/Git-Bisect-Demo.gif" width="700" caption="Git Bisect 執行範例 Gif" >}}

**大略操作步驟**

1. 執行開始。
2. 標記一個好的 Commit 與一個壞的 Commit。
2. 標記 Git 自動切換的 Commit 是好的或壞的。
3. 成功找到第一個有問題的 Commit。

## 基本指令

這邊只列出基本的指令，更詳細可洽[官方文件](https://git-scm.com/docs/git-bisect)。

**開始與停止**

```shell
git bisect start # 開始搜尋
git bisect start HEAD 51dfd5c # 開始搜尋，並標記 HEAD 為壞的，51dfd5c 為好的。

git bisect reset # 停止搜尋
```

**標記 Commit**

```shell
git bisect bad # 標記目前版本為壞的
git bisect bad a15fd5c # 標記 a15fd5c 為壞的

git bisect good # 標記目前版本為壞的
git bisect good a15fd5c # 標記 a15fd5c 為好的

git bisect skip # 跳過目前版本

git bisect reset a15fd5c # 重置 a15fd5c 標記
```

**查看與重播紀錄**

```shell
git bisect log # 查看操作記錄
git bisect replay <file-path> # 自動執行 git bisect log 所記錄的內容

git bisect visualize # 視覺化檢視
```

## 範例

現在我們有一個專案，完整的 Git log 如下，假設在最新的 Commit `2795664` (Add feature 7) 發現了一個 Bug，想找到首次出現這個 Bug 的 Commit。

```shell
2795664 2021-04-31 | Add feature 7 (HEAD -> master) [Puck Wang]
aef2888 2021-03-15 | Add feature 6 [Puck Wang]
6eb2847 2021-02-29 | Add feature 5 [Puck Wang]
0675162 2021-01-04 | Add feature 4 [Puck Wang]
1fd2362 2020-12-24 | Fix bug [Puck Wang]
93a6c55 2020-11-31 | Update README.md [Puck Wang]
5de573a 2020-09-22 | Add feature 3 [Puck Wang]
b297615 2020-06-05 | Add feature 2 [Puck Wang]
3c7a4b7 2020-04-31 | Add feature 1 [Puck Wang]
51dfd5c 2020-02-29 | Init Commit [Puck Wang]
```

經測試發現，這個 Bug 是在 `3c7a4b7` (Add feature 1) 是不存在的，所以我們將 `2795664` (HEAD) 標記為壞的，`3c7a4b7` 標記為好的。

執行完就可以看到回應說剩幾個 Commit 及步驟。

```shell
$ git bisect start HEAD 3c7a4b7

Bisecting: 3 revisions left to test after this (roughly 2 steps)
[1fd23627446a40e6a4899168967236d7e7ff44b5] Fix bug
```

此時 Git 也自動 Checkout 到 `1fd2362` 了，接下來就重複測試有無 Bug，並標記正常或錯誤即可。

經測試後發現 `1fd2362` 是正常的，所以標記正常。

```bash
Puck@Puck-MBPR:git-bisect-test <1fd2362>
$ git bisect good

Bisecting: 1 revision left to test after this (roughly 1 step)
[6eb2847db93f3a059ade76a41ca42b533a03c854] Add feature 5
```

經測試後發現 `6eb2847` 是有問題的，所以標記錯誤。


```bash
Puck@Puck-MBPR:git-bisect-test <6eb2847>
$ git bisect bad

Bisecting: 0 revisions left to test after this (roughly 0 steps)
[0675162a1b00bf8d2cdd535d4fc80e3a31196a8e] Add feature 4
```

重複幾次後，就可以看到回應找到第一個有問題的 Commit 了 (`<xxxxx> is the first bad commit` )。

```bash
Puck@Puck-MBPR:git-bisect-test <0675162>
$ git bisect bad

0675162a1b00bf8d2cdd535d4fc80e3a31196a8e is the first bad commit
commit 0675162a1b00bf8d2cdd535d4fc80e3a31196a8e
Author: Puck Wang <me@puckwang.com>
Date:   Fri Apr 31 21:52:51 2021 +0800

    Add feature 4

:100755 100755 af74d9400639d947d035d1b85067ff08c6565da9 94663fe868dbc3a50b5683fe476b9a64aba89a82 M	test.sh
```

知道第一個有問題的 Commit 後，就可以進一步去查是什麼原因造成這個 Bug。

如果要離開搜尋，需要執行 `git bisect reset`。

## 重播操作

在前面有提到可以用 `git bisect log` 查看操作記錄，我們可以將它存成檔案，再利用 `git bisect replay` 進行重播。

儲存至 bisect-log.txt：
```shell
$ git bisect log > bisect-log.txt
```

使用 bisect-log.txt 進行重播：
```shell
$ git bisect replay bisect-log.txt

We are not bisecting.
Bisecting: 3 revisions left to test after this (roughly 2 steps)
[1fd23627446a40e6a4899168967236d7e7ff44b5] Fix bug
0675162a1b00bf8d2cdd535d4fc80e3a31196a8e is the first bad commit
commit 0675162a1b00bf8d2cdd535d4fc80e3a31196a8e
Author: Puck Wang <me@puckwang.com>
Date:   Fri Apr 31 21:52:51 2021 +0800

    Add feature 4

:100755 100755 af74d9400639d947d035d1b85067ff08c6565da9 94663fe868dbc3a50b5683fe476b9a64aba89a82 M	test.sh
```

## 使用腳本自動化執行

如果你有一個腳本或程式可以判斷這個 Commit 是否正常，就可以用 `git bisect run` 去自動地找到第一個有問題的 Commit。

它會使用 **Exit code** 去判斷好壞，規則如下：
- 0：代表正常
- 1~127：代表錯誤
- 125：代表跳過

### 範例

這個範例是使用一個腳本 `test.sh` 來模擬正常與否，並利用 `exit 1` 來模擬錯誤。

開始前都要先標記一個好的與一個壞的 Commit：
```shell
git bisect start HEAD 3c7a4b7
```

使用 `git bisect run` 並於參數中設定檢查指令，執行後就可以看到他自動檢查 Commit 有沒有問題，並找到第一個有問題的 Commit。
```shell
$ git bisect run ./test.sh
running ./test.sh
OK!
Bisecting: 1 revision left to test after this (roughly 1 step)
[6eb2847db93f3a059ade76a41ca42b533a03c854] Add feature 5
running ./test.sh
Error!
Bisecting: 0 revisions left to test after this (roughly 0 steps)
[0675162a1b00bf8d2cdd535d4fc80e3a31196a8e] Add feature 4
running ./test.sh
Error!
0675162a1b00bf8d2cdd535d4fc80e3a31196a8e is the first bad commit
commit 0675162a1b00bf8d2cdd535d4fc80e3a31196a8e
Author: Puck Wang <me@puckwang.com>
Date:   Fri Apr 31 21:52:51 2021 +0800

    Add feature 4

:100755 100755 af74d9400639d947d035d1b85067ff08c6565da9 94663fe868dbc3a50b5683fe476b9a64aba89a82 M	test.sh
bisect run success
```

## 總結

**Git Bisect** 讓我們更方便地去 Debug，可以省去大量的時間，且又是內建於 Git 的工具，不用另外安裝。

而在 CI/CD 中，也可以在發生錯誤時，使用 `git bisect run` 搭配自動化測試，自動地找出有問題的 Commit 並通知作者。

## 參考資料

- Git - git-bisect Documentation
    - https://git-scm.com/docs/git-bisect
