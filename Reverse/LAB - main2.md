# main2
###### tags: `Reverse`
## 解題流程

這題跟 main1 一樣是藏了 function 在 init array，不過因為事 dll 檔，init array 名字看起來不一樣，用一樣的方式找到 init array，可以看到有一個 RunFunc1，因該就是使用者自己加的 function 沒錯。

![](https://i.imgur.com/xd40NB5.png)

點進去看看，F5 反編譯後可以看到不同於 main1，這個 function 是有傳入 a1 及 a2 兩個參數的。
![](https://i.imgur.com/bS9IEQL.png)

然後其實 init function 是可以吃 argc 跟 argv 的，又看到有個 if a1==2，合理判斷一個是 a1 是 argc，a2 是 argv，按 y 分別重新定義一下型別跟命名。

![](https://i.imgur.com/XSJtybh.png)

可以看到 argv 第一個參數會做 atoi 再除 17，等於 25734 的話就拿 src 迴圈做 XOR，但這邊 src 不是字串，長度 21 也不知道哪裡來的，不如我們就試試用 python 算 argv\[1\] 給多少除 17 會等於 25734，讓程式執行起來看看，沒想到就看到 FLAG 了！
```python=
    print(25734*17)
```
![](https://i.imgur.com/TJoQGs6.png)

## FLAG 截圖

![](https://i.imgur.com/TJoQGs6.png)