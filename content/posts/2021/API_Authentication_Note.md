---
id: "de86cadfde35e863344d5a06419bd5f3bb4037e3"
title: "Web Api 驗證方式筆記"
description: "這是一篇紀錄關於 API 驗證的筆記，內容為我有碰過的 API 驗證方式。"
date: 2021-12-19T21:40:58+08:00
draft: false
featureImage: "/images/hero-main.jpg"
images:
    - "/images/og_image.png"
categories:
    - 軟體開發
    - 資訊安全
tags:
    - Api
---

非公開的 Web API 一定會進行驗證來確保呼叫者是允許使用的，在這篇文章中將紀錄我遇過的幾種驗證方式。

<!--more-->

## API Key

這是最簡單也最最方便的一種方式，它是由 API 提供者隨機產生一個字串給呼叫者，而呼叫者送出的 Request 必須包含這個隨機字串讓 API 提供者可以驗證其身份，而這個用於驗證的隨機字串就叫做 API Key。

雖然可以只使用 Key 就知道是哪個呼叫者呼叫的，但也有些 API 提供者也會要求也要多帶 Client Id 進行驗證。

### 放置位置

呼叫時要如何帶給 API 提供者，一般是由 API 提供者去指定一個位置，有些會放在 Header 或 Query String 中，也有少部分直接放在 Body 裡面。

- **放在 Header 中**：最常見的一種方式，有的會直接用標準的 Header 名稱 (如 `Authorization`)，或是自己指定一個名稱 (如 `X-Api-Key`)。
- **放在 Query String 中**：此種方式就是直接將 Key 帶在 Url 的 Query String 中，如 `?key=<API_KEY>`，但 Web Server 那邊 Log 可能要調整避免儲存到 Key。
- **放在 Body 中**：這一種我比較少看過，就是直接把 Key 包含在 Body 裡面。

