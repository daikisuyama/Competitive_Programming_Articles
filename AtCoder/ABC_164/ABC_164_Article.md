# [A問題](https://atcoder.jp/contests/abc164/tasks/abc164_a)

wがs以上かどうかで場合分けをします。

```python:A.py
s,w=map(int,input().split())
print("unsafe" if w>=s else "safe")
```

# [B問題](https://atcoder.jp/contests/abc164/tasks/abc164_b)

高橋くんの攻撃により青木くんの体力はc→c-b、青木くんの攻撃より高橋くんの体力はa→a-dとなるので、制約も緩いことから攻撃をシミュレートして先に体力が0以下になった方が負けです。

下記のコードでは、checkによりどちらの攻撃のターンかを保存しています。

```python:B.py
a,b,c,d=map(int,input().split())
check=True
while True:
    if check:
        c-=b
        if c<=0:
            print("Yes")
            break
    else:
        a-=d
        if a<=0:
            print("No")
            break
    check=not check
```

#[C問題](https://atcoder.jp/contests/abc164/tasks/abc164_c)

よくあるパターンです。被りなしで何通りかなので、setを使います。

```python:C.py
n=int(input())
s=set()
for i in range(n):
    s.add(input())
print(len(s))
```

#[D問題](https://atcoder.jp/contests/abc164/tasks/abc164_d)

まず、サンプルを見ると、**パターンが少ない**ことに気づきます。そして、**iからjの文字列を愚直に全て書き出すとO($N^2$)で間に合わない**ということも明らかです。

このもとで、$i$文字目から$j$文字目までの数$n$が2019の倍数であるかの判定に、**区間の端を固定する発想**を応用します。つまり、「$i$文字目から最後の文字までの数($k$)」と「$j-1$文字目から最後の文字までの数($l$)」を考え、$k-l$が2019の倍数であるかを判定すれば良いです(∵**2019は2および5と互いに素**)。つまり、$k,l$が2019を法とした時に合同であるかを判定すれば良いです。

よって、$x$文字目から最後の文字までの数$|S|$通りについて2019で割った余りがそれぞれ何通りあるかを辞書に格納し、それぞれの余りzがy通りある際には$\_yC\_2$で$(i,j)$の組み合わせが求まります。この$z$を0\~2018で考えて和を考えれば良いのですが、この方法では**$i$文字目から最後の文字までの数が2019の倍数であるか**を考慮していません。したがって、先ほどの場合に$z$が0となる場合を答えとして足す必要があります。

```python:answerD.py
s=input()
l=len(s)
se=dict()
k=0
for i in range(l-1,-1,-1):
    k+=(pow(10,l-1-i,2019)*int(s[i]))
    k%=2019
    if k in se:
        se[k]+=1
    else:
        se[k]=1

ans=0
for j in se:
    i=se[j]
    if i>1:
        ans+=(i*(i-1)//2)
    if j==0:
        ans+=i
print(ans)
```
