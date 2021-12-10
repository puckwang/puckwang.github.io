---
id: "1833f4bd719597a2f907cd5ac87f8906bb75709d"
title: "Git 進階應用 Submodule 與 Subtree，使用它們來拆分專案"
description: "在開發過程中，專案隨著時間變得越來越肥，不時還生出子專案，此時就會遇到需要各專案共用一些 Code 的部分。本篇文將說明使用 Git Submodule 與 Git Subtree 實現匯入子專案的差異及它們的使用方法。 Git Submodule 教學、 Git Subtree 教學"
date: 2020-03-18T21:29:12+08:00
draft: false
featureImage: "/images/hero-main.jpg"
images:
    - "/images/og_image.png"
categories:
    - 版本控制
    
tags:
    - Git
    - Git Submodule
    - Git Subtree
    - 子模組
    - 匯入
    - 模組化
    - 專案拆分

---

在開發過程中，專案隨著時間變得越來越肥，不時還生出子專案，此時就會遇到需要各專案共用一些 Code 的部分，如果共用的部分是用 `複製貼上` 的方式去同步，那勢必一定會造成兩邊不同步，維護困難。

本篇文將分享 Git Submodule 與 Git Subtree 的差異及它們的使用方法。

<!--more-->

以下將以 `SubRepo` 代表要被匯入的 Repository，而 `SuperRepo` 則是把 SubRepo 匯入的 Repository。

## Submodule vs Subtree

Submodule 與 Subtree 兩個都是可以將 `SubRepo` 加入 `SuperRepo` 的解決方法，但怎麼解決的及實際使用上都有所差異。

簡單來說 Submodule 是用像指標的方式，將 SubRepo 的 HASH 紀錄在 SuperRepo 中，而 Subtree 則是以副本的方式把 SubRepo 某版複製一份到 SuperRepo。

用表格看可能就更清楚了：

{{< table >}}

|                 | Submodule                                | SubTree                                   |
|-----------------|------------------------------------------|-------------------------------------------|
| Cost            | 僅佔用 .gitmodule                        | 佔用等同 SubRepo 的大小的空間             |
| Clone SuperRepo | 需多步驟                                 | 原指令                                    |
| Push to SubRepo | **容易**，視為兩個獨立的 Repo，可以直接 push |  **不容易**，因為不知道 SubRepo log，還要比對 |
| pull to SubRepo | **不容易**，需執行另外執行指令                   | **容易**，因為就只是 Pull SuperRepo           |
| 簡單形容          | 一個 Repo 中的另一個 Repo                | 跟原本 Repo 合併，視為一個子目錄           |

{{< /table >}}

另外也可以用一句話的方式描述：

- **Submodule**: 較易 push，較不易 pull，不佔空間，因為它只紀錄 HASH。
- **Subtree**: 較不易 push，較易 pull，不佔空間，因為是副本。

<br>

### Git Submodule 簡介

