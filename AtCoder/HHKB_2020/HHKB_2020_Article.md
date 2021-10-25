# 感想

二番目に良いパフォでしたが、目指しているのはここではないので悔しいです。D問題を飛ばしてE問題に移れたのは良かったのですが、**D問題は丁寧に考えれば解けたはずの問題なので悔しいです**。

# [A問題](https://atcoder.jp/contests/hhkb2020/tasks/hhkb2020_a)

Yに等しい場合はupperメソッドで変換します。

```python:A.py
s,t=input(),input()
if s=="Y":
    print(t.upper())
else:
    print(t)
```

# [B問題](https://atcoder.jp/contests/hhkb2020/tasks/hhkb2020_b)

問題文の読解に時間がかかり焦りました。要約すれば、縦2マスまたは横2マスを選択した時にいずれのマスも散らかってないマスである場合の数を求める問題です。

縦2マスか横2マスを選択します。この時、反転させて考えれば縦2マスは横2マスとみなせるので元の行列の横2マスの判定と、反転させた行列の横2マスの判定をそれぞれ行って和を求めます。

```python:B.py
h,w=map(int,input().split())
s=[list(input()) for i in range(h)]
t=[["" for j in range(h)] for i in range(w)]
for i in range(h):
    for j in range(w):
        t[j][i]=s[i][j]
ans=0
for i in range(h):
    for j in range(w-1):
        if s[i][j]=="." and s[i][j+1]==".":
            ans+=1
for i in range(w):
    for j in range(h-1):
        if t[i][j]=="." and t[i][j+1]==".":
            ans+=1
print(ans)
```

# [C問題](https://atcoder.jp/contests/hhkb2020/tasks/hhkb2020_c)

コドフォで良く出る$mex$を求める問題です。$i$番目までに出てきた数を保存する配列$check$と$i$番目の時点での解(出てきてない数の中の最小値)の$mi$を保持しておきます。

この時、**$mi$は単調増加する**ので、$check[mi]!=0$のときは$check[mi]=0$となる$mi$が見つかるまでインクリメントをし、$check[mi]=0$のときは$mi$を出力すれば良いです。

```python:C.py
n=int(input())
p=list(map(int,input().split()))
check=[0]*200001
mi=0
for i in range(n):
    check[p[i]]+=1
    while True:
        if check[mi]>0:
            mi+=1
        else:
            break
    print(mi)
```

# [D問題](https://atcoder.jp/contests/hhkb2020/tasks/hhkb2020_d)

まず、赤い正方形と青い正方形を**複数置くものだと誤読**していました。赤い正方形と青い正方形は一つずつなので、片方を固定した時のもう片方の並べ方を式変形によって$O(1)$で求めることを考えます。ここで、(赤い正方形の一辺:A)$\geqq$(青い正方形の一辺:B)として**赤い正方形を固定して青い正方形を動かす**ことを考えます。この時、赤い正方形は左下の角が$(i,j)\ (0 \leqq i \<N-A,0 \leqq i \<N-A)$にあるものとします。

![](/AtCoder/HHKB_2020/HHKB_2020_1.png)

よって、上記の赤,緑,黄色,青の長方形に青の正方形が入る場合の数を考えれば良いです。また、それぞれ四箇所の重なる部分に青の正方形が含まれる場合は引いて計算する必要があります。ここで、長方形の部分及び重なる部分はそれぞれ対称性より合計をとると等しいので、以下の赤の長方形から青の長方形を引いたものを四倍することで答えは求まります。

![](/AtCoder/HHKB_2020/HHKB_2020_2.png)

(1)赤の長方形について

青の正方形と青の正方形の横幅を考えれば、$B \leqq i \leqq N-A$が成り立ちます。この時、含まれる青の正方形の数は下記のように計算できます。

```math
\begin{align}
&\sum_{i,j}(N-B-1)(i-B+1)\\
&=(N-B-1)\sum_{i,j}(i-B+1)\\
&=(N-B-1)\sum_{j}(\sum_{i}(i-B+1))\\
&=(N-B-1)\sum_{j}(1,2,…,N-A-B+1)\\
&=(N-B-1)\sum_{j}\frac{(N-A-B+1)(N-A-B+2)}{2}\\
&=(N-B-1)(N-A+1)\frac{(N-A-B+1)(N-A-B+2)}{2}\\
\end{align}
```

