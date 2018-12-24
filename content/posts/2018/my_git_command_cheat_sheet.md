---
title: "我的 Git 實用指令小抄"
date: 2018-12-24T13:59:13+08:00
Lastmod: 2018-12-24T13:59:13+08:00
draft: false
description: "使用 Git 也有一段時間了，除了常用的 commit 等常用基本指令外，還有許多很實用的指令，在這邊做一下紀錄。"
tags: ["Command"]
categories: ["Git", "筆記"]
---

使用 Git 也有一段時間了，除了常用的 commit, push 等常用基本指令外，還有許多很實用的指令，在這邊做一下紀錄。

<!--more-->

# Basic

## Init
```git
git init  #初始化 Git 專案。
```

## Remote
```git
git remote  #列出遠端倉庫清單
git remote add <name> <url>  #新增一筆遠端倉庫
git remote get-url <name>  #取得某筆遠端倉庫的 URL
git remote set-url <name> <newurl>  #設定某比遠端倉庫的 URL
```

## Add
```git
git add <pathspec>... #將一個或多個已變更的檔案新增至「待提交」
```
  * `-A, --all`: 全部檔案。
  
  * `-f, --force`: 允許新增已經被忽略的檔案。

## Remove
```git
git rm --cached <file>...  #將檔案從歷史紀錄中完整刪除 ps:多用於刪除機密檔案
```

## Commit
```git
git commit  #提交變更。
```

  * `-m, --message <message>`: 加上 Commit Message，預設是開啟指定編輯器。
  
  * `--amend`: 修改上一筆的提交。
  
## Status
```git
git status  #查看目前檔案的狀態。
```

## Log
```git
git log  #查看提交歷史紀錄。

```

## Reset
```git
git reset [<commit>]  #重置狀態。
```

  * `--soft`: 僅重置 `HEAD` 與 `Index`，工作目錄檔案的變更不會被重置。
  
  * `--hard`: 重置 `HEAD`、`Index`與工作目錄。

## Branch
```git
git branch  #查看目前分支清單
git branch <name>  #建立新分支
```

  * `-d, --delete`: 刪除已合併的分支。
  
  * `-D`: 刪除分支，即使他未合併。
  
  * `-m, --move`: 更名分支。
    
  * `-M`: 更名分支，即使目標分支已存在。

## Checkout
```git
git checkout <branch>  #切換分支。
git checkout [<branch>] <file>  #還原工作目錄的檔案。
```

  * `-b <branch>`: 建立新分支並切換過去。
  
  * `-B <branch>`: 建立新分支或重置已存在的分支並切換過去。

## Cherry-Pick
```git
git cherry-pick <commit>...  #取得 Commit 到目前分支
```

## Stash
```git
git stash  #將未提交的檔案暫時放置臨時區
git stash list  #查看 Stash 放了哪些東西
git stash pop   #將最上層的一筆資料返回工作目錄
```

## Rebase
```git
git rebase [<branch>]  #將該分支接到目標分支最上層

範例:
#此時就會將目前所在分支的 commit 接到 master 最前面，也就是讓目前所在分支保持最新狀態
git rebase master

```

  * `-i`: 編輯每個 Commit 的操作。

# Advanced

未完待續.....