Submodule 最早出現在 [v1.5.3](https://github.com/git/git/blob/53f9a3e157dbbc901a02ac2c73346d375e24978c/Documentation/RelNotes/1.5.3.txt) (可能吧？)，根據 Release Note 中描述，出現的目的就是用於管理 super-project 中的子專案，可能因為它比較老，網路上的文件相對就比較多了。

它會將指定的 SubRepo 版本 Clone 到指定的路徑，並將這個版本的 HASH 紀錄在 SuperRepo 中。它有自己的 .git 目錄有自己的操作範圍，只要進到這個路徑就等於是進到另一個 Repo 了，外面只會記得目前在哪個版本。

<br>

### Git Subtree 簡介

Subtree 出現的時間我就更不確定了，在 [v1.5.2](https://github.com/git/git/blob/53f9a3e157dbbc901a02ac2c73346d375e24978c/Documentation/RelNotes/1.5.2.txt) 跟 [v1.7.11](https://github.com/git/git/blob/53f9a3e157dbbc901a02ac2c73346d375e24978c/Documentation/RelNotes/1.7.11.txt) 都有出現，且好像一開始目的是用於合併專案的 (?

它一樣是將指定的 SubRepo 版本 Clone 到指定目錄，但差別在它會送一個 merge commit 到 SuperRepo，實際上看起來無故冒出一條線跟 SuperRepo 合併，然後它們就合而為一了。

大概就是像這樣：

{{< mermaid >}}

graph LR
    subgraph SubRepo
        A1[abd7417] --> B1
        B1[c690741] --> C1
        C1[ff35922] --> D
    end
    subgraph SuperRepo
        A[7542f2f] --> B
        B[9db0cd9] --> C
        C[506df19] --> D
        D[Merge] --> F
        F[5d43e32] --> E
        E[d6cbc13]
    end

{{< /mermaid >}}

## 如何使用 Git Submodule ?

### 章節目錄

- [新增 Submodule](#新增-submodule)
- [Pull SuperRepo](#pull-superrepo)
- [Pull Submodule (更新 Submodule 版本)](#pull-submodule-更新-submodule-版本)
- [Push Submodule](#push-submodule)
- [刪除 Submodule](#刪除-submodule)
- [其他人 Clone SuperRepo](#其他人-clone-superrepo)
- [將現有的專案拆出 Submodule](#將現有的專案拆出-submodule)


### 建立 Submodule
要將一個 SubRepo 使用 Submodule 的方式加到 SuperRepo，流程清單如下：

1. `git submodule add <repo url> <folder>`
2. `git commit`
3. `git push`

接下來我們看實際執行畫面：

首先先執行 `git submodule add` 去加入 SubRepo
```sh
$ git submodule add git@github.com:puckwang/SubRepo.git sub-repo
Cloning into '~/sub-repo'...
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (3/3), done.
```

接下來執行 `git status` 就可以看到有新增了兩個未提交的檔案

```sh
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	new file:   .gitmodules
	new file:   sub-repo
```

- `.gitmodules` 是用來記錄 Module 的路徑以及 Repo url。
- `sub-repo` 放 Module 的目錄，雖然會有這個，但其實不會實際將整個 SubRepo 提交出去。

最後記得要 `commit` 和 `push`，另外要注意的是，因為 SuperRepo pull 是不會更新 Submodule，要自己手動[更新](#更新-Submodule)。

```bash
git commit
git push
```

### Pull SuperRepo
如果 SuperRepo 所記錄的 Submodule 版本有更新，執行 `pull` 時是不會自動也去把它更新，且執行 `status` 時會看到提示
```bash
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   sub-repo (new commits)
```

此時就要執行以下指令將 Submodule 更新，執行完再看 `status` 就不會有提示了。
```bash
git submodule update --recursive
```


### Pull Submodule (更新 Submodule 版本)
如果 Submodule 有新的 Commit，在 SuperRepo 執行 pull 是不會更新的，要自己手動執行指令：
```sh
$ git submodule update --recursive --remote

remote: Enumerating objects: 4, done.
remote: Counting objects: 100% (4/4), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (3/3), done.
From github.com:puckwang/SubRepo
   1a39dc5..c9c4974  master     -> origin/master
Submodule path 'sub-repo': checked out 'c9c4974e887f2362cc9f7f9d4c90b19891969d67'
```
- `--recursive`: 代表如果遞迴執行，Submodule 中如果有 Submodule，有一併更新。
- `--remote`: 使用遠端的分支來更新，而不是 SuperRepo 所記錄的 HASH (更新 Submodule 版本用)。

或者是直接到 Submodule 裡面執行 pull 也可以。

執行完後就會發現 SuperRepo 中的 Submodule 目錄會被標示 `(new commits)`，所以要記得 push。
```sh
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   sub-repo (new commits)

no changes added to commit (use "git add" and/or "git commit -a")
```


### Push Submodule
只要把 Submodule 當作一般 Repo 操作就可以了，但當 Submodule 有新的 `commit` 時，SuperRepo 所記錄的 HASH 都會改變，所以如果順序錯了話，會多一次 `commit`。

比較好的順序如下：

1. 在 Submodule commit 變更。
2. 在 Submodule push 剛剛的 commit。
3. 在 SuperRepo commit 變更 (這裡會包含新的 Submodule HASH)。
4. 在 SuperRepo push 或是使用 `git push --recurse-submodules=on-demand` 就可以省略步驟 2。

### 刪除 Submodule
首先用以下指令將 submodule 紀錄刪除
```bash
git submodule deinit -f <submodule folder>
```

接下來刪除 submodule 的 `.git`，他會在 SuperRepo 的 `.git` 中的 `modules` 中。
```bash
rm -rf .git/modules/path/to/submodule
```

最後一步是刪除實體的目錄
```bash
git rm -f path/to/submodule
```

做完以上動作記得也要 `commit` 與 `push`。

```bash
git commit
git push
```


### 其他人 Clone SuperRepo 
其他人在 `Clone` 含有 Submodule 的專案時，要記得在 `clone` 指令後面加上 `--recurse-submodules`，否則預設是不會將 submodule `clone` 下來的。
```bash
git clone <repo url> --recurse-submodules
```

如果是已經 `clone` 的也可以執行以下指令將 Submodule 拉下來
```bash
git submodule init
git submodule update

# or

git submodule update --init
```

### 將現有的專案拆出 Submodule
這個部分應該是大多數人都會遇到的，因為很難在專案一開始就切的這個完美，一定是長到一定的量，維護時覺得不行了才開始來切。

要把現有的專案中其中的某個目錄切成 Submodule 其實不難，也不會遺失紀錄。

首先執行拆解的部分，為了保險起見，請另外 `Clone` 一個乾淨的專案。
```bash
git clone <Super Repo>
```

接下來刪除 origin，因為等等會把 Submodule 推上自己的遠端倉庫，所以這邊就先刪掉原本的 SuperRepo 的遠端倉庫。
```bash
git remote rm origin
```

這個是可選的步驟，如果想要保留 `commit` 就要執行這個動作，他會過濾掉其他 `commit`，只保留我們要的 Submodule 的 `commit`，這就是為什麼一開始要另外 `clone` 一個新的。
```bash
git filter-branch --subdirectory-filter <SubModule folder path> -- --all
```

執行過程會因為專案大小而有所不同，大一點的專案，且那個 Submodule 包含很多 `commit` 時，就會執行一段時間。
```bash
$ git filter-branch --subdirectory-filter app -- --all

Rewrite 9ca4f14172ddcdf1aa9af5b69338bb41eb56162c (143/152) (6 seconds passed, remaining 0 predicted)
```

執行完後應該會只剩所選路徑的 `commit`，並會發現整個根目錄只會剩剛剛所選路徑裡面的東西。

檢查完沒問題後，就可以把要放 Submodule 的倉庫加進來，並推上去了。
```bash
git remote add origin <SubModule Repo>
git push
```

接下來將我們切出來的 SubRepo 加回 SuperRepo，先用 `git rm` 刪除原本的目錄，在用 `git submodule` 把他加回來，就像前面提到 "[加入 Submodule 步驟](#加入-Submodule-步驟)" 一樣
```bash
git rm -r <folder>
git submodule add <git repository B url> <folder>
```

確認完沒問題，能跑的都能跑了，要記得 `commit` 與 `push`
```bash
git commit
git push
```

## 如何使用 Git Subtree ?

我覺得 Subtree 比 Submodule 操作上更簡單，也更容易理解。

### 章節目錄

- [新增 Subtree](#新增-subtree)
- [Pull SuperRepo](#pull-superrepo-subtree)
- [Pull Subtree (更新 Subtree 版本)](#pull-subtree-更新-subtree-版本)
- [Push Subtree](#push-subtree)
- [刪除 Subtree](#刪除-subtree)
- [其他人 Clone SuperRepo](#其他人-clone-superrepo-subtree)
- [將現有的專案拆出 Subtree](#將現有的專案拆出-subtree)

### 新增 Subtree
想要把 SubRepo 用 Subtree 的方式加到 SuperRepo 的話，可以執行以下指令，你必須設定幾個參數， Subtree 的路徑、SubRepo 的遠端倉庫連結及一個版本。

它會將 SubRepo Clone 下來到指定的路徑，並將他 Merge 進我們的 SuperRepo 當前的 HEAD。

```bash
git subtree push --prefix <folder path> <repo url> <ref>
```

> 小技巧，可以將遠端倉庫 Url 設別名，就像 origin 那樣，就不用每次都要輸入那麼長。


實際執行會是這樣
```bash
$ git subtree add --prefix subtree git@github.com:puckwang/SubRepo.git c9c4974e887f2362cc9f7f9d4c90b19891969d67

git fetch git@github.com:puckwang/SubRepo.git c9c4974e887f2362cc9f7f9d4c90b19891969d67
warning: no common commits
remote: Enumerating objects: 6, done.
remote: Counting objects: 100% (6/6), done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 6 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (6/6), done.
From github.com:puckwang/SubRepo
 * branch            c9c4974e887f2362cc9f7f9d4c90b19891969d67 -> FETCH_HEAD
Added dir 'subtree'
```

使用 `git status` 後，會發現自動發了一個 Merge commit。
```bash
$ git status

*   ae3f960 2020-03-19 | Add 'subtree/' from commit 'c9c4974e887f2362cc9f7f9d4c90b19891969d67' (HEAD -> master) [Puck Wang]
|\
| * c9c4974 2020-03-18 | Create feature.txt [Puck Wang]
| * 1a39dc5 2020-03-18 | Initial commit [Puck Wang]
* 6922999 2020-03-19 | Update sub repo (origin/master, origin/HEAD) [Puck Wang]
* 6cb24ae 2020-03-19 | Update sub repo [Puck Wang]
* 811f46a 2020-03-19 | Update subrepo [Puck Wang]
* 73f473c 2020-03-18 | Add sub-repo [Puck Wang]
* 12b9412 2020-03-18 | Initial commit [Puck Wang]
```

然後，就沒有了... 記得 `push`。
```bash
git push
```

### Pull SuperRepo{#pull-superrepo-subtree}

不影響，他被視為一個資料夾而已，直接下 `git pull`。

### Pull Subtree (更新 Subtree 版本)

跟[新增](#新增-subtree)的語法差不多，就只是把 'add' 改成 'pull'，其他參數都一樣，執行後他會直接幫你執行 commit 並跳到輸入 message 的畫面。

```
git subtree pull --prefix <folder path> <repo url> <ref>
```

實際執行的範例

```bash
$ git subtree pull --prefix subtree git@github.com:puckwang/SubRepo.git master
remote: Enumerating objects: 12, done.
remote: Counting objects: 100% (12/12), done.
remote: Compressing objects: 100% (8/8), done.
remote: Total 11 (delta 4), reused 6 (delta 2), pack-reused 0
Unpacking objects: 100% (11/11), done.
From github.com:puckwang/SubRepo
 * branch            master     -> FETCH_HEAD
Merge made by the 'recursive' strategy.
 subtree/123.txt        | 0
 subtree/abc.txt        | 0
 subtree/abc1.txt       | 0
 subtree/feafeat123.ttt | 1 +
 subtree/feature2.txt   | 1 +
 5 files changed, 2 insertions(+)
 create mode 100644 subtree/123.txt
 create mode 100644 subtree/abc.txt
 create mode 100644 subtree/abc1.txt
 create mode 100644 subtree/feafeat123.ttt
 create mode 100644 subtree/feature2.txt
```

跟剛剛一樣，我們用 `git status` 檢視，可以看到像這樣的圖，像一個分支一直重複被合併進來。

```bash
*   6ba2777 2020-03-19 | Merge commit 'b9f546c57282516102e93e9a5315a7287393e805' (HEAD -> master) [Puck Wang]
|\
| * b9f546c 2020-03-19 | Create feafeat123.ttt [Puck Wang]
| * be3de4b 2020-03-19 | Add 123 [Puck Wang]
| * c47d874 2020-03-19 | abc1.txt [Puck Wang]
| * c2b399c 2020-03-19 | abc.txt [Puck Wang]
| * c86fe17 2020-03-19 | Create feature2.txt [Puck Wang]
* |   ae3f960 2020-03-19 | Add 'subtree/' from commit 'c9c4974e887f2362cc9f7f9d4c90b19891969d67' (origin/master, origin/HEAD) [Puck Wang]
|\ \
| |/
| * c9c4974 2020-03-18 | Create feature.txt [Puck Wang]
| * 1a39dc5 2020-03-18 | Initial commit [Puck Wang]
* 6922999 2020-03-19 | Update sub repo [Puck Wang]
* 6cb24ae 2020-03-19 | Update sub repo [Puck Wang]
* 811f46a 2020-03-19 | Update subrepo [Puck Wang]
* 73f473c 2020-03-18 | Add sub-repo [Puck Wang]
* 12b9412 2020-03-18 | Initial commit [Puck Wang]
```

### Push Subtree

前面比較時有提到 Subtree 再回推時會比較難處理，而他的順序如下：

1. 在 SuperRepo commit 變更
2. 在 Subtree push (這會執行比較久) 

而 `push` 的指令也跟前兩個一樣，改前面而已，後面都一樣。

```
git subtree push --prefix <folder path> <repo url> <ref>
```

實際執行範例，過程中會看到他在比對 commit，為多越慢，所以我才說他是比較難處理的。
```bash
$ git subtree push --prefix subtree git@github.com:puckwang/SubRepo.git master
git push using:  git@github.com:puckwang/SubRepo.git master
Counting objects: 2, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (2/2), 237 bytes | 237.00 KiB/s, done.
Total 2 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To github.com:puckwang/SubRepo.git
   b9f546c..b7107b3  b7107b38f5416503718a846cf5c85f1ecb5bb1f0 -> master
```

### 刪除 subtree

跟 Submodule 相比，刪除 Subtree 算非常間單，就只要把資料夾刪掉就可以了。

### 其他人 Clone SuperRepo{#其他人-clone-superrepo-subtree}

不影響，他被視為一個資料夾而已，可直接執行 `git clone`。

### 將現有的專案拆出 Subtree

這個部分也比 Submodule 簡單一點，兩個指令。

首先先把要切出來的資料夾切出來。
```bash
git subtree split --prefix <folder path>
```

接下來直接把他推上遠端倉庫就可以了。
```bash
git subtree push --prefix <folder path> <repo url> <ref>
```

## 結論
在實際實用兩種方式後，我是比較喜歡 Submodule，因為他不佔空間，實際也就是另一個 Repository，而 Subtree 是整個併進來，就不太喜歡。

但實際上要用那種方式，還是要視當下需求再決定。

## 參考文章

- https://stackoverflow.com/questions/17413493/create-a-submodule-repository-from-a-folder-and-keep-its-git-commit-history
- http://yutin.logdown.com/posts/188306-git-subtree-total-addendum-library
- https://codewinsarguments.co/2016/05/01/git-submodules-vs-git-subtrees/
- https://stackoverflow.com/questions/1260748/how-do-i-remove-a-submodule/36593218#36593218
