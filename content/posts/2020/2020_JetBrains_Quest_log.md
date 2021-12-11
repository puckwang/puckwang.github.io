---
id: "f5fe3bd8df3bcd7bd56a54eb486ec4171f9086ec"
title: "2020 JetBrains Quest 官方解謎全記錄"
description: "這是一篇解謎的紀錄文章，對這種解謎有興趣的可以進來看看"
date: 2020-03-09T23:23:00+08:00
draft: false
images:
    - "/images/og_image.png"
categories:
    - 活動紀錄
    
tags:
    - 彩蛋
    - JetBrains
    - Quest
aliases:
    - /post/2020/git-submodule-vs-subtree/
---

就在台灣時間 2020/3/9 20:04，JetBrains 的官方 Twitter 發了一篇推文如下

{{< tweet jetbrains 1236986174075482113 >}}

而我看到這個，就馬上燃起我的解謎魂了！也就打一篇解題的過程來紀錄一下。

接下來我將一步一步的紀錄我的解題步驟，不過當看到這篇文章時應該已經過了活動期限囉！畢竟不能破人家的梗！

<!--more-->

## 第一個問題

我們在 JetBrains 粉絲團的貼文中得知了一串神秘的字串，這是這次活動的第一個問題的入口。

```
48 61 76 65 20 79 6f 75 20 73 65 65 6e 20 74 68 65 20 73 6f 75 72 63 65 20 63 6f 64 65 20 6f 66 20 74 68 65 20 4a 65 74 42 72 61 69 6e 73 20 77 65 62 73 69 74 65 3f
```

### 解碼

看起來他是一個編碼後的字串，而且每個數字都是十六進制，有可能是十六進制的 ASCII。

在這邊就直接使用 Browser 的 Console 用 js 去將這個字串以空白分割後轉成文字吧。

```js
let source = '48 61 76 65 20 79 6f 75 20 73 65 65 6e 20 74 68 65 20 73 6f 75 72 63 65 20 63 6f 64 65 20 6f 66 20 74 68 65 20 4a 65 74 42 72 61 69 6e 73 20 77 65 62 73 69 74 65 3f';

source.split(' ').map(i => parseInt(i, 16)).map(i => String.fromCharCode(i)).join('')
// "Have you seen the source code of the JetBrains website?"
```

果然，他是一段文字轉乘 ASCII 再轉成十六進制。

我們循這個個線索到 JetBrains 官網的 source code，找到了第二個關的提示。

{{< img src="/images/2020/jetBrains_task_2.png" caption="JetBrains 官網的 source code" width="500" >}} 

```
<!--
      O
{o)xxx|===============-
      O

Welcome to the JetBrains Quest.

What awaits ahead is a series of challenges. Each one will require a little initiative, a little thinking, and a whole lot of JetBrains to get to the end. Cheating is allowed and in some places encouraged. You have until the 15th of March at 12:00 CET to finish the 3 quests.
Getting to the end of each quest will earn you a reward.
Let the quest commence!

JetBrains has a lot of products, but there is one that looks like a joke on our Products page, you should start there...
It’s dangerous to go alone take this key: Good luck! == Jrrg#oxfn$

                 O
-===============|xxx(o}
                 O
-->
```

內文一開始說明了活動時間及結尾有獎勵，後段有給出了往下一關的提示及一個 Key: `Good luck! == Jrrg#oxfn$`。

從後半段提示中得知，我們要先去 JetBrains 產品頁面中一個看起來是開玩笑的產品！

### 難道是新產品 JK?

JK? 他們有這個產品嗎 XD? 這也太容易了吧，平時就有在關注的我，一進來就看到一個陌生的 Logo。

{{< img src="/images/2020/jetBrains_websit_products.png" caption="JetBrains 官網的產品頁面" width="500" >}} 

接續上一段我們找到了 JK 這個假產品，點擊它後就出現了往下一題的提示訊息！

{{< img src="/images/2020/jetBrains_quest_jk.png" caption="JK?" width="500" >}} 

在此得到了一個連結 `https://jb.gg/###`，其中 `###` 為 500~5000 質數的數量。

訊息最後出現了 `Good luck!`，不知道是有意還無意，我們先記住它，看看後面會不會用到。

經過程式計算後，500~5000 共有 669-95 = 574 個質數，也就是 `###` 為 574，而完整連結為 `https://jb.gg/574`。

### 神秘的編號 MPS-31816

開啟連結後，我們被導向到了 PyCharm Help 的一個頁面，也拿到了往下一個挑戰的提示！

{{< img src="/images/2020/jetBrains_pycharm_help.png" caption="JetBrains 官網的產品頁面" width="500" >}}

圖中有一個字卡，裡面有 YouTrack 的 Logo 及一個編號 `MPS-31816`。

看起來是要叫我們到他們的 YouTrack 找 `MPS-31816` 這個編號的 issue。


