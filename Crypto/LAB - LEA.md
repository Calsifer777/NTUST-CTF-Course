# LAB - LEA

###### tags: `資安` `crypto`

## 題目解析
這題在做的是 LEA(Length Extrnsion Attack)，一樣給了 source code 和 remote server。

輸入 username，若是 Admin 或 Guest 會返回 Session_id 跟 (username&&password&&sessionID) 做完 Hash 產生的 Mac，若隨便輸入 username，就會要你設密碼，一樣返回 Session_id 跟 Mac，然後讓你輸入mac&&*session&&cmd，只要 session_id in *sess and cmd == "flag"。

而我們要做的就是透過其返回的 Hash Mac，以 LEA 方式塞入一些 padding 讓我們的 flag cmd 指令能被認證是正確的。

```python=0
#!/usr/bin/env python3
import os,random,sys,string
from math import sin
from secret import FLAG, SECRET_PASSWORD
from hashlib import sha256

USERS = {}
USERS[b'Admin'] = SECRET_PASSWORD
USERS[b'Guest'] = b'No FLAG'

def MyHash(s):
	A = 0x464c4147
	B = 0x7b754669
	C = 0x6e645468
	D = 0x65456173
	E = 0x74657245
	F = 0x6767217D
	def G(X,Y,Z):
		return (X ^ (~Z | ~Y) ^ Z) & 0xFFFFFFFF
	def H(X,Y):
		return (X << Y | X >> (32 - Y)) & 0xFFFFFFFF
	X = [int((0xFFFFFFFE) * sin(i)) & 0xFFFFFFFF for i in range(256)]
	s_size = len(s)
	s += bytes([0x80])
	if len(s) % 128 > 120:
		while len(s) % 128 != 0: s += bytes(1)
	while len(s) % 128 < 120: s += bytes(1)
	s += bytes.fromhex(hex(s_size * 8)[2:].rjust(16, '0'))
	for i, b in enumerate(s):
		k, l = int(b), i & 0x1f
		A = (B + H(A + G(B,C,D) + X[k], l)) & 0xFFFFFFFF
		B = (C + H(B + G(C,D,E) + X[k], l)) & 0xFFFFFFFF
		C = (D + H(C + G(D,E,F) + X[k], l)) & 0xFFFFFFFF
		D = (E + H(D + G(E,F,A) + X[k], l)) & 0xFFFFFFFF
		E = (F + H(E + G(F,A,B) + X[k], l)) & 0xFFFFFFFF
		F = (A + H(F + G(A,B,C) + X[k], l)) & 0xFFFFFFFF
	return ''.join(map(lambda x : hex(x)[2:].rjust(8, '0'), [A, F, C, B, D, E]))

def verify(*stuff):
	return MyHash(b'&&'.join(stuff)).encode()


def main():
    username = input('Welcome to our system!\nPlease Input your username: ').encode()
    if b'&' in username:
        print('nope')
        exit(-1)
    if username in USERS:
        password = USERS[username]
    else:
        password = input("Are you new here?\nLet's set a password: ").encode()
        USERS[username] = password
        
    print(f'Hello {username.decode()}')
    session = bytes.hex(os.urandom(10)).encode()
    print(f'Here is your session ID: {session.decode()}')
    print(f'and your MAC(username&&password&&sessionID): {verify(username,password,session).decode()}')

    while True:
        mac, *sess, cmd = bytes.fromhex(input('\nWhat do you want to do? ')).split(b'&&')
        if mac == verify(username,password,*sess,cmd) and session in sess[0]:
            if cmd == b'flag':
                if username == b'Admin':
                    print(FLAG)
                    return
                else:
                    print('Permission denied')
            elif cmd == b'exit':
                print('exit')
                break
            else:
                print('Unknown command.')
        else:
            print('Refused!')
    print('See you next time.')
        

if __name__ == '__main__':
    main()
```

