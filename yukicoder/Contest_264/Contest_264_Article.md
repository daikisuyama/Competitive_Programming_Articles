# 感想

今回も焦って方針を定めきることができませんでした。

冷静になればできる問題が多いので、**方針を整理する力が必要**と感じています。

# [A問題](https://yukicoder.me/problems/no/1217)

変える文字に注目すれば良いです。

```python:A.py
t="abcdefghijklmnopqrstuvwxyz"
s=input()
for i in range(26):
    if s[i]!=t[i]:
        print(t[i]+"to"+s[i])
        break
```

#[B問題](https://yukicoder.me/problems/no/1218)

これもA問題と同様に方針を立てやすいと思います。  
制約から$x$(もしくは$y$)は1~$z$-1しか取り得ないので、**全探索**をします。

```python:B.py
n,z=map(int,input().split())
for x in range(1,z):
    y=int((z**n-x**n)**(1/n))
    if x**n+y**n==z**n:
        print("Yes")
        break
else:
    print("No")
```

# [C問題](https://yukicoder.me/problems/no/1219)

A,B問題までは悪くなかったのですが、方針を詰め切ることができませんでした。

まず、**ナイーブにシミュレーションを行う**ことを考えます。この時、$a[i]\>i$となる時はその$a[i]$のぶんは少なくとも**移動させることができない**ので、**任意の$i$で$a[i] \leqq i$**が成り立つ必要があります(✳︎1)。また、この条件はシミュレーション中も常に成り立つ必要があるので、**$a[i]=i$が成り立つ中で最小の$i$から順にシミュレーション**を行えば良いです(✳︎2)。

ここで、ナイーブなシミュレーションは$O(N^2)$ですが、ナイーブなシミュレーションでは0~$i$-1までの全てに+1しているので、**$i$番目より前の要素にいくつぶん+1を行ったか**を保存しておけば$O(N)$で効率良く計算できます。また、このために大きい$i$から順に石の移動が可能かを考えますが、これは**シミュレーションの順番とは異なります**($\because$ (✳︎2))。しかし、合計で移動してくる石の個数はシミュレーションの順番によらず同じなので、**石の数さえ合っていれば**シミュレーションは都合の良いように行うことができます。

ここで、$i$番目のマス目を見たときに石がいくつ分移動してきているかを$c$とすると、**そのマス目から移動する石は$a[i]+c$**となります。このとき、$(a[i]+c)\%i=0$であればこのマスから移動させることができます。また、移動させることができるとき、$c=c+\frac{a[i]+c}{i}$が成り立ちます。これを任意のマス目について考え、成り立っていればYesでそうでなければNoとなります。

(✳︎1)…問題文をよく読んだらこの条件は常に成り立っていました…。反省です。

```python:C.py
from itertools import dropwhile
n=int(input())
a=list(dropwhile(lambda x:x==0,list(map(int,input().split()))[::-1]))[::-1]
n=len(a)
if any(a[i]>i+1 for i in range(n)):
    print("No")
    exit()
c=0
for i in range(n-1,-1,-1):
    if (c+a[i])%(i+1)!=0:
        print("No")
        break
    c+=(c+a[i])//(i+1)
else:
    print("Yes")
```

# [D問題](https://yukicoder.me/problems/no/1220)

そもそもの確率の計算を間違えていました…。

確率の計算を行うと、$F,S$はそれぞれ以下のようになります。

```math
\begin{align}
F&=\frac{M \times _NC_K}{_{NM}P_K}\\
S&=\frac{(N-K+1) \times M^K}{_{NM}P_K}
\end{align}
```

さらに、$F\>S$は以下のようになります。

```math
\begin{align}
&\frac{M \times _NC_K}{_{NM}P_K}>\frac{(N-K+1) \times M^K}{_{NM}P_K}\\
&\leftrightarrow M \times _NC_K>(N-K+1) \times M^K\\
&\leftrightarrow _NC_K>(N-K+1) \times M^{K-1}
\end{align}
```

