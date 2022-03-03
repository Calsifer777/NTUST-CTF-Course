# Lab - DNS Lookup Tool
###### tags: `資安` `Web`

## 題目解析
題目提供的網站如下，點擊 Magic 可以看到 host + 自己的輸入，點擊 Source Code 可以看到網頁的原始碼。
![](https://i.imgur.com/IBuJC7v.png)

## 解題流程
在 Source Code 中我們可以看到比較重要的一段的 php block。

![](https://i.imgur.com/FXetIeJ.png)

可以看到我們在輸入框輸入的字串會被直接帶進 shell_exec 的 function，並接在 host 指令之後，以一對 ' ' 把我們的輸入包起來轉成字串，一般來說是要我們輸入網址做 host 指令查詢，但我們可以做 injection 的動作，用 ' 結束第一個 '，再用 ";" 分隔指令，做 ls -al，最後用另一個 ' 結束後面的 ' 。

![](https://i.imgur.com/jrBAqNp.png)

可以看到我們成功執行了 ls 指令。

再來就是要找FLAG，FLAG 通常會放在跟目錄或 home 底下，我們就直接去根目錄看看，看到一個 flag_44ebd3936a907d59，應該沒錯就是他了，檢視該檔案就可以看到 FLAG 了

## FLAG截圖

![](https://i.imgur.com/zHFd083.png)