# main3
###### tags: `Reverse`
## 解題流程
這題跟前面 main 系列題目差不多概念，只是這題是將 function 藏在 finish array 裡，而且還有 3 個 function，其中兩個 12BA 跟 11DE 看起來就是使用者寫的 function。

![](https://i.imgur.com/YLvB0GX.png)

![](https://i.imgur.com/dw7zWxR.png)

![](https://i.imgur.com/dFtE7zq.png)

另外為了確定這 3 個 function 在 finish 時的執行順序，用 gdb 在 3 個 fuction 的 entry 設斷點，可以得知執行順序是由下往上，執行到第二個結束，就可以看到 FLAG 在 rdi 蹦出來了。

![](https://i.imgur.com/dqKQuXA.png)

![](https://i.imgur.com/fMQU3dI.png)

為什麼會在 rdi 看到，可以看一下 11DE 的 function，v3 ~ v7 其實應該都是 src 的陣列後面部分，這樣對下來也剛好是 46 bytes，符合迴圈拿去做 XOR 的 index，而 XOR 出來後轉成字串就是 FLAG 了，在看一下 assembly 的部分，可以看到結果的 dest 是有 lea rdi 的，所以 rdi 指向的 dest 就是 FLAG 字串了！

![](https://i.imgur.com/gZ0OJYu.png)

## FLAG 截圖
![](https://i.imgur.com/wSZ3PGZ.png)
