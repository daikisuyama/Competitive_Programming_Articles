# 感想

C問題の題意を正確に読み取ることができませんでした…。

# [A問題](https://atcoder.jp/contests/abc174/tasks/abc174_a)

30度以上かどうかをチェックします。

```python:A.py
print("Yes" if int(input())>=30 else "No")
```

# [B問題](https://atcoder.jp/contests/abc174/tasks/abc174_b)

距離を求めます。また、ルートで小数になるのが不安なので、二乗しました。

```python:B.py
n,d=map(int,input().split())
ans=0
for i in range(n):
    x,y=map(int,input().split())
    ans+=(x**2+y**2<=d**2)
print(ans)
```

# [C問題](https://atcoder.jp/contests/abc174/tasks/abc174_c)

**$K$の倍数は$K$で割った余りが0**です。また、$7,77,777,…$の$i$番目を$a_i$とすると、$a\_{i+1}=10 \times a\_i +7$になります。したがって、$a\_j,a\_l$の余りが同じであった場合は$a\_{j+1},a\_{l+1}$の余りも同じになります。よって、$a_{K+1}$までで**$K$で割った余りが被るものが存在する**ので、$a\_1$\~$a\_k$までを探せば$K$で割った余りが0のものを探すことができます。また、逆にそこまでで0が出てこなかった場合は$K$で割った余りが0になるものは存在しません。

```python:C.py
k=int(input())
now=7%k
if now==0:
    print(1)
    exit()
for i in range(1,k):
    now=10*now+7
    now%=k
    if now==0:
        print(i+1)
        break
else:
    print(-1)
```


# [D問題](https://atcoder.jp/contests/abc174/tasks/abc174_d)

最終的に"赤赤…赤赤/白白…白白"という並びになることは容易にわかります。まずは与えられた石の列を**貪欲に変える**ために、左から赤に変えていきます。また、赤石を変える必要はないので**白石の時のみ**を考えます。この時、その白石を赤石に変えることを考えますが、その石の色を塗り替える代わりにその石より右側ににある赤石と入れ替えれば**同時に二つの石の色を変えられる**ので効率が良いです。さらに、入れ替える回数が少なくなるように赤石と入れ替えるには**一番右側にある赤石と入れ替える**のが適切であるのはわかります。これを繰り返す中で右側に赤石が一つもない状況が作り出せればその時点での回数が最小の操作回数となります。

```python:D.py
from collections import deque
n=int(input())
c=input()
r=deque()
lr=0
for i in range(n):
    if c[i]=="R":
        lr+=1
        r.append(i)
ans=0
for i in range(n):
    if lr==0:
        break
    if c[i]=="W":
        if i<r[-1]:
            ans+=1
            r.pop()
            lr-=1
        else:
            break

print(ans)
```

# [E問題](https://atcoder.jp/contests/abc174/tasks/abc174_e)

丸太が長いほど切る回数が少ないという単調性を持ち、その丸太の切る回数の最小値を求めたいため、二分探索を実装すれば良いです。

ここで、**二分探索では最小(or最大)のもの以上で常に成り立つ条件を考える**必要があり、「最長の長さ(切り上げ)を$x$以下にするのに少なくとも必要な丸太の切る回数が$K$以下となる」ことがここでは条件となります。つまり、それぞれの丸太に対して`ceil(a[i]/x)-1`を計算して合計したものが$K$以下であれば良いです。ただし、下記のコードの`l,r`については**`l`には必ず成り立たない値**,**`r`には必ず成り立つ値**を設定するので、注意が必要です。

```python:E.py
n,k=map(int,input().split())
a=list(map(int,input().split()))
def f(x):
    global a,n,k
    if x==0:
        return 10**10
    ret=0
    for i in range(n):
        ret+=(-((-a[i])//x)-1)
    return ret

l,r=0,10**10
while l+1<r:
    x=l+(r-l)//2
    if f(x)<=k:
        r=x
    else:
        l=x

print(r)
```
