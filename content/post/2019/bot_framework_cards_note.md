---
title: "Microsoft Bot Framework 內建 Cards 種類"
description: "在開發聊天機器人時，現今給使用者的資料呈現方式除了一般的字串形式外，另外一種較常出現的就是以卡片形式呈現，運用卡片形式呈現不僅讓使用者更容易閱讀，也不會造成聊天頻道看起來很雜亂。"
date: 2019-01-07T12:29:01+08:00
draft: false
images:
    - "/images/og_image.png"
categories:
    - 筆記
    - bot framework
tags:
    - bot framework
    - bot
    - Card
---

在開發聊天機器人時，現今給使用者的資料呈現方式除了一般的字串形式外，另外一種較常出現的就是以卡片形式呈現，
運用卡片形式呈現不僅讓使用者更容易閱讀，也不會造成聊天頻道看起來很雜亂。

Bot Framework 以內建許多卡片供開發者使用，不僅支援多個平台，也很容易使用。

<!--more-->

## 如何發送卡片訊息？
可以直接呼叫 `IDialogContext` 中的 `MakeMessage()` 方法取得回覆訊息的物件，
然後將你要發送的卡片建立出來，再轉成 `Attachment` 的物件後新增進 Attachments 中就完成了，可參考下方 Code。
```csharp
private async Task AfterMessageReceivedAsync(IDialogContext context, IAwaitable<CardsDemoAction> result)
{
    // 建立回覆 Message
    var message = context.MakeMessage();
    
    // 建立 Hero Card
    var heroCard = new HeroCard()
    {
        Title = "Splatoon 2",
        Text = "Happy New Year.  by Puck",
        Images = new List<CardImage>()
        {
            new CardImage("https://imgur.com/hwylGzp.jpg")
        },
        Buttons = new List<CardAction>()
        {
            new CardAction()
            {
                Type = ActionTypes.OpenUrl,
                Title = "Open",
                Value = "https://imgur.com/hwylGzp.jpg"
            }
        }
    };
    
    // 設定 Layout Type
    message.AttachmentLayout = AttachmentLayoutTypes.Carousel;
    
    // 將 Hero Card 轉成 Attachment 後放入 List
    message.Attachments.Add(heroCard.ToAttachment());
    
    // 送出訊息
    await context.PostAsync(message);

    ...
}
```

## 卡片種類
* Hero Card: 包含標題、內文、大圖片及按鈕等。
* Receipt Card: 包含標題以及收據常用內容等。
* Thumbnail Card: 包含標題、子標題、內文、小圖片及按鈕等。
* Signin Card: 包含標題及按鈕等。
* Animation Card: 類似 Hero Card 但圖片為 Gif 等。
* Audio Card: 包含標題、子標題及音樂等。
* Video Card: 包含標題、子標題及影片(可為 YouTube 連結)等。
* Adaptive Card: 可自訂版面的卡片，官方有提供工具做設計，可直接匯入 JSON 檔或自己寫 Code 等。

## Hero Card

可以用在像要呈現如 Facebook 圖片貼文樣式的時候。