このまま計算を行うと**それぞれのクエリで$O(K)$ほどかかってしまう**ので事前計算により高速化を図りますが、このままだと掛け算なので巨大な数となってオーバフロー及び大量の計算時間を必要とします。そこで、**対数を使ってより計算しやすい足し算に掛け算を直します**。つまり、$F\>S$は以下のようになります。

```math
\begin{align}
&N! \div (N-K)! \div K!>(N-K+1) \times M^{K-1}\\
&\leftrightarrow \sum_{i=1}^{N}{\log{i}}-\sum_{i=1}^{N-K}{\log{i}}-\sum_{i=1}^{K}{\log{i}}>\log{(N-K+1)}+(K-1)\log{M}
\end{align}
```

ここで、$\log$の部分は事前に累積和($a[i]:=\sum_{j=1}^{i}{\log{j}}$)を求めておけば良く、これは$O(N\_{max})$で求めることができます($N\_{max}$は高々$10^5$です。)。

以上よりそれぞれのクエリを$O(1)$で求めることができるので、全体の計算量は$O(N\_{max}+Q)$となります。

```python:D.py
from math import log2
from itertools import accumulate
x=[0]+[log2(i) for i in range(1,10**5+1)]
a=list(accumulate(x))

for _ in range(int(input())):
    n,m,k=map(int,input().split())
    ans=["Flush","Straight"]
    print(ans[a[n]-a[n-k]-a[k]>(a[n-k+1]-a[n-k])+(k-1)*(a[m]-a[m-1])])
```

# [E問題](https://yukicoder.me/problems/no/1221)

まず、繋がっている辺の本数の情報が関わるので、**DFSまたはBFS**を使いそうです。さらに、頂点上でそれぞれスコアが決まり貪欲に決められず、それぞれの頂点は存在するかどうかの二つの状態しか持たないので、**木上でのDP**が有効です。

そして、DFSによる木上のDPの場合、**ある頂点にいるときにその頂点の子をを頂点とする部分木の情報**をすでに知っており、**今見ている頂点と子を繋ぐ辺**に注目すれば実装できそうなので、ここではDFSによる実装を行います。

DPは以下のようにおきます。

$dp[i][j]:=$($i$を根とする部分木において最大のスコア)
(ただし、$j=0$のときはその頂点は消され、$j=1$のときはその頂点は消されていない。)

ここで、今見ている頂点を$i$、子の頂点を$j$としてDPの遷移を考えます。また、DFSにより$dp[j]$はすでに求まっています。

(1)$dp[i][0]$への遷移

$i$を追加しても$a[i]$が増えるだけです。

$$dp[i][0]=a[i]+\sum_j{max(dp[j][0],dp[j][1])}$$

(2)$dp[i][1]$への遷移

それぞれの$j$に注目したときに、$j$が存在すれば$i$とどちらも存在するので、$b[i]+b[j]$だけ増えます。

$$dp[i][1]=\sum_j{max(dp[j][0],dp[j][1]+b[i]+b[j])}$$

(✳︎)葉にいる場合(初期化)

$$dp[i]=[a[i],0]$$

```python:E.py
from sys import setrecursionlimit
setrecursionlimit(10**7)
n=int(input())
a=list(map(int,input().split()))
b=list(map(int,input().split()))
tree=[[] for i in range(n)]
for i in range(n-1):
    u,v=map(int,input().split())
    tree[u-1].append(v-1)
    tree[v-1].append(u-1)
check=[False]*n
dp=[[0,0] for i in range(n)]

def dfs(i):
    global n,a,b,tree,check,dp
    #頂点iを消さない場合
    dp[i]=[a[i],0]
    for j in tree[i]:
        if not check[j]:
            check[j]=True
            dfs(j)
            dp[i][0]+=max(dp[j][0],dp[j][1])
            dp[i][1]+=max(dp[j][0],dp[j][1]+b[i]+b[j])

check[0]=True
dfs(0)
print(max(dp[0]))
```
