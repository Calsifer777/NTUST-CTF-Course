# LAB - LAT
###### tags: `資安` `crypto`

## 題目解析
這題是一個 Toy model 的加密方式，他的構造方法雖然跟 Lattices 沒關係，但可以用 Lattices的特性去解。

題目給了 chall.py 和 outpu.txt，chall.py 就是加密的 source code，output.txt 內含 public 對 (q, h) 和 encrypt_flag 的值。

而我們要做的事就是透過 Lattices 的特性去解出密鑰 (f, g)，反向解出 FLAG。
```python=0
from Crypto.Util.number import getPrime, inverse, bytes_to_long
import random
import math

FLAG = b'FLAG{????????????????????}'


def gen_key():
    q = getPrime(512)
    upper_bound = int(math.sqrt(q // 2))
    lower_bound = int(math.sqrt(q // 4))
    f = random.randint(2, upper_bound)
    while True:
        g = random.randint(lower_bound, upper_bound)
        if math.gcd(f, g) == 1:
            break
    h = (inverse(f, q)*g) % q
    return (q, h), (f, g)


def encrypt(q, h, m):
    assert m < int(math.sqrt(q // 2))
    r = random.randint(2, int(math.sqrt(q // 2)))
    e = (r*h + m) % q
    return e


def decrypt(q, h, f, g, e):
    a = (f*e) % q
    m = (a*inverse(f, g)) % g
    return m


public, private = gen_key()
q, h = public
f, g = private

m = bytes_to_long(FLAG)
e = encrypt(q, h, m)

print(f'Public key: {(q,h)}')
print(f'Encrypted Flag: {e}')

```
## 解題流程
首先將 Fh = G + qR 以兩個 vector 加法表示成 F(1, h)-R(0, q) =，(F, G)，因此創建 v1 和 v2，然後兩邊做高斯消去 (Gaussian Lattices Reduction)，如果 V2 比 V1 還短就交換，算 m = round(v1*v2/(v1*v1))，如果 m==0 代表不用繼續做，如果不是，就把 v2 = v2 - m * v1，這樣就可以得到兩個最短的 vector，而這兩個 vector 其實就是密鑰，解出來有可能是負的，但我們要的是距離所以正負沒差。

有私鑰後進行解密，算完是一個 long，轉成 byte 後就可以看到 FLAG 了！
```python=0
q, h = 0, 0
encrypt_flag = 8989708726503367404715754689730782388959095528339234074784589050444870239608140046626770182336762701788963731736424741343914003600890982230995409715448250
with open("output.txt", "r") as f:
    tmp = f.read()[13:-1].split("\n")
    q = int(tmp[0].split(",")[0])
    h = int(tmp[0].split(",")[1][:-1])

v1 = vector((1, h))
v2 = vector((0, q))

while True:
    if v2.norm() < v1.norm():
        v1, v2 = v2, v1
    m = round(v1*v2/(v1*v1))
    if m == 0:
        break
    v2 = v2 - m * v1
f,g = v1

flag = long_to_bytes(decrypt(q, h, f, g, encrypt_flag))
print(flag)
```
## FLAG截圖
![](https://i.imgur.com/TBRhLjx.png)
