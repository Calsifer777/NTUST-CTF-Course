# LAB - LSB
###### tags: `資安` `crypto`

## 題目解析
題目給了一個 server 的 source code 跟 remote server，從 source code 可以看到是做 RSA 加密。

輸入欲解密的數字，server 會返回解密後 mod3 的結果，LSB 是做 mod2，這邊做 mod3 的話我們只要把運算也改成 mod3 就沒問題了。

```python=0
import random
from Crypto.Util.number import *

FLAG = open('./flag', 'rb').read()

def pad(data, block_size):
    padlen = block_size - len(data) - 2
    if padlen < 8:
        raise ValueError
    return b'\x00' + bytes([random.randint(1, 255) for _ in range(padlen)]) + b'\x00' + data

def main():
    p = getPrime(512)
    q = getPrime(512)
    n = p * q
    e = 65537
    d = inverse(e, (p - 1) * (q - 1))

    m = bytes_to_long(pad(FLAG, 128))
    c = pow(m, e, n)
    print(f'n = {n}')
    print(f'c = {c}')

    while True:
        c = int(input())
        m = pow(c, d, n)
        print(f'm % 3 = {m % 3}')

try:
    main()
except:
    ...
```

## 解題流程
連上 remote server，取得返回的 n 跟 c，設定好 3 mod n 的反元素。
```python=0
from pwn import *
from Crypto.Util.number import *

r = remote('edu-ctf.csie.org', 42071)

q = r.recvline()
n = int(q[4:])
q = r.recvline()
c = int(q[4:])
e = 65537

a = inverse(3, n)
m = 0
b = 0
i = 0
f = 0
```
每次送 (3^-ie)C 給 server 解密，server 會返回 (3^-i)m modn 再 mod3 的結果，因此我們要的數字就會是 mm = r - a * 3 modn 的反元素 * 前一個算出來的數字，若是第一個數字，這邊就會是 0，mod 出來的 r 直接是我們要的第一個數字，而出來的數字乘回 3^i 次方就可以去掉 3 modn 的反元素，得到明文的各進位數字。

全部加起來直到 mm 連續 10 個 0 以上就知道明文數字已經全部轉換完畢，最後再 long 轉 byte 就可以看到 FLAG 了！
```python=0
from pwn import *
from Crypto.Util.number import *

r = remote('edu-ctf.csie.org', 42071)

q = r.recvline()
n = int(q[4:])
q = r.recvline()
c = int(q[4:])
e = 65537

a = inverse(3, n)
print(a)
print("---------")
print((a * 3)%n)
print("---------")
m = 0
b = 0
i = 0
f = 0

while True:
    # print(str(pow(a, i*e, n)* c % n).encode())
    r.sendline(str(pow(a, i*e, n)* c % n).encode())
    q = r.recvline()
    # print(q)
    mm = (int(q.split()[-1]) - (a * b) % n) % 3
    # print(mm)
    if mm == 0:
        f += 1
        if f == 10:
            break
    else:
        f = 0
    b = (a*b + mm) % n
    m = 3 ** i * mm + m
    i += 1
print(m)
print("FLAG：")
print(long_to_bytes(m))
```
## FLAG 截圖：
![](https://i.imgur.com/NQ8f2k3.png)

