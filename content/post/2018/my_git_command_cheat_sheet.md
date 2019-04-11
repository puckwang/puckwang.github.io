---
title: "Git 常用指令與設定筆記"
date: 2018-12-24T12:59:13+08:00
draft: false
description: "使用 Git 也有一段時間了，除了常用的 commit 等常用基本指令外，還有許多很實用的指令，在這邊做一下紀錄。"
images:
    - "/images/og_image.png"
tags: 
    - Command
    - Config
    - 設定
    - 指令
    - featured
categories: 
    - Git
    - 筆記
---

使用 Git 也有一段時間了，除了常用的 commit, push 等常用基本指令外，還有許多很實用的指令，在這邊做一下紀錄。

<!--more-->

# 本地相關
### Init
```bash
git init  #初始化 Git 專案。
```

### Add
```bash
git add <pathspec>... #將一個或多個已變更的檔案新增至「待提交」
```
  * `-A, --all`: 全部檔案。

  * `-f, --force`: 允許新增已經被忽略的檔案。

### Remove
```bash
git rm  #刪除檔案並加入 Staged
```
  * `--cached`: 移除已進入 Staged 的檔案，保留本地檔案
  * `-f`: 移除已進入 Staged 的檔案，並移除本地檔案

> 這邊移除並不會真正的從 Git 移除，只會讓他在接下來不會再被加入 Staged，如果要真正移除檔案，請看我之後會寫的文章。

### Commit
```bash
git commit  #提交變更。
```

- `-m, --message <message>`: 加上 Commit Message，預設是開啟指定編輯器。

- `--amend`: 修改上一筆的提交。

- `-S`: 使用 GPG 簽署此 Commit。

### Status
```bash
git status  #查看目前檔案的狀態。
```

### Log
```bash
git log  #查看提交歷史紀錄。

```

- `--verify-signatures`: 顯示 GPG 簽署相關資訊。

### Reset
```bash
git reset [<commit>]  #重置狀態。
```

  * `--soft`: 僅重置 `HEAD` 與 `Index`，工作目錄檔案的變更不會被重置。

  * `--hard`: 重置 `HEAD`、`Index`與工作目錄。

### Branch
```bash
git branch  #查看目前分支清單
git branch <name>  #建立新分支
```

  * `-d, --delete`: 刪除已合併的分支。

  * `-D`: 刪除分支，即使他未合併。

  * `-m, --move`: 更名分支。

  * `-M`: 更名分支，即使目標分支已存在。

### Checkout
```bash
git checkout <branch>  #切換分支。
git checkout [<branch>] <file>  #還原工作目錄的檔案。
```

  * `-b <branch>`: 建立新分支並切換過去。

  * `-B <branch>`: 建立新分支或重置已存在的分支並切換過去。

### Cherry-Pick
```bash
git cherry-pick <commit>...  #取得 Commit 到目前分支
```

### Stash
```bash
git stash  #將未提交的檔案暫時放置臨時區
git stash list  #查看 Stash 放了哪些東西
git stash pop   #將最上層的一筆資料返回工作目錄
```

### Rebase
```bash
git rebase [<branch>]  #將該分支接到目標分支最上層

# 範例:
#此時就會將目前所在分支的 commit 接到 master 最前面，也就是讓目前所在分支保持最新狀態
git rebase master
```

  * `-i`: 可以自訂對每個 Commit 的操作及調整其順序，而操作命令及縮寫如下 (可以打第一個縮寫代替)。
    * `p, pick` = 使用這個 commit。 (default)
    * `r, reword` = 使用這個 commit，但編輯 commit 訊息。
    * `e, edit` = 使用這個 commit，但暫停 rebase 來編輯這個 commit。
    * `s, squash` = 使用這個 Commit，但與前一個 commit 合併。
    * `f, fixup` = 類似 squash，差在會捨棄這個 commit 的訊息。
    * `x, exec` = 執行後面接的的 Shell
    * `d, drop` = 移除這個 commit

> 如果使用 Vim 來編輯，`:wq` 存檔離開就可以完成 rebase

### Merge
```bash
git merge <commit>...  #將指定 Commit 合併到目前所在分支
```

### Show
```bash
git show <object>...  #顯示詳細資訊
```

### Submodule
```bash
git submodule  #列出子模組清單
git submodule add <repository> [<path>]  #新增某遠端倉庫為子模組，執行後會自動 clone
git submodule init [<path>...]  #初始化子模組
git submodule sync [<path>...]  #同步
```

### Verify GPG
```bash
git verify-commit <commit>  # 驗證 Commit 的 GPG 簽署。
git verify-tag <commit>  # 驗證 Tag 的 GPG 簽署。
```

### Reflog
```bash
git reflog # 它會紀錄所有執行等指令，如果 Commit 不見，短時間內都還可以用它查。
```

# 遠端倉庫相關
### Remote
```bash
git remote  #列出遠端倉庫清單
git remote add <name> <repository>  #新增一筆遠端倉庫
git remote get-url <name>  #取得某筆遠端倉庫的 URL
git remote set-url <name> <newurl>  #設定某比遠端倉庫的 URL
```

### Clone
```bash
git clone <url>  #下載遠端倉庫，預設遠端倉庫名稱為 origin
```

### Push
```bash
git push  #將本地的提交推到到預設的遠端倉庫
git push <repository>
```

  * `-f, --force`: 強制推送，也就是強制覆蓋遠端的 commit (例如如果已經將 commit 推上去了，但又修改它，就必須要加上這個才推得上去)。
  * `--force-with-leas`: 跟 `-f` 同功能但比它安全，它會去檢查遠端有沒有其他人的新提交，如果有就會不能推，才不會發生蓋掉別人的紀錄的事情發生。

### Pull
```bash
git pull  #將預設的遠端倉庫同步下來
git pull <repository>
```

  * `-r, --rebase[=<false|true|preserve|interactive>]`: 如果設為 true 時，當遠端與本地有差異，使用 rebase 而不使用 merge。

# 全域設定檔
預設路徑: `~/.gitconfig`
```bash
[user]
	name = Puck Wang
    email = s9801077@gmail.com
[pull]
    #這個設定了之後，pull 時就可以不用加參數
	rebase = true  
[alias]  
    #一大堆的別名，讓打指令更快速
	ff = flow feature start
	fh = flow hotfix start
	pl = pull
	ps = push
	psf = push --force-with-lease
	br = branch
	co = checkout
	cb = checkout -b
	cd = checkout develop
	cm = commit
	st = status
	sh = stash
	mr = merge
	sh = show
	rb = rebase
	rbc = rebase --continue
	a = add
	aa = add *
	l = log

	#以下是網路上找來的 log 指定，但用蠻久了，來源不清楚
	ll = log --pretty=format:'%h %ad | %s%d [%Cgreen%an%Creset]' --graph --date=short
	lo = log --oneline
	lg = log --graph --pretty=format:'%Cred%h%Creset %ad |%C(yellow)%d%Creset %s %Cgreen(%cr)%Creset' --abbrev-commit --date=short
	lgg = log --graph --abbrev-commit --decorate --format=format:'%C(bold blue)%h%C(reset) - %C(bold cyan)%aD%C(reset) %C(bold green)(%ar)%C(reset)%C(bold yellow)%d%C(reset)%n''          %C(white)%s%C(reset) %C(dim white)- %an%C(reset)' --all

[core]
    #設定套用全域忽略的設定檔
	excludesfile = /Users/xxxxxx/.gitignore_global  
```

`.gitignore_global`
```shell
.DS_Store
.idea
.vs
```
