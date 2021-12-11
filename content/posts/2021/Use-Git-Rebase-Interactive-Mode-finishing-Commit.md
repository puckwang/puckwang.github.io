---
id: "fe76417f8cf478d210f4278e2e79e8db6fef3d4a"
title: "使用 Git Rebase Interactive 模式整理 Commit"
description: "利用 git rebase 的互動模式，可以讓你簡單的去調整 Commit 的順序；或是拆分過大的 Commit；也可以刪除不必要的 Commit"
date: 2021-07-19T21:09:02+08:00
draft: false
featureImage: "/images/hero-main.jpg"
images:
    - "/images/og_image.png"
categories:
    - 版本控制
tags:
    - Git
aliases:
    - /post/2021/use-git-rebase-interactive-mode-finishing-commit/
---

{{< img src="/images/2021/git-rebase.png" width="600" caption="Git Rebase" >}}

利用 git rebase 的互動模式，可以讓你簡單的去調整 Commit 的順序；或是拆分過大的 Commit；也可以刪除不必要的 Commit。

<!--more-->

## 什麼是 Git Rebase

Git Rebase 是一個內建的 git 指令，可以用來將 Commit 重新提交到新的基礎。

下面這個範例中有一個 feature 分支是從 master 開出來，而 master 也有新的提交。

```
         E---F---G feature
        /
   A---B---C---D master
```

我們可以執行在 feature 分支執行 `git rebase master`，就可以把在 feature 的 Commit 以 D 為基礎重新套用，結果如下：

```
                 E'---F'---G' feature
                /
   A---B---C---D master
```

預設執行 Rebase 是會套用全部的 Commit，如果我們使用互動模式 (`git rebase [-i | --interactive]`) 就可以針對個別 Commit 去自訂操作。

## 進入互動模式選單

在原本的指令中，加入 `-i` 或是 `--interactive` 就可以進入互動模式。

```
git rebase -i <起始 Commit>
```

執行後，預設會使用 vim 開啟互動模式的介面，調整完後執行 `:wq` 存擋關閉 vim，就會依照指定的指令執行。

上半部未註解的部分會由上至下列出從指定的 Commit 到 HEAD 之間的所有 Commit，一行代表一個 Commit 及對應的操作，從頭依序 **操作指令**、**Hash** 及 **Commit 訊息**。

下半部有註解的部分，是一些指令的簡介及縮寫，使用時就可以直接參考不用去特別記它的指令及用法。

```shell
pick d34548f Add feature 1
pick 98fb1b9 Add feature 2
pick cbf941f Add feature 3
pick 1499a17 Add feature 4
pick 3e14876 Add feature 5

# 重定基底 bda1985..1499a17 到 bda1985（6 個提交）
#
# 指令:
# p, pick <提交> = 使用提交
# r, reword <提交> = 使用提交，但修改提交說明
# e, edit <提交> = 使用提交，進入 shell 以便進行提交修補
# s, squash <提交> = 使用提交，但融合到前一個提交
# f, fixup <提交> = 類似於 "squash"，但捨棄提交說明日誌
# x, exec <指令> = 使用 shell 執行指令（此行剩餘部分）
# b, break = 在此處停止（使用 'git rebase --continue' 繼續重定基底）
# d, drop <提交> = 刪除提交
# l, label <label> = 為目前 HEAD 打上標記
# t, reset <label> = 重設 HEAD 到該標記
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       建立一個合併提交，並使用原始的合併提交說明（如果沒有指定
# .       原始提交，使用備註部分的 oneline 作為提交說明）。使用
# .       -c <提交> 可以編輯提交說明。
#
# 可以對這些行重新排序，將從上至下執行。
#
# 如果您在這裡刪除一行，對應的提交將會遺失。
#
# 然而，如果您刪除全部內容，重定基底動作將會終止。
```

在上半部 Commit 的部分可以看到預設指令都是 `pick` (使用提交)，我們可以藉由更改這個指令來對任意的 Commit 進行編輯或刪除之類的操作。

需要注意的是開頭指令及 Hash 的部分，都要輸入正確否則它會無法辨識，而 Commit 訊息的部分就沒差了，不會影響執行。

常用的指令說明：

- `pick`：預設都是這個指令，代表會使用這個 Commit。
- `reword`：使用這個 Commit，但是執行到此 Commit 時會開啟 vim 供更改 Commit 訊息。
- `edit`：使用這個 Commit，但是執行到此 Commit 時會暫停，直到執行 `git rebase --continue`。
- `squash`：將這個 Commit 與前一個 Commit 合併，訊息也會合併。
- `fixup`：與 `squash` 相同，但會捨棄這個 Commit 的訊息。
- `drop`：刪除這個 Commit，結果同直接刪除行。

