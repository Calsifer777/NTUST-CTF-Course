# LAB - 喵 site
###### tags: `資安` `Web`

## 題目解析
主頁就是一個可愛的彩虹貓貓，上方有一個導覽列，還有 About 跟 Admin 另外兩頁。

![](https://i.imgur.com/YWCoiN8.png)

About 就是一個簡單的網頁介紹。

![](https://i.imgur.com/5Gvmo3q.png)

Admin 則是一個登入畫面，看來我們想拿到 FLAG 就要成功從這邊登入。

![](https://i.imgur.com/pVItMmH.png)

## 解題流程
可以看到頁面網址上有傳送 page=inc/home，看起來後端似乎是 include 我們傳的字串來打開頁面，所以我們試試看傳不存在的檔案名。

![](https://i.imgur.com/JIXLaIZ.png)

從錯誤訊息查看，果然他是去 include page 參數的字串加 .php 後輟來取得相應頁面，因此接下來可以嘗試做 LFI，去讀我們想要知道的 Admin 頁面資訊。

![](https://i.imgur.com/bit8grv.png)

既然知道他是 PHP 的 Website，我們就讓 include 過 PHP filter 去讀取 Admin.php 頁面做 base-64 的 encode，就可以看到返回的一串字串。(有點太長，截一點就好)

PHP filter：
```
http://splitline.tw:8400/?page=php://filter/read=convert.base64-encode/resource=admin
```
![](https://i.imgur.com/KGQ3TEy.png)

再來就把它複製到 CyberChef 做 base-64 decode，就可以看到 Admin.php 的 Source code 了，看到 username 跟 password 都直接寫死在裡面，直接複製出來登入就拿到 FLAG 了！
![](https://i.imgur.com/PVs2ZTA.png)

## FLAG截圖
![](https://i.imgur.com/zSijtbz.png)
