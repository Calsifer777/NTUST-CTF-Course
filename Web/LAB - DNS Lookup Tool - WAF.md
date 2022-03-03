# LAB - DNS Lookup Tool - WAF
###### tags: `資安` `Web`

## 題目解析
這題基本就跟前一次 DNS Lookup Tool 的 LAB 一樣，給你輸入框，要你輸入網址，後端會做 **"host <你的輸入> ;"** 的指令來查詢 host，只不過這題有一個 blacklist，若你輸入 blacklist 裡的字元或字串，就會被 block 掉。

![](https://i.imgur.com/kP3PexW.png)

![](https://i.imgur.com/gqLCymp.png)

## 解題流程
由於我們沒辦法單純閉合 '，並用 ";" 來分隔指令直接 cat 出 FLAG，所以得想辦法繞過 blacklist。

![](https://i.imgur.com/bdFOxlH.png)

而這就要用到 linux 的 cmd substitution，在 linux 指令下用 $(cmd)，可以把 cmd 執行後的 stdout 存成變數。

![](https://i.imgur.com/MIZtan9.png)

透過這樣的方式我們就可以將我們執行的 cmd 結果當作網址拿來做 host 指令，當然這網址一定是錯的，但這樣我們就可以透過錯誤訊息看到我們 cmd 的執行結果。

再輸入框輸入 '"$(ls /)"'，閉合前後單引號，中間塞 cmd substitution，去根目錄找找看 FLAG，但根目錄 ls 輸出太長，丟到 host 指令會錯誤，所以我們看不到結果。

![](https://i.imgur.com/oFAyGOO.png)

所以我們輸入 '"$(ls /f*)"'，用 * 擴展文件名來找找看有沒有 FLAG 檔案，果真有一個。

![](https://i.imgur.com/PDbQhje.png)

最後就直接 '"$(cat /f*)"'，就可以看到 FLAG 了！最後輸入 FLAG 要把 "/" 去掉，那是後面特殊字元的跳脫符。

## FLAG截圖
![](https://i.imgur.com/IceEvXa.png)
