# FILE note \- \(R\)
###### tags: `Pwn`
## 解題流程
- 先看一下 sc，可以看到最前面讀了 FLAG 檔案中的 FLAG，但又 memeset 把讀到 buf 上的 FLAG 自串又清掉了，且最後 malloc 了一個 buf 給我們。

    ![](https://i.imgur.com/pNflY7p.png)

- 在來是 3 個動作選單，分別是 note 的 RWO，其中 open 開一個 tmp file，read 用 get()，可以看出來有 BOF，write 是將 node_buf 寫到開的 tmp fd 裡。

    ![](https://i.imgur.com/iB593S0.png)

- 用 pwngdb 開起來可以看到 Heap 情況，分別是 tcache、fd、fd 的 buf、node_buf 4 個 entry。
- 用 opem note 會再多一個 chunk。  
- 因為 node_buf 是在最後面宣告的，所以新 malloc 的 tmp file 為只會是接續在 node_buf 的。
- 所以我們其實可以用 BOf 來去做一個我們想要的 file structure，只要知道 FLAG 的位址，控制一下 BUF 得 ptr 與 end，就可以讀到 FLAG。

    ![](https://i.imgur.com/AEMwt7B.png)

- 了解大致流程後，首先要透過 note_buf 往上找到原 FLAG 的位址，發現是拿 note_buf 位址 - 0x5570bc3ab480 = 0x1010，所以 FLAG 的位址可以用 note_buf - 0x1010 表示。
- 測一下從 note_buf 到 tmp file，我們會要蓋多少個字(0x210)。

    ![](https://i.imgur.com/wJqPkjO.png)


- 另外也要設定一下隨意讀的條件 flags。

    ![](https://i.imgur.com/Fl8BNer.png)

- 接著就是按照 file structure 設定 IO read 跟 IO write buffer 的 ptr 與 end 位址，接在 BOF 後，最後用 3 選項 read 出 tmp file 內容就可以順利印出 FLAG 拉！

    ![](https://i.imgur.com/d0YeMnH.png)

    ![](https://i.imgur.com/UtZLt0t.png)

### FLAG 截圖
![](https://i.imgur.com/Om0IIZK.png)
