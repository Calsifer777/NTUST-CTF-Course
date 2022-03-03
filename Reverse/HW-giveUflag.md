# giveUflag
###### tags: `Reverse`
## 解題流程

- 首先用 ida 把 giveUflag.exe 開起來，從 main 可以看到 call 了兩個 function，第一個是初始化，第二個 sub_40184c() 是我們要關注的。

    ![](https://i.imgur.com/A5Iqdzl.png)

- sub_40184c() 進來又 call 了兩個 function，先點 sub_401940() 進去看。
    ![](https://i.imgur.com/0aiJyYl.png)

- 可以很明顯的看出他就是在追 PEB 的 LDR 參數，透過 InMemoryOrderModuleList.Flink 這個 linking list 按照記憶體內排序順序去追 import 進來的 module，wcsicmp() 是字串比對，ak 追進去發現是 k 字元，也就是迴圈會比到第一個 BaseDLLName 有 k 的 module，然後 return DLLBase 位址。

    ![](https://i.imgur.com/twMQKrg.png)

    ![](https://i.imgur.com/lP8qiUo.png)

- 用 x64dbg 開啟，在 return 的地方下個斷點，看他回傳值是什麼，從 rax 可以看到是 kernel32.dll，確實是 giveUflag.exe 有 import 的 dll。

    ![](https://i.imgur.com/gkzheL9.png)

    ![](https://i.imgur.com/hzm7hTH.png)

    ![](https://i.imgur.com/UiL0OrN.png)

- 再來回頭看第二個 sub_40153() 做了什麼，重點在 25 行的 for 迴圈，可以看到 a1 是傳入的參數，也就是 kernel32.dll 在記憶體的 DLLBase，做一些操作運算後用 stricmp() 與 sleep 字串比對，目的是尋訪 kernel32.dll export 的 function 找到 sleep() 這個 function 位址然後 break。

    ![](https://i.imgur.com/0aiJyYl.png)

    ![](https://i.imgur.com/D9crvoW.png)

    ![](https://i.imgur.com/gFg0In3.png)

- 實際在符號地方點選 kernel32.dll，搜尋 sllep 字串也確實能找到這個 export function。

    ![](https://i.imgur.com/nMwrrT5.png)

- 所以也可以設個斷點在字串比對的地方，確認看看是不是在找 sleep()，可以從 rcx、rdx 得知確實是在比對 function name，而 rcx 的 function 確實就是 kernel32.dll export 的。

    ![](https://i.imgur.com/TSIAOpW.png)

    ![](https://i.imgur.com/YyvOhvJ.png)

- 接下來就能推測 for 迴圈下面的部分就是 3 個 sleep()，都是傳入一個極大的數字，讓 process 卡在那無法往下執行，下面對 buffer 做 XOR 運算明顯就很可疑，再來就是我們要怎麼想辦法跳過那 3 個卡住的 sleep()，在 puts() 下個斷點，看 buffer 運算後的結果。

    ![](https://i.imgur.com/Mr8ZkHf.png)

- 一開始是想說竟然 sleep() 會卡住，那我把字串比對後的 je 改成 jne 就可以不跑到 sleep 就，程式確實是順利的在第一次迴圈就跳出去，卻在呼叫 function call 的時候觸發 Exclusive 終止程式了，想也是，別的 function 用法本來就不一定會是傳 int 參數的。

    ![](https://i.imgur.com/FPWIjy0.png)

- 竟然不能動 function ptr 的部分，就改傳入的時間參數就好啦，通通改成 0x01，讓 sleep() 只都只停 1 毫秒， F9 執行下去就能看到 buffer 變動後的 FLAG 拉！

    ![](https://i.imgur.com/U3SbRKC.png)

    ![](https://i.imgur.com/VuZBLiB.png)

## FLAG 截圖

![](https://i.imgur.com/gh49WIc.png)