(2)青の長方形について
先ほどの$B \leqq i \leqq N-A$に加え、$B \leqq j \leqq N-A$も成り立ちます。この時、含まれる青の正方形の数は下記のように計算できます。

```math
\begin{align}
&\sum_{i,j}(j-B-1)(i-B+1)\\
&=\sum_{i}(i-B+1)\times \sum_{j}(j-B+1)\\
&=(\frac{(N-A-B+1)(N-A-B+2)}{2})^2\\
\end{align}
```

上記の計算は$(j-B-1),(i-B+1)$と**それぞれ$i,j$に依存する**ので、和として分離することを用います。一方の項に$i,j$が含まれる場合は分離できないことに注意が必要です。

以上の(1)から(2)を引いたものを四倍して求めれば良いです。また、Pythonを使えば多倍長整数なのでオーバフローは気にしなくて大丈夫で、最後に$10^9+7$で割った余りを求めます。

```python:D.py
mod=10**9+7
for _ in range(int(input())):
    n,a,b=map(int,input().split())
    if a<b:
        a,b=b,a
    x=(n-a-b+1)*(n-a-b+2)//2*(n-a+1)*(n-b+1)*4
    y=((n-a-b+1)*(n-a-b+2)//2)**2*4
    if a+b>n:
        print(0)
        continue
    print((x-y)%mod)
```

# [E問題](https://atcoder.jp/contests/hhkb2020/tasks/hhkb2020_e)

まず、総和とあるので、**照らされるマスが$2^k$通りのうち何回現れるか**を考えます。ここで、あるマスに照明を置いた時に照らされるマスはそのマスから連続した散らかってないマスになります。ここで、いくつかのマスに照明をおいた時、複数の照明があるマスを照らす可能性があります。この時はそのマスを1回として解に加える必要があるので、**そのマスを$x$個の照明が照らせると仮定**してそのマスが照らされるマスとして何回現れるか考えました。ここで、**$x$個のいずれかの照明がついていればそのマスは照らされます**。よって、照明を置くマスの選び方が$2^k$通りでその$x$個のいずれのマスも選ばない選び方が$2^{k-x}$通りなので、少なくとも一つ選ぶ選び方は$2^k-2^{k-x}$通りです($\because$ 包除原理)。

よって、それぞれのマスについていくつのマスから照らされるかを求めて2乗の前計算も行うことで、総和を$O(HW)$で求めることができます。ここで、**あるマスを照らすマスは行または列を共有**しており、行列を転置させれば同様なので、**行を共有するマス**のうちいくつのマスから照らされるかをまずは考えます。

ここで、ある行に$y$個だけ連続した散らかってないマスがあった時、**そこに含まれる任意のマスについて行を共有して照らされるマスの数は$y$**となります。したがって、itertoolsのgroupby関数を使って連続した散らかってないマスをまとめることで、$O(HW)$でそれぞれのマスでの$y$を求めることができます。同様に列を共有するマスのうちいくつのマスから照らされるかを求めて$y$と足し合わせた値から1を引くことで($\because$**そのマスを二回カウント**している)それぞれのマスにおける$x$が求まります。

```python:E.py
mod=10**9+7
h,w=map(int,input().split())
s=[list(input()) for i in range(h)]
k=0
for i in range(h):
    for j in range(w):
        if s[i][j]==".":
            k+=1
t=[["" for j in range(h)] for i in range(w)]
for i in range(h):
    for j in range(w):
        t[j][i]=s[i][j]
po=[0]*(k+1)
po[0]=1
for i in range(1,k+1):
    po[i]=po[i-1]*2
    po[i]%=mod
#行でつながっている部分
from itertools import groupby
check=[[0]*w for i in range(h)]
for i in range(h):
    now=0
    for key,group in groupby(s[i]):
        l=len(list(group))
        if key=="#":
            now+=l
        else:
            for j in range(now,now+l):
                check[i][j]+=l
            now+=l
for i in range(w):
    now=0
    for key,group in groupby(t[i]):
        l=len(list(group))
        if key=="#":
            now+=l
        else:
            for j in range(now,now+l):
                check[j][i]+=l
            now+=l
ans=0
for i in range(h):
    for j in range(w):
        #print(k,k-check[i][j])
        if check[i][j]!=0:
            ans+=(po[k]-po[k-check[i][j]+1])
        ans%=mod
print(ans)
```
