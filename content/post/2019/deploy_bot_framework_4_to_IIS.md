---
title: "Deploy Bot Framework 4 to IIS"
description: "在使用 Bot Framework 開發時，一定會遇到要部署至 IIS 的狀況，但官方文件就只有寫部署到 Azure 的方法，雖然方法與部署 .Net Core 差不多，但沒注意到細節踩雷也會很煩。內文為部署 Bot Framework 4 至 IIS 的方法。"
date: 2019-02-13T19:00:47+08:00
draft: false
images:
    - "/images/og_image.png"
categories:
    - bot framework
tags:
    - bot framework v4
    - chatbot
    - IIS
    - deploy
---

在使用 Bot Framework 開發時，一定會遇到要部署至 IIS 的狀況，但官方文件就只有寫部署到 Azure 的方法，
雖然部署 Bot Framework 4 的方法與部署 .Net Core 差不多，但沒注意到細節踩雷也會很煩。

<!--more-->

# 目錄
- [環境設置](#env-set)
    - [1. 安裝 Windows Server Hosting Bundle](#env-set-1)
    - [2. 建立新的 IIS 應用程式集區](#env-set-2)
    - [3. 建立新的 IIS 網站](#env-set-3)
- [部署](#deploy)
    - [1. 新增 Production 環境資料](#deploy-1)
    - [2. 發布 Bot 專案](#deploy-2)
    - [3. 複製至 IIS 網站根目錄](#deploy-3)
    - [4. 檢查 AspNetCoreModule 載入是否正常](#deploy-4)
- [驗證](#test)
    - [於瀏覽器中測試](#test-1)
    - [於 Bot Framework Emulator 4 中測試](#test-2)
- [Troubleshooting](#troubleshooting)

# 環境設置 {#env-set}
### 1. 安裝 Windows Server Hosting Bundle {#env-set-1}
IIS 需要另外安裝 `.NET Core Windows Server Hosting Bundle` ，請至官方下載點選擇適當的版本後，下載 `Runtime & Hosting Bundle` 並執行安裝。

連結：https://dotnet.microsoft.com/download/archives

<img src="/images/2019/bot_framework_4_ws_hosting_boundle_index.png" width="400">
<br>

### 2. 建立新的 IIS 應用程式集區 {#env-set-2}
建立一個新的 `應用程式集區` 其中設定的部分， .NET CLR 版本務必要改成 `沒有 Mamnged 程式碼` 。

<img src="/images/2019/bot_framework_4_IIS_app_pool_create.png" width="300">
<br>

### 3. 建立新的 IIS 網站 {#env-set-3}

新建一個網站或是在一個已存在網站中建立虛擬目錄，不管是哪種方式，都要將 `應用程式集區` 設定成在上一步所建立的那個應用程式集區。另外其他的設定如 `實體目錄` 、 `繫結` 就請自行判斷設定。

<img src="/images/2019/bot_framework_4_IIS_create_website.png" width="400">
<br>

# 部署 {#deploy}
### 1. 新增 Production 環境資料 {#deploy-1}
`Startup.cs` 會依照環境不同去讀 `BotConfiguration.bot` 中的設定，平常開發時是讀 `development` ，而發行後是讀 `Production` ，但預設是沒有資料的，所以要自己手動新增。

`BotConfiguration.bot` 範例如下，原本在 `services` 中只會有 `development` 一個設定，請依照自己的需求新增 `production` 版本的設定：
```
{
  "name": "PuckBot",
  "services": [
    {
      "type": "endpoint",
      "name": "development",
      "endpoint": "http://localhost:3978/api/messages",
      "appId": "",
      "appPassword": "",
      "id": "1"
    },
    {
      "type": "endpoint",
      "name": "production",
      "endpoint": "http://<Your_server_ip>/api/messages",
      "appId": "",
      "appPassword": "",
      "id": "1"
    }
  ],
  "padlock": "",
  "version": "2.0"
}
```

> 這裡可以直接用 Bot Framework Emulator 4 設定，也可以用 IDE 開起來改。

### 2. 發布 Bot 專案 {#deploy-2}
在 Visual Studio 上方選擇 `建置` -> `發行 xxx` ，接下來請選擇發布目標為 `資料夾` ，並確定好其他如 `.net core 版本` 等設定皆無誤，之後就可以執行發行的動作了。

<img src="/images/2019/bot_framework_4_vs_publish.png" width="400">
<br>

等它執行完後，預設會輸出到這個路徑 `./bin/Release/netcoreapp2.1/` (版本號會依照剛剛設定有所不同)。

### 3. 複製至 IIS 網站根目錄 {#deploy-3}
接下來請將 `./bin/Release/netcoreapp2.1/publish/` 全部的檔案複製到之前新增的 IIS 網站的根目錄中，記得重啟網站。

### 4. 檢查 AspNetCoreModule 載入是否正常 {#deploy-4}
到 IIS 網站中，選擇 `模組` ，之後檢查一下有沒有一個叫 `AspNetCoreModule` 的模組，如果沒有，請檢查 `.NET Core Windows Server Hosting Bundle` 是否正確安裝。

<img src="/images/2019/bot_framework_4_IIS_check_model.png" width="400">
<br>

# 驗證 {#test}
### 於瀏覽器中測試 {#test-1}
部署完後可於瀏覽器中查看網站是否功能正常，如果看到 `Your bot is ready!` 的字樣就可以了。

<img src="/images/2019/bot_framework_4_successful.png" width="400">
<br>

### 於 Bot Framework Emulator 4 中測試 {#test-1}
執行 Bot Framework Emulator 4 ，並開啟之前所設定的 `BotConfiguration.bot` 檔案，它就會讀到 Production 的設定了，並可以直接使用它來建立連線。

注意：在連線前記得要將 Bot Framework Emulator 4 設定好 [ngrok](https://ngrok.com/)，否則會沒辦法建立連線。

# Troubleshooting {#troubleshooting}

### IIS 4xx / 5xx
如果在網站看到的是 IIS 的錯誤，請檢查權限、根目錄、應用程式集區等設定有沒有設定正確。

### Runtime Error
如果是看到 `An error occurred while starting the application.` 這個錯誤，請至 `web.config` 中將 `stdoutLogEnabled` 設為 `true` 去看 log 檢查是為什麼錯。

> 預設 log 路徑為 `.\logs\stdout` ，如果 logs 不存在，它就不會自己連結，請務必要自己建立 logs 資料夾，他才會自己產生 log 檔。

---

**參考資料**

- https://stackify.com/how-to-deploy-asp-net-core-to-iis/
- https://poychang.github.io/troubleshoot-an-error-occurred-in-asp-dot-net-core-on-iis/

