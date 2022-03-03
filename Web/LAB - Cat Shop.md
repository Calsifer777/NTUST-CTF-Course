# LAB - Cat Shop
###### tags: `資安` `Web`

## 題目解析
一進來就可以看到 FLAG 字樣，看起來要用我們的錢錢去買到他，但我們只有 65536。
![](https://i.imgur.com/XssaIKS.png)

## 解題流程
我們看到 FLAG 的商品，但重點是不能購買，因此可以觀察一下前兩個的 Orange Cat 跟 Rainbow Cat 按下購買後的跳轉，打開 F12，看到其網址分別是 /item/5428 跟 /item/5429，因此我們合理懷疑 FLAG 是 item/5430

![](https://i.imgur.com/GR39zCX.png)

直接在網址後輸入 /item/5430，成功跳進購買頁面。

![](https://i.imgur.com/h2sROdQ.png)

但我們錢錢不夠，買不起 FLAG，F12檢查看看，他是用 POST 去發送一個 form-data，因此我們可以找找 input tag，發現有一個 hidden 的 input ，name 還是 cost，value 也是 FLAG 的錢錢，我們直接把 costs 改成 0 元，他送出去的 form-data 的 cost 應該就會是 0 元了。

![](https://i.imgur.com/xJ1APb7.png)

果然是跟我想的一樣，成功購買 FLAG，回到首頁就可以看到 FLAG 了！。

![](https://i.imgur.com/uBgUGru.png)

## FLAG截圖

![](https://i.imgur.com/aylLWSw.png)
