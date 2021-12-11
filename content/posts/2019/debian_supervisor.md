---
id: "611308f248a37d8004cbc7c07cdb985a9cbda9bf"
title: "Debian 安裝與配置 Supervisor"
description: "利用 Supervisor 去管理程序，支援自動重啟與多程序"
date: 2019-05-08T11:44:21+08:00
draft: false
images:
    - "/images/og_image.png"
categories:
    - 伺服器維運
    
tags:
    - Supervisor
    - Process Manager
    - Debian
    - Services
    - Laravel
    - Queue
    - Linux
aliases:
    - /post/2019/debian_supervisor/
---

最近為了讓在伺服器上的 Laravel Queue Worker 更方便去管理，不用每次重啟伺服器都要上去重新執行，
所以要來安裝官方文件有提到到 `Supervisor` 這個服務。

Supervisor 是個用 Python 寫的一個程序管理服務，支援自動重啟、執行多程序 ...etc。

<!--more-->

## Install
```bash
apt-get install supervisor
```

## 建立設定檔
前往 `/etc/supervisor/conf.d` 建立監看程序的設定檔 `xxxxx.conf`

```conf
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /home/laravel/prod/artisan queue:work --sleep=3 --tries=3
autostart=true
autorestart=true
user=root
numprocs=8
redirect_stderr=true
stdout_logfile=/home/laravel/log/prod/worker.log
```
- `[program:xxxx]`: 告訴 supervisor 這是個要被管理的程式的設定檔，`:` 後面接著一個自己指定的名稱。
- `process_name`: 所執行程序的命名方式。
- `command`: 啟動程序的指令。
- `autostart`: 啟動 supervisor 時是否自動啟動，預設為 `true`。
- `autorestart`: 自動重啟程序，可為`false` 、 `unexpected` 或 `true` ，如果為 `unexpected` 
則會去檢查 exit code，是屬於 `exitcodes` 才會重啟，而如果為 `true` 則會無條件自動重啟。
- `user`: 執行程序的使用者。
- `numprocs`: 執行程序數，如果大於 1 ，則 `process_name` 需包含 `%(process_num)` ，預設為 1。
- `redirect_stderr`: 將 stderr 導向到 supervisor 的 stdout 文件中，預設為 `false`。
- `stdout_logfile`: 設定 stdout 要輸出的路徑 (需建立好資料夾，否則啟動會失敗)。

詳細設定可至[官方文件](http://supervisord.org/configuration.html)。


建立完後要重啟服務
```
service supervisor restart
```

檢查服務狀態
```
service supervisor status 
```

## supervisorctl

檢查狀態
```
supervisorctl status
```

重讀設定檔
```
supervisorctl reread  # 僅重讀
supervisorctl update  # 會重啟動應程序
```

啟動/停止/重啟 程序
```
supervisorctl start <name>
supervisorctl stop <name>
supervisorctl restart <name>
```
如果要一次重起所有程序，可以在名稱中使用萬用字元，如 `laravel-worker:*` 。



