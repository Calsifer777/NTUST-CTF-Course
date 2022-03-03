# LAB - Pohel
###### tags: `資安` `crypto`

## 題目解析
這一題是簡單的對數加密，g 是 2，先找質數 n，然後找一個 n-1 以下的 k 做 mod，計算 $g^k$ $mod(n)$，題目有個 output 給了 g, n, c 跟 encrypt 的 FLAG。
```python=0
#!/bin/env python3

import gmpy2
import random
import hashlib


def gen():
    while True:
        p = [gmpy2.next_prime(random.randrange(1<<32)) for _ in range(16)]
        o = 2
        for pi in p:
            o *= pi
        n = o + 1
        if gmpy2.is_prime(n):
            g = 2
            if pow(g, o // 2, n) == n - 1:
                return g, n


def main():
    g, n = gen()
    k = random.randrange(n - 1)
    c = pow(g, k, n)

    with open('flag.txt', 'rb') as f:
        flag = f.read()
    k = hashlib.sha512(str(k).encode('ascii')).digest()
    enc = bytes(ci ^ ki for ci, ki in zip(flag.ljust(len(k), b'\0'), k))

    print('g =', g)
    print('n =', n)
    print('c =', c)
    print('flag =', enc.hex())


if __name__ == '__main__':
    main()

```
## 解題流程
這題很明顯就是要你做 DLP 算出模 p 的對數加密的 k，這邊先對 n-1 做 factor() 找出因數，再來就是做 bsgs 枚舉因數的所有對數可能，然後用 crt(中國剩餘定理)最後根據不同因數的同餘推出 k。
```python=0
f = [fi for fi in f]
x = []
for fi in f:
    gi = pow(g, (n-1)//fi, n)
    hi = pow(c, (n-1)//fi, n)
    xi = bsgs(gi, hi, (0, fi))
    x.append(xi)

x = crt(x, f)
print(x)
pow(g, x, n) == c
```
之後就是簡單的 XOR 加密，因此再拿 k 過 Hash 後跟 encrypt FLAG 再做一次 XOR 就可以看到 FLAG 了。
```python=0
k = hashlib.sha512(str(x).encode('ascii')).digest()
flag_cracked = bytes(ki ^^ ci for ki, ci in zip(k, bytes.fromhex(flag)))
flag_cracked
```
## FLAG截圖
![](https://i.imgur.com/Tp0BNrq.png)