相關實例：
- [Imgur](https://apidocs.imgur.com/#authorization-and-oauth)：使用公共資料相關的 API，只需將 ClientId 帶在 Header 的 `Authorization` 中即可。
- [Google Cloud APIs](https://cloud.google.com/docs/authentication/api-keys?hl=zh-tw#using_an_api_key)：部分允許使用 API Key 的 API 只需將 Key 帶在 Query String 中即可，如 `?key=<API_KEY>`。

### 在 Server 端只儲存 Hash 結果

API Key 就跟我們一般使用者密碼一樣是用於驗證身份，那我們同樣也可以把儲存密碼的方式拿來儲存 API Key。

你可以只儲存經過 Hash 的值，然後在驗證時用 Hash 的值來比對而非原始的值。如果需要提示呼叫者目前的 Key，則可以只儲存前幾個字元供識別即可。

當然這不是一個一定要做的事，因為就算 API Key 外洩也只能在同一個 API 提供者使用，不像密碼可能會出現多個網站共用的情性。

## API Token

API Token 與 API Key 相似，跟 Key 的差別在 Token 可能是經過編碼或加密產生的，格式可能是公開的格式 (JWT) 或是只有他們自己知道的格式。我們可以把它視為一個由 API 提供者所發的身份證，呼叫者呼叫 API 時或附上它，而 API 提供者可直接透過它知道呼叫者的身份及權限等資訊。

通常在 OAuth 通過驗證取得 access token 時，這個 token 就是以 JWT 的方式產生。

### JSON Web Tokens (JWT)

JWT 是由 Header、Payload 及 Signature 三個部分組成，彼此間使用 `.` 連結：

- **Header**：JWT 第一個部分，用來標示類型為 JWT 及紀錄簽章演算法的類型 (SHA256, RSA...)，是使用 Base64 編碼的 JSON 格式。

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

- **Payload**：JWT 第二個部分，用來存放資料的地方，可以 Claim 名稱可以用[標準](https://www.iana.org/assignments/jwt/jwt.xhtml)的也可以自訂，一樣也是使用 Base64 編碼的 JSON 格式。

```json
{
    "sub": "1234567890",
    "name": "Puck Wang",
    "admin": true
}
```

- **Signature**：JWT 第三個部分，是一個訊息的簽章，用於檢查前兩部分是否有被修改。

```shell
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret
)
```

將三個部分組合起來後就會像這樣

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IlB1Y2sgV2FuZyIsImFkbWluIjp0cnVlfQ.KcGicYB-emFBnUrXIViwDzCML6FB8W8QfD1lttRV0t4
```

## HMAC Signature (簽章驗證)

在這個方式中，API 提供者一樣會隨機產生一個 API Key 給呼叫者，但在呼叫 API 時並不是直接帶 API Key，而是利用 API Key 並使用 HMAC 演算法計算後產生的 Signature，在 API 提供者收到請求後也用同樣的方式去產生 Signature，如果兩者相同就算是通過驗證。

計算 Signature 時的訊息格式可以自己訂，驗證時用同樣格式就可以。而 HMAC 演算法部分就要看原生語言支援了，大部分應該都有 MD5、SHA1、SHA256、SHA384...，建議用 SHA256 以後的可能比較安全，因為 MD5 和 SHA-1 已經不安全了。

為了避免被彩虹表攻擊及防止重送攻擊 (replay attack)，通常會在訊息裡面包含時間的資訊，而 API 提供者驗證時，就會規定這個時間必須在 XX 時間以內，以此限制這個簽章有效時間。但這樣沒有完全防止重送攻擊，只是縮短可能的時間而已，所以除了加入時間資訊外，還會加入其他資訊來達到更完善的防護。

在 HMAC 演算法 方面，有些 API 提供者會支援多個演算法讓呼叫者選擇，而呼叫者就可以選擇一個比較方便產生的，並在後續紀錄於 Header 中讓 API 提供者知道是哪個演算法。

### 放置位置

比較常見的做法是放在 Header 中，但有些會直接放在 `Authorization`，有些會用自訂 Header 來放。

- 使用 `Authorization` 的範例

```
Authorization: HMAC-SHA256 ClientId=<ClientId>&Timestamp=<Timestamp>&Signature=<Signature>
```

- 使用自訂 Header 的範例

```
x-ClientId: <ClientId>
x-Timestamp: <Timestamp>
x-Signature: <Signature>
```

### 最簡單的一個範例

在這個範例中，會需要用到的資料：

- `ClientId`：識別呼叫者身份。
- `Timestamp`：發送 Request 當下時間搓 (Unix timestamp)，不可與伺服器時間相差 XX 時間。

首先將它們依照指定格式組成一個字串後，接下來拿 `API Key` 作為 Key 使用 HMAC-SHA256 算出 `Signature`。

```
HMACSHA256("<ClientId><Timestamp>", <ApiKey>) 
```

最後把 `ClientId`、`Signature` 及 `Timestamp` 三個值包含在 Request 裡面，那 API 提供者就可以用同樣的方式去算出 `Signature`，如果兩者相等且 `Timestamp` 也在指定時間以內，就代表通過驗證。

這個範例還是可能會有被重送攻擊的風險，接下來將在訊息中加入更多資訊以更完整的防止重送攻擊。

### 利用 Timestamp 及 Nonce 來防止重送攻擊

這個方法中，每個 Request 都必須包含一個隨機值 `Nonce`，而 API 提供者則會檢查在一段時間內不允出現重複的 `Nonce` 來達到防止重送攻擊。

在**呼叫者** 部分，需要準備的資料：

  - `ClientId`: 識別呼叫者身份。
  - `Timestamp`：發送 Request 當下時間搓 (Unix timestamp)，不可與伺服器時間相差 XX 時間。
  - `Nonce`：隨機值，可以直接使用 UUID 或時間 (ms)，指定時間內不可重複。

首先將它們依照指定格式組成字串後，接下來拿 `API Key` 作為 Key 使用 HMAC-SHA256 計算出 `Signature`，再把 `ClientId`、 `Nonce`、`Timestamp` 及 `Signature` 四個值包含在 Request 裡面，範例 Header 如下：

```
x-ClientId: <ClientId>
x-Timestamp: <Timestamp>
x-Nonce: <Nonce>
x-Signature: HMACSHA256("<ClientId><Timestamp><Nonce>", <ApiKey>) 
```

在 **API 提供者** 部分，除了需要檢查 Signature 是否相同及 Timestamp 是否過期外，還要多檢查在 XX 時間內 Nonce 有沒有重複，如果重複就要視為不通過驗證。

Nonce 及 Timestamp 的時間限制是相同的，且大部分為 5 ~ 15 分鐘，而 Nonce 長度不限，只要在時間內不容易碰撞即可。

這個方法雖然已大幅度降低重送攻擊的風險，比較麻煩的地方就是必須用 Memory Cache 或 Redis 去紀錄 Nonce 一定的時間來用於檢查有沒有重複，這就增加了一些執行成本。

### 加入更多資料驗證來防止重送攻擊

除了使用 Nonce 以外，你也可以將 Query String、Body 甚至 Header 也都加入訊息作為驗證依據，讓攻擊者就算拿到 Signature 也只能執行相同的操作不能拿去執行其他操作，藉此縮小風險。

在**呼叫者** 部分，需要準備的資料：

- `ClientId`: 識別呼叫者身份。
- `Timestamp`：發送 Request 當下時間搓 (Unix timestamp)，不可與伺服器時間相差 XX 時間。
- `Nonce`：隨機值，可以直接使用 UUID 或時間 (ms)，指定時間內不可重複。
- `QueryString`：Query String。
- `Body`：Body 經過 Hash 計算後的值。
- `URL`：Request URL。
- `Method`：Request Method。

首先將它們依照指定格式組成字串後，接下來拿 `API Key` 作為 Key 使用 HMAC-SHA256 計算出 `Signature`，再把 `ClientId`、 `Nonce`、`Timestamp` 及 `Signature` 帶在 Request 的 Header 裡面：

```
x-ClientId: <ClientId>
x-Timestamp: <Timestamp>
x-Nonce: <Nonce>
x-Signature: HMACSHA256("<ClientId><Timestamp><Nonce>", <ApiKey>) 
```

在 **API 提供者** 部分，只要檢查以下有任意一個不通過，就代表不通過驗證：

1. `Timestamp` 已超過指定時間範圍。
2. `Nonce` 已重複。
3. `Signature` 不相等。

### 相關實例

- [Amazon Signature Version 4](https://docs.aws.amazon.com/general/latest/gr/sigv4-create-canonical-request.html)
- [Line Pay Api](https://pay.line.me/jp/developers/apis/onlineApis?locale=zh_TW)

## OAuth

> 待補

## 參考資料

- https://jwt.io/introduction
- https://security.stackexchange.com/questions/248212/rest-api-is-protected-by-hmac-based-authentication-what-does-including-url-path
