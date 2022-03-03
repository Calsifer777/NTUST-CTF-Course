# Cpp_Traditional_GUI
###### tags: `Reverse`
## 解題流程

這題主要考點是如何在 code 整體龐大的情況下快速去抓到重點 code 做 reverse，快速了解整個程式在幹嘛。

首先執行檔案，看到彈出視窗，按下確認後進入到要求輸入帳號密碼的 GUI，得知這是一隻 MFC(Microsoft Foundation Class) 撰寫的桌面應用程式。

![](https://i.imgur.com/KmKszgM.png)

![](https://i.imgur.com/dyssAsy.png)

### 找尋重點 code (registerClass)
- 我們知道 MFC 會 call registerClass 家族 API 來註冊一個 window 對象，所以可以用 ida 打開在 import 搜尋一下有沒有 registerClass 相關家族 API，可以看到有一個 RegisterClassExW 的 API 被引入，點擊可以在 data 看到外部函數地址，再按 x 查看引用，直接導到再 main 對應 call 的地方。

    ![](https://i.imgur.com/vfIKVqX.png)

    ![](https://i.imgur.com/SBVp7mg.png)

    ![](https://i.imgur.com/CU5Dx8g.png)

- 按空白鍵查看 call RegisterClassExW 的指令記憶體位址是 base + 0x10E0，再用 X64dbg 開啟檔案後在相同地方設下斷點，將程式 run 下去，觀察呼叫 registerClass 的記憶體狀態。

    ![](https://i.imgur.com/LJ7YTT4.png)

    ![](https://i.imgur.com/pOTWXCy.png)


- 查一下 RegisterClassExW msdn，可以看到傳入 RegisterClassExW function 只有一個參數，是個指向 WNDCLASS structure 的 ptr。

    ![](https://i.imgur.com/hTGiCYF.png)

- WNDCLASS structure 就是要註冊的 window 的 structure，內容參數長這樣，我們要關注的是第二個 WNDPROC 的回調函式，程式就是在這邊定義使用者對 window 的各種操作觸發何種事件的。

    ![](https://i.imgur.com/aami5jc.png)

### 找尋 WNDPROC 回調函式
- 所以如果我們能找到 WNDPROC 回調函式位址，透過 ida 反編譯應該就能看懂 window 啟動後與使用者的交互。

- 我們回到 x64dbg，剛剛前面對 RegisterClassExW 下斷點執行後的記憶體狀態。

    ![](https://i.imgur.com/pOTWXCy.png)

- 第一個參數應該就是指向 WNDCLASS structure 的 ptr，在資料視窗 ctrll + G 輸入 rcx，跳到 rcx 指向的位址，WNDCLASS structure 第一個參數是 UINT style，因此我們往後從第 9 個 byte開始看，應該就是指向 WNDPROC 回調函式的位址的 ptr (注意是 Little Endian)。

    ![](https://i.imgur.com/JrKocVG.png)

- 選 8 byte 起來右鍵 在反組譯視窗中跟隨 QWORD (若只是點選 在反組譯視窗中跟隨，會直接跳到第一個 byte 儲存位址，而不會把整個 QWORD 當 ptr 跳過去)，就可以看到 WNDPROC 回調函式的起始位址了。

    ![](https://i.imgur.com/ePyx7zp.png)

- 減掉 base 後回到 ida 找對應位址就能找到 WNDPROC 回調函式，F5 反編譯一下，看到一大坨 code，往下滑看到 switch()，switch 裡有 DestroyWindow API (按下 X 事件) 跟 GetWindowTextW (取的 dialog text)，可以推測這邊應該就是判斷使用者的哪種操作觸發什麼事件。

    ![](https://i.imgur.com/7BPFOJh.png)

- 兩個 GetWindowTextW 應該就分別是取帳號跟密碼沒錯了，下個斷點測看看，GetWindowTextW 的第二個參數是 lpaddress，一樣在資料視窗 ctrl + G 跟隨 rdx，F8 步過看一下 GetWindowTextW 執行完後是不是真的有寫入帳號密碼。

    ![](https://i.imgur.com/0owVCUy.png)

    ![](https://i.imgur.com/QOZfLVJ.png)

- 能看到確實是有的。

    ![](https://i.imgur.com/QoNcGQY.png)


- 再下面 while 就是判斷帳號密碼正確性的判斷。

    ![](https://i.imgur.com/YmGg7Ni.png)

- 追進 word_140014720 跟 aToyz2k7e3 就可以知道正確帳號密碼了，word_140014720 是 utf-8 的 data，ida 好像沒辦法解析，可以自行轉換，或其實也可以在 x64dbg 看一下就知道了，x64dbg 能自動轉換出來。

    ![](https://i.imgur.com/sjsv8hB.png)

- 最後就是輸入正確帳號密碼就能拿到 FLAG 拉！

## FLAG 截圖
![](https://i.imgur.com/0yE0G09.png)
