# Rop2win
###### tags: `Pwn`
## 解題流程
- 首先一樣來看 sc，可以看到有 seccomp_rule 的 syscall 限制，只能呼叫 RWO(read、write、open) 3 個 syscall。
- 程式有給輸入 filename、ROP buffer，和一個能做到 BOF 的 read()。

    ![](https://i.imgur.com/AFzOekJ.png)

- 很明顯就是要我們用 ROP chain，透過 RWO 讀到 FLAG 檔案。
- 首先我們要找到需要的 ROP gadget 的 address。
- 透過 ROPgadget 工具，找此 binary 可能的 ROP gadget 將輸出導到一檔案，用搜尋找出我們需要的 rdi、rsi、rdx、rax、syscall、leave_ret 的 gadgets 位址，。

    ![](https://i.imgur.com/rumZd5V.png)

    ![](https://i.imgur.com/6igyeuY.png)

- 再來就是用 python 寫一個腳本做 ROP chain，可以看到我們找出所有需要的 ROP gadgets 了。
- 我們先定義好 ROP chain 的整個流程，先是 open('/home/rop2win/flag', 0) 打開 FLAG 檔案。
- 再來因為預設開啟的 fd 0、1、2 分別是 stdin、stdout、stderr，因此可以知道新 open 的 fd 代號應該是 3。
- 再去 read(3, fn, 0x30) 讀出 FLAG 字串，最後再 write(1, fn, 0x30) 來把 FLAG 寫到 stdout。
- 而 send ROP 時前面多加的 'A'\*0x8 就只是塞垃圾，因為 stack 最底放的是 ret addr，但這裏我們不需要再跳到別處。
- 再來就是觸發 ROP，再 BOF 塞 'A'\*0x20，再塞 rbp 跟 ret addr，讓 pop rbp 時 rbp 跳到我們 ROP 的 buuffer，leave ret 時 再 leave ret 一次。

    ![](https://i.imgur.com/uYo0RJm.png)

- 執行下去就可以看到 FLAG 拉~~

## FLAG 截圖
![](https://i.imgur.com/Jsl2DkM.png)
