---
id: "d585ab632dbffd266343d697206343914f3550d1"
title: "改用 SQL Server 或其他 Database 來儲存 Bot State"
description: "Bot Framework 4 官方內建的 Storage 並沒有 SQL Server，僅支援開發用的 MemoryStorage 和 Azure 上的 CosmosDb 與 Blob。"
date: 2019-03-15T23:52:21+08:00
draft: false
images:
    - "/images/og_image.png"
categories:
    - 軟體開發
tags:
    - bot framework v4
    - chatbot
    - state
    - SQL Server
    - dotnet
aliases:
    - /post/2019/bot_framework_4_state_change_storage_db/
---

Bot Framework 4 官方內建的 Storage 並沒有 SQL Server，
僅支援開發用的 `MemoryStorage` 和 Azure 上的 `CosmosDb` 與 `Blob`。

因工作上需要，於是就想辦法把寫一個支援 SQL Server 的 `Storage`。

<!--more-->

我存取 DB 是使用 [EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore) 所以如果沒有裝，要先用 NuGet 裝一下。

如果不想用 `EntityFrameworkCore` 的人可以直接跳到 Source Code 的部分看，並自己改一下。

## 建立 DB 存取所需資料
首先先建立用來儲存 Bot State 的 Model，只有兩個欄位 `Key` 跟 `Data`。
{{< gist puckwang 24c4ef6689d406552edffc23dbaae2e0 "BotStateStorage.cs" >}}

接下來要建立 建立 EntityFramework 的 DBContext，記得將 `_dbConnectString` 改成自己的，如果不是用 SQL Server 的人 `UseSqlServer` 也要成對應的。
{{< gist puckwang 24c4ef6689d406552edffc23dbaae2e0 "BotContext.cs" >}}

寫到這邊 Entity Framework 必要的東西都有了，記得執行下面兩個指令去產生 DB 的 Table。
```bash
dotnet ef migrations add InitialCreate
dotnet ef database update
```

DB 的部分都完成了，接下來換 Storage 的部分。

## 替換 Storage

首先先建立 SqlStorage，這份是參考 [`CosmosDbStorage.cs`](https://github.com/Microsoft/botbuilder-dotnet/blob/master/libraries/Microsoft.Bot.Builder.Azure/CosmosDbStorage.cs)，因為它相依於 IStorage，所以改起來算好改，要改的東西也就 CRUD 而已，蠻簡單的。
{{< gist puckwang 24c4ef6689d406552edffc23dbaae2e0 "SqlStorage.cs" >}}

最後將 `Startup.cs` 中建立 Accessors 的地方改放入我們剛剛寫的 `SqlStorage` 

```
 IStorage dataStore = new SqlStorage();
 var accessors = new BotAccessors(dataStore);
``` 

執行 State 儲存後就會發現 Table 有資料了

![](https://i.imgur.com/DcRT2uH.png)

最後，對於這個改法有更好建議的人，也歡迎留言指教，讓它更好。