### 部分可用參數
* Title: 標題 (string)
* Text: 內文 (string)
* Images: 圖片 (List<[CardImage](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-api-reference?view=azure-bot-service-4.0#cardimage-object)>)
* Buttons: 按鈕 (List<[CardAction](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-api-reference?view=azure-bot-service-4.0#cardaction-object)>)

更多詳細內容可洽 [官方API](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-api-reference?view=azure-bot-service-4.0#herocard-object) 文件。

### 程式碼
```csharp
var heroCard = new HeroCard()
    {
        Title = "Splatoon 2",
        Text = "Happy New Year.  by Puck",
        Images = new List<CardImage>()
        {
            new CardImage("https://imgur.com/hwylGzp.jpg")
        },
        Buttons = new List<CardAction>()
        {
            new CardAction()
            {
                Type = ActionTypes.OpenUrl,
                Title = "Open",
                Value = "https://imgur.com/hwylGzp.jpg"
            }
        }
    };
```
### 結果
<img src="/images/2019/bot_framework_hero_card_demo.png" alt="Hero Card Demo Image" width="500">

## Receipt Card

是一個很方便的內建收據樣式，支援購買訊息可用於顯示購物日期或統一編號等購物資訊，不過可惜的一點是明明數量都輸入了卻不能自動算總計。

### 部分可用參數
* Title: 標題 (string)
* Total: 總計 (string)
* Items: 物品清單 (List<[ReceiptItem](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-api-reference?view=azure-bot-service-4.0#receiptitem-object)>)
* Facts: 購買訊息清單 (List<[Fact](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-api-reference?view=azure-bot-service-4.0#fact-object)>)
* Tax: 稅 (string)
* Vat: 消費稅 (string)
* Buttons: 按鈕 (List<[CardAction](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-api-reference?view=azure-bot-service-4.0#cardaction-object)>)

更多詳細內容可洽 [官方API](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-api-reference?view=azure-bot-service-4.0#receiptcard-object) 文件。

### 程式碼
```csharp
var receiptCard = new ReceiptCard()
    {
        Title = "購物清單",
        Total = "256.0",
        Tax = "1.0",
        Vat = "5.0",
        Items = new List<ReceiptItem>()
        {
            new ReceiptItem()
            {
                Title = "美式咖啡",
                Subtitle = "1杯",
                Price = "120.0",
                Quantity = "1",
                Image = new CardImage("https://via.placeholder.com/150/e65100/FFFFFF/?text=C")
            },
            new ReceiptItem()
            {
                Title = "全麥土司",
                Subtitle = "2條",
                Price = "65.0",
                Quantity = "2",
                Image = new CardImage("https://via.placeholder.com/150/fbc02d/FFFFFF/?text=T")
            }
        },
        Facts = new List<Fact>()
        {
            new Fact()
            {
                Key = "日期",
                Value = DateTime.Now.ToShortDateString()
            },
            new Fact()
            {
                Key = "編號",
                Value = "AB01234567"
            },
            new Fact()
            {
                Key = "購買人",
                Value = "Puck"
            }
        },
        Buttons = new List<CardAction>()
        {
            new CardAction()
            {
                Type = ActionTypes.ImBack,
                Title = "確認",
                Value = "OK"
            }
        }
    };
```
### 結果
<img src="/images/2019/bot_framework_receipt_card_demo.png" alt="Hero Card Demo Image" width="500">

## Thumbnail Card

當想使用清單式顯示但圖片又不想要 Hero Card 那麼大時，就可以使用這個來顯示。

### 部分可用參數
* Title: 標題 (string)
* Text: 內文 (string)
* Subtitle: 子標題 (string)
* Images: 圖片 (List<[CardImage](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-api-reference?view=azure-bot-service-4.0#cardimage-object)>)
* Buttons: 按鈕 (List<[CardAction](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-api-reference?view=azure-bot-service-4.0#cardaction-object)>)
* Tap: 點擊卡片所觸發的 Action ([CardAction](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-api-reference?view=azure-bot-service-4.0#cardaction-object))

更多詳細內容可洽 [官方API](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-api-reference?view=azure-bot-service-4.0#thumbnailcard-object) 文件。

### 程式碼
```csharp
var thumbnailCard = new ThumbnailCard()
    {
        Title = "Splatoon 2",
        Text = "Happy New Year",
        Subtitle = "by Puck",
        Tap = new CardAction()
        {
            Type = ActionTypes.OpenUrl,
            Title = "Tip",
            Value = "https://imgur.com/hwylGzp.jpg"
        },
        Images = new List<CardImage>()
        {
            new CardImage("https://imgur.com/hwylGzp.jpg")
        },
        Buttons = new List<CardAction>()
        {
            new CardAction()
            {
                Type = ActionTypes.ImBack,
                Title = "Like",
                Value = "Like"
            }
        }
    };
```
### 結果
<img src="/images/2019/bot_framework_thumbnail_card_demo.png" alt="Hero Card Demo Image" width="500">


## SignIn Card

如果只想顯示按鈕群，就可以使用這種卡片。

### 部分可用參數
* Text: 說明文字 (string)
* Buttons: 按鈕 (List<[CardAction](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-api-reference?view=azure-bot-service-4.0#cardaction-object)>)

更多詳細內容可洽 [官方API](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-api-reference?view=azure-bot-service-4.0#signincard-object) 文件。

### 程式碼
```csharp
var signinCard = new SigninCard()
    {
        Text = "請選擇操作: ",
        Buttons = new List<CardAction>()
        {
            new CardAction()
            {
                Type = ActionTypes.ImBack,
                Title = "新增",
                Value = "Add"
            },
            new CardAction()
            {
                Type = ActionTypes.ImBack,
                Title = "修改",
                Value = "Edit"
            },
            new CardAction()
            {
                Type = ActionTypes.ImBack,
                Title = "刪除",
                Value = "Delete"
            }
        }
    };
```
### 結果
<img src="/images/2019/bot_framework_sigin_card_demo.png" alt="Hero Card Demo Image" width="500">

## Animation Card

跟 Hero Card 類似，但如果圖片想要用 gif 就要用這種卡片。

### 部分可用參數
* Title: 標題 (string)
* Subtitle: 子標題 (string)
* Text: 說明文字 (string)
* Media: Gif 圖片 (List<[MediaUrl](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-api-reference?view=azure-bot-service-4.0#mediaurl-object)>)
* Buttons: 按鈕 (List<[CardAction](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-api-reference?view=azure-bot-service-4.0#cardaction-object)>)
* Autoloop: 自動循環 (boolean)
* Autostart: 自動播放 (boolean)

更多詳細內容可洽 [官方API](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-api-reference?view=azure-bot-service-4.0#animationcard-object) 文件。

### 程式碼
```csharp
var animationCard = new AnimationCard()
    {
        Title = "Thank You!",
        Subtitle = "Demo, by Puck",
        Autoloop = true,
        Autostart = true,
        Media = new List<MediaUrl>()
        {
            new MediaUrl("https://media.giphy.com/media/osjgQPWRx3cac/giphy.gif")
        }
    };
```
### 結果
<img src="/images/2019/bot_framework_animation_card_demo.gif" alt="Hero Card Demo Image" width="500">

## Audio Card

可播放音樂的卡片。

### 部分可用參數
* Title: 標題 (string)
* Subtitle: 子標題 (string)
* Text: 說明文字 (string)
* Media: 音樂檔 (List<[MediaUrl](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-api-reference?view=azure-bot-service-4.0#mediaurl-object)>)
* Buttons: 按鈕 (List<[CardAction](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-api-reference?view=azure-bot-service-4.0#cardaction-object)>)
* Autoloop: 自動循環 (boolean)
* Autostart: 自動播放 (boolean)

更多詳細內容可洽 [官方API](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-api-reference?view=azure-bot-service-4.0#audiocard-object) 文件。

### 程式碼
```csharp
var audioCard = new AudioCard()
    {
        Title = "卡農 - 帕赫貝爾",
        Subtitle = "Demo, by Puck",
        Media = new List<MediaUrl>()
        {
            new MediaUrl("https://upload.wikimedia.org/wikipedia/commons/6/62/Pachelbel%27s_Canon.ogg")
        },
        Autoloop = true,
        Autostart = true,
    };
```
### 結果
<img src="/images/2019/bot_framework_audio_card_demo.png" alt="Hero Card Demo Image" width="500">

## Video Card

可播放影片的卡片。

### 部分可用參數
* Title: 標題 (string)
* Subtitle: 子標題 (string)
* Text: 說明文字 (string)
* Media: 影片檔，可以是 Youtube 連結 (List<[MediaUrl](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-api-reference?view=azure-bot-service-4.0#mediaurl-object)>)
* Buttons: 按鈕 (List<[CardAction](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-api-reference?view=azure-bot-service-4.0#cardaction-object)>)
* Autoloop: 自動循環 (boolean)
* Autostart: 自動播放 (boolean)

更多詳細內容可洽 [官方API](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-api-reference?view=azure-bot-service-4.0#videocard-object) 文件。

### 程式碼
```csharp
var videoCard = new VideoCard()
    {
        Title = "Splatoon 2 Soundtrack",
        Media = new List<MediaUrl>()
        {
            new MediaUrl("https://youtu.be/rZaT-PpDAl8")
        },
        Subtitle = "Videmo Demo, by Puck",
        Autoloop = true,
        Autostart = true,
    };
```
### 結果
<img src="/images/2019/bot_framework_video_card_demo.png" alt="Hero Card Demo Image" width="500">

## Adaptive Card

如果內建的卡片都不符合需求，可以使用 Adaptive Card 來自己設計符合需求的卡片。使用上有兩種方法，分別為使用 JSON 匯入以及用 API 自己組合，
如果是固定的內容可以選用 JSON 就好了，而如果想要內容可以動態輸入，那就用 API 物件自己刻比較好。

另外，官方也有提供線上工具供開發者可以方便設計 [連結](https://adaptivecards.io/designer/)，想自己寫 Code 的也可以參考 [API](https://docs.microsoft.com/en-us/adaptive-cards/authoring-cards/card-schema)

```csharp
AdaptiveCard adaptiveCard = new AdaptiveCard()
    {
        Body = new List<AdaptiveElement>()
        {
            new AdaptiveTextBlock()
            {
                Text = title,
                Color = AdaptiveTextColor.Accent,
                Size = AdaptiveTextSize.Medium,
                Weight = AdaptiveTextWeight.Bolder
            },
            new AdaptiveTextBlock()
            {
                Text = dateTime.ToLocalTime().ToString(CultureInfo.InvariantCulture),
                Size = AdaptiveTextSize.Small,
                Spacing = AdaptiveSpacing.Small,
                Color = AdaptiveTextColor.Dark,
                HorizontalAlignment = AdaptiveHorizontalAlignment.Left
            },
            new AdaptiveTextBlock()
            {
                Text = body,
                Wrap = true
            }
        },
        Version = "1.0"
    };

    Attachment attachment = new Attachment()
    {
        ContentType = AdaptiveCard.ContentType,
        Content = adaptiveCard  // 如果是要用 JSON 的方式匯入，直接接再這邊就可以了，不用像上面一樣。
    };
```

<img src="/images/2019/bot_framework_adaptive_card_demo.png" alt="Hero Card Demo Image" width="500">

## 卡片的排版
當卡片有兩張以上在同一筆訊息時，就會需要排版，內建有水平跟垂直兩種排版方式。

### List (垂直)
```csharp
message.AttachmentLayout = AttachmentLayoutTypes.List;
```
<img src="/images/2019/bot_framework_list_demo.png" alt="Hero Card Demo Image" width="500">

### Carousel (水平)
```csharp
message.AttachmentLayout = AttachmentLayoutTypes.Carousel;
```
<img src="/images/2019/bot_framework_carousel_demo.png" alt="Hero Card Demo Image" width="500">