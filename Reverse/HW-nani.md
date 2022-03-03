# nani
###### tags: `Reverse`
## 解題流程

- 這題首先將 nani.exe 丟到 DIE 可以發現有加 UPX 殼，版本是 3.96，下載 3.96 的 UPX 去殼後得到原始 exe 檔，可以開始用 ida 逆了。

    ![](https://i.imgur.com/pX25y3q.png)
    
    ![](https://i.imgur.com/dQwraal.png)


- ida 開啟後看一下 main，除了第一個像初始化，啥都看不懂。

    ![](https://i.imgur.com/8q0xu5B.png)
    
- 沒關係，用 X64dbg 開啟，在第二跟第三個 function 下個斷點，觀察一下執行狀況，可以看到輸出了兩個字串，嘲諷了一下我們。

    ![](https://i.imgur.com/YVVixda.png)

- 回到 ida，第二個 function 參數追進去就是 Unpack me 的 string，推測應該就是單純的 print() 了，第三個就比較可疑，看起來除了 print() 外，還將 RIP jump 到某個地方了。
 
    ![](https://i.imgur.com/Le23wCv.png)
    
- jump 的位置追進去可以看到，明明是跳到 0x4019C9，ida 解析的 function 卻從 19C8 開始，看來是 ida 解錯，多將一個前面的 byte 看成指令了，自己按 U，然後在19C9 按 P 從新反組譯成 function。

    ![](https://i.imgur.com/VmX0ZWv.png)

- ok，看起來正常多了，而且看到關鍵字 IsDebuggerPresent()，一種 Anti-Reverse 機制，若檢測到是用 Debugger 開啟，就會執行一些操作，通常是直接 exit() 或檔案自毀。

    ![](https://i.imgur.com/ANcR46C.png)

    ![](https://i.imgur.com/mRBvSdb.png)

- 這是後就要用到 SkyllaHide 拉，一個強大的反各種 Anti-Reverse 機制的外掛，從 github 下載就可以在 x64dbg 上安裝，可以看到能反的 Anti-Reverse 機制之多，這邊就把 BeingDebugged 的部分勾起來，就不怕程式檢查了。
- 
![](https://i.imgur.com/WQl58ys.png)

- 避開 IsDebuggerPresent() 後再往下看，有呼叫一個 sub_4019F1()，點進去還有一層，再點進去可以看到，毫無疑問是對 VM 環境的偵測，一樣是一種 Anti-Reverse 技術，這邊因為我是直接再 主機跑，所以不會有被偵測為 VM 的問題，下面 for 迴圈比對的部分，會比對上面各種 VM 的 cpuid，若比對都沒有，i >= v10，就會進入下一步的 function，若偵測到為 VM 環境，就會 break 輸出字串並結束。

    ![](https://i.imgur.com/L2CStc0.png)
    
    ![](https://i.imgur.com/qi8g7Uv.png)

- 點進 sub_4017DF()看看，其實是看不太懂，不過可以看到 overflow_error，是 exception 的基底函式。

    ![](https://i.imgur.com/nsaEFu9.png)
    
    ![](https://i.imgur.com/xInoluF.png)

- 切換到 x64dbg 下斷點跑跑看，可以看到我們確實沒被檢測為 VM，準備進 sub_4017DF()，但繼續 F9 下去程式卻直接結束，什麼都看不到。

    ![](https://i.imgur.com/lWfqxbk.png)

- F7 進入 function 再 F8 步過執行後，可以確認是執行 sub_4A0D40() 後程式結束，再用相同方式追進去可以發現是裡面的 sub_40C9D0() 觸發 exception，可以推測是由 sub_4017DF() 為進入點觸發的 exception 再搞事。

    ![](https://i.imgur.com/X3Kihyb.png)

- 而 x64 程式追 exception handler 的方式是用 PEbear 開啟 exe，跟隨 Optional Hdr 的 Exception table，然後看程式 exception 觸發的進入點位址，從 ida 看到 sub_4017DF() 的 RVA 是 17DF，再 exception table 中查找包含該位址的 begin、end 區段，在UnwindinfoAddress 右鍵跟隨 RVA。

    ![](https://i.imgur.com/4PsrtYr.png)

- 而 Unwind_INFO 的 structure 紀錄如下，前兩個 byte 是一些資訊，第三跟四個 byte 是後面 code array 的長度，code array 的 UNWIND 資料型態是兩 byte，長度由前面 CountOfCodes 決定，接著 0x00 補滿為 4 的倍數，後面再接 Exception Handler。

![](https://i.imgur.com/L8wEfml.png)

- 所以我們這邊的 Exception Handler 依照結構可以得知 RVA 是 0x16FB。

    ![](https://i.imgur.com/kQa9FlY.png)

- 回到 ida 找我們的 Exception Handler，可以看到 ida pro 很 'pro'，解出了這個 Exception Handler，點進去看到熟悉的字眼 VirtualProtect()，看起來是將 byte_4015AF 之後的那一大坨 data XOR 運算後，用 VirtualProtect() 在記憶體內將該區塊轉成能執行的權限，當作指令來執行。
    ![](https://i.imgur.com/ap7bStb.png)
    
    ![](https://i.imgur.com/EQjsKEu.png)

- 在 0x16FB 設斷點可以看到真的有跑到，在 VirtualProtect() 轉成執行權限的地方 0x15AF 下斷點也能看到有執行到，表示確實被更改權限並準備執行了。

    ![](https://i.imgur.com/q7ezmlf.png)
    
    ![](https://i.imgur.com/4LoOPMc.png)

- 用 F8 不過發現他是個迴圈在做某些事情，看右邊的 RDX 隨著迴圈執行次數可以看到似乎是熟悉的 FLAG 字串順序。
- 
    ![](https://i.imgur.com/AefJmhk.png)

- 在最後 exit() 的地方下個斷點，直接 F9執行下去，就拿到 FLAG 了！

    ![](https://i.imgur.com/DTWQf10.png)
    
## FLAG 截圖
![](https://i.imgur.com/kxc6Sp9.png)
