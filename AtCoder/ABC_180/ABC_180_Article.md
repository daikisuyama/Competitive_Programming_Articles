# 感想

（不備があったので、省略します。）

# [A問題](https://atcoder.jp/contests/abc180/tasks/abc180_a)

$n\< a$のときは問題が不成立ですが、制約より$n \geqq a$が成り立ちます。

```python:A.py
n,a,b=map(int,input().split())
print(n-a+b)
```

# [B問題](https://atcoder.jp/contests/abc180/tasks/abc180_b)

問題文の通りに実装します。実は入力をそのまま絶対値に変換しても実装可能です。

```python:B.py
n=input()
x=list(map(lambda y:abs(int(y)),input().split()))
print(sum(x),sum([i*i for i in x])**0.5,max(x))
```

# [C問題](https://atcoder.jp/contests/abc180/tasks/abc180_c)

分配した際にシュークリームが余ってもいいと誤読して$[\frac{n}{k}]$の値の候補を求める問題だと勘違いしてました。

余らないように分配するので約数を列挙するだけです。

```python:C.py
def m():
    n=int(input())
    d=[]
    for i in range(1,int(n**0.5)+1):
        if n%i==0:
            d+=[i]
            if i!=n//i:d+=[n//i]
    d.sort()
    return d
for i in m():print(i)
```

#　[D問題](https://atcoder.jp/contests/abc180/tasks/abc180_d)

A倍するかBを足すかのいずれの選択かをします。前者と後者で最適な選択をしますが、**前者が一度最適でなくなった場合はそれ以降は最適なのは常に後者**となります。また、これは変化量を$\times A$と$+B$で比べればわかります。

よって、$x$をA倍するのが最適な場合($\leftrightarrow$$x$にBを足した値よりも$x$をA倍した値の方が小さい場合)、を考えます。このとき、$x\times A \leqq x+ B$を比較の条件式で使うのは当然ですが、**$x \<y$を満たすように条件**を置くことも必要です。ループの中を丁寧に実装すれば、ループを抜けた後に$B$を足す計算も$ans+[\frac{y-x-1}{b}]$を求めるだけになります。

```python:D.py
x,y,a,b=map(int,input().split())
ans=0
while True:
    if x*a<=x+b:
        if x*a>=y:
            print(ans)
            exit()
        else:
            ans+=1
            x*=a
    else:
        #こっから+
        #x*a>x+b
        #x<y
        break
print(ans+(y-x-1)//b)
```
