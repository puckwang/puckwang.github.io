---
title: ".Net 使用 FluentMigrator 遷移資料庫"
description: "Fluent Migrator 是一個 .Net 的資料庫遷移 (Migration) 框架套件，提供 Code-First 的方式去管理資料庫結構，並可將其納入專案的版控中。"
date: 2020-06-30T17:59:32+08:00
draft: false
featureImage: "/images/hero-main.jpg"
images:
    - "/images/og_image.png"
categories:
    - 軟體開發
tags:
    - dotnet
    - .net-core
    - 套件
    - database
---

Fluent Migrator 是一個 `.Net` 的資料庫遷移 (Migration) 框架套件，其他如 `Laravel` 及 `Ruby on Rails` 也有類似的套件。遷移就像是資料庫的版本控制一樣，提供 `Code-First` 的方式去管理資料庫結構，並可將其納入專案的版控中。

<!--more-->

官網：https://fluentmigrator.github.io/

**一個建立 Users Table 的 Migration 範例:**

```cs
[Migration(1)]
public class CreateUserTable : Migration
{
    public override void Up()
    {
        Create.Table("Users")
            .WithColumn("Id").AsInt32().PrimaryKey().Identity()
            .WithColumn("Name").AsString(255);
    }

    public override void Down()
    {
        Delete.Table("Users");
    }
}
```

可能會有人想說「為什麼不直接用 Entity framework 就好？」，EF 提供完整的 ORM，而這個套件只提供「遷移」這部分，算是另一種選擇，如果有專案因為效能問題不想使用 EF 但又想要有類似機制，就可以使用這個套件。

### 使用 Migration 框架與使用 SQL 的比較

{{< table >}}
|     | 使用 Migration 框架 | 使用 SQL |
| --- | --- | --- |
| 支援不同的資料庫 | 套件有支援的都可以使用，寫一份就可以通用，且較不易出錯 | 要為每個資料庫個別去寫 SQL，需寫很多份，還要一個個去驗證執行是否正確 |
| 自動化執行 | 支援與 App 一同執行或使用 CLI 的方式 | 需自己處理 |
| 執行順序 | 可自訂每個 Migration 的版本，會照版本依序執行 | 無   |
| 紀錄  | 執行的紀錄將紀錄在 DB 某個 Table 中 | 無   |
| Rollback | 可 Rollback 到任意版本 | 需自己另外寫一份 SQL |
{{< table />}}

## 安裝 Fluent Migrator

這個套件有把執行跟寫分開，依照自己需求安裝就可以了，像是如果是要用 CLI 去執行的就可以只安裝寫 Migrations 的部分就好。

- 純寫 Migrations 需安裝:
    - `FluentMigrator`
