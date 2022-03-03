# FILE note \- \(W\)
###### tags: `Pwn`
## 解題流程
- 這題跟第一題隨意讀得很像，不過不同的是他沒有開檔，而是多 malloc 了一個 debug_secret。
- 而選單部分也多了一個 load_note，且每次做完選單動作都會過一個 \_check_debug_secret 函式，如果 debug_secret 等於後面那個，他就會開啟一個 shell。
- 我們的目的就是要透過隨意寫將 debug_secret 寫成那個字串，拿到 shell。

    ![](https://i.imgur.com/NWLoFsu.png)

    ![](https://i.imgur.com/Bz8AJPP.png)

- 首先我們一樣要先抓到 debug_secret 的位址，用 note_buf 與 debug_secret 相減後可以看到相差 0x30 (記得 0x10 的 header)。
- 所以 debug_secret 會是 note_buf 位址 - 0x30。

    ![](https://i.imgur.com/wFRxrQ3.png)

- 然後一樣透過 case 1、2 觸發 BOF。
- 建構隨意讀的 file structure，這次要關注的是 \_IO_write_ptr 與 \_IO_write_end 的位址(記得 fileno 這次要改 0)。
    ![](https://i.imgur.com/B3SfTVS.png)

    ![](https://i.imgur.com/UtZLt0t.png)

- 最後透過 case 4 load_note() 將 note_buf 裡存好的字串寫進 debug_secret。 
- 腳本跑下去就拿到 shell 了，FLAG 在 /home/filenote_w/flag 裡。
    
## FALG 截圖
![](https://i.imgur.com/MDS6B5L.png)