找到了編號為 `MPS-31816`的 issue：https://youtrack.jetbrains.com/issue/MPS-31816

{{< img src="/images/2020/jetBrains_MPS-31816.png" caption="MPS-31816" width="500" >}}

從文中我們得到了一個密文，是跟前面給的 key 有關。

```
Qlfh$#Li#|rx#duh#uhdglqj#wklv#|rx#pxvw#kdyh#zrunhg#rxw#krz#wr#ghfu|sw#lw1#Wklv#lv#rxu#lvvxh#wudfnhu#ghvljqhg#iru#djloh#whdpv1#Lw#lv#iuhh#iru#xs#wr#6#xvhuv#lq#Forxg#dqg#iru#43#xvhuv#lq#Vwdqgdorqh/#vr#li#|rx#zdqw#wr#jlyh#lw#d#jr#lq#|rxu#whdp#wkhq#zh#wrwdoo|#uhfrpphqg#lw1#|rx#kdyh#ilqlvkhg#wkh#iluvw#Txhvw/#qrz#lw“v#wlph#wr#uhghhp#|rxu#iluvw#sul}h1#Wkh#frgh#iru#wkh#iluvw#txhvw#lv#‟WkhGulyhWrGhyhors†1#Jr#wr#wkh#Txhvw#Sdjh#dqg#xvh#wkh#frgh#wr#fodlp#|rxu#sul}h1#kwwsv=22zzz1mhweudlqv1frp2surpr2txhvw2
```

### 再次解碼

我們回頭看之前的 key，`Good luck! == Jrrg#oxfn$`。

看起來後面的 `Jrrg#oxfn$` 是 `Good luck!` 的密文，但用什麼加密呢？

先嘗試判斷是不是用古典密碼學的凱撒加密法。

經過人工計算後，看起來是 `轉成 ASCII 後加三`，我們嘗試用程式驗證一下。
```js
'Good luck!'.split('').map(i => i.charCodeAt()).map(i => i + 3).map(i => String.fromCharCode(i)).join('')
// "Jrrg#oxfn$"
```

果不其然，現在我們將這個規則套用至剛剛的密文，做解密的動作。

```js
let ciphertext = 'Qlfh$#Li#|rx#duh#uhdglqj#wklv#|rx#pxvw#kdyh#zrunhg#rxw#krz#wr#ghfu|sw#lw1#Wklv#lv#rxu#lvvxh#wudfnhu#ghvljqhg#iru#djloh#whdpv1#Lw#lv#iuhh#iru#xs#wr#6#xvhuv#lq#Forxg#dqg#iru#43#xvhuv#lq#Vwdqgdorqh/#vr#li#|rx#zdqw#wr#jlyh#lw#d#jr#lq#|rxu#whdp#wkhq#zh#wrwdoo|#uhfrpphqg#lw1#|rx#kdyh#ilqlvkhg#wkh#iluvw#Txhvw/#qrz#lw“v#wlph#wr#uhghhp#|rxu#iluvw#sul}h1#Wkh#frgh#iru#wkh#iluvw#txhvw#lv#‟WkhGulyhWrGhyhors†1#Jr#wr#wkh#Txhvw#Sdjh#dqg#xvh#wkh#frgh#wr#fodlp#|rxu#sul}h1#kwwsv=22zzz1mhweudlqv1frp2surpr2txhvw2';

ciphertext.split('').map(i => i.charCodeAt()).map(i => i - 3).map(i => String.fromCharCode(i)).join('');
// "Nice! If you are reading this you must have worked out how to decrypt it. This is our issue tracker designed for agile teams. It is free for up to 3 users in Cloud and for 10 users in Standalone, so if you want to give it a go in your team then we totally recommend it. you have finished the first Quest, now it’s time to redeem your first prize. The code for the first quest is “TheDriveToDevelop”. Go to the Quest Page and use the code to claim your prize. https://www.jetbrains.com/promo/quest/"
```

中間還插入工商 XD

第一個問題就到這裡結束了，獎品是 所有產品 3 個月序號，蠻不錯的。

最後，在寄來的 Mail 中得知下一個問題是在 2020/03/11 19:00 (`1583924400`) 公布。

原文如下：
```
Excited? You should be. The next quest will be at 1583924400 on our social media.

never odd or even,

The JetBrains Quest team
```

## 第二個問題


到了預告的時間，官方如期的貼出了第二個問題的提示，看起來是幾個前後顛倒的文字。

{{< tweet jetbrains 1237694815283879943 >}}

### 前後顛倒的文字

我們將他反轉過來後，得到結果如下：

```
The product domain-specific languages are specified, is where the next of quest hides. The case where Dutch tax is processed, is a place you should show interest. Look hard as when text is white, sometimes it is out of sight. Cmd/Ctrl+A helps.
```

