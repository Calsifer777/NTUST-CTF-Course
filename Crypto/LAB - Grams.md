# LAB - Grams
###### tags: `資安` `crypto`

## 題目解析
首先題目給了 output.txt、ngrams.json、solve.py 和 vigenere.py。

ouput.txt 內容為過 Vigenere 加密的字串 y 和一個與 y 相同密鑰 Hash 後 XOR 的 FLAG 資訊。
```
y = 45d6ukumip,ppi9c8loq9lt89iz1mgdu22.w 6u.0tdhv02szkibnb0bk2b,mi,58bc9so 1.f8b,vs7 bdvznq6jrpa 99bz6n5ukbapc5mdg8ub9t7.77hg.1qbx32u4ytx,7nw89df3g04pw01goz2s8gu4jhewc3ss5qnxe6aq slhp50yc.w,1htje430 l5 x 0sjj76a23drbih7mt2qdf,10pbtb.hua,dbv3tbi203zzn3sy8ga7q,o349qwy0.8d5zeh,31x0ol0pain413 8iu,rbza2mkz,k9izl6gs6nju 2nbbbyf145d6ukocywcdrqti87dq9lt13g.0d5kb6267bvqo5d1m80 8,imqt5dc4r98kjdosc 5cgduj z
enc = 996d2614786cde14a4f429648b738bbe0868bcd0445e24c913b5d6302c0e8d38e1cfcedbdd8956cec71622caceeafd5c3004555b11ab0249858208556154d9b7
```

ngrams 就是字詞在句中的機率，是一字典檔。

![](https://i.imgur.com/xexD9pW.png)

vigenere.py 是加密 Source code，solve.py 給了一些可以用來解題的函式，因此我們要做的事就是想辦法猜出加密的 key，再拿來 Hash後跟 enc XOR，就能還原出FLAG。

## 解題流程
先隨機生成一些 key candidate，然後 decrypt y 可以得到一字串，再接著透過 ngrams 計算字串分數，之後對 key 前幾名高分的 key 做類似基因演算法的 shuffle 跟 crossover，再重複前面步驟。理論上分數越高，decrypt 出來的字串會越符合原本未加密的 y，當然肯定還是會有一些差錯，最後再人工矯正一些單字，讓其變成正確的句子。

這邊 key size 是 80，population 大小是 7000，迭代 2000 次，可以依據情況調整。
```python=1 
keys = np.array(initialize(7000)) #80 index tuple
scores = np.array([])
for i in keys:
    scores = np.append(scores, fitness(tuple(i)))
keys = keys[scores.argsort()[::-1]][:600]
for m in range(2000):
    np.random.shuffle(keys)
    for i in range(len(keys)//2):
        child = np.array(crossover(keys[i*2], keys[i*2+1], 0.7))
        keys = np.concatenate((keys,  [child]))
    np.random.shuffle(keys)
    for i in range(len(keys)//2):
        keys[i] = mutate(keys[i])
    scores = np.array([])
    for i in keys:
        scores = np.append(scores, fitness(tuple(i)))
    keys = keys[scores.argsort()[::-1]][:600]
    scores = scores[scores.argsort()[::-1]][:600]
    print(m, int(scores[0]), decrypt(ctx, keys[0]))
    print(keys[0])
```
2000 次後得到一個看起來比較順眼的字串，大概跟明文有 87% 像了。

![](https://i.imgur.com/oAqSgUS.png)

自行修改單字讓句子完整，存在 plain2.txt。
```
gimli is a 384 bits permutation designed to achieve high security with high performance across a broad range of platforms. it is currently in round 2 of the nist lightweight cryptographic project, the submission consisting of an authenticated encryption and a cryptographic hash function. in this paper, we focus on the gimli cipher, which performs authenticated encryption with associated data.
```
有了明文跟密文，要解 key 就很簡單，charset 是密鑰範圍，遞迴密鑰去做加密後比對密文，就可以得到 key。拿到 key 後 Hash 再跟 enc 做 XOR，FLAG就出來啦！
```python=0
import string
import hashlib

charset = string.ascii_lowercase+string.digits+',. '
charset_idmap = {e: i for i, e in enumerate(charset)}

with open('../plain2.txt') as f:
    plain = f.read().strip()
print(len(plain))
# for i, c in enumerate(plain):
#     print(str(i)+":"+str(c))
with open('../output.txt') as f:
    ctx = f.readline().strip()[4:]
    enc = bytes.fromhex(f.readline().strip()[6:])
# ctx = [charset_idmap[c] for c in ctx]
print(len(ctx))
# print(ctx)

ksz = 80
plain = [charset_idmap[c] for c in plain]
key = [0 for i in range(ksz)]
# print(key)

def decrypt_key(plain, ctx):
    N, ksz = len(charset), 80
    for i in range(len(ctx)):
        for j in range(len(charset)):
            # print(i)
            if (plain[i]+ j)%N == charset.index(ctx[i]):
                key[i%ksz] = j
    # return ''.join(charset[(c + key[i % ksz]) % N] for i, c in enumerate(plain))

decrypt_key(plain, ctx)
print(key)
with open("../output.txt") as f:
    ctx = f.readline().strip()[4:]
    enc = bytes.fromhex(f.readline().strip()[6:])
    print(enc)
    print(''.join(charset[k] for k in key).encode('ascii'))
k = hashlib.sha512(''.join(charset[k] for k in key).encode('ascii')).digest()
print(k)
enc = bytes(ci ^ ki for ci, ki in zip(enc.ljust(len(k), b'\0'), k))
print('flag =', enc.decode("ascii", "ignore"))
```
## FLAG 截圖
![](https://i.imgur.com/klI7L8z.png)
