# HellWindows
###### tags: `Reverse`
## 解題流程

- 用 ida 開啟可以在 function window 查到有兩個 TlsCallback function，windows 下的機制，會在 main 前執行。

- 兩個 function 分別對 aTh3Pa55wd 這個參數修改一次值，至於執行順序目前不知道，等等再來看。

    ![](https://i.imgur.com/7xZKduK.png)

    ![](https://i.imgur.com/LzBrAJZ.png)

- 再來看看 main 的部分，可以看到 argc 需要 >= 2，也就是要傳入一個參數，傳入的一個參數會跟 aTh3Pa55wd 做字串比對，比對成功才繼續往下執行，下面就是對 aOtflagHelloMyF 做一些運算，最後會告訴你將 FLAG 放在 FS 了，理論上進 FS 找一下應該就能看到 FLAG。

    ![](https://i.imgur.com/aPesxJu.png)

    ![](https://i.imgur.com/i9FR9bC.png)

    ![](https://i.imgur.com/X3gkucg.png)

- 用 x64dbg 能發現 aTh3Pa55wd 最後的值是 Yoasobi，在命令列傳入 Yoasobi 字串，就能順利執行了。

    ![](https://i.imgur.com/KDf6VW0.png)

- 但這邊我透過 teb()去找卻沒看到 FLAG 字樣，應該是因為 64 bit 是 teb 是 GS[0] 開始，不是 FS，試了好久都找不到 FS register 位址，這邊可能要用雙機測試以 kernel debug 的方式看才能訪問到 FS 區塊，所以最終沒能解出來。

    ![](https://i.imgur.com/7AdFK12.png)


