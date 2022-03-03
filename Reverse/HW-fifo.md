# fifo
###### tags: `Reverse`
## 解題流程

- 首先用 ida 開啟 fifo 檔案，在 start 看到熟悉得 fini 跟 init，跟進去看一下確認 init array 與 fini array 沒有藏 function ptr。

    ![](https://i.imgur.com/3SY5jDR.png)

    ![](https://i.imgur.com/L0LqYK8.png)

- 接著進 main 按 F5 反編譯一下，可以看到很大一坨 code，稍微順一下流程，path 跟 file 分別是路徑跟檔名，但用 python 直接對其 hex 轉 byte 會發現它並不能轉乘 ASCII 字元，sub_12E9 點進去可以看到是對傳入的變數每個 byte 做一些 XOR 的運算，而做完之後就 open file 變數的檔案，可以合理推測 sub_12E9 就是將前面經過混淆的 path、file 等變數做還原，得到正常的值。

    ![](https://i.imgur.com/O9Hs0aF.png)

- 用 gdb 打開在 sub_12E9 執行後的 open() 下個斷點 (透過 assembly 查位址，由 base 往上加)，可以看到欲傳入 open() 的第一個 file 參數存在 rdi，成功看到還原的 file name，果然 sub_12E9 是在做混淆還原。

    ![](https://i.imgur.com/yb7dllc.png)

    ![](https://i.imgur.com/2otH6fW.png)

- 所以還原個變數後在該目錄下創建一檔案，並寫入一些 data，然後關閉檔案。

- 接著繼續往下看，這邊 fork() 了一個子進程，並且根據 execve()定義，透過 state structure 設定 argv[] 參數，所以這邊 fork() 出來的子進程把前面創建的 /tmp/khodsmeogemgoe 當執行檔來執行。

    ![](https://i.imgur.com/drkplR5.png)

    ![](https://i.imgur.com/cQBpTaz.png)

## 父進程
- 前面 fork() 返回的 PID > 0 是父進程，不會進 if，因此再往下看，父進程 mkdir 一個 path，這個 path 變數也是前面還原回來的，並且透過 mkfifo 在該路徑下又創建了一個 fifo 檔案，讓進程間可以溝通，接著父進程 open() 該檔案並寫入 data，供後續子進程讀取。

    ![](https://i.imgur.com/AdCJxD0.png)

- 一樣在 open() 的地方設斷點，可以看到 mkdir 成功創建 /tmp/bnpkevsekfpk3 資料夾，mkfifo 成功建立 /tmp/bnpkevsekfpk3/aw3movsdirnqw 的 fifo 檔案，open() 也準備根據 還原後的 v15 變數來讀取並寫入 fifo 檔案。
    ![](https://i.imgur.com/Dq7sfAP.png)

- 整個 fifo 檔案流程執行完，多了 /tmp/khodsmeogemgoe 及 /tmp/bnpkevsekfpk3 資料夾，然而 /tmp/bnpkevsekfpk3/aw3movsdirnqw 檔案卻消失了，父進程後續也沒有動 aw3movsdirnqw 檔案，可以合理推測應該是子進程搞的鬼。

### 子進程
- 我們來看看子進程做了什麼事，將 khodsmeogemgoe 拖到 ida 看看 (子進程 execve()，把 khodsmeogemgoe 當執行檔執行)，出現 sp analyze 問題一樣先 u 選好位址再 p，反編譯後可以看到 khodsmeogemgoe 在做的事是將 aw3movsdirnqw data 讀出存到 buf，並把 aw3movsdirnqw 檔案刪掉，最後 sub_1209 點進去可以看到跟原本 fifo 檔案 sub_12E9 很像的 XOR 還原 code，還原的 buf data 接著被拿去當成 function 執行，這邊就很可疑，可以在這邊 call buf 的地方下個斷點觀察一下記憶體狀態。

    ![](https://i.imgur.com/hsjkMFc.png)

- 用 ```ps aux | grep khodsme``` 指令揪出已被自動執行的子進程，再 kill 掉。

    ![](https://i.imgur.com/Y9XVVAo.png)

- 然後在原 fifo gdb 的 execve() 下個斷點，讓子進程 fork() 之後不要直接去執行 /tmp/khodsmeogemgoe，注意 gdb 下 ```set follow-fork-mode child``` 的指令，不然 gdb 預設是關注父進程的，會讓子進程自動跑完，而 fifo 檔案是有管道的概念，被讀過後 data 會消失。

    ![](https://i.imgur.com/k4YHgf8.png)

- 確認沒執行的 aw3movsdirnqw 進程後單獨用 gdb 將 khodsmeogemgoe 開起來，設好 call buf 的斷點。

    ![](https://i.imgur.com/evO9xdr.png)

- 執行下去就看到 FLAG 了，看來從 aw3movsdirnqw 讀入的 data 在經過 sub_12E9 與 v5 data 做 XOR 還原後就是正確 FLAG 了，會看到 FLAG 是因為 rdi 是 function 第一個參數存的 register，這邊是指向 v5 陣列的地址，v5 改變後 rdi 就也看的到改變後的值了！

    ![](https://i.imgur.com/S8AI02x.png)

## FLAG 截圖

![](https://i.imgur.com/V2Ajjkw.png)
