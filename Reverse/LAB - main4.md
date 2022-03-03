# main4
###### tags: `Reverse`
## 解題流程
這題跟 main3 整體流程是一樣的，只是它變成了 32 bit，必須用 32 bit ida pro 來開啟，我們一樣找到 finish array，發現有 3 個 function。

![](https://i.imgur.com/6Z4Pgs1.png)

在 gdb 設斷點，發現執行順序是由下往上，第三個 function 沒做什麼可疑的事，第二個 sub_124D 跟 第一個 sub_130E 就明顯是人為的了。
![](https://i.imgur.com/x6DngUt.png)

![](https://i.imgur.com/B311Las.png)

![](https://i.imgur.com/3gqzXDl.png)

而程式一樣執行完第二個 function 就可以看到一點 FLAG 的跡象，不過並不完整。

![](https://i.imgur.com/n0jKrBm.png)

到這邊我們可以判斷他是在 sub_124D 做 XOR 得出 FLAG 的，而在反編譯的 code 可以看到他做完後是 copy 到 byte_4040 這個 buffer 也就是 dest。

![](https://i.imgur.com/uNxlhni.png)

所以我們在 push src 下個指令設一個斷點，按下 ni 應該能看到 strcpy 執行後 retrun 到 rax 的 src，就可以拿到 FLAG 了。

![](https://i.imgur.com/cRIw5DX.png)

![](https://i.imgur.com/tPy9LIl.png)

## FLEG 截圖

![](https://i.imgur.com/RbVLEBE.png)
