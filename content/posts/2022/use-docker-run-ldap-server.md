---
id: "5cfe6ac5519bd6450d414c0f1cc876241f5edf4e"
title: "使用 Docker 建立 LDAP 伺服器及管理介面"
description: "在開發專案時，很常會需要使用 LDAP 伺服器來處理帳號與密碼的驗證，此時就可以使用 Docker 來為開發環境架設一個 LDAP 伺服器。"
date: 2022-05-16T04:17:10+08:00
draft: false
featureImage: "/images/hero-main.jpg"
images:
    - "/images/og_image.png"
categories:
    - Docker
tags:
    - LDAP
---

在開發專案時，很常會需要使用 LDAP 伺服器來處理帳號與密碼的驗證，此時就可以使用 Docker 來為開發環境架設一個 LDAP 伺服器。

<!--more-->

本文會使用到 Docker，如果未安裝的人，請至 [Docker 官網](https://www.docker.com/get-started/) 下載安裝檔並安裝，這邊就不贅述了。

後續會使用 [osixia/openldap](https://github.com/osixia/docker-openldap) 這個 Docker Image 來建立 LDAP Server，另外還會使用
[osixia/phpldapadmin](https://github.com/osixia/docker-phpLDAPadmin) 這個 Docker Image 來建立 LDAP 管理介面 (phpLDAPadmin)。

## TL;DR

以下指令可建立一個 LDAP Server 及 phpLDAPadmin，所有設定為預設值。

```bash
docker run --name ldap-server \
        --hostname ldap-server \
		-p 389:389 -p 636:636 \
		--detach \
		osixia/openldap:latest
		
docker run --name ldap-admin \
    -p 6443:443 \
    --link ldap-server:ldap-host \
    --env PHPLDAPADMIN_LDAP_HOSTS=ldap-host \
    --detach \
    osixia/phpldapadmin:latest
```

**預設的連線資訊：**

- Base DN: `dc=example,dc=org`
- Port: 389 / 636 (TLS)
- 管理介面：<a href="https://localhost:6443/" target="_blank">https://localhost:6443/</a>
- Admin DN: `cn=admin,dc=example,dc=org`
- Admin Password: `admin`

## 建立 LDAP Server

執行以下指令就可以建立一個最基本的 LDAP Server：

```bash
docker run --name ldap-server \
        --hostname ldap-server \
		-p 389:389 -p 636:636 \
		--detach \
		osixia/openldap:latest
```

**參數說明**

- `--name <name>`：設定 Docker 容器的名稱，在上面的指令中是設為 `ldap-server`。
- `--hostname <name>`：設定 Docker 容器的主機名稱，這個為了下面要新增管理介面使用，在上面的指令中是設為 `ldap-server`。
- `-p <Local Port>:<Container Port>`：設定要綁定到本機的通訊埠，在上面的指令中是設為 389 與 636。
- `--env LDAP_TLS_VERIFY_CLIENT="try"`：設定 LDAP Server 的驗證客戶端憑證的類型為 `try`，這是為了開發時方便而設定，不需要再匯入客戶端憑證，目前沒有加到指令中，有需要的人請自行新增。

**預設的連線資訊：**

- Port: 389 / 636 (TLS)
- Admin DN: `cn=admin,dc=example,dc=org`
- Admin Password: `admin`

### 解決 LDAP Client 不信任憑證

因為這個 Image 預設是使用自簽憑證，使用 TLS 方式連線時會出現遠端憑證無效的錯誤。

如果您有使用自己的憑證也讓客戶端信任這個憑證，就不需做這個步驟。

1. 執行 `docker cp ldap-server:/container/service/slapd/assets/certs/ldap.crt .` 將自動產生的自簽憑證複製到本機的當前目錄中。
2. 安裝 ldap.crt 憑證到本機中，並設定為信任憑證 (或單獨設定信任 X509 初級規則)。

### 使用自訂的憑證

如果不想使用容器自動產生的自簽憑證，可以自己建立憑證或去申請免費憑證，再掛到 LDAP-Server 裡面即可。

- CA 憑證檔案路徑：`/container/service/slapd/assets/certs/ca.crt`
- LDAP 憑證檔案路徑：`/container/service/slapd/assets/certs/ldap.crt`
- LDAP 憑證 Key 檔案路徑：`/container/service/slapd/assets/certs/ldap.key`

您可以使用 `--volume /path/ssl:/container/service/slapd/assets/certs` 的方式映射本機目錄到容器中，以此來更改憑證。

### 其他自訂設定

可以透過環境變數來自訂一些 LDAP 的設定，可以透過 `--env <name>="<value>"` 的指令來設定環境變數。

- `LDAP_ORGANISATION`：設定 LDAP 的組織名稱，預設為 `Example Inc.`。
- `LDAP_DOMAIN`：設定 LDAP 的組織域名，預設為 `example.org`。
- `LDAP_ADMIN_PASSWORD`：設定 LDAP 的管理員密碼，預設為 `admin`。
- `LDAP_TLS_VERIFY_CLIENT`：設定 LDAP 是否驗證客戶端憑證，如想關掉可以設為 `try`，預設為 `demand`。
- `LDAP_TLS_CRT_FILENAME`：LDAP SSL 憑證檔案名稱，預設為 `ldap.crt`。
- `LDAP_TLS_KEY_FILENAME`：LDAP SSL 憑證 Key 檔案名稱，預設為 `ldap.key`。
- `LDAP_TLS_CA_CRT_FILENAME`：CA 憑證檔案名稱，預設為 `ca.crt`。

這邊只列出幾個常用的，更詳細可洽[原始文件](https://github.com/osixia/docker-openldap#defaultstartupyaml)。

### 故障排除

**無法使用 TLS (636) 連線，但使用 389 Port 時可正常連線**

1. 請檢查 636 Port 是否有通 (可使用 telnet 檢查)。
2. 預設會需求驗證客戶端憑證，如需關閉，請在執行指令時加上 `--env LDAP_TLS_VERIFY_CLIENT="try"` 來調整為可選驗證。

**使用 TLS (636) 連線時，出現遠端憑證無效的錯誤**

- 請檢查憑證是否已在受信任，也可參考 [解決 LDAP Client 不信任憑證](#解決-ldap-client-不信任憑證)。

## 建立 LDAP 管理介面

這裡所使用的管理介面是 phpLDAPadmin，如果有自己的工具，這裡就可以直接跳過了。

執行以下指令來建立 phpLDAPadmin：

```bash
docker run --name ldap-admin \
		-p 6443:443 \
		--link ldap-server:ldap-host \
        --env PHPLDAPADMIN_LDAP_HOSTS=ldap-host \
        --detach \
		osixia/phpldapadmin:latest
```

**參數說明**

- `--name <name>`：設定 Docker 容器的名稱，在上面的指令中是設為 `ldap-admin`。
- `--link <name>:<alias>`：連結到其他 Docker 容器，在上面的指令中是設為 `ldap-server:ldap-host` 來連結到本文中建立的 LDAP Server。
- `--env PHPLDAPADMIN_LDAP_HOSTS`：設定 LDAP Server 的主機名稱，在上面的指令中是設為 `ldap-host`。
- `-p <Local Port>:<Container Port>`：設定要綁定到本機的通訊埠。

執行完成後，就可以在 <a href="https://localhost:6443/" target="_blank">https://localhost:6443/</a> 的網址上看到管理介面，輸入 Admin 的 DN 及 Password 即可登入使用。

憑證無效的問題可以直接忽略，或是參考「[解決 LDAP Client 不信任憑證](#解決-ldap-client-不信任憑證)」的方式來解決。

{{< img src="/images/2022/phpLDAPadmin.webp" caption="phpLDAPadmin 登入畫面" >}} 

## 參考資料

- https://github.com/osixia/docker-openldap
- https://github.com/osixia/docker-phpLDAPadmin
