# market
###### tags: `Pwn`
## 解題流程
- 題目再 free admin 的地方順序上出了問題，導致 admin->secret free 的其實是 tcache 的 整個 structure 的位址。

    ![](https://i.imgur.com/9uOdDvP.png)

- 下面是一些可以對 heap 做操作的 interaction。

    ![](https://i.imgur.com/AgYkrAJ.png)

- 會變成 free 一個 tcache 的 整個 structure，而該 structure 再 free 之後的 key 仍然指向 tcache 的 整個 structure。

- 因此只要我們嘗試拿回指向自己本身的整個 tcache structure，就可以擁有整個 tcache structure。
- 再去做 partial overite，拿回該 structure 下 free 掉在 tcache 的 admin entry data 段塞 16byte 的 A，就會因為中間沒有 null byte，把原 admin->secret 一起印出來，就拿到 FLAG 了！

    ![](https://i.imgur.com/k1T5wea.png)

## FLAG 截圖
![](https://i.imgur.com/Hr8UmLw.png)