# heapmath
###### tags: `Pwn`
## 解題流程

- 一起來快樂的計算 Heapmath，我解了 1 小時 : )
- 首先要找出各種 malloc size 的 chunk free 之後在 tcache 存的 linking list 順序，總之就是以 stack 形式放在 malloc size - 8(下一個 chunck 前 8 byte 可以放) + 10(chunk header)，並對齊 0x10 的 chunk list。 

    ![](https://i.imgur.com/Y5vEdib.png)

- 在來是要計算 chunk 的 address，這裡就看上面 B ~ C 中間宣告的 chunck 實際 malloc 了多少空間，記得也是 malloc size - 8 + 10 對齊 0x10，然後全部加起來在加到 B 的位址上就可以了。

    ![](https://i.imgur.com/PiOkN4y.png)

- 下一題是要計算 index 下的值，這題很簡單，X malloc 0x40，Y 也 malloc 0x40，且 type 都是 unsigned long，x86-64 下一個 unsigned long 8 byte，所以 X、Y 各是長度 10 的陣列，Y 在 X 之後宣告，位址是連續下去的，所以沒毛病，X 在 X\[9\] 結束，Y 從 X\[10\]開始。

    ![](https://i.imgur.com/iXCjkJB.png)

- 接著是計算 tcahe fd，可以看到先 free(X) 再 free(Y)，所以 chunk list 是 Y --> X，Y的 fd 會是指向 X 的 data 段位址，所以是 Y 的 data 段 - 0x10(header) - 0x40(x data 段 size)。

    ![](https://i.imgur.com/XWkaBsz.png)

- 接著是計算 fastbin fd 算法就不同，chunk list 一樣是 Y --> X，但 Y 的 fd 會是指向 X 的 header 段位址，所以是 Y 的 data 段 - 0x10(header) - 0x50(x data 段 size + 0x10 header)。
- 答對就看到 FLAG 拉！
 
    ![](https://i.imgur.com/nSFqaMb.png)

## FLAG 截圖
![](https://i.imgur.com/nSFqaMb.png)