## 解題流程
首先我們要先建構能繼續擴展的 MyHash()，命名為 MyHash2()，A~F 就是 IV，也就是前一次的 Hash 值，不過代入時要注意順序，然後要注意的是因為我們是接續第一個 block 再做一次 Hash，等價於 (username&&password&&sessionID+padding&&msg+padding+all_length) 的 Hash，因此 all_lenth 地方要加上前面 block 的長度，其他部分基本都一樣。
```python=0
def MyHash2(origin_hash, s, start):
    A = int(origin_hash[0:8], 16)
    B = int(origin_hash[24:32], 16)
    C = int(origin_hash[16:24], 16)
    D = int(origin_hash[32:40], 16)
    E = int(origin_hash[40:48], 16)
    F = int(origin_hash[8:16], 16)
    print(A)
    def G(X,Y,Z):
       return (X ^ (~Z | ~Y) ^ Z) & 0xFFFFFFFF
    def H(X,Y):
        return (X << Y | X >> (32 - Y)) & 0xFFFFFFFF
    X = [int((0xFFFFFFFE) * sin(i)) & 0xFFFFFFFF for i in range(256)]
    s_size = len(s)
    s += bytes([0x80])
    if len(s) % 128 > 120:
        while len(s) % 128 != 0: s += bytes(1)
    while len(s) % 128 < 120: s += bytes(1)
    s += bytes.fromhex(hex((s_size + start) * 8)[2:].rjust(16, '0'))
    for i, b in enumerate(s):
        k, l = int(b), i & 0x1f
        A = (B + H(A + G(B,C,D) + X[k], l)) & 0xFFFFFFFF
        B = (C + H(B + G(C,D,E) + X[k], l)) & 0xFFFFFFFF
        C = (D + H(C + G(D,E,F) + X[k], l)) & 0xFFFFFFFF
        D = (E + H(D + G(E,F,A) + X[k], l)) & 0xFFFFFFFF
        E = (F + H(E + G(F,A,B) + X[k], l)) & 0xFFFFFFFF
        F = (A + H(F + G(A,B,C) + X[k], l)) & 0xFFFFFFFF
    return (''.join(map(lambda x : hex(x)[2:].rjust(8, '0'), [A, F, C, B, D, E]))).encode()
```
對於前一個 block 的 Hash Mac，長度上只有 password 是不知道的，(username&&password&&sessionID) 不算 password 跟 padding 的長度是 29，也就是密碼要大於 90 才有可能進到第 2 個 block，所以應該是只有 1 個 block 沒錯。

而因為在輸入cmd的時候有個特性session in *sess 這邊可以讓 *sess = [padding, session]，他對到 session 有在裡面，cmd就會順利被執行。

最後只要用一開始拿到的mac接 (&&session&&cmd) 接下去算hash就可以拿到新的而且正確的mac讓我們去傳送。
```python=0
for length in range(29, 100):
    conn = remote("edu-ctf.csie.org", "42073")
    conn.sendline(b"Admin")
    print(conn.recvline())
    print(conn.recvline())
    session_id = conn.recvline().split(b": ")[1].strip(b"\n")
    print("session_id:", session_id)
    mac = conn.recvline().split(b": ")[1].strip(b"\n")
    print("mac length: ", len(mac))
    cmd = b"flag"
    padding = session_id + bytes([0x80]) + bytes([0x00]) * (120-length-1) + bytes.fromhex(hex(length * 8)[2:].rjust(16, '0'))
    print("padding: ", len(padding))
    mac = MyHash2(mac, b"&&" + session_id + b"&&" + cmd, 128)
    print("mac length: ", len(mac))
    print(conn.recvuntil(b"?"))
    print((b'&&'.join([mac, padding, session_id, cmd])))
    conn.sendline(b"&&".join([mac, padding, session_id, cmd]))
    a = conn.recvall()
    print(length, a)
    if b"FLAG" in a:
        break
    conn.close()
```
照理來說是這樣啦，我覺得流程上也沒有問題，但不知道為什麼送過去都沒有收到返回，我試了很多次，嘗試各種方式，最後發現一個問題是 *sess 因為是指標，跟 session 的形式是不一樣的。

![](https://i.imgur.com/4buKJNF.png)

可以看到理論上同樣的 byte，但感覺一個被看成 String，一個被看成 16 進位 byte，導致在比對時比對時不是 True，我後來有嘗試做 hex string 轉換再 encode()，就可以比對成功，但再傳到 remote server 還是一樣沒能收到值，感覺中間轉換的地方出了點問題，我盡力了QQ，最終沒能解出來。

![](https://i.imgur.com/P9gbPaq.png)