## 範例

這裡將透過一個範例來展示幾個常用操作。

原始的 Commit 如下：

```shell
* 655082e 2021-07-19 | Add feature 6 (HEAD)
* 69a3e14 2021-07-19 | Add feature 5-1
* b99b647 2021-07-19 | Add feature 4
* b2e9250 2021-07-19 | Add feature 5
* 8ad13c6 2021-07-19 | Add feature 3
* c3e8936 2021-07-19 | Fix feature 2 bug
* e4e3c6f 2021-07-19 | Add feature 2
* 9fa7ff7 2021-07-19 | Add feature 1
* bda1985 2021-07-19 | Init Commit
```

需求：

1. 交換 `Add feature 4` 及 `Add feature 5` 的順序。
2. 合併 `Add feature 2` 及 `Fix feature 2 bug` (不留訊息)。
3. 刪除 `Add feature 5-1`


#### Step 1. 執行 git rebase

執行以下指令開啟 git rebase 互動模式，並指定 Rebase `bda1985` 之後的 Commit：

```
git rebase -i bda1985
```

執行後，將顯示以下資訊

```shell
pick 9fa7ff7 Add feature 1
pick e4e3c6f Add feature 2
pick c3e8936 Fix feature 2 bug
pick 8ad13c6 Add feature 3
pick b2e9250 Add feature 5
pick b99b647 Add feature 4
pick 69a3e14 Add feature 5-1
pick 655082e Add feature 6

# 重定基底 bda1985..655082e 到 bda1985（8 個提交）
# ...
```

#### Step 2. 交換 `Add feature 4` 及 `Add feature 5` 的順序

直接將 `Add feature 4` 及 `Add feature 5` 兩行交換順序。

```shell
pick 9fa7ff7 Add feature 1
pick e4e3c6f Add feature 2
pick c3e8936 Fix feature 2 bug
pick 8ad13c6 Add feature 3
pick b99b647 Add feature 4
pick b2e9250 Add feature 5
pick 69a3e14 Add feature 5-1
pick 655082e Add feature 6

# 重定基底 bda1985..655082e 到 bda1985（8 個提交）
```

#### Step 3. 合併 `Add feature 2` 及 `Fix feature 2 bug`

為了將 `Fix feature 2 bug` 合併至 `Add feature 2`，更改 `Fix feature 2 bug` 的指令為 `fixup` (與上一個合併但不保留訊息)。

```shell
pick 9fa7ff7 Add feature 1
pick e4e3c6f Add feature 2
fixup c3e8936 Fix feature 2 bug
pick 8ad13c6 Add feature 3
pick b99b647 Add feature 4
pick b2e9250 Add feature 5
pick 69a3e14 Add feature 5-1
pick 655082e Add feature 6

# 重定基底 bda1985..655082e 到 bda1985（8 個提交）
```

#### Step 4. 刪除 `Add feature 5-1`

因為要刪除 `Add feature 5-1` 這個 Commit，所以將這一行直接移除。

```shell
pick 9fa7ff7 Add feature 1
pick e4e3c6f Add feature 2
fixup c3e8936 Fix feature 2 bug
pick 8ad13c6 Add feature 3
pick b99b647 Add feature 4
pick b2e9250 Add feature 5
pick 655082e Add feature 6

# 重定基底 bda1985..655082e 到 bda1985（8 個提交）
```

#### Step 5. 套用 Rebase 指令

在 Vim 中執行 `:wq` 就可以關閉互動模式，並執行 Rebase。

執行結果如下，可以看到已完成上面列的需求。

```shell
* 31d6978 2021-07-19 | Add feature 6 (HEAD -> master)
* 5e4f588 2021-07-19 | Add feature 5
* 04b8a95 2021-07-19 | Add feature 4
* 17fc05f 2021-07-19 | Add feature 3
* a30783f 2021-07-19 | Add feature 2
* 9fa7ff7 2021-07-19 | Add feature 1
* bda1985 2021-07-19 | Init Commit
```

執行過程動畫：

{{< img src="/images/2021/git-rebase-demo.gif" width="850" caption="Git Rebase Demo" >}}


## 總結

利用 Git Rebase 的互動模式介面，可以快速的整理 Commit，讓我們的變更紀錄更容易理解。但 rebase 的操作是會變更整個歷史，要注意一點是 **不要在共用的分支進行 rebase**，會造成其他人的歷史紀錄亂掉。


## 參考資料

- Git - git-rebase Documentation
    - https://git-scm.com/docs/git-rebase
