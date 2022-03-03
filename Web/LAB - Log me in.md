# LAB - Log me in
###### tags: `資安` `Web`

## 題目解析
畫面上看到就是個簡單的登入頁面，點下 magic 可以看到他根據輸入搜尋帳號密碼的 SQL 指令，明顯就是可以用 SQL Injection 去做破解。
![](https://i.imgur.com/0v12jRs.png)

## 解題流程
首先要閉合 username 的單引號根括弧，然後把後面比對 password 的部分註解掉，中間補個一定為 True 的條件 or 起來，密碼就算便打個字串，就能通過密碼檢查，拿到 FLAG 了！
```
') or "aaa"="aaa" --
```
![](https://i.imgur.com/teHXd2f.png)

## FLAG 截圖
![](https://i.imgur.com/ImB4pLi.png)
