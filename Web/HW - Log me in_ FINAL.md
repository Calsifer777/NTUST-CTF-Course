# HW - Log me in: FINAL
###### tags: `資安` `Web`

## 題目解析
題目是一個登入頁面，有提示說 debug page 可以看到 source code。
![](https://i.imgur.com/19k4ytO.png)

![](https://i.imgur.com/j2ZDCKs.png)

## 解題流程
要看到 debug page 其實用預設 guest 登入稍微等一下，server 應該有設 timeout 時間，過一段時間重新整理，連線因為 server 的 timeout 斷掉，就會出現 debug page，而且錯誤也有噴出來給給你看，其實後來測試 sql injection 也發現你只要故意送錯的 sql 語法，他就會噴錯了，也不用等時間重新整理。

![](https://i.imgur.com/EZPHftL.png)

從 debug 可以看到 source code，且也會標註錯誤發生的地方，看起來很明顯就是存在 sql injection 的漏洞，再來就是使勁戳了，當然他有 WAF，所以並不是那麼好戳，有很多東西要想辦法繞過。

試了好一陣子，union、select、or、and 這些 sql 語法的關鍵詞都會被擋，所以可以在中間加上\/\**\/，繞過 WAF 又可以讓 sql 語法順利執行，space 的話可以用 tab 替換 (從可以打 tab 的地方複製過來)，另外單引號前面用一個 \ 跳脫讓 waf 加的 \ 失效，理論上就可以控制 sql 了，但我似乎還差一步，應改哪裡出了點小問題，感覺 sql 語法還是錯的，但我自己有架 sql server 起來測，模擬他建了一個資料表，是可以的，也能順利做到 union select 跟 order 的語法。
```
\'	un/**/ion	se/**/lect	1,1	#
```
![](https://i.imgur.com/sGsRaYP.png)

## FLAG 截圖