---
id: "2da7472f-3f45-4f48-f142-df146844cbe6"
title: "執行 npm install 時出現 UNABLE_TO_VERIFY_LEAF_SIGNATURE 錯誤"
description: "最近使用 Windows 的同事有一天突然遇到執行 `npm install` 時出現 `UNABLE_TO_VERIFY_LEAF_SIGNATURE` 的錯誤。"
date: 2024-05-11T14:17:10+08:00
draft: false
featureImage: "/images/hero-main.jpg"
images:
    - "/images/og_image.png"
categories:
    - 問題解決
tags:
    - NodeJs
    - npm
    - 憑證錯誤
---

最近使用 Windows OS 的同事有一天突然遇到執行 `npm install` 時出現 `UNABLE_TO_VERIFY_LEAF_SIGNATURE` 的錯誤。

<!--more-->

## 問題說明

執行 `npm install` 後出現錯誤訊息大概如下：

```sh
npm ERR! code UNABLE_TO_VERIFY_LEAF_SIGNATURE
npm ERR! errno UNABLE_TO_VERIFY_LEAF_SIGNATURE
npm ERR! request to https://registry.npmjs.org/@liff/core /-/@liff/core-1.2.2.tgz failed, reason: unable to verify the first certificate

npm ERR! A complete log of this run can be found in: C:\Users\puck_wang\AppData\Local\npm-cache\_logs\2024-04-25T06_26_38_657Z-debug-0.log
```

為什麼使用 npm 官方的套件來源會有憑證問題 (`UNABLE_TO_VERIFY_LEAF_SIGNATURE`) 呢？

把錯誤訊息中的 URL 用瀏覽器開起來後發現，原來是防毒軟體的「網路過濾功能」把憑證換掉了，才導致 npm 出現不信任憑證的問題。

{{< img src="/images/2024/ssl-filter-ca.webp" width="400" caption="ESET SSL Filter CA" >}} 

## 解決方式

我們可以把 CA 憑證匯出為 Base64 單一憑證檔，然後在用 NodeJS 的環境變數 `NODE_EXTRA_CA_CERTS` 設定信任這個憑證即可。

以 Chrome 為例，請連到網站後點選「網址列左側設定」 >「已建立安全連線」>「憑證有效」>「詳細資訊」>「匯出」，
存檔類型記得選擇「Base64-encoded-ASCII single certificate」，存擋路徑隨意。

匯出憑證後，再設定環境變數 `NODE_EXTRA_CA_CERTS` 為憑證的路徑就可以了，設定完記得重開 Terminal。

環境變數設定方式：

- Linux / MacOS 的使用者：在使用的 Terminal 的 `.bashrc` 或 `.zshrc` 加入 `export NODE_EXTRA_CA_CERTS=/path/to/trusted/CA.pem`
- Windows 的使用者：可以到電腦的進階設定的介面新增環境變數。

## 參考資料

- https://github.com/the-last-byte/ESET-NPM-Breakage-Fix?tab=readme-ov-file
