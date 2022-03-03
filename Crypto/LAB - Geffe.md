# LAB - Geffe
###### tags: `資安` `crypto`

## 題目解析
題目有給一個 correlation.py 和 output.txt。

correlation.py 是加密的 Source code，output.txt 是在 server上先跑過一次的輸出，前面的一坨 binary list 是 key 做 nLFSR 後產生的 256 bit的 stream，IV 是 AES 加密的初始向量，encrypted_flag 是以相同 key 對 FLAG 做 AES 加密後的密文。
```pyhton=0
[0, 1, 1, 0, 0, 0, 1, 1, 0, 1, 1, 0, 0, 0, 0, 0, 0, 1, 1, 1, 0, 0, 1, 1, 0, 0, 1, 0, 1, 1, 0, 0, 0, 1, 1, 1, 0, 1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 1, 1, 0, 1, 1, 0, 1, 1, 1, 0, 0, 1, 0, 0, 1, 0, 1, 1, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 0, 1, 0, 1, 0, 0, 1, 1, 1, 0, 0, 1, 0, 0, 1, 1, 1, 1, 0, 1, 0, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 1, 0, 0, 1, 1, 0, 1, 1, 0, 1, 0, 0, 1, 1, 1, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 0, 1, 1, 0, 0, 0, 1, 0, 0, 0, 1, 1, 0, 0, 0, 1, 0, 0, 1, 0, 1, 1, 1, 1, 0, 0, 0, 0, 1, 1, 0, 0, 0, 1, 0, 0, 1, 0, 0, 1, 1, 0, 0, 1, 1, 1, 0, 0, 0, 0, 1, 0, 1, 0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 0, 0, 1, 1, 1, 1, 1, 0, 0, 1, 1, 0, 0, 1, 0, 1, 1, 0, 0, 0, 1, 0, 1, 1, 0, 0, 1, 1, 0, 0, 1, 0, 1, 0, 1, 1, 1, 1, 0, 1]
{'iv': 'cd2832f408d1d973be28b66b133a0b5f', 'encrypted_flag': '1e3c272c4d9693580659218739e9adace2c5daf98062cf892cf6a9d0fc465671f8cd70a139b384836637c131217643c1'}
```
因此我們要做的就是想辦法從 256 bit 的 stream 回推出 key，有了 key 再做 AES 解密就能拿到 FLAG。

## 解題流程
從 correlation.py 中可以看出 nLFSR 的輸出是由 3 個 LFSR 控制的，key 長度是 69，3 個 LFSR 的 state 就是 key 分區段丟進去。
```python=0
class Geffe:
    def __init__(self, key):
        assert key.bit_length() <= 19 + 23 + 27 # shard up 69+ bit key for 3 separate lfsrs
        key = [int(i) for i in list("{:069b}".format(key))] # convert int to list of bits
        self.LFSR = [
            LFSR(key[:19], [19, 18, 17, 14]),
            LFSR(key[19:46], [27, 26, 25, 22]),
            LFSR(key[46:], [23, 22, 20, 18]),
        ]

    def getbit(self):
        b = [lfsr.getbit() for lfsr in self.LFSR]
        return b[1] if b[0] else b[2]
```
而這樣形式的 nLFSR 有一特性是 output 會分別跟 2、3 的 LFSR 有 75% 的相似度，利用這特性我們就可以透過 bit 的組合，跟 output 做比較來推出 key，下面舉例從 2 的 LFSR 回推 key，3 也是同樣做法。
```python=0
key_len = 27

for b in range(key_len):
    print(str(b)+':')
    for c in itertools.combinations(range(key_len), b):
        key_candidate = [1- stream[i] if i in c else stream[i] for i in range(key_len)]
        lfsr = LFSR(key_candidate, [27, 26, 25, 22])
        s = [lfsr.getbit() for _ in range(256)]
        matches = sum(x==y for x,y in zip(s, stream))
        if matches>=180:
            print(key_candidate)
            print(s)
            break
```
有了 2 跟 3 的 key，要推出 1 的 key 就很簡單，一樣用 bit 的組合去試，將推出的 key1 與 key2、key3 結合就是完整的 key，用一個 list 存所有可能的 key，然後每次做AES解密，需要試 2^19 次方，但比起原本的 2^69 次方快的多了，最後就可以成功拿到 FLAG 了！
```python=0
def decrypt_flag(key):
    sha1 = hashlib.sha1()
    sha1.update(str(key).encode('ascii'))
    key = sha1.digest()[:16]
    # print(key)
    iv = bytes.fromhex("cd2832f408d1d973be28b66b133a0b5f")
    encrypted_flag = bytes.fromhex("1e3c272c4d9693580659218739e9adace2c5daf98062cf892cf6a9d0fc465671f8cd70a139b384836637c131217643c1")
    # print(encrypted_flag)
    cipher = AES.new(key, AES.MODE_CBC, iv)
    plaintext = cipher.decrypt(encrypted_flag).decode("ascii", "ignore")
    if "FLAG" in plaintext:
        print("FLAG:"+plaintext[:-2])
        
key1=[0 for i in range(19)]
key2 = [0, 1, 1, 0, 0, 0, 1, 0, 0, 1, 1, 0, 0, 0, 0, 0, 0, 1, 0, 1, 0, 0, 0, 1, 1, 0, 1]
key3 = [0, 1, 1, 0, 1, 0, 1, 1, 0, 1, 1, 0, 1, 0, 1, 0, 0, 0, 1, 0, 1, 0, 1]   
total_key = key1+key2+key3
possible_key = []
for b in range(len(key1)):
    for c in itertools.combinations(range(len(key1)), b):
        tmp = copy.deepcopy(key1)
        for i in c:
            tmp[i] = 1
        possible_key.append(tmp+key2+key3)

print("possible_key_num: "+str(len(possible_key)))
for i in possible_key:
    decrypt_flag(int("".join(str(x) for x in i), 2))
```
## FLAG 截圖
![](https://i.imgur.com/4RhBnwr.png)