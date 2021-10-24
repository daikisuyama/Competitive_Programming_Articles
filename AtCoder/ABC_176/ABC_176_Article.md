# 感想

D問題が通せずC++の実装でバグをだし酷い状況ではありましたが、E問題を通してギリギリで耐えました。

# [A問題](https://atcoder.jp/contests/abc176/tasks/abc176_a)

一度に$x$個焼けるので、$\lceil \frac{n}{x} \rceil$だけの回数焼けば良いです。また、焼く時に$t$分だけかかるので、$\lceil \frac{n}{x} \rceil \times t$を出力します。

```python:A.py
n,x,t=map(int,input().split())
y=-((-n)//x)
print(y*t)
```

# [B問題](https://atcoder.jp/contests/abc176/tasks/abc176_b)

一文字ずつを数字に変換して和を考えます。したがって、一文字ずつを`int`で変換します。

```python:B.py
n=input()
ans=0
for i in n:
    ans+=int(i)
print("Yes" if ans%9==0 else "No")
```

# [C問題](https://atcoder.jp/contests/abc176/tasks/abc176_c)

前から順に見て踏み台が必要か及び踏み台の高さを決めれば良いです。つまり、$a\_{i-1}>a\_i$の時、踏み台の高さを$a\_{i-1}-a\_i$として順に見ていけば良いです。

```python:C.py
n=int(input())
a=list(map(int,input().split()))
ans=0
for i in range(1,n):
    if a[i]<a[i-1]:
        ans+=(a[i-1]-a[i])
        a[i]=a[i-1]
print(ans)
```

# [D問題](https://atcoder.jp/contests/abc176/tasks/abc176_d)

グリッド探索問題において最小のワープ魔法の回数を求めたいため、通常のBFSやDFSを用いれば良さそうです。しかし、普通に実装すると**移動Bの$5 \times 5$のマスの探索で時間がかかる**ので、効率化を図ります。

まず、移動Aではワープ魔法の回数を使わないので、**移動Aのみで辿ることができる場合は移動Aが最適**であるということです。したがって、**移動Aを優先的に選んで探索する**ようなBFSまたはDFSを考えます。DFSはその探索順序を恣意的に変更するというのが難しいので、BFSを使って考えます。BFSの場合、queueを使って**後ろに挿入**し、**前から順に探索**していきます。しかし、この場合は挿入されたものを順に辿っていくだけです。したがって、ここではBFSでdequeを使い、移動Aをする場合は前に挿入し、移動Bをする場合は後ろに挿入することで、この優先順を実現することができます。また、**コストが0の辺で繋がる場合は前に挿入し、コスト1の辺で繋がる場合は後ろに挿入するようなBFS**を**01-BFS**と呼びます。

また、**移動AとBのいずれもコストを消費する移動**なので、**ダイクストラ法**により解くのもかなり自然な解法であると考えられます。つまり、移動Aをコスト0の辺、移動Bをコスト1の辺とおき、最小のコストでの移動を考える問題に帰着すれば良いです。よって、**コストは非負**であることと合わせれば、**グリッドを頂点とみたダイクストラ法**により$O(HW \log{HW})$で計算することができます。

```python:D.py
#辺の重み(変化量)が0である場合は01DFSって特別なやつ
#0の場合はそこでBFSできる
import sys
from collections import deque
sys.setrecursionlimit(10**7)
input=sys.stdin.readline
h,w=map(int,input().split())
c=[int(i)-1 for i in input().split()]
d=[int(i)-1 for i in input().split()]
s=[input() for i in range(h)]
inf=100000000
dp=[[-1 if s[i][j]=="#" else inf for j in range(w)] for i in range(h)]
dp[c[0]][c[1]]=0
now=deque([[c[0],c[1]]])
#グリッド上のbfsはfor文で
def bfs():
    global h,w,s,dp,now
    while len(now):
        i,j=now.popleft()
        for k,l in [[i-1,j],[i+1,j],[i,j-1],[i,j+1]]:
            if 0<=k<h and 0<=l<w:
                if dp[k][l]!=-1:
                    if dp[k][l]>dp[i][j]:
                        dp[k][l]=dp[i][j]
                        now.appendleft([k,l])
        for k in range(i-2,i+3):
            for l in range(j-2,j+3):
                if 0<=k<h and 0<=l<w:
                    if dp[k][l]!=-1:
                        if dp[k][l]>dp[i][j]+1:
                            dp[k][l]=dp[i][j]+1
                            now.append([k,l])
bfs()
print(dp[d[0]][d[1]] if dp[d[0]][d[1]]!=inf else -1)
```


# [E問題](https://atcoder.jp/contests/abc176/tasks/abc176_e)

まず、**爆破対象の個数が最大の行(MAXR)及び列(MAXC)を選択することが最適**で、答えはMAXR+MAXCです。しかし、その**交差するグリッド上に爆破対象がある場合**、答えはMAXR+MAXC-1です。ここで、爆破対象の個数が最大でない行または列を選択するとその行及び列に含まれる爆破対象の個数はMAXR+MAXC-1以下になるので、**爆破対象の個数が最大の行及び列を選ぶべき**です。

ここで、爆破対象の個数が最大の行及び列は**複数ある可能性**があり、それぞれを格納した配列を`xr`,`xc`とします。この時、これらの行と列の組み合わせは`len(xr) × len(xc)`だけあります。この中で爆破対象が交差する部分にないような行と列の組み合わせが少なくとも一つ存在するかを探します。また、`len(xr) × len(xc)`は最大で$(3 \times 10^6)$通りあるので、**全ての組み合わせを試すのが難しい**です。

したがって、行と列の組み合わせを全て試すのではなく、**逆にそれぞれの爆破対象がその組み合わせに含まれるか**を探します。つまり、`xr`,`xc`をsetに変換し、それぞれの爆破対象のインデックスがこのsetに含まれるかを見ていくことで、この組み合わせの中に爆破対象がいくつあるかを求めることができます(`check`)。`check`が`len(xr) × len(xc)`よりも小さい場合は交差する部分に爆破対象が存在しないような行と列の選び方が少なくとも一つ存在するのでMAXR+MAXCが解で、`check`が`len(xr) × len(xc)`と等しい場合はMAXR+MAXC-1が解となります。


```python:E.py
h,w,m=map(int,input().split())
item=[list(map(int,input().split())) for i in range(m)]
row=[0]*h
col=[0]*w
for i in range(m):
    x,y=item[i]
    row[x-1]+=1
    col[y-1]+=1
mr,mc=max(row),max(col)
xr=set([i for i in range(h) if row[i]==mr])
xc=set([i for i in range(w) if col[i]==mc])
check=len(xr)*len(xc)
for i in range(m):
    r,c=item[i]
    if r-1 in xr and c-1 in xc:
        check-=1
print(mr+mc if check>0 else mr+mc-1)
```
