# FILE note \- \(X\)
###### tags: `Pwn`
## 解題流程
- 這題跟前面 FILE note \- \(X\) 很像，不過這次沒有 \_check_debug_secret 函式可以讓我們直接拿到 shell，而是要透過修改 file structure vtable 內的 function ptr 到 execve plt 的 one_gadget，來拿到 shell。

- sc 可以看到這次 print 的不是 note_buf 的地址，而是 print 這個 function ptr。

    ![](https://i.imgur.com/Ht7mVOG.png)

- 因此首先我們可以用 ELF 讀遠端 server 目錄下的 libc.so.6，找出 printf 在 lib 中的 offset 來算出 libc 的 base。

    ![](https://i.imgur.com/aq0QEx5.png)

- 算出 libc base 後，也用一樣方式算出 \_IO_file_jumps 的 base。
- 透過前面隨意讀的 BOF 搭配 file structure，將 vtable 在 file 讀寫時會呼叫到的 \IO_file_jumps function 的 got 改寫成 execve 的 plt(one_gadget)。
- 這樣就可以拿到 FLAG 了！
- 理論上是這樣啦，我看程式碼跟讀檔的部分也是沒問題，但腳本 run 下去他卻說找不到 libc.so.6 這個檔案，不知道是不是講師拿掉了，所以我最終沒拿到 FLAG QQ。