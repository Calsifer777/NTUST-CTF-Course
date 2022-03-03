# LAB - Log me in: Revenge
###### tags: `資安` `Web`

## 題目解析
畫面上看到就是個簡單的登入頁面，點下 magic 可以看到他根據輸入搜尋帳號密碼的 SQL 指令，而跟前面 Log me in 不同的地方從 Source code 可以看到，他是用輸入的 username 直接去 select，取出 SQL result 的密碼再跟輸入密碼比對，因此直接註解掉 SQL 語法是沒用的，要想辦法讓他返回 \[username, password] 形式。

![](https://i.imgur.com/Kf4oVX1.png)

## 解題流程
既然我們不知道 admin 帳號的真實密碼，我們只要想辦法讓 sql 搜尋結果出來的帳號密碼是我們能控制的，就可以輕鬆用假帳號密碼登入。

這邊就要用到 SQL 的 UNION 指令，UNION select 可以將兩次 select 做聯集，就像是在做 OR 語法，如果紀錄存在於第一個 select 或第二個 select，就會被取出，不過要注意的是兩邊 column 數及資料型態必須相同。

```
xxxxx') union select 'admin', 'all_my_god' --
```
![](https://i.imgur.com/IrBIEoq.png)

密碼就是我們自行 UNION select 的 "all_my_god"。

## FLAG 截圖
![](https://i.imgur.com/apTjnPN.png)
