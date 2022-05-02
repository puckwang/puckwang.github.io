---
id: "a26e7947125913b1eaa0dfe7896c8b92c90df79e"
title: "為 Yup 自訂驗證加上支援 Typescript"
description: "我們在使用 Yup 時，很常會使用 Yup.addMethod 來新增自訂的驗證方式，但如果專案是 Typescript 的卻沒有加上適當的 Type，會使編譯時出現錯誤。 "
date: 2022-05-02T14:11:09+08:00
draft: false
featureImage: "/images/hero-main.jpg"
images:
    - "/images/og_image.png"
categories:
    - 軟體開發
tags:
    - Front-end
    - 資料驗證
    - Yup
    - Typescript
---

我們在使用 Yup 時，很常會使用 `Yup.addMethod` 來新增自訂的驗證方式，但如果專案是 Typescript 的卻沒有加上適當的 Type，會使編譯時出現錯誤。 

這時候我們為加上正確的 Type，不只不會出現錯誤，使用上也更方便。

<!--more-->

## 為自訂驗證方法新增支援 Typescript

1. 首先我們先使用 `Yup.addMethod` 建立一個驗證方法，以下使用「驗證是否為有效 json」為範例。

**yup.ts**

```typescript
import * as Yup from 'yup';

Yup.addMethod(Yup.string, 'json', function (
    this: Yup.DateSchema,
    message?: string
) {
    return this.test('json', message || '這不是一個有效的 Json', function (this: TestContext, value: string) {
        return isEmpty(value) || (typeof value === 'string' && isValidJson(value));
    });
});
```

2. 接下來只要透過 `declare module 'Yup'` 來宣告這個驗證方法的 Type 即可。如果自訂驗證是新增在 `string()` 則介面部分就必須指定 `StringSchema`，其他型態的也是如此，且一次就可以宣告多個方法。

**yup.ts**

```typescript
declare module 'Yup' {
    interface StringSchema {
        /**
         * 確認字串是否是 Json 格式
         */
        json (message?: string): StringSchema;
    }
}
```

3. 最後要記得在剛剛設定 `declare` 的檔案，最後使用 `export` 去匯出 Yup，並在專案的頂層去載入它，以下範例用 `Index.tsx`。

**yup.ts**

```typescript
export default Yup;
```

**Index.tsx**

```tsx
import './lib/yup';

const Index: React.FC = () => {
    return <App />;
}
```

使用時就跟其他內建的 Yup 驗證方式一樣。

```typescript
import { string } from 'yup';

const schmea = object({
    json: string().json()
})
```

這樣為自訂驗證新增 Type 後，不僅使用上會有自動補全，也可利用 IDE 找到所有有用到這個驗證方法的地方。

## 完整 Code

**yup.ts**
```typescript
import * as Yup from 'yup';

declare module 'Yup' {
    interface StringSchema {
        /**
         * 確認字串是否是 Json 格式
         */
        json (message?: string): StringSchema;
    }
}

Yup.addMethod(Yup.string, 'json', function (
    this: Yup.DateSchema,
    message?: string
) {
    return this.test('json', message || '這不是一個有效的 Json', function (this: TestContext, value: string) {
        return isEmpty(value) || (typeof value === 'string' && isValidJson(value));
    });
});

export default Yup;
```

## 參考資料

- https://github.com/jquense/yup/issues/312
