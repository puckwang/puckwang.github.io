---
id: "745804413fb73a691e632edf27da1a0e30f6ed47"
title: "使用 Github Actions 來自動化部署 Hugo 到 Github Pages"
description: "利用 Github Actions 在將 Commit 推上 Github 後，自動化執行 Hugo 建置並部署結果到 Github Pages"
date: 2020-03-14T21:22:30+08:00
draft: false
images:
    - "/images/og_image.png"
categories:
    - DevOps
    - 精選
tags:
    - Github Actions
    - Hugo
    - 部署
    - CI/CD
    - 自動化
    - 建置
    - Workflows
    
---
{{<mermaid>}}
graph LR;
    A[Self] -->|Commit| B
    B[Github] --> C
    C[Github Actions] -->|Build/Deploy| D
    D[Github Pages]
{{< /mermaid >}}

本文說明如何利用 Github Actions 在將 Commit 推上 Github 後，自動化執行 Hugo 建置並部署結果到 Github Pages。

<!--more-->

以往如果想在 Github 處理 CI/CD 的部分，大多專案或團隊都會去使用 `TravisCI`，我在使用上是感到蠻不方便的，有些設定沒有跟 Github 直接整合。

接下來將利用這兩個 Actions 來達到自動化部署 Hugo：

- [peaceiris/actions-hugo](https://github.com/peaceiris/actions-hugo): 負責設定 Hugo 與建置專案。
- [peaceiris/actions-gh-pages](https://github.com/peaceiris/actions-gh-pages): 負責將建置結果部署到 Github Pages。

## 建立第一個 Workflows

可以到 Github 專案中，選擇 Actions 頁籤，再點 `Set up a workflow yourself`，就會進到建立檔案的畫面，路徑為 `.github/workflows/main.yml`，
就算不從這邊，用建立檔案的方式也會到達同一個畫面，詳細可以去看 ["Configuring workflows"](https://help.github.com/en/actions/configuring-and-managing-workflows/configuring-and-managing-workflow-files-and-runs)。

首先先為這個 workflow 取個名字，我這邊就直接取叫做 `Github-Pages`。

```yaml
name: Github-Pages
```

接下來要來寫觸發的時機與分支，我這裡是將這個 workflow 的觸發時間設為 "在推送到此分支時 (`push`)"，並將 `develop` 設為要觸發的分支。

```yaml
...
on:
  push:
    branches:
      - develop
```

## 撰寫第一個 Job

接下來建立第一個 Job，我這裡將這個 Job 命名為 `deploy`，並再 `runs-on` 選項設定這個 Job 要跑在 `ubuntu-18.04`上。

```yaml
...
jobs:
  # Job 名稱，可自訂。
  deploy: 
    # 設定這個 Job 使用哪個 Runner
    runs-on: ubuntu-18.04
    
    # 要執行的步驟
    steps:
```

接下來寫將 Repo checkout 下來的步驟，這裡直接使用這個官方 actions [actions/checkout@v1](https://github.com/actions/checkout) 就可以完成了。

如果 Hugo 的 theme 是使用 submodules 的方式加入的，要記得加上 `submodules: true`，不然會缺少 Theme。

```yaml
...
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true
```

接下來加入 Hugo 的部分，這裡直接使用 [peaceiris/actions-hugo](https://github.com/peaceiris/actions-hugo)，而這個 actions 文件齊全也有在維護，算是很好的選項。

參數說明: 

 - `hugo-version`: 指定 Hugo 版本
 - `extended`: 使用 Hugo extended

Hugo 建置的部分，就依自己的需求寫了，基本上把平常手動建置的指令複製過來就好。

```yaml
...
    steps:
      ...

      - name: Setup Hugo # Step name
        uses: peaceiris/actions-hugo@v2 # Step 使用的 actions
        with: # 參數設定
          hugo-version: '0.57.2'
          extended: true

      - name: Build
        run: hugo --gc --minify --cleanDestinationDir  # 自己寫執行的步驟
```

再來就要寫將 Hugo 建置出來的靜態網站發佈到 Github Pages 的步驟，在這裡也使用同一作者個 actions [peaceiris/actions-gh-pages](https://github.com/peaceiris/actions-gh-pages)。

參數說明

 - `github_token`: 設定推送的 Token，直接設為 `${{ secrets.GITHUB_TOKEN }}` 就可以了，`GITHUB_TOKEN` 會由 actions 去處理。
 - `publish_dir`: 設定要發布的資料夾，預設為 `public`。
 - `publish_branch`: 設定目標分支，預設為 `gh-pages`。
 - `force_orphan`: 在目標分支，只保留最後一個提交，預設為 `false`。
 - `CNAME`: 設定 Github Pages Custom Domain 使用，其實也可以將 CNAME 新增在 hugo 的 static 就可以了。
 - `user_name` & `user_email`: 設定 Commit 的 name 與 email 資料。
 
 只有 `github_token` 是必填 (三個 token 擇一，詳洽 actions 文件)，都是選填。
 
```yaml
...
    steps:
      ...

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_branch: master
          force_orphan: true
```

到這裡就完成設定了，儲存檔案後在目標分支推送 Commit，就可以 Actions 看到在執行了，也可以在 Commit 上看到執行狀態。

## 完整 Workflows 設定檔

```yaml
name: Github-Pages

on:
  push:
    branches:
      - develop

jobs:
  deploy:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v1
        with:
          submodules: true

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.57.2'
          extended: true

      - name: Build
        run: hugo --gc --minify --cleanDestinationDir

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_branch: master
          force_orphan: true
```

## 結語

利用 Github Actions 將以往要手動執行建置的步驟改為自動去執行，雖然節省的時間不多，但可以減少手動執行時出錯的機會，也讓自己可以專注於寫作。

最後分享一下這個 Blog 的 [Workflows](https://github.com/puckwang/puckwang.github.io/blob/develop/.github/workflows/main.yml) 與 [Actions 的畫面](https://github.com/puckwang/puckwang.github.io/actions)。
