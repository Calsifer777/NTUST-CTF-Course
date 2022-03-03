# main1
###### tags: `Reverse`
## 解題流程
用 ida 打開看到有定義 finish, init 跟 main 的 function pointer。

![](https://i.imgur.com/xjY4wru.png)

點進 main 看起來是沒甚麼可疑的點。

![](https://i.imgur.com/W53tot4.png)

再來進 init 看看，有沒有 function 藏在 init array，可以看到他按照記憶體位址順序呼叫 init array 中的 function ptr。

![](https://i.imgur.com/ecBtgQy.png)

可以看到 init array 的部分有兩個 function ptr，正常應該只有一個，所以就是藏了一個 function 在這裡，sub_11E0 沒什麼可疑的地方，點進去 sub_1328 還有再 call sub_12CF 跟 sub_11EA。

![](https://i.imgur.com/Bcr6B9s.png)

最終可以看到有一些熟悉的字樣，應該就是使用者自行建立的 fuction 沒錯。

![](https://i.imgur.com/rovrTcC.png)

F5 反編譯可以看出就是在對一組 21 byte 的東西做迴圈 XOR，用 python 模擬他在做的事，就可以拿到 FLAG 了！

![](https://i.imgur.com/TShHMKa.png)

![](https://i.imgur.com/jlcBT0N.png)

## FLAG 截圖

![](https://i.imgur.com/cvF6OZZ.png)
