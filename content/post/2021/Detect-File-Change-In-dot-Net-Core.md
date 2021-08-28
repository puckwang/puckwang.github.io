---
id: "4fb997c89db4db7c8c53e9a2dcd27ee5b985e6fd"
title: ".Net Core 偵測指定檔案或目錄變化時自動執行某些動作"
description: "本文將分享在 .Net Core 當指定檔案或目錄變化時，自動重新載入或執行某些動作。"
date: 2021-08-28T21:18:35+08:00
draft: false
featureImage: "/images/hero-main.jpg"
images:
    - "/images/og_image.png"
categories:
- 軟體開發
tags:
- dotnet
- .net-core
---

本文將分享在 .Net Core 當指定檔案或目錄變化時，自動重新載入或執行某些動作。

<!--more-->

本文包含 「`appsettings.json`」、「常見設定格式」及「其檔案或目錄」三種目標的偵測變化的方法，可以依照自己需求選擇。

雖然針對不同的目標有不同的方法，但都是利用 [IChangeToken](https://docs.microsoft.com/zh-tw/dotnet/api/microsoft.extensions.primitives.ichangetoken) 來做通知，所以首先要先了解 IChangeToken 是什麼。

## 關於 IChangeToken 介面

在 IChangeToken 介面中的 [`RegisterChangeCallback(Action<Object>, Object)`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.primitives.ichangetoken.registerchangecallback) 方法可以用來註冊一個 Callback，當有變化時就會自動呼叫這個 Callback，但註冊是一次性的，所以每次呼叫完的要重新註冊。

`RegisterChangeCallback` 第一個參數為要呼叫的 Callback，它必須是一個方法且含有一個 state 的參數，而第二個參數就是設定呼叫時要傳入的 state 參數值，不過如果沒有用到的話，設為 `null` 即可。

以下範例是從 `GetChangeToken()` 方法取得 `IChangeToken` 後，註冊一個`Callback` 的方法，且在每次變更時重新註冊一次，讓他每次變更都會執行：

```csharp
void Callback(object state)
{
    GetChangeToken().RegisterChangeCallback(Callback, null);

    // Changed ...
}

GetChangeToken().RegisterChangeCallback(Callback, null);
```

另一種不需要重新註冊的寫法，就是利用靜態類別 `ChangeToken` 所提供的靜態方法來註冊 Callback，說明如下：
 
- [`OnChange<TState>(Func<IChangeToken>, Action)`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.primitives.changetoken.onchange)：
  - 第一個參數 `Func<IChangeToken>` 為取得 `IChangeToken` 的方法
  - 第二個參數 `Action` 為欲呼叫的 Callback。
- [`OnChange<TState>(Func<IChangeToken>, Action<TState>, TState)`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.primitives.changetoken.onchange)：功能與 `ChangeToken.OnChange<TState>(Func<IChangeToken>, Action)` 相同，只是差別在可以指定在呼叫時傳入一個 `TState`。

以下範例是上一個範例的簡化版本

```csharp
ChangeToken.OnChange(GetChangeToken, () =>
{
    // Changed ...
});
```

## 偵測 `appsettings.json` 變化

如果你只是要在 `appsettings.json` 有變化時重新載入，那可以不用做其他設定，因為現在預設都已經是會自動重新載入了。

但如果你想要另外執行一些動作，就可以透過 DI 取得 `IConfiguration` 後呼叫 `GetReloadToken()` 取得 `IChangeToken`。

以下範例是每次 `appsettings.json` 變更時發送 Log：

```csharp
ChangeToken.OnChange(configuration.GetReloadToken, () =>
{
    logger.LogInformation("Configuration changed");
});
```

## 使用 `IConfiguration` 來自動載入或偵測變化

如果你的檔案是 Json、Xml 或 Ini 格式，且要做的事情就只是 **重新載入** 而已，可以直接使用 [`ConfigurationBuilder`](https://docs.microsoft.com/zh-tw/dotnet/api/microsoft.extensions.configuration.configurationbuilder) 來建立 `IConfiguration`，就可以如同原本的 `appsettings.json` 一樣自動載入了，
且使用方式也一樣。

使用流程如下：

1. 設定 File Provider 或是 Base Path 其中一個。
    - File Provider 可以直接從 `IHostEnvironment.ContentRootFileProvider` 取得或是自己建立 (`new PhysicalFileProvider(<root_directory>)`)。
    - Base Path 可以從 `Directory.GetCurrentDirectory()` 取得或是自己指定。
2. 使用 `AddXXXFile()` 來加入要載入的檔案，且將 `reloadOnChange` 參數設為 `true`。
    - Json：`AddJsonFile()`
    - Xml：`AddXmlFile()`
    - Ini：`AddIniFile()`
3. 呼叫 `Build()` 建立並取得 `IConfiguration`。

範例如下：

```csharp
var configuration = new ConfigurationBuilder()
    .SetFileProvider(env.ContentRootFileProvider)
//  .SetBasePath(Directory.GetCurrentDirectory())
//  .AddIniFile("config.ini", false, reloadOnChange: true)
//  .AddXmlFile("config.xml", false, reloadOnChange: true)
    .AddJsonFile("config.json", false, reloadOnChange: true)
    .Build();
```

如果你也想要另外執行一些動作，那方法就跟 [偵測 `appsettings.json` 變化](#偵測-appsettings-json-變化) 裡面提到的一樣，範例如下：

```csharp
var configuration = new ConfigurationBuilder()
    .SetFileProvider(env.ContentRootFileProvider)
    .AddJsonFile("config.json", false, reloadOnChange: true)
    .Build();
    
ChangeToken.OnChange(configuration.GetReloadToken, () =>
{
    logger.LogInformation("Configuration changed");
});
```

## 偵測指定檔案或目錄變化

這一個方法是最通用的，可以針對某個檔案、某個目錄或是指定副檔名的檔案來偵測變化。

首先要先取得 File Provider，可以直接從 `IHostEnvironment.ContentRootFileProvider` 取得或是自己建立 (`new PhysicalFileProvider(<root_directory>)`)。

再來就是利用 File Provider 的 `Watch()` 方法來偵測變化，傳入的唯一參數是一個過濾的字串，可以直接指定名稱或使用 `*` 或 `**` 來過濾。

以下範例是偵測專案根目錄的 `Configs` 資料夾內有變化時發送 Log：

```csharp
var fileProvider = env.ContentRootFileProvider;

ChangeToken.OnChange(() => fileProvider.Watch("Configs/**"), () =>
{
    logger.LogInformation("Target changed");
});
```

## 參考資料

- Configuration in ASP.NET Core
  - https://docs.microsoft.com/zh-tw/aspnet/core/fundamentals/configuration
- Detect changes with change tokens in ASP.NET Core
  - https://docs.microsoft.com/en-us/aspnet/core/fundamentals/change-tokens