- 有要執行 Migrations 需安裝:
    - `FluentMigrator.Runner`
    - 目標資料庫的 [ADO.NET](http://ADO.NET) Data Provider
        - SQLite: `Microsoft.Data.Sqlite`
        - PostgreSQL: `Npgsql`
        - ...

## 撰寫 Migration

建立一個名為 `20200616_CreateUserTable` 的檔案，在裡面寫一個 Class，他繼承 `Migration` 及實作 `Up()` 跟 `Down()` 兩個方法。

> 檔名跟 Class 名稱不限制，但建議檔名前面版本，便於檢視。

接下來為這個 Class 加上 `[Migration]` Attribute，如 `[Migration(<自訂版本>)]`，版本可自訂，用於決定執行順序。

在 `Up()` 中攥寫這個 Migration 要執行的動作，而 `Down()` 則是寫 `Up()` 反向的操作，是用來清除 `Up()` 所執行的效果。

```cs
[Migration(1)]
public class CreateUserTable: Migration
{
    public override void Up()
    {
        // ...
    }

    public override void Down()
    {
    	// ...
    }
}
```

這個套件提供使用 Fluent Interface 的方式去定義要執行的操作 (如建表、修改欄位...等)，幾乎可以不需要看文件就可以依照 IDE 提示寫出來。

假設我要建立帶有 id 及 name 欄位的 users 資料表，那我可以這樣寫：

- 在 `Up()` 中，使用 `Create` 去建立一個名為 users 的資料表，接下來使用 `WithColumn` 去定義兩個欄位。
- 在 `Down()` 中，使用 `Delete` 去刪除這個資料表。

```cs
[Migration(1)]
public class CreateUserTable: Migration
{
    public override void Up()
    {
        Create.Table("users")
	        .WithColumn("id").AsInt32().PrimaryKey().Identity()
	        .WithColumn("name").AsString();
    }

    public override void Down()
    {
    	Delete.Table("users");
    }
}
```

> 一個 Migration 沒有限制只能做一件事，像是我可以一個 Migrstion 就建很多個資料表。

### Fluent Interface 支援的操作

它命名的非常直覺，幾乎可以不用看文件直接照著打就可以完成。詳細也可參考 [Fluent Syntax Schema Overview](https://fluentmigrator.github.io/articles/technical/fluent-api.html)。

支援的操作及範例如下：

#### 建立

- 進入點: `Create`
- 用途: 用於 **建立** 資料表、欄位、Index、Unique 約束...
- 範例:

```cs
Create.Table("users")
    .WithColumn("id").AsInt32().PrimaryKey().Identity()
    .WithColumn("name").AsString();
```

#### 編輯

- 進入點: `Alter`
- 用途: 用於 **編輯** 資料表、欄位、Index、Unique 約束...
- 範例:

```cs
Alter.Table("users").AddColumn("account").AsString().Unique();
```

#### 改名

- 進入點: `Rename`
- 用途: 用於 **重新命名** 資料表、欄位、Index、Unique 約束...
- 範例:

```cs
// 將 users 改為 accounts
Rename.Table("users").To("accounts");

// 將 users.name 改為 users.fullname
Rename.Column("name").OnTable("users").To("fullname");
```

#### 刪除

- 進入點: `Delete`
- 用途: 用於 **刪除** 資料表、欄位，資料、Index、Unique 約束...
- 範例:

```cs
// 刪除資料表
Delete.Table("users");

// 刪除欄位
Delete
	.Column("column1")
	.Column("column2")
	.FromTable("users");
	
// 刪除資料
Delete
	.FromTable("users")
	.Row(new { name = "Admin" }); // 刪除 name = 'Admin' 的 rows
```
#### 新增資料
- 進入點: `Insert`
- 用途: 用於新增資料
- 範例:

```cs
Insert
	.IntoTable("users")
	.Row(new { name = "Admin" });
```

#### 更新資料
- 進入點: `Update`
- 用途: 用於更新資料
- 範例:

```cs
Update
	.Table("users")
	.Set(new { name = "Administrator" })
	.Where(new { name = "Admin" });
```

#### 執行 SQL
- 進入點: `Execute`
- 用途: 用於執行原生 SQL
- 範例:

```cs
Execute.Script("myscript.sql");
Execute.EmbeddedScript("UpdateLegacySP.sql");
Execute.Sql("DELETE TABLE users");
```

#### 判斷資料庫類型
- 進入點: `IfDatabase()`
- 用途: 當有些操作只想要在特定資料庫類型執行時，就可以用它來判斷。
	- 資料庫 [識別碼](https://fluentmigrator.github.io/articles/multi-db-support.html#supported-databases)。
- 範例:

```cs
// 只在 PostgreSQL 才建立 users
IfDatabase("Postgres")
	.Create.Table("users")
    .WithColumn("id").AsInt32().PrimaryKey().Identity()
    .WithColumn("name").AsString();
```

## 執行 Migration

執行的方式有提兩種供選擇，分別為 In-Process 與 CLI，如果沒有特殊需求，官方推薦是使用 In-Process 的方式執行 Migrations。

(未完待續)
