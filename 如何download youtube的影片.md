# yt-dlp

## 基本使用
```
yt-dlp -f bestvideo+bestaudio --merge-output-format mp4 https://www.youtube.com/watch?v=XXXX
```


`yt-dlp` 支援匯入 cookies，這在下載需要登入驗證的影片（例如 YouTube 私人影片、會員限定影片、Bilibili、Twitter 等）時非常重要。

以下是 **完整詳細步驟** 教你如何將瀏覽器中的 Cookie 匯入到 `yt-dlp` 使用。

---

## 🧭 第一步：匯出瀏覽器的 Cookie

### ✅ 方法一：使用擴充套件《**Get cookies.txt**》

這是最推薦、最簡單的方式，支援 Chrome / Edge / Firefox：

1. 安裝擴充套件：
    
    - [Chrome 版](https://chrome.google.com/webstore/detail/get-cookiestxt/gmpjjijgfeckehgoecbpihgfblgklbph)
        
    - [Firefox 版](https://addons.mozilla.org/en-US/firefox/addon/cookies-txt/)
        
2. 安裝完後，打開你要下載影片的網站（例如 YouTube、Bilibili）。
    
3. 點擊瀏覽器工具列中的 **Get cookies.txt** 圖示，會自動產生 `.txt` 檔案，下載儲存下來（例如 `youtube.com_cookies.txt`）。
    

---

### ✅ 方法二：手動從開發者工具匯出（不推薦，麻煩又容易漏）

如果你不想裝擴充套件，也可以這樣手動抓 Cookie：

1. 在影片頁面按 F12 開啟開發者工具。
    
2. 選擇「Application（應用程式）」→ 左邊選「Cookies」→ 點選該網站。
    
3. 右鍵複製每一行的內容（如 `name=value`），最後組成一長串格式，貼成 `.txt` 儲存，格式如下：
    

```txt
cookie1=value1; cookie2=value2; cookie3=value3
```

（這方法不建議，除非你很熟悉 Cookie 機制）

---

## 🧪 第二步：用 yt-dlp 匯入 Cookie

假設你已經有了 cookie 檔案，例如 `youtube.com_cookies.txt`，指令如下：

```bash
yt-dlp --cookies youtube.com_cookies.txt "https://www.youtube.com/watch?v=xxxxxx"
```

這樣 yt-dlp 會以你的登入狀態執行下載，例如可下載：

- 僅限會員的影片
    
- 私人影片
    
- 年齡限制內容
    
- 登入才能看到的串流
    

---

## 🔒 注意事項

- cookie 有有效期限，如果你換裝置或登出帳戶，之前匯出的 cookie 檔可能會失效，要重新匯出。
    
- 若你是登入 Google 帳號，也可能啟用了雙重驗證，那麼 cookie 的使用可能有限制（特別是敏感帳戶）
    
- 不建議分享你的 cookie.txt 檔案，因為它包含你的登入身份，任何人都可以假冒你。
    

---

## 🔁 補充：結合 ffmpeg 自動合併影片與音訊

有些影片是分開下載（video + audio），你可以這樣整合下載與轉檔：

```bash
yt-dlp -f bestvideo+bestaudio --merge-output-format mp4 \
--cookies youtube.com_cookies.txt \
"https://www.youtube.com/watch?v=xxxxxx"
```

---

如果你願意提供你要下載的平台（YouTube / Bilibili / 其他），我可以給你專屬參數範例。是否需要我幫你測試網址的可下載性呢？