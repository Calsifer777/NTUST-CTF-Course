# 3xc3pt
###### tags: `Reverse`
## 解題流程

- 用 ida 開起來可以看到 main function 有個 exception 的 handler sub_140001100()，因為我用的是 ida pro，所以 ida 直接幫我解析出來了。

    ![](https://i.imgur.com/lNlrJIB.png)

- 如果沒有 ida pro 就需要用 PEbear，從 Exception Directory 去追蹤，看 exception 是落在哪個區間，進一步找到 handler 位址，來自己解析 Exception function。

    ![](https://i.imgur.com/YkixKRF.png)

    ![](https://i.imgur.com/lXMecb5.png)

- 點進 sub_140001100() 後，可以看到很大坨的兩個陣列賦值，然後對 CommandLine 指標迴圈往下做一些運算操作，最後餵進 CreateProcessW() 這個 create new process thread 的 function 中。

    ![](https://i.imgur.com/TNUAYT2.png)

- 我們想看 CommandLine 運算完最後餵進 CreateProcessW() 的值就用 x86dbg 開啟，在 CreateProcessW() 的地方下個斷點，F9 執行後就看到原來運算完的 CommandLine 就是 FLAG 拉！

    ![](https://i.imgur.com/oqyxNbf.png)

    ![](https://i.imgur.com/xVSPN6F.png)

## FLAG 截圖

![](https://i.imgur.com/EPIXIVE.png)
