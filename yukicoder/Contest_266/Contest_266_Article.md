# 感想

Dが解けなくて焦りましたが、よく考えれば式変形だけで解くことができます。また、E問題の発想は出たのですが、計算量の解析ができませんでした。

# [A問題](https://yukicoder.me/problems/no/1229)

トライは最大で$[\frac{n}{5}]$回、コンバージョンは最大で$[\frac{n}{2}]$、ペナルティーゴールは最大で$[\frac{n}{3}]$回となります。また、$N$は100以下なので全探索すれば良いです。ただし、コンバージョンの回数$\leqq$トライの回数であることに気をつけます。

```python:A.py
n=int(input())
ans=0
for i in range(n//5+1):
    for j in range(n//2+1):
        for k in range(n//3+1):
            ans+=(i*5+j*2+k*3==n and i>=j)
print(ans)
```

#[B問題](https://yukicoder.me/problems/no/1230)

まず、宝箱の確率は並べ替えても答えには関係がないので、$p \leqq q \leqq r$とします。

この時、ある宝箱に印をつけて**STAYかCHANGEを行います**。STAYを選択する場合は**確率が最大の宝箱にSTAYするのが最大**なので、ダイヤモンドを手に入れる確率は選択した宝箱にダイヤモンドがある確率に等しく$\frac{r}{p+q+r}$です。CHANGEを選択する場合は**確率が最小の宝箱からCHANGEするのが最大**なので、ダイヤモンドを手に入れる確率は選択していない宝箱にダイヤモンドがある確率に等しく$\frac{q+r}{p+q+r}$です。

よって、答えは$max(\frac{r}{p+q+r},\frac{q+r}{p+q+r})=\frac{q+r}{p+q+r}$となります。

```python:B.py
p,q,r=sorted(list(map(int,input().split())))
print((q+r)/(p+q+r))
```

# [C問題](https://yukicoder.me/problems/no/1231)

まず、10の倍数であるかが重要なので、それぞれのカードの$mod\ 10$を取って考えます。また、1枚のカードを選ぶことによって$mod\ 10$の値が変わり、**最大の選ぶ枚数を状態として**持つDPを考えます。この時以下のようにDPをおきます。

$dp[i]:=$(余りを$i$にする時の最大枚数)

また、遷移はカード$A\_j$を選ぶ時、以下のようになります。

$dp[(i+A\_j)\%10]+=dp[i]$

さらに、**遷移が一方向ではない**ので、カード$A\_j$を選択した時の**$dp$の更新はまとめて行う必要**があります。従って、それぞれのカード$A\_j$の更新は一旦$dp\_sub$に保存しておいて、カード$A\_j$の選択が終わってから$dp$に記録していく必要があります。

```python:C.py
ans=0
n=int(input())
check=[0 for i in range(10)]
a=list(map(int,input().split()))
for i in a:
    check[i%10]+=1
dp=[0 for i in range(10)]
for i in range(10):
    for j in range(check[i]):
        dp_sub=[0]*10
        for k in range(10):
            if k==0:
                dp_sub[(i+k)%10]=dp[k]+1
            elif dp[k]!=0:
                dp_sub[(i+k)%10]=dp[k]+1
        for k in range(10):
            dp[k]=max(dp[k],dp_sub[k])
        #print(dp_sub)
#print(dp)
print(dp[0])
```

# [D問題](https://yukicoder.me/problems/no/1232)

**素数と累乗の関係の問題**なので、**フェルマーの小定理**を用いて考えます。フェルマーの小定理は以下のようです。

```math
pを素数とし、aをpと互いに素の整数とした時、a^{p-1} = 1\ (mod \ p) \\
また、これを示すには二項定理からa^{p} = a\ (mod \ p)を示せれば良い。
```

まず、フェルマーの小定理にそのまま値を代入すると$2^{p\_i-1} = 1\ (mod \ p\_i)$…①が成り立つので、この式を変形します。また、**2と$p\_i$が互いに素であることが必要**ですが、これは$p\_i=2$の時のみでこの時は$x=2$で成り立ちます。ここで、**①の両辺の累乗**をとった時に右辺の値は変わりませんが、左辺の値は変わります。つまり、①の両辺の$t$乗を取れば、$2^{t(p\_i-1)} = 1 \ \ (mod \ p\_i)$が成り立ちます。また、求めたいのは$2^{t(p\_i-1)} = t(p\_i-1) \ \ (mod \ p\_i)$となる時です。よって、**二辺を連立**すれば$t(p\_i-1)=1 \ \ (mod \ p\_i)$が成り立ちます。これを変形することで$p\_i-t=1  \ \ (mod \ p\_i)$より$t=p\_i-1$とすればこれを満たし、$x=(p\_i-1)^2$は$10^{18}$より小さいので確かに問題の条件を満たします。

以上より、題意を満たすのは$p\_i=2$の時$x=2$で$p \neq 2$の時$x=(p\_i-1)^2$となります。

```python:D.py
for i in range(int(input())):
    n=int(input())
    print(2 if n==2 else (n-1)**2)
```

# [E問題](https://yukicoder.me/problems/no/1233)

まず、任意の$i,j$についての$A\_i \% A\_j$の値の和を求めます。このような**総和の問題ではそれぞれの値に注目する**と良いですが、余りのままだと難しいので**言い換え**を考えます。つまり、$A\_i \% A\_j=A\_i-[\frac{A\_i}{A\_j}] \times A\_j$と言い換えます。この時、$A\_i$については$n\sum_{i=1}^{n}A\_i$なので、$O(N)$で求められますが、$[\frac{A\_i}{A\_j}] \times A\_j$は簡単には求まりません。

ここでは、$A\_j$を固定すると、**$[\frac{A\_i}{A\_j}] \times A\_j$が一致するような$i$が存在する可能性**があるので、$A\_j$を固定して考えます($\because$一致すると、**まとめて計算できる**ことがほとんどなので)。また、値が一致して$[\frac{A\_i}{A\_j}]=k$となる時、$A\_j \times k \leqq A\_i < A\_j \times (k+1) $となるので、**度数分布のイメージでまとめて**数えていきます。

この際、数列$A$を昇順で並べておけば、ある$k$に相当する数がいくつあるかは**二分探索を用いて探す**ことが可能です。また。計算の効率化のために、ある$A\_j$で固定した時の総和は辞書で保存しておき、同じ数が出てきた場合は辞書に保存された数を用います。さらに、それぞれの$A\_j$で最大$[\frac{A\_{max}}{A\_j}]$通り存在し、それぞれの検索に$O(\log{n})$かかるので、全体の計算量は$O( \sum \_{j=1} ^{n} [\frac{A\_{max}}{A\_j}] \log{n})=O(A_{max} \log{n} \sum \_{j=1} ^{n} [\frac{1}{A\_j}])$となります。よって、調和級数の考え方より$O(\sum \_{j=1} ^{n} [\frac{1}{A\_j}])=O(\log{A_{max}})$なので、全体の計算量は$O(A_{max} \log{n} \log{A_{max}})$となります。


```python:E.py
n=int(input())
a=list(map(int,input().split()))
a.sort()
from bisect import bisect_left,bisect_right
ans=0
d=dict()
for i in range(n):
    if a[i] in d:
        ans+=d[a[i]]
        continue
    ans_sub=0
    j=i
    while j!=n:
        x=a[j]//a[i]
        b=bisect_left(a,(x+1)*a[i],lo=j)-1
        ans_sub+=(x*(b-j+1)*a[i])
        j=b+1
    d[a[i]]=ans_sub
    ans+=ans_sub
print(sum(a)*n-ans)
```
