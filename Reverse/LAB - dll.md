# dll
###### tags: `Reverse`
## 解題流程
- 首先用 ida 開啟 usedll.exe，再 main function F5 反編譯，點進 sub_401690 沒什麼東西，初始化一個靜態變數為 1，就 return 了，應該是再做一個初始化。

- 再來配置了一個 0x1000 長度的 buffer，print 出 main function 的位址，然後讀取使用者輸入，可以看到是讀取兩個數字(%d)，所以這邊是要傳入兩個參數，然後把這兩個參數丟到 YoDllFunc1()，最後印出 buffer 內容。

    ![](https://i.imgur.com/vvAPO1S.png)

- 追看看 YoDllFunc1()，發現他是外部引入函式。

    ![](https://i.imgur.com/WRAHs0d.png)

- 去 import 看一下可以得知是從 hiDll 引入的。

    ![](https://i.imgur.com/vEOBSmN.png)

- 用 ida 打開 hiDll，從 export 找到 YoDllFunc1()，追進去看可以看到，他對比對第一個傳入數字，符合就在 buffer 第1個 element 存 70 (F) 再做 YoDllFunc2()，做完再對第 18~24 的 element 做一些運算操作 。
- 
    ![](https://i.imgur.com/74U5YXy.png)
    
    ![](https://i.imgur.com/AftFO9N.png)
    
- 那我們就需要再追進去 YoDllFunc2() 看看他做了些什麼，一樣是個外部引入函式，從 otherDll 引入的。

    ![](https://i.imgur.com/rdiy42a.png)

- 一樣用 ida 打開 otherDll，從 export 找到 YoDllFunc2()，追進去看可以看到在做的事情跟 YoDllFunc1() 是差不多的。

    ![](https://i.imgur.com/KngwNey.png)

- 所以整體流程應該是要輸入兩個數字，經由 YoDllFunc1() 跟 YoDllFunc2() 比對成功後，去修改 buffer 的值，那我們就用 x64dbg 開啟 usedll.exe ，在一開始 main 的 puts(buffer) 下個斷點，也可以同步在 YoDllFunc1() 跟 YoDllFunc2() 下斷點看有沒有確實進入和比對數字狀況。
- 
    ![](https://i.imgur.com/dj6mBYb.png)

- F9 執行到要輸入參數的地方，輸入在 YoDllFunc1() 跟 YoDllFunc2() 看到的數字，F9 執行到 puts(buffer) 的斷點，就可以看到 FLAG 了！

    ![](https://i.imgur.com/vKvaF3I.png)
    
    ![](https://i.imgur.com/AP4ZHnY.png)
    
## FLAG 截圖

![](https://i.imgur.com/K5EZZfI.png)
