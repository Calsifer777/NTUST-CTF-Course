# LAB - POA
###### tags: `資安` `crypto`

## 題目解析
## 題目解析
題目給了一個 server.py，是一個 CBC mode 的 AES 解密系統，傳密文他會幫你解密，但只能得知解密後的 padding 符不符合格式，符合就返回 YES，不符合就返回 NO，padding 格式是明文後補 0x80 再補 0x00 到 16 的倍數，unpad 就是檢查到有 0x80，其後面的 bit 是不是都 0x00，如果是就符合格式。
```python=0
def pad(data):
    padlen = 16 - len(data) % 16
    return data + int('1' + '0' * (padlen * 8 - 1), 2).to_bytes(padlen, 'big')
    
def unpad(data):
    for i in range(len(data) - 1, len(data) - 1 - 16, -1):
        if data[i] == 0x80:
            return data[:i]
        elif data[i] != 0x00:
            raise PaddingError
    raise PaddingError

def encrypt(plain):
    # random iv
    iv = os.urandom(16)
    # encrypt
    aes = AES.new(key, AES.MODE_CBC, iv) 
    cipher = aes.encrypt(pad(plain))
    return iv + cipher

def decrypt(cipher):
    # split iv, cipher
    iv, cipher = cipher[:16], cipher[16:]
    # decrypt
    aes = AES.new(key, AES.MODE_CBC, iv) 
    plain = aes.decrypt(cipher)
    return unpad(plain)
    
def main():
    print(f'cipher = {encrypt(flag).hex()}')
    while True:
        try:
            decrypt(bytes.fromhex(input('cipher = ')))
            print('YESSSSSSSS')
        except PaddingError:
            print('NOOOOOOOOO')
        except:
            return
```
另外提供了 remote server 可以戳，戳一下可以看到返回的東西，IV (前16 byte) 跟密文。
```
b'\xdb\x8a^\xd7PQ\x1d\xb3\\\x9c\xed\x91\xb9\xd8\t\xd89\xf6\x92\x91%\xa2]\x85\xbc\xa8\xc2-\xdc\x94M\x89\x7f@\xac\x1c\xdf\x91<\x0ehm\\l\xcf\x7fN\x8e'
```

因此我們要做的就是透過 POA(padding oracle attack) 來推出中間值，再跟提供的 IV 做 XOR 就可以推回明文了。

## 解題流程
密文只有 32 byte (48-16) 所以是兩個 block 而已，對 block 裡的 16 byte，我們一個一個爆搜，每個試 0~255 (2^8 一個 byte)，先 XOR 0x80 這樣若返回 yes 我們的 j 就會直接是我們的中間值。

把我們的爆搜 IV += block 密文，丟回 server 他就會幫我們解密，若為 yes，就把 j 記下，做完整個 block (16 byte) 後會得到完整的中間值，再拿來跟 IV 做 XOR，就會得到該 block 的密文。
```python=0
from pwn import *

r = remote("edu-ctf.csie.org", 42070)
q = r.recvline()
cipher = bytes.fromhex(q.split(b' ')[-1][:-1].decode())
print(cipher)

def xor(a, b):
    return bytes(i^j for i, j in zip(a, b))

for block in range(2):
    ok = b''
    for i in range(16):
        for j in range(256):
            c = b'\x00'*(15-i)+bytes([(j^0x80)])
            if i == 15:
                c = b'\x00'*16 + c
            c += ok
            c += cipher[16*block + 16:16 * block +32]
            p = c.hex().encode()
            r.sendlineafter(b'cipher = ', p)
            q = r.recvline()
            if q != b'NOOOOOOOOO\n':
                ok = bytes([j])+ok
                print(xor(cipher[:16*block+16][-i-1:], ok))
                break

r.interactive()
```
## FLAG 截圖
![](https://i.imgur.com/iazzkzF.png)

