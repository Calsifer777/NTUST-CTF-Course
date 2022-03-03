# HW - DVD Screensaver
###### tags: `資安` `Web`
## 題目解析
題目給了蠻多提示，也有給 source code，裡面還包含了 dockerfile 跟 docker-compose.yml，可以自行把網站架起來測試。

![](https://i.imgur.com/j6KTEeW.png)

題目就是要你以某個特定帳號登入，你就可以看到 FLAG。

![](https://i.imgur.com/cjwKDWu.png)

## 解題流程
題目是讀取登入輸入的帳號密碼，做 sql 查詢後取帳號跟透過 os.Getenv 取得的 SECRET_KEY 環境變數做一個 signed cookie 返回，再導到登入後頁面驗證 cookie ，並做 sql 查詢對應 username FLAG 欄位。

整題的思路的話首先要想辦法拿到 secret key，拿到 secret key 後我們就可以偽造 cookie，再用偽造的 cookie 讓我們能成功以特殊帳號身分登入拿到 FLAG。

把主要程式碼 app.go 看過一遍，看到 os.ReadFile 可以很猜到 static 的頁面應該存在 path traversal，讓我們能去挖到 process environ 的資訊。

![](https://i.imgur.com/OcA5QSM.png)

試了一陣子嘗試用 ../ 回到前一層存取檔案，都存取不到，會直接跳回登入頁面，只有 static 路徑下的東西讀取得到。

![](https://i.imgur.com/d0RcCFj.png)

![](https://i.imgur.com/1VUNyO2.png)


從 go http 的 doc 可以看到 Handler 實際上如果你給的不是規範形式 url，他會重新導回規範路徑，除非使用 CONNECT method，他才會不改變 path 跟 host 做一次內部 redirect 到該 url，，所以理所當然我們在網址上想透過 ../ 訪問非規範 url 會讀不到東西，因為 url 出去預設是 GET method，他就直接導回 static/，存取不到東西就又導回根目錄。

![](https://i.imgur.com/pedHwgM.png)

所以我們用 curl 發送 CONNECT method 的封包，另外根據 RFC3986 定義之 url 的 normalization，curl 預設是會把 url 中的 ../ 給移除，因此要在再加上 --path-as-is 參數才可以不做 normalizatioin，順利把 path traversal url 送出去。
```
curl -X CONNECT --path-as-is http://dvd.chal.h4ck3r.quest:10001/static/../../etc/passwd
```
![](https://i.imgur.com/WhoCcuZ.png)

沒問題可以順利存取到，再來就是要找 Environmental variables (環境變數)了，我們要找的不是系統的 etc/environ，是 process 的 proc/self/environ，直接讀的話會告訴你用存檔的方式，所以把 environ 儲存成 a.txt，打開來看就可以看到 secret key 了。

![](https://i.imgur.com/WzaAp2g.png)

![](https://i.imgur.com/CUnuXEy.png)

再來是要想想怎麼偽造正確的 session (cookie)，想從登入階段下手，查到正確含有 FLAG 的帳號其實不太可能，從 login 部分的 source code 可以看到有對我們帳號密碼的輸入做 regex rule 的判斷，幾本上沒法做到什麼 sql 語法的銜接。

![](https://i.imgur.com/OaJwd1Y.png)

因此我們應該換個方向想，看看根目錄也就是檢查登入的部分，現在我們有 seret_key，因此 cookie 是可控的，cookie 從上面 login 部分也可以看出來 cookie 參數只有 username，那是不是可以控制 username 在根目錄部分做 sql injection，讀到含有 FLAG 的帳號。

![](https://i.imgur.com/bYXEuU5.png)

```
' UNION SELECT username , flag FROM users where flag like '%FLAG{%
```

透過將上面 sql 當成 username，就能去 select  FLAG 形式的字串，也就可以找到正確的 username 了，所以我們就把 secret_key 跟 injection 的 username 寫進 source code，透過以下指令架起 local server。
```
docker-compose build
docker-compose up
```
![](https://i.imgur.com/QELujSo.png)


![](https://i.imgur.com/arTOtYg.png)

從 local 端用預設 guest 帳號登入，透過 burpsuite proxy 攔截 request，複製 cookie 到正式 server 上修改再 forward，就可以偽造 cookie，拿到 FLAG 了！

![](https://i.imgur.com/bDoZGjq.png)

## FLAG 截圖
![](https://i.imgur.com/3BOvVvG.png)