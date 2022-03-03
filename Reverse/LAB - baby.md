# baby
###### tags: `Reverse`
## 解題流程
首先看一下 start function，可以看到有一個 system call，根據 x86-64 sys call，rax 是 sys call number，確認一下 x86 sys call table，發現 ida 解析錯物，應該是 sys_open 而不是 sys_wait，那前面對 register 的操作應該就是 open 的參數了。

![](https://i.imgur.com/jPTiw8N.png)

![](https://i.imgur.com/7pIfADC.png)

查一下 open 的 man，可以看到有兩中呼叫方式，argc=4 or argc=3，照 x86-64 sys call register 儲存順序，得知 rdi 為 path 參數，rsi flag 分別為 O_WRONLY(檔案唯寫)、O_CREAT (無文件則創建)、O_EXCL (文件存在出錯返回)，rdx 是 mode arg，設定檔案權限 777。

![](https://i.imgur.com/LqM5IJZ.png)

檔案開起後儲存在 [rbp+100h] 的 stack 位址，rax xor 自己 inti 為 0，r8 要搭配下面區塊看，其實就是 fot 迴圈中的 i=0，r12 存一連續 byte data 的初始 offset(address)，用來跟 "NotFLAG{Hello_Baby_Reverser}" 字串 xor 出 FLAG 的，jnz 的話會去看 ZF flag 是不是為1，如果不為 1 就走紅線跳到 loc_40107c，比較 r8 跟 29，若不為 0 (ZF=1)，就會進 loc_40105B，其實這邊看的出來就是一個 for(i=0; i<30; i++) 迴圈。

![](https://i.imgur.com/AxcyAqj.png)

迴圈裡做的是 func_write 那個 block，r10 讀取固定字串 "NotFLAG{Hello_Baby_Reverser}"，r11 每次存 r10 的一個字元 byte，再 XOR r12 也就是另一組固定的 bytes，一樣一次取一個 byte，最後把結果存到 [rbp+200h]，這個位址，然後做 func_write。

![](https://i.imgur.com/aJRMhbR.png)

根據 write 的 man 對照參數可以發現 func_write 一樣一次一個 byte寫進 fd 檔案。
![](https://i.imgur.com/07sfIlY.png)

接下來要找出 FLAG，我們可以用 gdb，再 sys_write 的地方下斷點，這樣就可以看每次是要寫什麼字元進去。

![](https://i.imgur.com/biMuQGA.png)
寫檔的 buf 是 rsi，因次我們就關注 rsi 的值可以看到 gdb 還非常好心的告訴你這似乎是一個 "F" ASCII 字元，持續執行下去就能看到完整的 FLAG 了。
![](https://i.imgur.com/sUWDHfT.png)

## FLAG 截圖
![](https://i.imgur.com/8r7Iw9v.png)
