# LAB - HNP

###### tags: `資安` `crypto`

## 題目解析
這題在做的是橢圓曲線的數位簽章加解密，題目給了 server source code 跟 remote server。

可以看到機會有 3 次，option 有 3 個，exit 不管他，1 是給他 msg 做數位簽章，但不能輸入 "Kuruwa"，2 是做數位簽章解密，如果簽章認證成功且 msg 是 "Kuruwa" 就給你 FLAG。

```python=0
#!/usr/bin/env python3
from random import randint
from Crypto.Util.number import *
from hashlib import sha256
from ecdsa import SECP256k1
from ecdsa.ecdsa import Public_key, Private_key, Signature

FLAG = open("flag", 'r').read()

E = SECP256k1
G, n = E.generator, E.order

d = randint(1, n)
k = randint(1, n)
pubkey = Public_key(G, d*G)
prikey = Private_key(pubkey, d)
print(f'P = ({pubkey.point.x()}, {pubkey.point.y()})')

for _ in range(3):
    print('''
1) talk to Kuruwa
2) login
3) exit''')
    option = input()
    if option == '1':
        msg = input('Who are you?\n')
        if msg == 'Kuruwa':
                print('No you are not...')
        else:
            h = sha256(msg.encode()).digest()
            k = k * 1337 % n
            sig = prikey.sign(bytes_to_long(h), k)
            print(f'sig = ({sig.r}, {sig.s})')

    elif option == '2':
        msg = input('username: ')
        r = input('r: ')
        s = input('s: ')
        h = bytes_to_long(sha256(msg.encode()).digest())
        verified = pubkey.verifies(h, Signature(int(r), int(s)))
        if verified:
            if msg == 'Kuruwa':
                print(FLAG)
            else:
                print('Bad username')
        else:
            print('Bad signature')
    else:
        break
```
## 解題流程
我們要想辦法取得 "Kuruwa" 簽章的 r, s，但 option 1 加密時又不能輸入 "Kuruwa"，從 source code 看到 k 雖然初始值是 random，但其之後更新的 k 都是做 k*1337 mod(n)，因此我們其實可以根據以下算法，代入兩次簽章加密的 r ,s做聯立，就可以解出 d，如果能解出 d，在確定 k 後只要自行寫一個相同的簽章加密就可以得到 r, s。

簽章加密得到的 s 與 d 的關係式
```
s*k = h + d*r
```
而我們可以先輸入兩次隨便 input，拿到兩次 (r, s)，做兩式聯立的化簡，s, r, h 都已知，k 是倍數 mod 關係，因此可以導出一元一次 d 的關係式。
```
s1k1 = h1 + dr1
s2*1337*k1 = h2 + dr2
s1/1337*s2 = (h1 + dr1) / (h2 + dr2)
s1h2 + d (s1r2) = 1337*s2h1 + d (1137*s2r1)
d = (s1h2 - 1337*s2h1) / ((1337*s2r1) - s1r2)
```
根據 s1k1 = h1 + dr1 順便也可以解出 k1 來去推我們第三次簽章需要的 k3。
```
k1 = (h1 + dr1) / s1
```
如上所述，兩次簽章的 r, s。

![](https://i.imgur.com/UiI4V6a.png)

算出 d 及 k1 的程式碼，h1, h2要特別注意轉成 long。
```python=0
from hashlib import sha256
from ecdsa import SECP256k1
from Crypto.Util.number import *
from ecdsa.ecdsa import Public_key, Private_key, Signature
E = SECP256k1
n = E.order

h1 = bytes_to_long(sha256('1'.encode()).digest())
h2 = bytes_to_long(sha256('2'.encode()).digest())
print(h1)
print(h2)
r1, s1 = (80008438807091165108615174829599971936129753597170016673150452774846238575026, 55697401886876568024796117294543289714754236449455555252622618269228780971163)
r2, s2 = (52838727272077736177613175949315187278950643955622671491134409778359715435970, 42141158621193015262436608548441979216382271186265858380994862471347238445231)
#s1k1 = h1 + dr1
#s2*1337*k1 = h2 + dr2
#s1/1337*s2 = (h1 + dr1) / (h2 + dr2)
#s1h2 + d (s1r2) = 1337*s2h1 + d (1137*s2r1)
#d = (s1h2 - 1337*s2h1) / ((1337*s2r1) - s1r2)

#k1 = (h1 + dr1) / s1
d_ans = (s1*h2 - 1337*s2*h1) * inverse_mod((1337*s2*r1) - s1*r2, n) % n
print(d_ans)
k1 = (h1 + d_ans*r1) * inverse_mod(s1, n)%n
print(k1)
```
有了 d 跟 k1 我們就可以寫出跟 sever 一樣的簽章加密，他的 k 是先乘再加密，我們前面輸入了兩次簽章加密，因此第三次的 k 就是 k1 做兩次 *1337 mode(n)，mag 給 "Kuruwa" 過 Hash 記得轉 byte 就可以產生出 r, s了。
```python=0
G = E.generator
pubkey = Public_key(G, d_ans*G)
prikey = Private_key(pubkey, d_ans)
h = bytes_to_long(sha256("Kuruwa".encode()).digest())
k = k1
for i in range(2):
    k = k * 1337 % n
sig = prikey.sign(h, k)
(sig.r, sig.s)
```
最後拿 r, s 給 server 做 option 2 認證，輸入 "Kuruwa" 就可以拿到 FLAG 拉！

## FLAG截圖
![](https://i.imgur.com/mETKfDo.png)