看起來又是要找一個他們的產品，且是在特定領域用的，還有提示位置在跟 "荷蘭稅務" 有關的地方，且要反白才看得到，真好心 XD


剛好我知道他們有一個產品客群就是特定領域，它叫做 MPS，進到產品頁面後在下方 `Who is using MPS?` 找到了跟 "荷蘭稅務" 有關的文章了！

{{< img src="/images/2020/jetBrains_MPS_bottom.png" caption="MPS" width="500" >}}

點擊 `Read MPS case study` 開啟 PDF，反白後就看到一個空白區域居然有字，不過貼到其他地方才看得到。
{{< img src="/images/2020/jetBrains_MPS_case_study.png" caption="MPS Case Study PDF" width="500" >}}

內文如下，看起來也是去找他們年度報告有關於 `18,650` 的地方。
```
This is our 20th year as a company, we have shared numbers in our JetBrains Annual report, sharing the section with 18,650 numbers will progress your quest.
```
### JetBrains Annual report

進到了 [JetBrains 年度報告後](https://www.jetbrains.com/company/annualreport/2019/)，開始尋找有關 18,650 的地方。

這裡就比較難了，將它的所有數字加起來剛好就是 18,650，再根據上一個提示說要分享它。

{{< img src="/images/2020/jetBrains_year_annual.png" caption="7th Annual Hackathon" width="500" >}}

接下來就可以在分享的訊息中看到下一關的提示了。

{{< img src="/images/2020/jetBrains_twitter_quest_3.png" caption="Sharing Message" width="500" >}}

提示文如下，依照提示進入連結找尋 `lego brainstorms project `

```
I have found the JetBrains Quest! Sometimes you just need to look closely at the Haskell language, Hello,World! in the hackathon lego brainstorms project 
https://blog.jetbrains.com/blog/2019/11/22/jetbrains-7th-annual-hackathon/ 
```

很簡單就可以在內文中看到一個標題是 `Lego BrainStorms`，那邊有一張圖，圖中剛好有 `Hello,World` 的字樣，而那個區塊上方還有一段迷樣文字。

{{< img src="/images/2020/jetBrains_lego_brainstorms.png" caption="Lego Brainstorms 圖片" width="500" >}}

官方也很好心的把字放在圖片的 `alt` 中，不用一個一個打出來，直接複製就好了。

### 人工英文解碼

```
d1D j00 kN0w J378r41n2 12 4lW4Y2 H1R1N9? ch3CK 0u7 73h K4r33r2 P493 4nD 533 1f 7H3r3 12 4 J08 F0r J00 0R 4 KW357 cH4LL3n93 70 90 fUr7h3r @ l3457.
```

又是一個加密後的字串，不過似乎可以透過字去分析是哪個單字，但依照規則全文替換，大概就可以得出結果：

```
DiD yoo kNow JetBrains is alWaYs HiRiNg? cheCK out teh Kareers Page anD see if tHere is a JoB For Joo oR a KWest cHaLLenge to go fUrther @ least.
```

到這裡看得懂的就可以直接往下一步了，看不懂的也能藉助 Google 翻譯，讓這個句子更完整：

```
DiD you kNow JetBrains is alWaYs HiRiNg? cheCK out the Careers Page anD see if tHere is a JoB For Joo oR a KWest cHaLLenge to go fUrther @ least.
```

### 招聘 Fearless Quester 

我們進到求職頁面後，就可以看到有一個職缺叫做 [Fearless Quester](https://www.jetbrains.com/careers/jobs/fearless-quester-356/)。

而在裡面就有一個標題是說 `To progress with your quest what you’ll need:`，內文如下：

1. To check out what we have for game developers.
2. Be geeky enough to remember how you used to cheat at Konami games.
3. Try cheating on the page.


分別代表的意思是：

1. JetBrains 提供給遊戲開發者的一個[解決方案](https://www.jetbrains.com/gamedev/)
2. Konami 有個很有名的遊戲祕技指令，就是 `上上下下左右左右BA`
3. 在 (1) 那個畫面使用 (2) 作弊指令

照著它意思做之後就可以開啟一個打方塊的遊戲，把方塊敲完就會看到結束的訊息了。

{{< img src="/images/2020/jetBrains_quest_2_end.png" caption="打方塊遊戲" width="500" >}}

到網站輸入 Code 後，就可以收到第二個問題的序號了。

Mail 結尾也附上第三個問題公布的地點為 `Twitter`，但好像沒有給時間，是叫我們要訂閱。

```
A little @jetbrains bird will deliver the next clue – so do some birdwatching,
```

## 第三個問題

官方在 2020/03/13 19:00 發出了第三個問題的提示，跟前兩次一樣都是間隔一天。

提示看起來是一個編碼 or 加密過的文字，常玩的人看到這種都先丟 Base64 試試看

{{< tweet jetbrains 1238420744817782784 >}}

### 超常見的編碼方法

果然如我所料，成功解碼了，它就是用 Base64 編碼的。而內容看起來是要我們去官方的 Instagram。
```
Have you seen the post on our Instagram account?
```

進到 Instagram 後，我們就在官方貼文上得到以下文字，看來又是一個產品巡禮，但似乎沒那麼單純。

```
Welcome to the final Quest! You should start on the Kotlin Playground: https://jb.gg/kotlin_quest
P.S. If you don’t know about the #JetBrainsQuest, it’s not too late to find out.
```


我們進到連結後，就看到了以下畫面。看起來是跟第一天一樣的加密訊息，只是官方已經幫你寫好 Code 了，還特地留了 `TODO` 呢！

{{< img src="/images/2020/jetBrains_kotlin_playground.png" caption="Kotlin Playground" width="500" >}}

我們從 1 開始試，一直試到 3 就解出有意義的訊息了，內容如下：

```
We have been working 22/7 on the video for the first episode of the PhpStorm EAP. If we gave you a clue, it would be easy as pi.
```

### 這個梗感覺埋很久了

依照提示訊息來到 [PhpStorm EAP Episode 1](https://youtu.be/OtQuAr3n87c?t=190) 的 3:14 處，看到了相關的提示連結： [https://jb.gg/31415926](https://jb.gg/31415926)

{{< img src="/images/2020/jetBrains_PhpStorm_EAP_Ep1.gif" caption="PhpStorm EAP Episode 1" width="500" >}}

進入連結後，就是一個問答遊戲了，只要錯一個就要重來且限時 1 分鐘，不過題目總數蠻少的，搜尋一下應該就可以過了。

{{< img src="/images/2020/jetBrains_quest_game.png" caption="JetBrains Quest Game" width="500" >}}

全部答對後，就得到以下的提示訊息。

```
Almost there! The last challenge is in the Tips of the Day of a specific IntelliJ IDEA Community version from our latest build page in Confluence, but… there is a catch. You have to know which version to look for. To find the build number, you need sight beyond sight:

. Not Everything Today Does All You Could Ask. Lessons Learned From Other Relevant Solutions, Possibly Even Another Kind Emerge. Risking Sometimes Being Liberal Or Generous Proves Ordinary Simple Tests Infinitely More Annoying. Get Examining Hidden Initial Designated Early Symbols. They Have Everything Needed, Except Xerox, To Completely Level Up Everything.
```

看起來是要去看 IDEA Community 某一個版本裡面的 Tips of the Day，就可以找到提示了。

下面也提示說想要知道版本號，先解出下方這個文章的正確意思。我們留意一下最後一句話。

> you need sight beyond sight

再仔細看一看文章內容，發現英文每個單字字首都是大寫，但英文不會每個字開頭都大寫啊！

所以我們取出每個大寫的字，就拼出了以下提示訊息：

```
.NET DAY CALL FOR SPEAKERS BLOG POST, I MA GEH IDES THE NEXT CLUE
```

接下來我們來到了 [.NET Day Call for Speakers](https://blog.jetbrains.com/dotnet/2020/02/13/jetbrains-net-day-online-2020-call-speakers/) 的文章，尋找是哪一版的線索。

### 又一個藏在 Source Code

仔細看了一下，從表面上看，除日期外沒有任何線索，但從裡面看就發現了官方又把線索放在 html 裡面。

看起來我們要找的是這個版本 [201.6303](https://confluence.jetbrains.com/display/IDEADEV/IDEA+2020.1+Quest+Build+Edition)

{{< img src="/images/2020/jetBrains_dotNet_day_callforxxxx.png" caption=".NET Day Call for Speakers Image" width="500" >}}

把 201.6303 版本的 IDEA 下載下來安裝後，開起來到 `Tips of the Day` 的視窗，按了下一個好幾次終於看到最終的謎題：

{{< img src="/images/2020/jetBrains_201_6303_tips_of_the_day.png" caption="Tips of the Day of 201.6303" width="500" >}}

拿到了最後 Key 的提示了，Key 就是費氏數列第 50,000,000 位的前四後四碼。

看起來不難，跑一下程式就可以算出來了，網路上也有很多[原始碼](https://gist.github.com/yodalee/4e221b081be4b367e9c7ef328ada7db5#file-fastfib-c)可以參考。

{{< img src="/images/2020/jetBrains_quest_end.png" caption="claim your prize" width="500" >}}

## 結語

這三個問題不會說很難，也不會太簡單，剛好都是思考一下就可以想到的，有些還是可以跑程式暴力破解。但我覺得這個活動帶來的宣傳效果不小，特別是將自己的產品融入問題之中，玩家在解題過程就會順勢認識產品。
希望明年也還有類似活動，有得玩又有獎品可以拿 ：）
