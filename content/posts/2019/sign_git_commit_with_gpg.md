---
id: "f817530ba30930024b066e8771dd27f3da080f6f"
title: "利用 GPG 簽署 git commit"
description: "利用 GPG 簽署 git commit，讓 Github 顯示綠色小勾勾"
date: 2019-04-03T16:04:42+08:00
draft: false
images:
    - "/images/og_image.png"
categories:
    - 精選
    - 版本控制
    
tags:
    - Github
    - Gitlab
    - GPG
    - Verified
    - git
    - 身份識別
    
---

GPG 全名為 GNU Privacy Guard 也可以簡稱為 GnuPG，他是一個加密軟體，但也可以用來驗證身份。

今天就要來將我們 git commit 加上 GPG 簽署，讓它 Push 到 Github 等代管平台後，別人可以確定這份 Commit 是你提交的。

<!--more-->

## 安裝
### Homebrew 
如果系統是 MacOS 的人，且有裝 Homebrew 就可以直接用它來安裝，套件名稱為 [gnupg](https://formulae.brew.sh/formula/gnupg)
```
brew install gnupg
```

### Other
對於其他系統或者是沒有 Homebrew 的人，可以直接到 GnuPG [官方網站](https://www.gnupg.org/download/#binary)下載安裝檔。

## 產生 GPG keys

> 如果有不想打指令的人，可以跳過這段，改使用[軟體](#use_GPG_Suite)去產生 GPG Keys。

請打開終端機 (`terminal`) 執行以下指令去產生 Key，如果在這裡出現 `找不到指令` 的錯誤，請檢查是否有安裝成功。

```shell
gpg --full-generate-key
```

執行結果如下，過程中會詢問一些問題，請依照自己需求回答，當然也可以都使用預設設定。

首先會要選擇 Key 的類型，基本上用預設的 RSA & RSA 就可以了。
```
請選擇你要使用的金鑰種類:
   (1) RSA 和 RSA (預設)
   (2) DSA 和 Elgamal
   (3) DSA (僅能用於簽署)
   (4) RSA (僅能用於簽署)
你要選哪一個? 1
```

再來會要輸入 Key 的長度，github 是建議 4096 位元。
```
RSA 金鑰的長度可能介於 1024 位元和 4096 位元之間.
你想要用多大的金鑰尺寸? (2048) 4096
你所要求的金鑰尺寸是 4096 位元
```

接下來會詢問 Key 的有效期限，沒有特殊需求選 `金鑰不會過期` 就可以了。
```shell
請指定這把金鑰的有效期限是多久.
         0 = 金鑰不會過期
      <n>  = 金鑰在 n 天後會到期
      <n>w = 金鑰在 n 週後會到期
      <n>m = 金鑰在 n 月後會到期
      <n>y = 金鑰在 n 年後會到期
金鑰的有效期限是多久? (0) 0
金鑰完全不會過期
```

最後會跟你確認是否有問題。
```
以上正確嗎? (y/N) Y
```

確認完後會要你輸入基本資料及安全密碼。
```shell
GnuPG 需要建構使用者 ID 以識別你的金鑰.

真實姓名: <Name>
電子郵件地址: <Email>
註釋:
你選擇了這個使用者 ID:
    "<Name> <<Email>>"

變更姓名(N), 註釋(C), 電子郵件地址(E)或確定(O)/退出(Q)? O
```

到這邊資料都輸入完了，等它產生完就會看到如下的畫面。
```shell
我們需要產生大量的隨機位元組. 這個時候你可以多做一些事情
(像是敲打鍵盤, 移動滑鼠, 讀寫硬碟之類的)
這會讓隨機數字產生器有更多的機會獲得夠多的亂數.
我們需要產生大量的隨機位元組. 這個時候你可以多做一些事情
(像是敲打鍵盤, 移動滑鼠, 讀寫硬碟之類的)
這會讓隨機數字產生器有更多的機會獲得夠多的亂數.
gpg: /Users/user/.gnupg/trustdb.gpg: 建立了信任資料庫
gpg: 金鑰 xxxxxxxxxxxxxxx 已標記成徹底信任了
gpg: 目錄 '/Users/user/.gnupg/openpgp-revocs.d' 已建立
gpg: revocation certificate stored as '/Users/user/.gnupg/openpgp-revocs.d/xxxxxxxxxxxxxxx.rev'
公鑰和私鑰已建立及簽署.

pub   rsa4096 2019-04-02 [SC]
      0D69E11F12BDBA077B3726AB4E1F799AA4FF2279
uid                      "<Name> <<Email>>"
sub   rsa4096 2019-04-02 [E]
```
這個就是 GPG Keys 的 Fingerprint (指紋) `0D69E11F12BDBA077B3726AB4E1F799AA4FF2279` ，
而他與 `Long key ID` 及 `Short key ID` 的關係如下表:
```shell
Fingerprint: 0D69E11F12BDBA077B3726AB4E1F799AA4FF2279
Long key ID:                         4E1F799AA4FF2279
Short key ID:                                A4FF2279
```

可見兩個不同長度的 ID 只是截取 Fingerprint 的部分方便辨識而已，當有碰撞時就會變長了。

## 提交公鑰到 Github / Gitlab / Bitbucket

使用 `gpg --list-secret-keys` 去顯示出所有包含公要與私鑰的 GPG Keys。

將要使用的 Fingerprint (指紋) 複製起來，以下範例的 Fingerprint 為 `0D69E11F12BDBA077B3726AB4E1F799AA4FF2279` 。

```shell
$ gpg --list-secret-keys

gpg: 正在檢查信任資料庫
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: 深度: 0  有效:   1  已簽署:   0  信任: 0-, 0q, 0n, 0m, 0f, 1u
/Users/user/.gnupg/pubring.kbx
---------------------------------
sec   rsa4096 2019-04-02 [SC]
      0D69E11F12BDBA077B3726AB4E1F799AA4FF2279
uid           [ultimate] <Name> <<Email>>
ssb   rsa4096 2019-04-02 [E]
```

執行 `gpg --armor --export <Fingerprint>` ，承上範例的 Fingerprint 如下
```shell
gpg --armor --export 0D69E11F12BDBA077B3726AB4E1F799AA4FF2279
```

它會輸出公鑰請將它貼到你的 Github / Gitlab / Bitbucket 設定中的 GPG Key 中。

公鑰長得像這樣，但長度會依產生時設定而定：
```shell
-----BEGIN PGP PUBLIC KEY BLOCK-----
OVG48HWL9GV5P8HG1ZE51VUAJR47WF9H5UGRGX01ZB9I6LVRV55
B7SF1KY5H026LT7J9FL9G67I1EMPR4UIJAA6W3QPWRB3QGYRJ0DP
AMR2XJAI69NM5LBIZJR7GHY7JPI5O1OA07D008M7BDSTP3VMS0ND
FWB1BPVR277S0611F6NQJCSJLRBTW9CJ54WN31ZK3JR0N
=SSDQ

-----END PGP PUBLIC KEY BLOCK-----
```

## 提交包含簽署的 Commit
執行 `git config --global user.signingkey <Fingerprint>` 為 git 設定 Fingerprint。承之前範例的 Fingerprint 如下
```shell
# 僅限此倉庫
git config user.signingkey 0D69E11F12BDBA077B3726AB4E1F799AA4FF2279

# 全域設定
git config --global user.signingkey 0D69E11F12BDBA077B3726AB4E1F799AA4FF2279
```

接下來在每次 Commit 時加上 `-S` 就可以簽署了。
```shell
git commit -S -m "Add xxx.txt"
```

如果不想每次都加上 `-S` 可以做以下設定
```shell
# 僅限此倉庫
git config commit.gpgsign true

# 全域設定
git config --global commit.gpgsign true
```

在 Commit 過程中會要求要輸入 GPG 的安全密碼。

推上去遠端倉庫後，就可以在 Commit 紀錄中看到被標示為 `Verified ` 了

{{< img src="https://i.imgur.com/rvXVAk0.png" caption="Github commit 已驗證的截圖" >}} 

## GPG Suite {#use_GPG_Suite}
懶得打指令的也可以用 GPG Suite 來產生與管理 GPG Keys，不過圖形化介面的東西就不詳細介紹了。

GPG Suite 官網: [https://gpgtools.org/](https://gpgtools.org/)

{{< img src="https://gpgtools.org/images/screenshots/gka-key-list.1506349762.png" caption="官方提供的截圖" >}} 

## 故障排除
如果在下 `git commit` 後出現這個錯誤：
```
error: gpg failed to sign the data
fatal: failed to write commit object
```
請嘗試以下兩種解法

### 解法一

請執行 `echo "test" | gpg --clearsign` 檢查看看是 git 的問題還是 gpg 的問題。

如果可以執行正常，請檢查 git 設定是否正確，否之如果出現這個錯誤：
```
gpg: 簽署時失敗: Inappropriate ioctl for device
gpg: [stdin]: clear-sign failed: Inappropriate ioctl for device
```

請將 `export GPG_TTY=$(tty)` 加到你的 `.bashrc` 或 `.zshrc`，之後重啟終端機或執行重讀指令 `. ~/.zshrc` or `. ~/.bashrc`

再執行一次 `echo "test" | gpg --clearsign` 看看是成功執行。

### 解法二

安裝 pinentry-mac
```
brew install pinentry-mac
```

將以下設定新增至 `~/.gnupg/gpg.conf`
```
no-tty
```

以下設定新增至 `~/.gnupg/gpg-agent.conf`
```
pinentry-program /usr/local/bin/pinentry-mac
```

清除背景執行的 `gpg-agent`
```
killall gpg-agent
```

執行 `echo "test" | gpg --clearsign` 看看是成功執行。


