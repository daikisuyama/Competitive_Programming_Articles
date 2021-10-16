# 感想

F問題はそこまで高いレベルではなかったのですが、解き方に気づいたもののコンテスト時間内に解けなかったため、非常に悔しかったです。


# [A問題](https://atcoder.jp/contests/abc153/tasks/abc153_a)

ceil関数で体力をh以下にするのに必要な回数を計算するだけです。

```python:A.py
import math
h,a=map(int,input().split())
print(int(math.ceil(h/a)))
```

# [B問題](https://atcoder.jp/contests/abc153/tasks/abc153_b)

合計の必殺技で殺せるかどうかで判断すれば良いです。

```python:B.py
h,n=map(int,input().split())
a=[int(i) for i in input().split()]
print("Yes" if sum(a)>=h else "No")
```

# [C問題](https://atcoder.jp/contests/abc153/tasks/abc153_c)

必殺技で体力の高いモンスターから倒していけば良いです。

```python:C.py
n,k=map(int,input().split())
h=[int(i) for i in input().split()]
h.sort(reverse=True)

if k>=n:
    print(0)
else:
    print(sum(h[k:]))
```

# [D問題](https://atcoder.jp/contests/abc153/tasks/abc153_d)

この問題では、分裂した際に同じ体力のモンスターが生まれることに注目します。つまり、初めに体力$n$のモンスターがいた時、下記のようにモンスターの体力は減っていって最後に体力が1になった時にその次の攻撃で0になります。

$$n→[\frac{n}{2}],[\frac{n}{2}]→[\frac{[\frac{n}{2}]}{2}],[\frac{[\frac{n}{2}]}{2}],[\frac{[\frac{n}{2}]}{2}],[\frac{[\frac{n}{2}]}{2}]$$

つまり、モンスターの分裂の回数を$k$としたとき、モンスターへの攻撃の合計数は$2^0+2^1+2^2+…+2^{k-1}$になります。よって、$k$を求めた上で攻撃の合計数の$2^k-1$を求めれば良いです。

```python:answerD.py
h=int(input())
cnt=0
while True:
    h=h//2
    cnt+=1
    if h==0:
        break
print(2**cnt-1)
```

# [E問題](https://atcoder.jp/contests/abc153/tasks/abc153_e)

**個数が無制限のナップザックの問題**に相当します。ここで、$dp[i]:=$(モンスターの体力を$i$減らすために必要な最小の魔力)と定義し、任意の要素を$inf$で初期化して$dp[0]=0$とします。そして、$inf$ではない要素の身を更新していけば良いです。

そして、最終的にモンスターの体力を最小の魔力で0以下にすれば良いので、モンスターの体力を$h$以上減らせる中での最小の魔力を求めれば良いです。

```python:answerE.py
h,n=map(int,input().split())
a,b=[],[]
for i in range(n):
    a_sub,b_sub=map(int,input().split())
    a.append(a_sub)
    b.append(b_sub)
inf=10000000000000
ma=max(a)
dp=[inf]*(h+1+ma)
dp[0]=0
for i in range(n):
    x=[a[i],b[i]]
    for j in range(h+1+ma):
        if dp[j]!=inf:
            if j+x[0]<=h+ma:
                dp[j+x[0]]=min(dp[j+x[0]],dp[j]+x[1])
            else:
                break
print(min(dp[h:]))
```

# [F問題](https://atcoder.jp/contests/abc153/tasks/abc153_f)

爆弾を使ってできるだけ効率よくモンスターを倒すことが目標です。ここで、モンスターを座標順に並べておきます。まず、一番左端にいるモンスターを倒すときは**区間の左端をそのモンスターに合わせる**ことでより効率よくモンスターを倒すことができ、このとき必要な爆破回数は$\lceil \frac{h[0]}{a} \rceil$になることは明らかです。また、この爆破に巻き込まれるモンスターを調べるには、**二分探索で区間の右端がどこにあるのかを調べる**ことでわかります。ここまではコンテスト中に考察することができました。

上記の方針の場合、巻き込んだモンスターの体力を減らすために入れ子のforループを使っているため、効率の悪い1つ目のコードとなります。ここで、[いもす法](https://qiita.com/DaikiSuyama/items/67547e14b47cd6360252)を使うと、区間の両端の端のみで効率よく更新を行うことができます。したがって、いもす法を利用する元で端から累積和をとって考えていき、モンスターの体力が爆発に巻き込まれてすでに0以下になっている場合は飛ばし、0より大きい場合は必要回数だけ爆発を行えば良いです。

以上を実装して、2つ目のコードのようになります。

```answerF_TLE.py
import bisect
import math
n,d,a=map(int,input().split())
xh=[list(map(int,input().split())) for i in range(n)]
xh.sort()
x=[xh[i][0] for i in range(n)]
h=[xh[i][1] for i in range(n)]
cnt=0
for i in range(n):
    #print(cnt)
    if h[i]>0:
        m=math.ceil(h[i]/a)
        cnt+=m
        k=bisect.bisect_right(x,x[i]+2*d,lo=i)
        for j in range(i,k):
            h[j]-=(m*a)
print(cnt)
```

```answerF.py
import bisect
import math
n,d,a=map(int,input().split())
xh=[list(map(int,input().split())) for i in range(n)]
xh.sort()
x=[xh[i][0] for i in range(n)]
h=[xh[i][1] for i in range(n)]
g=[0]*n
cnt=0
for i in range(n):
    if h[i]>0:
        m=math.ceil(h[i]/a)
        cnt+=m
        k=bisect.bisect_right(x,x[i]+2*d,lo=i)
        if i!=n-1:
            h[i]-=m*a
            g[i]-=m*a
            if k<n:
                g[k]+=m*a
            g[i+1]+=g[i]
            h[i+1]+=g[i+1]
    else:
        if i!=n-1:
            g[i+1]+=g[i]
            h[i+1]+=g[i+1]
print(cnt)
```
