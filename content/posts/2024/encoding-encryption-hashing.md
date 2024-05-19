---
id: "ed0bf4eb4e0e23eb47c116d0a9fd65d8c26cd7c4"
title: "關於編碼 (Encoding)、加密 (Encryption)與雜湊 (Hashing)"
description: "很常見到會把「編碼 (Encoding)」、「加密 (Encryption)」以及「雜湊 (Hashing)」這三個搞混的人。身為一個軟體開發者，這些基本的資料處理技術的定義應該要很清楚，否則開發出來的軟體就很常遇到資安問題。"
date: 2024-05-19T12:14:49+08:00
draft: false
featureImage: "/images/hero-main.jpg"
images:
  - "/images/og_image.png"
categories:
  - Software Development
tags:
  - Encoding
  - Encryption
  - Hashing
---

常見有人會把「編碼 (Encoding)」、「加密 (Encryption)」以及「雜湊 (Hashing)」這三者混淆，或者是誤用。

作為軟體開發者，應該對這些基本的資料處理技術有簡單的理解，否則開發出來的軟體很容易出現資安問題。

<!--more-->

## 編碼 (Encoding)

編碼主要目的是轉換資料的格式，以便在不同的系統間傳輸或儲存。

編碼的過程是可逆的，且能以眾所皆知的方法編碼及解碼，也因此它不是用來保護資料的機密性。

常見的編碼方式：Base64、URL Encode

### 使用 Base64 編解碼範例

使用 `base64` 指令執行編碼

```sh
$ echo -n 'Puck' | base64
UHVjaw==
```

使用 `base64` 指令執行解碼

```sh
$ echo -n 'UHVjaw==' | base64 -d
Puck
```

- `-d`：代表進行解碼的動作。

也可以直接從檔案讀取資料並輸出到檔案。

```shell
# 編碼
$ base64 -i plaintext.txt -o encoded.txt
$ cat encoded.txt
UHVjawo=

# 解碼
$ base64 -d -i encoded.txt -o plaintext.txt
$ cat plaintext.txt
Puck
```

## 加密 (Encryption)

加密的主要目的是用來保護資料的機密性，確保不會未經授權的人知道資料的內容。

加密的過程跟編碼一樣是可逆的，但它多加了金鑰的部分來限制只有知道金鑰的人才可以解密。

現今加密技術有分為「對稱加密」及「非對稱加密」兩種加密方式：

- 「對稱加密」在加解密時是使用 **同一組金鑰**。
- 「非對稱加密」在加解密時是使用 **不同的金鑰**，分別稱為公鑰及私鑰。

常見的加密方式：

- 對稱加密：AES、DES
- 非對稱加密：RSA、ECS、DSA

常見應用：HTTPS、檔案加密

### 使用 AES 加解密範例

使用 `openssl` 指令執行加密

```sh
$ echo "Puck" | openssl enc -aes-128-cbc -a -salt -k KEY_STRING
U2FsdGVkX18b92lQm7P+t/xfRtNwEE2aeBOABJZTKgQ=
```

- `enc`：表示使用 OpenSSL 加密的功能。
- `-aes-128-cbc`：表示使用 AES 128 CBC 的加密方式。
- `-a`：表示使用 Base64 格式輸出。
- `-salt`：表示使用增加隨機值來增強安全性。
- `-k <password>`：設定加密的金鑰字串。

使用 `openssl` 指令執行加密

```sh
$ echo "U2FsdGVkX18b92lQm7P+t/xfRtNwEE2aeBOABJZTKgQ=" | openssl enc -d -aes-128-cbc -a -salt -k KEY_STRING
Puck
```

- `enc`：表示使用 OpenSSL 加密的功能。
- `-d`：表示執行解密的操作。
- `-aes-128-cbc`：表示使用 AES 128 CBC 的加密方式。
- `-a`：表示使用 Base64 格式輸出。
- `-salt`：表示使用增加隨機值來增強安全性。
- `-k <password>`：設定加密的金鑰字串。

也可以直接從檔案讀取資料並輸出到檔案。

```shell
# 加密
$ openssl enc -aes-128-cbc -a -salt -k KEY_STRING -in plaintext.txt -out encrypted.txt
$ cat encrypted.txt
U2FsdGVkX1+eWPgBmQrtGPlfvFBB6C81xkdsS0UiA9w=

# 解密
$ openssl enc -d -aes-128-cbc -a -salt -k KEY_STRING -in encrypted.txt -out plaintext.txt
$ cat plaintext.txt
Puck
```

- `in`：表示資料來源的檔案。
- `out`：表示資料輸出的檔案。

## 雜湊 (Hashing)

雜湊主要目的在於產生資料的唯一識別碼。未來如果要比對原始資料就只需要比對這一組識別碼即可，如果識別碼相同代表原始資料相同。

雜湊過程是不可逆的，也就是使用正規手段沒辦法透過雜湊結果反推原始資料。

常見的雜湊方法：MD5、SHA 系列、HMAC、Blake2
常見應用：竄改檢查、資料庫密碼儲存

### 使用 AES 加解密範例

使用 `openssl` 指令執行雜湊

```sh
$ echo "Puck" | openssl dgst -sha256
a2ad93d2c9fd1ef000883ecb91e94b4d184e12143e751246a61227fa1fd74440
```

- `dgst`：表示使用 OpenSSL 資料摘要的功能。
- `-sha256`：表示使用 SHA256 雜湊演算法。

也可以計算整個檔案的雜湊值。

```sh
$ openssl dgst -sha256 plaintext.txt
SHA256(plaintext.txt)= a2ad93d2c9fd1ef000883ecb91e94b4d184e12143e751246a61227fa1fd74440
```

## 三種技術的比較

{{< table >}}

| 比較項目   | 編碼 (Encoding) | 加密 (Encryption) | 雜湊 (Hashing) |
|--------|------------|-----------------|----------|
| 可逆性    | O          | O               | X        |
| 安全性    | 低          | 中               | 高        |
| 執行花費時間 | 低          | 高               | 中        |
| 輸出資料大小 | 中          | 大               | 小        |
| 應用範圍   | 不同系統間傳輸    | 防止未經授權的存取       | 不需原始資料的資料比對 |

{{< /table >}}

