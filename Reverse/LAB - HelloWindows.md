# HelloWindows
###### tags: `Reverse`
## 解題流程

- 首先用 ida 開啟，透過在 funcion windows 搜尋 main 找到 main function。

    ![](https://i.imgur.com/vQ0LtC5.png)

- F5 反編譯後可以看到v6 與 v7 是連續位址，大小也都一致，應該是一個長度 19 的陣列，ida 解析錯誤，另外還看到關鍵 api VirtualProtect，一個傳入 virtual address 來修改記憶體中 process access 權限的 api。

    ![](https://i.imgur.com/qfjUe3e.png)

- VirtualProtect 傳入的前三參數分別是 lpaddress、size、protect option，protect option 0x40 代表的權限是 rwx，所以整個 VirtualProtect 在做的是將 lpaddres [rsp+30h] 開始 + 0x1000 的區段操作權限變為 rwx。

    ![](https://i.imgur.com/N9yEU4B.png)

    ![](https://i.imgur.com/H1s2A3J.png)

- 然後可以看到再下一步將 unk_140024A80 長度為 0xC9 的 data copy 至 memory 的 lpaddress 位址，等待將 lpaddress 當成 function 呼叫執行。

    ![](https://i.imgur.com/XylEFHt.png)

- 我們可以用 x64dbg 動態 debugger 在 call VirtualProtect 的地方設斷點，觀察程式運行，可以看到 VirtualProtect 傳入的第一個參數是在 rcx，指向 0x22D1C0BAFD0 這個位址。

    ![](https://i.imgur.com/s5cMu0Y.png)

    ![](https://i.imgur.com/vFg0UWI.png)

- 在記憶體映射可以找到這個位址的 page，F8 步過執行 VirtualProtect 後可以看到後面有再 call lpaddress [rsp+30]，當成 fuction 呼叫執行。

    ![](https://i.imgur.com/bCjMwdn.png)

    ![](https://i.imgur.com/0ODkeO8.png)

- F7 進去可以看到是執行到 AFD0，data 也確實被覆寫了 ( 這邊有重跑過 base 不一樣，所以前半部分位址沒對上 )。

    ![](https://i.imgur.com/mcOe9JZ.png)

- 回 ida 看看用來複寫的 unk_140024A80 data，已經知道這邊 data 覆蓋到 lpaddress 上當成 function 呼叫，因此按 C 反組譯再按 P 看能不能變成 function，。

    ![](https://i.imgur.com/bpQxmxn.png)

    ![](https://i.imgur.com/tZDhSDt.png)

- 成功反組譯成 function， 再 F5 反編譯看看，可以看到 v8 是個字串，傳進 a1 當參數，a1 往回追，就是v10 (printf 的 func ptr)。

    ![](https://i.imgur.com/SYtxJui.png)

    ![](https://i.imgur.com/MUfLUFX.png)

- 所以可以知道程式執行起來的 key: 是怎麼來的，下一個 a2(v6, %d) 應該就是 scanf 取值了。

    ![](https://i.imgur.com/zb5yklP.png)

- 從 lpaddress 轉成的 function 可以看到有一個 for 迴圈，這邊 ida 應該是解析錯誤，應該只是單存 i 從 1 到 19 的 for 迴圈，回追 a4 可以看到傳入參數是 19，而 a3 跟 a5 往回追分別是前面的 v6 跟 aNotFLAGXxxxxxx 這個 symbol 的字串，下面都是指標寫法，其實是  a3 指向的位址 + 4 * i 個 byte，也就是尋訪 a3 陣列，每次 -= scanf() 傳入的 int，再下一行也是尋訪 a5 陣列，然後 XOR 前面每次減完的 a3\[i\]，所以應該要正確的再 scanf() 的時候輸入某個 int key，理論上就可以 XOR 出正確的 FLAG 了！

    ![](https://i.imgur.com/YnRQTZ6.png)

    ![](https://i.imgur.com/myx1uvN.png)

    ![](https://i.imgur.com/iR7dpPm.png)

- v6 的第一個數字是 675767066 ，aNotFLAGXxxxxxx 的第一個字是 N(78)，XOR 完要變成F(70)，要 XOR 8，稍微計算一下 (675767066 - 8) = 675767058，這就是 key 值。

    ![](https://i.imgur.com/YnRQTZ6.png)

- 先在最後 printf aNotFLAGXxxxxxx 的地方下斷點，不然他印完就直接結束了，在 call lpaddress 的斷點 F7 進去再自動 F9 執行 scanf() 等待，輸入 key 值 675767058，enter 後程式會停在 printf aNotFLAGXxxxxxx 的斷點，就可以看到成功被 XOR 還原的 FLAG 了！

    ![](https://i.imgur.com/gOBFisB.png)

## FLAG 截圖

![](https://i.imgur.com/VZtsLeE.png)

![](https://i.imgur.com/696yKde.png)


