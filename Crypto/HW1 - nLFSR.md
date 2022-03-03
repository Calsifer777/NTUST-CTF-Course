# HW1 - nLFSR
###### tags: `資安` `crypto`

## 題目解析
題目給了一個 server.py 的 source code，還有一 remote server，錢錢從 1.2 開始，每次輸入會跟一個 nLFSR 輸出做比對，相同 +0.02，不同 -0.04，錢錢大於 2.4 你就可以拿到 FLAG，就是這麼簡單！(騙誰，一點都不簡單QQ)

先來看一下他的 y 是怎麼取的，random() 跑了 step() 42次，step() 就是 nLFSR 步驟，XOR poly 那邊可以看成是 state 每次在對應位置上做 bit XOR out，所以我們其實可以模擬這個動作，這樣實際上是告訴我們每次可以拿到 某些 state bit XOR 起來的值。

```python=0
import os


state = int.from_bytes(os.urandom(8), 'little')
poly = 0xaa0d3a677e1be0bf
def step():
    global state
    out = state & 1
    state >>= 1
    if out:
        state ^= poly
    return out
    

def random():
    for _ in range(42):
        step()
    return step()


money = 1.2
while money > 0:
    y = random()
    x = int(input('> '))
    if x == y:
        money += 0.02
    else:
        money -= 0.04
    print(money)
    if money > 2.4:
        print("Here's your flag:")
        with open('./flag.txt') as f:
            print(f.read())
        exit(0)
print('E( G_G)')

```

## 解題流程
因為state 是 8 byte 的 random，我們想要還原的話，可以生成 8x8 的 companion 矩陣去推，所以我們要先想辦法取得 64 bit output。

由於 sage 的 pwntool 我一直匯不進來，Discord 群提的方法好像也是 linux 環境下才有用的，剛好看到有人家寫好的 remote class 就直接拿來用了。

get_result() 是用來拿 64 bit output 的函式，每次丟 0 到 srever，看他吐的錢比前一次多還少，來判斷實際 output 是多少，在送的時候發現 server 對訊息的接收次數有限制，只能送 33 次，超過會直接中止連線，因此嘗試了幾次，後來發現以'\n'分隔，一次就傳 64 個 input 就可以了！
```python=0
class remote:
    def __init__(self, host, port):
        self.s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.s.connect((host, port))
        self.buffer = b''
    def recvuntil(self, text):
        text = self._convert_to_bytes(text)
        while text not in self.buffer:
            self.buffer += self.s.recv(1024)
        index = self.buffer.find(text) + len(text)
        result, self.buffer = self.buffer[:index], self.buffer[index:]
        return result
    def recvline(self):
        return self.recvuntil(b'\n')
    def recvlines(self, n):
        lines = []
        for _ in range(n):
            lines.append(self.recvline())
        return lines
    def _convert_to_bytes(self, text):
        if type(text) is not bytes:
            text = str(text)
        if type(text) is str:
            text = text.encode()
        return text
    def send(self, text):
        text = self._convert_to_bytes(text)
        self.s.sendall(text)
    def sendline(self, text):
        text = self._convert_to_bytes(text)
        self.send(text + b'\n')
    def sendafter(self, prefix, text):
        self.recvuntil(prefix)
        self.send(text)
    def sendlineafter(self, prefix, text):
        self.recvuntil(prefix)
        self.sendline(text)
        
def get_result():
    global money
    global result_64
    new_money = float(r.recvline()[2:].decode('ascii'))
#     print(new_money)
    if new_money > money:
        result_64.append(0)
    else:
        result_64.append(1)
    money = new_money
    
result_64 = []
poly = 0xaa0d3a677e1be0bf
F.<x> = GF(2^64)
r = remote('edu-ctf.csie.org', 42069)
money = 1.2
x = r.send(b'0\n'*64)
for i in range(64):
    get_result()
print(money)
print(result_64)
```
存好 64 bit 的 output 後，再來要生單次 step 的 companion 矩陣，companion 的組成是 poly 配上右上的單位矩陣，poly 依序放在每個 row 的第一個元素。

老實說我數學偏差，想了很久，也上網查很多證明，但都一知半解，也不太確定為啥 companion 是這樣放，只知道 companion 最終的格式是這樣，上課助教有介紹，我也就照著上課的格式生出了 companion 矩陣。
```python=0
def matrix_repr_step():  #step companion 矩陣
    global poly
    poly_vetor = vector(F.fetch_int(poly))
    step = []
    for i in range(64):
        row = [ poly_vetor[i] ] + [0] * 63
        if i < 63:
            row[i+1] = 1
        step.append(row)
    step = Matrix(GF(2), step)
    return step
```
有了 companion 矩陣基本就完成大半了，再來就是模擬前面 server 生出 64 bit output 的過程，因先 out 後再做 shift & xor，所以第一次的 state 在 out 時實際上只改變了 42 次，而該次 step 的 XOR 是影響下次的 out，因此除了第一次是 42 次方外，後面都是 43 次方。

建立一個 lsb 的 vector，用來跟 companion 矩陣相乘，每次更新次方數，如次就可以得到經過 64 次輸出的 left significant 的 companion 矩陣，在接著做 solve_right (companion * initial_state = Guess) 就解出 initial_state 了。
```python=0
def get_initial_state():
    step = step = matrix_repr_step()
    first = step ^ 42 #因先 out 後再做 shift & xor，所以第一次是 42 次
    other = step ^ 43 #之後是43
    lsb = vector([1] + [0] * 63).row() # left significant bit
    guess = result_64
    companion = []
    for i in guess:
        companion.append((lsb * first).list()) #得到經過 64 次輸出的 left significant 的 companion 矩陣
        first *= other
    #companion * initial state = guess
    guess = vector(guess)
    companion = Matrix(GF(2), companion)
    initial_state = companion.solve_right(guess)
    return initial_state
```
有 initial_state 後，就只要做 64 次 random()，接下來的 random() 結果就會是我們要的正確的 output，不過這時候解出來的 initial_state 是 binary vector 形式，而我們要做 server 的 nLFSR 需要把他轉成 int 形式。

ranom() 64 次後再 random() 100 次，並把每次結果用'\n'接起來傳到 server，就可以看到 FLAG 了！

```python=0
initial_state = get_initial_state()
print(initial_state)
state = ZZ(list(initial_state), base=2)
# state = a
solve = ""
poly = 0xaa0d3a677e1be0bf
def step():
    global state
    out = state & 1
    state >>= 1
    if out:
        state ^^= poly
    return out
    

def random():
    for _ in range(42):
        step()
    return step()

for i in range(64):
    random()
for i in range(100):
    solve += str(random()) + '\n'
print(solve.encode())

r.send(solve.encode())
for i in range(100):
    result = r.recvline().decode('ascii')
    print(result)
    if "FLAG" in result:
        break
```
## FLAG 截圖

![](https://i.imgur.com/ZYMiwBW.png)

