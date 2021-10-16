# 感想

全体的に焦ってしまったため、今回も良いとは言えない結果でした。また、最初の20分と最後の20分くらいだけしか集中できてないので、そこを改善していきたいです。

# [A問題](https://atcoder.jp/contests/abc165/tasks/abc165_a)

aからbまでkの倍数かを全て確かめれば良く、以下のようなコードになります。また、any関数を使えばより簡単に書けます。

```python:A.py
k=int(input())
a,b=map(int,input().split())
for i in range(a,b+1):
    if i%k==0:
        print("OK")
        break
else:
    print("NG")
```

# [B問題](https://atcoder.jp/contests/abc165/tasks/abc165_b)

1.01倍にして切り下げる操作をX円以上になるまで繰り返し、その操作の総和を数えます。

```python:B.py
import math
ans=0
x=int(input())
y=100
while y<x:
    ans+=1
    y=math.floor(1.01*y)
print(ans)
```

# [C問題](https://atcoder.jp/contests/abc165/tasks/abc165_c)

この問題では、一見すると$M^{10}$通りを全探索しなければなりませんが、適当な$N,M$で実験すると$1 \leqq A_1 \leqq A_2 \leqq … \leqq A_N \leqq M$という制約が厳しく、$M^{10}$通りより大幅に少ない場合の数となることがわかります。したがって、まずは全探索の方針で問題を解き始めます。

この時、$A\_1,A\_2,…,A\_N$を前から順に定めれば良く、DFSを使えば良いです。$A\_1,A\_2,…,A\_d$までを持つ配列(or文字列)、前述の$d$、$A\_d$の三つを引数とする関数を用意し、$d$が$N$にたどり着いた場合は数列の得点の計算を行い、Nにたどりついていない場合は$A\_d$以上で$M$以下の値が次の$A_{d+1}$となりうるので、それぞれの場合の関数を呼び出していけばいいです。

また、$A\_1,A\_2,…,A\_d$は配列ではなく文字列として保存しておくと実装しやすいかもしれません。つまり、$1 \leqq A\_1 \leqq A\_2 \leqq … \leqq A\_N \leqq M \leqq 10$を$0 \leqq A\_1 \leqq A\_2 \leqq … \leqq A\_N \leqq M-1 \leqq 9$としても同様の結果が得られるので、それぞれの数字は一桁で統一して文字列と見なすということです。

```python:C.py
n,m,q=map(int,input().split())
Q=[list(map(int,input().split())) for i in range(q)]
ans=0

R=[range(i,m) for i in range(m)]
def dfs(s,d,l):
    global n,m,Q,ans
    if d==n:
        ans_=0
        for k in Q:
            a,b,c,e=k
            if int(s[b-1])-int(s[a-1])==c:
                ans_+=e
        ans=max(ans,ans_)
    else:
        for i in R[l]:
            dfs(s+str(i),d+1,i)
dfs("",0,0)

print(ans)
```

```python:C.py
from itertools import combinations_with_replacement
n,m,q=map(int,input().split())
Q=[list(map(int,input().split())) for i in range(q)]
ans=0
for s in combinations_with_replacement(range(1,m+1),n):
    ans_=0
    for k in Q:
        a,b,c,d=k
        if s[b-1]-s[a-1]==c:
            ans_+=d
    ans=max(ans_,ans)
print(ans)
```

# [D問題](https://atcoder.jp/contests/abc165/tasks/abc165_d)

まず、xがBの倍数の場合を考えると0になり、**Bの剰余が小さいと計算結果は小さくなるのではないか**と推測しました。このもとで、$x=k \times B+l \ (0 \leqq l \leqq B-1)$とすると、下記の式変形が成り立ちます。


```math
\begin{align}
[\frac{Ax}{B}]-A \times [\frac{x}{B}]&=[Ak+\frac{Al}{B}]-A \times [k+\frac{l}{B}] \\
&=Ak+[\frac{Al}{B}]-Ak+ A \times [\frac{l}{B}] \\
&=[\frac{Al}{B}]+A \times [\frac{l}{B}] \\
&=[\frac{Al}{B}]
\end{align}
```

つまり、結局は$[\frac{Al}{B}]$の最大値を求める問題に帰着できます。ここで、$0 \leqq l \leqq B-1$の仮定および、$[\frac{Al}{B}]$が$l$について(広義)単調増加するので、$l=B-1$で最大になります。また、$n \leqq B-1$の場合はnで最大になるので、求める答えは$[\frac{A \times min(B-1,n)}{B}]$です。

```python:answerD.py
a,b,n=map(int,input().split())
print(a*min(b-1,n)//b)
```
