# HW2 - Single
###### tags: `資安` `crypto`

## 題目解析
這題做的是 Singular elliptic curve 的對稱式加密，而我們要想辦法根據提供的 A, B 兩點 (在橢圓取線上)，來推出私鑰 dA 或 dB，並用 A * dB 或 B * dA 來推出共同密鑰 K 來解密拿到 FLAG。

## 解題流程
p 跟 G 還有需要運用到的 Function 題目都有給。
```python=0
Point = namedtuple("Point", "x y")
O = 'INFINITY'

p = 9631668579539701602760432524602953084395033948174466686285759025897298205383
gx = 5664314881801362353989790109530444623032842167510027140490832957430741393367
gy = 3735011281298930501441332016708219762942193860515094934964869027614672869355
G = Point(gx, gy)
```

根據題目給的 A, B 及 Ellptic curve 的方程式 $y^2 = x^3 +ax + b$ $(mod$ $p)$  我們可以做聯立解出 A, B，在解 A, B 的時候我是用等式去做 sage 的 solve()，而等式表示方式有未知數不能直接加上 mod，所以就直接算出 A, B 再做 mod 也沒關係 (分配律)。
```python=0
A = Point(x=3829488417236560785272607696709023677752676859512573328792921651640651429215, y=7947434117984861166834877190207950006170738405923358235762824894524937052000)
B = Point(x=9587224500151531060103223864145463144550060225196219072827570145340119297428, y=2527809441042103520997737454058469252175392602635610992457770946515371529908)
var('a b dA')
# eql1 = A.y**2 == (A.x**3 + a*A.x + b)
# eql2 = B.y**2 == (B.x**3 + a*B.x + b)
R = IntegerModRing(p)
eql1 = A.y**2 == (A.x**3 + a*A.x + b)
eql2 = B.y**2 == (B.x**3 + a*B.x + b)
solve = solve([eql1, eql2], a, b)
print(solve[0][0])
print(solve[0][1])
a_ = solve[0][0].rhs()
b_ = solve[0][1].rhs()

a_ = -275016383774926851721212624019639682076610444999817292925761556473478467104557380603229079021607084567962875189343036694681346601571506658043468361664679235767672695134042777611112376850057005899771385151671152683687197711114971/1919245360971656758276872055812146488932461121894548581344882831233155956071
b_ = 945388432553440275588891974126814557759786334978152820501009429506913479731192389133937917877505655062337390378896175601871977186318477998152353093410094007462688877216305828698383883938838642117777891957368943704595374898505243826951033986105490709591192216329838534208543902094030894609142263028328140/1919245360971656758276872055812146488932461121894548581344882831233155956071

F = GF(p)
a = F(a_)
b = F(b_)
a, b
#(9605275265879631008726467740646537125692167794341640822702313056611938432994, 7839838607707494463758049830515369383778931948114955676985180993569200375480)
```
有了 a, b 之後再來我們要確認這個 Singular curve 的類型，把 a, b 帶回 Ellptic curve 方程式用 factor() 做因式分解，分解出來是 $(x-\alpha)^2(x-\beta)$ 的形式，就知道他是 Node 的 Singular curve，我們同時也知道了 $\alpha$ 跟 $\beta$。
```python=0
R.<x> = PolynomialRing(F)
print(a, b)
print(factor(x**3 + a*x + b))
#(x + 1706485822346415641443806104662801825943914230110363749830437374602647864828) * (x + 8778425668366493782038529472271552171423076833119284811370540338595974272969)^2
```
再來是最重要的解私鑰 dA，我們知道 Node 的 Singular curve 有一個特性，d 與 Ellptic curve 上一點 P 之間的關係為，$\phi(dP)$ = $\phi(P)^d$
Node 的 $\phi$ 公式如下：

$\phi(P(x,y))=\frac{y+\sqrt{\alpha-\beta}(x-\alpha)}{y-\sqrt{\alpha-\beta}(x-\alpha)}$

G是基點，output 給的 A, B 是基於 G 下去做 ECDLP 算出來的，所以這裡我們的 G = P, A = dP, d = dA，將其與 $\alpha$, $\beta$ 分別代入，解 DLP 便可解出 dA。

然後要特別注意整體運算流程都是在 $mod(p)$ 下做運算的，所以我設了一個基於 p 的 IntegerModRing，在其下做公式運算得出 $\phi(A)$, $\phi(G)$，又 $\phi(A)$ = $\phi(G)^{dA}$，最後 log 解出 dA。
```python=0
R = IntegerModRing(p)
alpha, beta = R(alpha), R(beta)
print(alpha, beta)
fi_A = R((A.y + (sqrt(alpha-beta) * (A.x-alpha)))/(A.y - (sqrt(alpha-beta) * (A.x-alpha))))
fi_G = R((G.y + (sqrt(alpha-beta) * (G.x-alpha)))/(G.y - (sqrt(alpha-beta) * (G.x-alpha))))
print(f'fi_A:{fi_A}, fi_G:{fi_G}')

dA = R(log(fi_A, fi_G))
dA
#1532487521612462894579517163606359285989568203515734083099567402780433190052
```
然後依照 dA 推出共同密鑰 k，FLAG 的加密就只是簡單的跟 k 的 Hash 做 XOR，因此得到 k 後過 Hash 在跟密文做一次 XOR就可以得到 FLAG 了！

不過最後這邊沒法在 sage 做，不知道為什麼在 sage 做會噴錯，似乎是 Symbolic Rings 的問題，我後來是移植所有需要參數到 python3 下運行就沒問題了！
```python=0
k = point_multiply(B, dA).x
k = hashlib.sha512(str(k).encode('ascii')).digest()
enc = bytes.fromhex("1536c5b019bd24ddf9fc50de28828f727190ff121b709a6c63c4f823ec31780ad30d219f07a8c419c7afcdce900b6e89b37b18b6daede22e5445eb98f3ca2e40")
dec = bytes(ci ^ ki for ci, ki in zip(enc.ljust(len(k), b'\0'), k))
dec
```
## FLAG截圖
![](https://i.imgur.com/7I29x9V.png)