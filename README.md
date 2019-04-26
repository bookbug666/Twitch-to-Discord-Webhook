# 自動化的開台通知
Discord目前還沒有辦法直接連結Twitch來達成開台通知的功能，但我們可以透過第三方程式來存取Twitch的API，再透過Discord的Webhook來達成。

以下是你會需要的東西：
+ Discord帳號
+ Twitch帳號
+ IFTTT的帳號

> 注意Zapier的Webhook不推薦，因為他有太多資料都還是未處理過的，像是遊戲就直接報Game ID而非遊戲、時間後面還有一串大部分人看不懂的東西。

# 第一步：註冊IFTTT的帳號
前往 https://ifttt.com/ 並註冊一個帳號，目前也可以透過綁定Google/Facebook帳號的方式來創建帳號。

當然，如果你已經有帳號就直接進行到下一步吧:)

# 第二步：建立Discord Webhook
+ 直接在伺服器圖示右鍵進入伺服器設定，找到Webhooks，點擊建立。

![創建Webhook-1](https://i.imgur.com/a0xTBkR.png)

點擊建立後應會看到以下畫面。

![創建Webhook](https://i.imgur.com/1ddw241.png)

**請不要將/webhooks之後部分公開，知道這串代碼的人有權限在你所設定的頻道發言，請注意你的隱私及伺服器安全。**

+ 名稱、頻道及Webhook圖示都是可以之後再做更改的，這裡我們不急著修改(當然，你想在這裡先改好也是可以的)。
+別忘了將這裏的Webhook網址複製到剪貼簿!

# 第三步：創建IFTTT Applet
~~官方稱作applet/recipe，但我實在是不喜歡食譜或是小程式的說法，這裡我維持原文，如果你有什麼更棒的說法可以告訴我。~~

1. 在My Applet頁面點擊Create Applet以創建一個新的(或是[點我](https://ifttt.com/create)直接進入頁面。)
2. 點擊藍色的大 "[+] This" 並選擇 "Twitch" (用滾輪找可能會比較久，這裡建議直接搜尋Twitch)
3. 連結你的Twitch帳號
4. 選擇 "New stream started by you" (就是你的開台通知)
> **注意：**
> 若是想要在自己的Server中製作別人的開台通知，目前IFTTT只有提供 "Stream going live for a channel you follow" 的功能，也就是你必須有追蹤該台才行。
>
> 而且比較糟糕的是，他必須要之前就在你的追蹤名單內而且近期有開過台，不然可能會出現不在下拉式選單的情況。

5. 點擊藍色的大 "[+] That" 然後選擇 "Webhooks"
6. 選擇 "Make a web request" (也只有這個選項)

![Make a request](https://i.imgur.com/PgfAYDB.png)

7. 接著依序在格子中填入:
   > **URL:** _[第二步中複製起來的網址]_
   > **Method:** POST
   > **Content type:** `application/json`
   > **Body:** _[底下有說明]_
8. 點擊 "Create Action"
9. 大功告成!

# 關於"Body"的部分
Webhooks的內容是相當有彈性的，然而大部分人並不想在Json下折騰數個小時，這裡我提供了一個現成的。
```json
{
  "content": "{{ChannelName}} 開台了!",
  "embeds": [{
    "title": "{{ChannelUrl}}",
    "url": "{{ChannelUrl}}",
    "color": 6570404,
    "footer": {
      "text": "{{CreatedAt}}"
    },
    "image": {
      "url": "{{StreamPreview}}"
    },
    "author": {
      "name": "{{ChannelName}} 開台了!"
    },
    "fields": [
      {
        "name": "正在玩",
        "value": "{{Game}}",
        "inline": true
      },
      {
        "name": "觀眾數",
        "value": "{{CurrentViewers}}",
        "inline": true
      }
    ]
  }]
}
```
當你完成的時候，成果應該會像這樣子。

![示意圖](https://i.imgur.com/5NaoOST.png)

## 自訂訊息內容
如果你想要更改訊息內容，把"XXX 開台了!" 改成想要的內容就好。
```json
{
  "content": "{{ChannelName}} 開台了!",
  "embeds": [{
```

## 同場加映：在嵌入區中加入個人檔案圖片
1. 打開Twitch
2. 滑鼠右鍵你的個人檔案圖片，點擊"複製圖片位址"

![enter image description here](https://i.imgur.com/dYAMj9y.png)

4. 將畫面中`<IMAGE_URL>`改成你剛剛所複製的圖片網址。
```json
"thumbnail": {
  "url": "<IMAGE_URL>"
},
```
5. 將剛剛那一串貼在 `image` 的下方，你的代碼會從
```json
    "footer": {
      "text": "{{CreatedAt}}"
    },
    "image": {
      "url": "{{StreamPreview}}"
    },
    "author": {
      "name": "{{ChannelName}} 開台了!"
    },
```
變成這樣
```json
    "footer": {
      "text": "{{CreatedAt}}"
    },
    "image": {
      "url": "{{StreamPreview}}"
    },
    "thumbnail": {
      "url": "<IMAGE_URL>"
    },
    "author": {
      "name": "{{ChannelName}} 開台了!"
    },
```

**注意：** 
Twitch的個人檔案圖片網址並非固定網址，當你變更圖片之後就必須將code內的網址更改。

# 已知問題
1. 使用{{StreamPreview}}的時候，Discord的圖片快取只會讀取一次，之後開台的樣子在Discord中都是同個圖片。

**解決辦法：目前沒有**
> 無論是改成 "https://static-cdn.jtvnw.net/previews-ttv/live_user_{{ChannelName}}-1280x720.jpg" 或是 "https://static-cdn.jtvnw.net/previews-ttv/live_user_[直接打上Username]-1280x720.jpg" ，實際上POST出來的時候都是同個網址，他還是會抓取一樣的網址。
 
# 心得
坦白說Twitch API的延遲也差不多就是4-5分鐘，現在Bot如Mee6都能做到一樣的事情而且你換頭貼還會自動抓取，實在是沒必要用這個。

~~我自己就是因為資料一樣跟Mee6一樣等4分鐘，加上Profile Picture一直有問題而乾脆改用Mee6了~~

而且Discord對於Webhook Bot是獨立的，也就是說他沒有User Tag，你也不能靜音他或是搜尋他說過的話，對於不想看到的人等於是強迫中獎。

此外，若是使用IFTTT等類似服務，資料會受限於IFTTT端的變數，若是想要時間更準確或是更多變化，我想還是自己用的Python或是C#/C++寫一隻Bot比較方便。

***本文作為技術演示以及備份用途使用。***
