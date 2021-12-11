---
id: "efeef2fb0319d438830a5f9e532ce42824c39107"
title: ".Net core 應用程式使用 Kestrel 部署並搭配 IIS 來反向 Proxy"
description: "Kestrel 是 .Net core 隨附的跨平台網頁伺服器，可以自己獨立運行，也可以搭配其他網頁伺服器的反向 Proxy。"
date: 2019-04-11T20:00:48+08:00
draft: false
images:
    - "/images/og_image.png"
categories:
    - 伺服器維運
    - 軟體開發
    
tags:
    - deploy
    - kestrel
    - IIS
    - .Net Core
aliases:
    - /post/2019/dot_net_core_kestrel_iis_reverse_proxy/
---

Kestrel 是 .Net core 隨附的跨平台網頁伺服器，可以自己獨立運行，也可以搭配其他網頁伺服器的反向 Proxy。

<!--more-->

支援以下案例:

- HTTPS
- 用來啟用 WebSockets 的不透明升級
- Nginx 背後的高效能 Unix 通訊端

## 執行

執行的指令上會依照發布的方式有所不同。

以 `Framework Dependent` 發行，執行指令為：

```sh
dotnet <Project_Name>.dll
```

以 `Self-Contained` 發行，執行指令為：
```sh
./<Project_Name>.exe
```

可用參數有
```
--urls <uri>    # 指定執行的 Uri，例如： --urls http://localhost:1234
```

更詳細設定說明可以至[官方文件](https://docs.microsoft.com/zh-tw/aspnet/core/fundamentals/servers/kestrel?view=aspnetcore-2.1#kestrel-options)。


## 設定 反向 Proxy (Reverse proxy)

在這邊是使用 IIS 作為 Reverse proxy server，設定好 `URL Rewrite` 就可以了，而其他伺服器也差不多相同。

到站台頁面點選 `URL Rewrite` -> 點選 `新增規則` -> 點選 `反向 Proxy` 你就會看到新增的畫面。

過程中應該會叫你安裝 `Application Request Routing` 。


欄位說明:

- 輸入規則: 輸入目的地的位址。
- 輸出規則: 設定將內部伺服器轉成 Reverse proxy server 的位址。

{{< img src="https://i.imgur.com/Mex7gbX.png" caption="新增規則" width="500" >}} 


按下確定就完成設定，可以進到規則裡看更詳細的設定。

在「比對 URL」的區域中，可以選擇比對的`模式` (Patten)，預設是將全部 (.*) 都 Rewrite，可以在右邊 `測試模式` 中測試設定是否正確。

而下方「動作」區域中用來設定對符合項目的操作，我們目的是 `反向 Proxy` 所以動作類行為 `重寫` (Rewrite)，而「重寫 URL」可以設定 Rewrite 的格式，`{R:1}` 這類的參數可以在上方的 `測試模式` 中看到對應的值。

{{< img src="https://i.imgur.com/pIv3wSF.png" caption="規則詳細" width="700">}} 

到這邊設定就完成了，未來當有人連入 `http://blog.puckwang.com/xxxx` 都會被 Rewrite 至 `http://localhost:5000/xxxx` 。

如果要指定某些路徑才 Rewrite，可以修在上面提到的 `模式` (Patten)，例如下面這個設定就是將 `http://blog.puckwang.com/projects/xxxx` Rewrite 至 `http://localhost:5000/xxxx` ，而其他路徑則不影響。

{{< img src="https://i.imgur.com/88UMy2x.png" caption="附加路徑" width="700">}} 


當然我們也可以去 `測試模式` 中檢查。

可以看到它正確的將 projects 後面的都放入 `{R:1}` 。

{{< img src="https://i.imgur.com/enrCynU.png" caption="測試模式" width="500">}} 

