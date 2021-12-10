---
id: "3bbb758f0271c88ede7523a8571a550c064b610f"
title: ".Net Core 安裝 Entity Framework Core 並使用 Migration 來建立 Table"
description: "EntityFramework 是一個實現 ORM 的一個工具，而 EntityFrameworkCore 則是它的輕量版，簡單來說就是可以在專案中寫好需要的 Model 後，再用它產生對應的 Table，不用再自己執行 SQL。"
date: 2019-02-24T08:52:55+08:00
draft: false
images:
    - "/images/og_image.png"
categories:
    - 軟體開發
    
tags:
    - EntityFramework
    - ORM
    - DB Migrate
    - Database
    - .Net Core
    - Code First
---

EntityFramework 是一個實現 [ORM](https://zh.wikipedia.org/wiki/%E5%AF%B9%E8%B1%A1%E5%85%B3%E7%B3%BB%E6%98%A0%E5%B0%84) 的一個工具，而 EntityFrameworkCore 則是它的輕量版，簡單來說就是可以在專案中寫好需要的 Model 後，再用它產生對應的 Table，不用再自己執行 SQL。
<!--more-->

# 安裝

安裝 Microsoft.EntityFrameworkCore

**使用 .NET Core CLI 安裝**
```sh
dotnet add package Microsoft.EntityFrameworkCore
```

**使用 NuGet 安裝**

在 [NuGet 套件管理員](https://docs.microsoft.com/zh-tw/nuget/tools/package-manager-ui) 搜尋套件，並安裝。

這邊要注意版本，如果太高會出現 `Unable to cast object of type 'ConcreteTypeMapping' to type 'Microsoft.EntityFrameworkCore.Storage.RelationalTypeMapping'.` 的錯誤，此時請緩降版本直到正常執行。

# 建立 Model 與 Context
以下是建立一個 User Model 的範例與 Context 內的設定方式。
 
**User.cs**
```C#
public class User
{
    public string UserId { get; set; }
    public string Account { get; set; }
    public string Name { get; set; }
    public string Phone { get; set; }
    public string Address { get; set; }

    public DateTime CreatedAt { get; set; }
    public DateTime UpdatedAt { get; set; }
}
```

說明：

- Model 通常為單數，當 EF 去建立 Table 時，會轉成複數。
- 每個 Model 都要有一個欄位當作主鍵，預設會嘗試使用 `Id` 或 `<Name>Id` ，如果要客製化名稱可以使用 `[key]` 來直接指定，沒有這個欄位時會噴錯。
- 每個公開的欄位將會自動產生對應的資料庫欄位，如要排除可以為欄位加上 `[NotMapped]` 。


在 Model 中可以直接指定實體的關聯，也可以利用 `Data Annotations` 來設定許多資料行欄位的屬性，未來再另行打成一篇筆記，如有先需更詳細說明可洽[官方文件](https://docs.microsoft.com/zh-tw/ef/core/modeling/)

**ExampleContext.cs**
```C#
public class ExampleContext : DbContext
{
    public DbSet<User> Users { get; set; }

    private string _dbConnectString = "Data Source=127.0.0.1;Initial Catalog=mydb;persist security info=True;user id=sa;password=Ab123456";

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer(_dbConnectString);
    }
}
```
說明：

- 需手動為每個 Model 新增對應的 DbSet。
- 常見的 DB 連線類型： (要安裝對應的[Database Provider](https://docs.microsoft.com/zh-tw/ef/core/providers/#adding-a-database-provider-to-your-application))
    - `UseSqlServer` : SQL Server
    - `UseSqlite` : SQLite
    - `UseNpgsql` : PostgreSQL
    - `UseMySQL` : MySQL

## 建立 Migration 

使用 Migration 來管理資料庫，不僅讓資料庫的變化納入版控管理，安裝環境或部署時也不用執行 SQL，直接下指令就可以裝好對應版本的資料庫，實在是非常方便的 ~~黑魔法~~。在 PHP 著名框架 Laravel 也是用 Migration 來管理資料庫的變更的。

執行 `dotnet ef migrations add InitialCreate` 讓程式自動依照剛剛所寫的 Model 與 Context 來產生對應的 Migration 檔案。

執行完後就會看到在專案根目錄下產生出 Migrations 的資料夾。

## 建立 Table
執行 `dotnet ef database update` 來套用剛剛產生的 Migration 套用至資料庫。

執行完後就可以連到資料庫查看是否已有產生剛剛設定的 Table

# 基本查詢

### 取得所有資料
```C#
using (var context = new ExampleContext())
{
    var users = context.Users.ToList();
}
```

### 取得指定的一筆資料
```C#
using (var context = new ExampleContext())
{
    var user = context.Blogs
        .Single(b => b.UserId == 1);
}
```

### 加入 Where 條件
```C#
using (var context = new ExampleContext())
{
    var users = context.Users
        .Where(b => b.Name.Contains("Puck"))
        .ToList();
}
```

更多查詢方法，可以參考[官方文件](https://docs.microsoft.com/zh-tw/ef/core/querying/)
