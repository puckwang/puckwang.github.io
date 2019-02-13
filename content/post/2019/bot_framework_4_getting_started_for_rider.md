---
title: "Bot Framework 4 入門筆記 for MacOS and Jetbrains Rider"
description: "Microsoft bot framework 4 在去年九月就發布了，直到最近我才有機會去學他，雖然有碰過 v3，但聽說架構改很大，底層也改用 .Net Core 了，現在趕緊把它補上。雖然官方是推薦用 VS ，但身為一個 Jetbrains IDE 的愛好者，當然是使用它來當作開發工具，現在就讓我們從建立專案開始吧。 "
date: 2019-01-29T18:01:42+08:00
draft: false
images:
    - "/images/og_image.png"
categories:
    - 筆記
    - bot framework
tags:
    - bot framework v4
    - chatbot
    - rider
    - macOS
---

Microsoft bot framework 4 在去年九月就發布了，直到最近我才有機會去學他，雖然有碰過 v3，但聽說架構改很大，
底層也改用 .Net Core 了，現在趕緊把它補上。

雖然官方是推薦用 VS ，但身為一個 Jetbrains IDE 的愛好者，當然是使用它來當作開發工具，現在就讓我們從建立專案開始吧。 

<!--more-->

# 安裝專案 Template

新的東西連 VS 預設都不會有 Template，更不用說是 Rider了。

在 macOS 可以使用 `dotnet` 來下載 Template 的，而可以在[這邊](http://dotnetnew.azurewebsites.net/Search/bot)找到 v4 版本的 Template ，
而它們名稱分別為 `Bot v4 - Echo (bot-echo)` 與 `Bot v4 - Basic (bot-basic)`，點進詳細頁面就可以看到下載的指令。

執行指令把他們下載下來。
```shell
dotnet new --install "Ltwlf.BotBuilderV4.Echo"
dotnet new --install "Ltwlf.BotBuilderV4.Basic"
```

下載完後就可以在 `新增專案` 中看到囉！

<img src="/images/2019/bot_framework_4_rider_template.png" width="400">

# 新增專案

使用 Rider 新增專案後，如果 `Microsoft.Bot.Builder` 版本是 `4.0.6`，會發生抓不到 Package 的[問題](https://github.com/Microsoft/botbuilder-dotnet/issues/1002)，把它相關的 Package 都升到最新版就可以了。

執行專案後，可以下載 [BotFramework-Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases) 來模擬與 Bot 對話，
安裝完執行後，點擊 `Open bot` 選擇專案資料夾中的 `BotConfiguration.bot` 檔案，就可以開始與 Bot 對話了。

<img src="/images/2019/bot_framework_4_emulator.png" width="500">

# 專案架構

整個專案的基本架構是 .Net Core，而已開始建好專案就有幾個重要的檔案：

1. `Startup.cs`: 整個專案的入口點，裡面會載入一些 Bot 的設定及註冊這個 Bot。
    * 在 Line 33-34 的部分載入了 `appsettings.json` 以及環境設定檔 `appsettings.<環境名稱>.json`
    * 如果要更改 `Storage` 的儲存方式，在 Line 59 的 `IStorage dataStore = new MemoryStorage();`
    * 在 Line 81-85 使用 `AddSingleton` 方法註冊了 `conversationState` 及 `userState`，讓全域都可以拿到他們
    * 在 Line 87 的 `services.AddBot<Bot>(options => {...}` 註冊了 `Bot` 這個 Bot
    * 在 Line 93 的 `var botConfig = BotConfiguration.Load(botFilePath ?? @".\BotConfiguration.bot", secretKey);` 是意思是以 `BotConfiguration.bot` 來設定 Bot
2. `CounterState.cs`: 在這個專案中，是用來記錄第幾個 Turn 的 State，未來想要記錄一些使用者資料，也是要建一個跟這個類似的 Class。
    * 詳細怎麼做的可以看這篇[文章](https://blog.alantsai.net/posts/2018/10/bot-framework-v4-5-how-to-manage-state)。
3. `Bot.cs`: 實際處理訊息的地方，可以看到 `OnTurnAsync` 這個方法，就是實際再處理訊息的地方。
    * 在 Line 45 的地方，就使用 `conversationState` 將 `CounterState` 建立起來。

<img src="/images/2019/bot_framework_4_file_struct.png" width="350">

如果在架構上還有不請楚的可以看這篇[文章](https://blog.alantsai.net/posts/2018/10/bot-framework-v4-4-see-how-echobot-is-constructed)，講解得蠻詳細的   




