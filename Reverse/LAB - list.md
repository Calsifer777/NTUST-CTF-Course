# list
###### tags: `Reverse`
## 解題流程

用 ida 開啟後可以看到反組譯的 main function 長這樣，可以看到註解有說未完成的部分是 XOR value1 跟 value2 這個東西，然後這邊 call 了一個 sub_1168 的 function。

![](https://i.imgur.com/BSH1Fb9.png)

點進 function 後可以看到他分出錯的訊息，空白鍵跳到對應 asssembly code 按 u 再按 p 就可以再正長的解析 function 了。

![](https://i.imgur.com/EW0dDsZ.png)

進到 function 後看到 malloc 直覺可以想到他是要 new 一個 structure，可以透過 structure 介面來慢慢建造 structure。

![](https://i.imgur.com/fgBl1cN.png)

最終解出來的兩個 structure 應該是長得像下面這樣。

![](https://i.imgur.com/adkct5q.png)

解出來的 sub_1168 function 像這樣，可以看到他是兩個長度 32 的陣列，分別對 0x5A 與 0xA5 做 XOR。

![](https://i.imgur.com/vKsWFmU.png)

![](https://i.imgur.com/fsLiycj.png)


所以我們可以推測這兩個應該就是 value1 跟 value2，再寫個 python 把它們做 XOR，就可以看到 FLAG 了！

![](https://i.imgur.com/J2auo8y.png)

## FLAG 截圖
![](https://i.imgur.com/iwRQ6Rr.png)
