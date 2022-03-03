# Got2win
###### tags: `Pwn`
## 解題流程
- 從 source code 可以看到讀了一個 flag 檔案，我們就是要想辦法讓這個讀近來的字串印出來。

    ![](https://i.imgur.com/Izy27by.png)

- 在 "Overwrite addr: " 後 sanf() 一個 addr 變數，可以輸入一 address，讓下面從 stdin read 新的 address 覆蓋前面 addr 上的值。

- 而後面會在從 stdout 輸入的值將 flag 覆蓋掉，但其實這邊的用意就是外部函式在動態載入會去 GOT 取 addr，透過覆蓋 GOT 的對應 addr 變成想要執行的外部函數 plt，就可以透過該 plt 抓到從對應 GOT 抓到欲執行的任意外部函式。

- 因此透過在 read 呼叫前將 read 的 GOT 值存 write plt，就能成功利用 wrtie plt 從 GOT 取得 write 的實際位址來執行，並將 flag 給印出來。

- 首先我們要找 read 的 GOT 位址，在 pwngdb 輸入 got 就可以看到 got 列表。

    ![](https://i.imgur.com/Ommj2Js.png)

- 再來透過 b write@plt 下呼叫 write 準備進入 gadget1 的斷點，查看位址。

    ![](https://i.imgur.com/GtwE5tU.png)

- 最後透過 python pwntools 利用前面 scanf() 及 read() 分別輸入 read 的 got 位址、write 的 plt 位址。

    ![](https://i.imgur.com/jJBdiAr.png)

- 執行後就能看到 FLAG 拉~

    ![](https://i.imgur.com/uJyDnyr.png)
## FLAG 截圖
![](https://i.imgur.com/uJyDnyr.png)
