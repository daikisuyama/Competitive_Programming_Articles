# 感想

今回は10分程度遅れ、環境構築も終わらず、といった状況で参加したため、十分なパフォーマンスを出すことができませんでした。

# [A問題](https://atcoder.jp/contests/abc182/tasks/abc182_a)

引き算をして出力するだけです。

```python:A.py
a,b=map(int,input().split())
print(2*a+100-b)
```

# [B問題](https://atcoder.jp/contests/abc182/tasks/abc182_b)

**最大となる数の候補が2\~1000しかないので**、全探索します。

ただし、全探索する際に(最大となる数,GCD度)のペアを持ちながらやる必要があります。

```python:B.py
n=int(input())
a=list(map(int,input().split()))
ans=[2,sum(i%2==0 for i in a)]
for j in range(1000,2,-1):
  x=[j,sum(i%j==0 for i in a)]
  if x[1]>ans[1]:
    ans=[x[0],x[1]]
print(ans[0])
```

# [C問題](https://atcoder.jp/contests/abc182/tasks/abc182_c)

余りが3なので、**桁の合計が3の倍数**になれば良いです。この時、**それぞれの桁を3で割った余りのみ保存**して除きたい桁の選び方を考えます。よって、ここでは$check[i]$:=(余りが$i$の桁数)とします。

まず、桁の合計が初めから0の場合は0を出力します。それ以外の場合は以下のように場合分けをします。

(1)桁の合計の余りが1の時

[1]$check[1] \geqq 1$の時$check[1]-=1$により桁の合計の余りは0となります。
[2]$check[1] \< 1$の時$check[2]-=2$により桁の合計の余りは0となります。
[3]いずれの場合においても$sum(check)=0$になる時は任意の桁を選んで消しているので題意を満たしません。この時は-1を出力します。また、この条件下では$check[2]\<2$となることが示せます。

(2)桁の合計の余りが2の時

[1]$check[2] \geqq 1$の時$check[2]-=1$により桁の合計の余りは0となります。
[2]$check[2] \< 1$の時$check[1]-=2$により桁の合計の余りは0となります。
[3]いずれの場合においても$sum(check)=0$になる時は任意の桁を選んで消しているので題意を満たしません。この時は-1を出力します。また、この条件下では$check[1]\<2$となることが示せます。

```python:C.py
n=input()
l=len(n)
check=[0]*3
for i in range(l):
  check[int(n[i])%3]+=1
if int(n)%3==0:
  print(0)
elif int(n)%3==1:
  if check[1]>=1:
    check[1]-=1
    if sum(check)==0:
      print(-1)
    else:
      print(1)
  else:
    check[2]-=2
    if sum(check)==0:
      print(-1)
    else:
      print(2)
else:
  if check[2]>=1:
    check[2]-=1
    if sum(check)==0:
      print(-1)
    else:
      print(1)
  else:
    check[1]-=2
    if sum(check)==0:
      print(-1)
    else:
      print(2)
```

# [D問題](https://atcoder.jp/contests/abc182/tasks/abc182_d)

誤読して**それぞれの移動は完全にしかできない**と勘違いしてました。その場合は累積和を二回取った後に最大値を考えるだけで何も工夫は要りません。また、以下の考察で使う二つの配列を用意します。$b[i]:=$($i$番目の完全な移動の移動距離)と$c[i]:=$($i$番目までの完全な移動の移動距離の累積最大値)です。

ここでは、**完全には移動しない場合を分けて考える**と良いです。つまり、$i(0 \< i \< n-2)$番目まで完全に移動したとすれば、**$i+1$番目は途中まででも良いので最大となるように移動すれば良い**です。つまり、式で表すなら$(\sum\_{j=0}\^{i}b[j])+c[i]$が$i$番目での解になります。これを任意の$i$で考えれば良いです。よって、$\sum\_{j=0}\^{i}b[j]$の値を管理しながら$i$を順に大きくすることで実装することができます。また、$b$の累積和を取れば楽に実装できます（2つ目のコード）。

また、一回も移動しない場合と全ての移動で完全に移動した場合が上記ではカバーできていないので、これらのケースを加えて最大値を求めます。

```python:D.py
n=int(input())
a=list(map(int,input().split()))
from itertools import accumulate
b=list(accumulate(a))
c=list(accumulate(b,func=max))
ans=[0,b[0]]
now=b[0]
for i in range(1,n):
  ans.append(now+c[i])
  now+=b[i]
ans.append(now)
print(max(ans))
```

```python:D_better.py
n=int(input())
a=list(map(int,input().split()))
from itertools import accumulate
b=list(accumulate(a))
c=list(accumulate(b,func=max))
d=list(accumulate(b))
ans=[0]+[d[i]+c[i] for i in range(n-1)]+[d[n-1]]
print(max(ans))
```

# [E問題](https://atcoder.jp/contests/abc182/tasks/abc182_e)

[HHKB プログラミングコンテスト 2020-E Lamps](https://atcoder.jp/contests/hhkb2020/tasks/hhkb2020_e)の類題です。まず、ある行の**ブロックまたは端で囲まれている部分**があったとします。この部分に**ランプがひとつでも置いてあれば**その部分は照らされます。逆にこの場合でなければランプで照らされることはありません。列の時も考えて、**いずれかの場合で照らされるマスの数が答え**です。行と列の場合はそれぞれ転置して考えれば良いので、行の時のみをまずは考えます。

まず、このような**連続した部分をまとめて扱いたい場合はランレングス圧縮**を利用します。つまり、初めの状態でランプの置かれているマスを1,ブロックの置かれているマスを0,何も置かれていないマスを-1とします。この時、ブロックがない部分を考えたいので、**あるマスが0であるかで圧縮**を行えば良いです。すなわち下図のようになります。

![](/AtCoder/ABC_182/ABC_182_1.png)

そして、上図のように圧縮を行った後、**圧縮した部分に1がひとつでも含まれていれば**実線の部分を1(照らされるマス)とすれば良いです。また、groupbyを使ってfuncに$lambda \  x:x==0$を指定すれば先ほどの圧縮を行うことができます。

```python:E.py
from itertools import groupby
h,w,n,m=map(int,input().split())
check1=[[-1]*w for i in range(h)]
check2=[[-1]*h for i in range(w)]
for i in range(n):
  a,b=map(int,input().split())
  check1[a-1][b-1]=1
  check2[b-1][a-1]=1
for i in range(m):
  c,d=map(int,input().split())
  check1[c-1][d-1]=0
  check2[d-1][c-1]=0
#print(check1)
#print(check2)
for i in range(h):
  x=[(key,list(group)) for key,group in groupby(check1[i],key=lambda x:x==0)]
  now=0
  for k,g in x:
    if not k and 1 in g:
      for j in g:
        check1[i][now]=1
        now+=1
    else:
      now+=len(g)
for i in range(w):
  x=[(key,list(group)) for key,group in groupby(check2[i],key=lambda x:x==0)]
  now=0
  for k,g in x:
    if not k and 1 in g:
      for j in g:
        check2[i][now]=1
        now+=1
    else:
      now+=len(g)
ans=0
for i in range(h):
  for j in range(w):
    if check1[i][j]==1 or check2[j][i]==1:
      ans+=1
#print(check1)
#print(check2)
print(ans)
```
