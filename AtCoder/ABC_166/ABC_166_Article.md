# 感想

（不備があったので、省略します。）

# [A問題](https://atcoder.jp/contests/abc166/tasks/abc166_a)

ABCかARCかを出力します。

```python:A.py
s=input()
print("ABC" if s=="ARC" else "ARC")
```

# [B問題](https://atcoder.jp/contests/abc166/tasks/abc166_b)

どれか一つでもお菓子を持っていればいいので、k個のお菓子を順に見てそのお菓子を持っていればそれぞれの人に対応する配列の要素を1に変えていきます。

このもとで、配列の要素が1でないものを数えれば良いので、以下のようなコードとなります。

```python:B.py
n,k=map(int,input().split())
ans=[0]*n
for i in range(k):
    d=int(input())
    a=list(map(int,input().split()))
    for j in a:
        ans[j-1]=1
print(n-sum(ans))
```

# [C問題](https://atcoder.jp/contests/abc166/tasks/abc166_c)

問題をよく読んでいなかったため、UnionFindと勘違いしていました。

$A\_j,B\_j$を結ぶ道jがあった時、$A\_j \leqq B\_j$の時は$A\_j$は良い展望台ではありません。また、$B\_j \leqq A\_j$の時は$B\_j$は良い展望台ではありません。

したがって、それぞれの展望台について初めは良い展望台(1)と仮定し、道の情報から良い展望台ではないことがわかったら0にすることで、最終的に良い展望台のみが1になり、総和を考えれば良いことになります。

```python:C.py
n,m=map(int,input().split())
h=list(map(int,input().split()))
x=[1]*n
for i in range(m):
    a,b=map(int,input().split())
    if h[a-1]>=h[b-1]:
        x[b-1]=0
    if h[a-1]<=h[b-1]:
        x[a-1]=0
print(sum(x))
```


# [D問題](https://atcoder.jp/contests/abc166/tasks/abc166_d)

まず、Aが負である時Bが正の場合は$A^5-B^5<0$でX(>0)に等しくならないので、Aが負の時はBは必ず負です。また、AもBも負で$A^5-B^5=X$になる場合、A,Bを入れ替えてAとBのどちらの符号も反対(正)にすればAとBは正の数の組として存在するので、**Aは正の数として良い**です。この時、$A\>B$も仮定できます。さらに、$B^5=A^5-X$よりAを決めればBをO(1)で求めることができるので、Aの範囲を調べることにしました。

まず、Aを固定した時、$A^5-B^5$の最小値は$A>B$の元では$B=A-1と$なります($\because$ Bが最大になれば良い)。さらに、$A^5-(A-1)^5$はAについて単調増加です。ここで例えば$A=10^3$とすると$B=10^3-1$であり、$A^5-B^5=4.990009995001 \times 10^{12}$となります。したがって、$A=10^3$の時$A^5-B^5> 10^{12} >X$なので、Aは$10^3$より小さいことが**必要です**。よって、条件を満たす$(A,B)$の組は少なくとも一つ存在することが保証されるので、Aの範囲は1から$10^3$まで考えれば**十分です**。

このもとで、$B=(A^5-X)\^{\frac{1}{5}}$ならば$B^5=A^5-X$であることから、それぞれの$A^5-X$について5乗根をとってBを求め、その数が整数であるかを確かめれば良いです。また、誤差を避けるために、$(A^5-X)^{\frac{1}{5}}$をint関数で整数に直したものを5乗して元の$A\^5-X$に一致するかで確認しました。この操作を繰り返し、一致する(A,B)の組が現れたらbreakすることで(A,B)の組を求めることができます。

```python:answerD.py
x=int(input())
for A in range(10**3):
    k=A**5-x
    if k>=0:
        l=int(k**0.2)
        if l**5==k:
            print(A,l)
            break
    else:
        k=-k
        l=int(k**0.2)
        if l**5==k:
            print(A,-l)
            break
```

# [E問題](https://atcoder.jp/contests/abc166/tasks/abc166_e)

片方決めればどうなるか？という方針の元で実験しているうちに**一旦立式してみよう**という考えに至ったので、ペアの二人の間に成り立つ関係式を立てました。つまり、番号Kと番号Lの参加者(組み合わせを考えるので$K\<L$として良い)がペアになる時、$A\_K+A\_L=L-K$が成り立ちます。この式はさらに式変形することで、$A\_K+K=L-A\_L$となります。

この時、身長は正なので、$K \geqq L$の時は$A\_K + K \geqq A\_K+L > -A\_L+L =L-A\_L$であり、先ほどの$A\_K + K= L- A\_L$は成り立ちません。よって、$A\_i + i$ ($i=1,2,…,N$),$j- A\_j$($j=1,2,…,N$)をそれぞれ計算して値が一致する$(i,j)$は**重複なく条件を満たすペア**として認められます。

以上より、$A\_i + i$ ($i=1,2,…,N$) , $j- A\_j$($j=1,2,…,N$)をそれぞれ辞書で管理し、両方に含まれる数に対して何通りあるかを計算すれば良いです。

```python:answerE.py
n=int(input())
a=list(map(int,input().split()))
c,d=dict(),dict()
for i in range(n):
    if i+1-a[i] in c:
        c[i+1-a[i]]+=1
    else:
        c[i+1-a[i]]=1
    if i+1+a[i] in d:
        d[i+1+a[i]]+=1
    else:
        d[i+1+a[i]]=1
ans=0
for i in c:
    if i in d:
        ans+=(c[i]*d[i])
print(ans)
```

# [F問題](https://atcoder.jp/contests/abc166/tasks/abc166_f)

まず、A+B+Cは一定であり、ゲームをする問題なので**状態に注目**すると良いです。

上記を考慮しつつ実験を行うと、貪欲にそれぞれの選択を決められることがわかります。すなわち、「できるだけ**小さい数は引かず大きい数は引く**」という方法であれば、**それぞれの選択で0を避ける**ことができます。

この時、**選択できる数のどちらも0であるために0を避けられないパターン**があることに注意が必要です。したがって、0を避けるだけでなく、その次の選択でこのパターンにならないようにしたいです。今の選択がX回目であるとすると、「X回目の選択で選ぶことのできない数が0(①)」かつ「X+1回目の選択に使用する文字列がX回目の選択に使用する文字列と違う(②)」かつ「X回目の選択とX+1回目の選択の両方で選ぶ数が1でその数に対してX回目に-1の操作を行う(③)」パターンが**X+1回目に選択できる数のどちらも0であるパターン**なので、①かつ②かつ③の場合に、X回目の選択とX+1回目の選択の両方で選ぶ数に対してX回目に+1の操作を行えば良いです。


```python:answerF.py
n,a,b,c=map(int,input().split())
s_=input().split()
ans=""
for i in range(n):
    s1,s2=s_[i]
    if s2=="B":
        if i!=n-1:
            if s_[i+1]=="AC" and c==0 and a==1:
                a,b,ans=(a+1,b-1,ans+"A")
            elif s_[i+1]=="BC" and c==0 and b==1:
                a,b,ans=(a-1,b+1,ans+"B")
            else:
                a,b,ans=(a-1,b+1,ans+"B") if a>b else (a+1,b-1,ans+"A")
        else:
            a,b,ans=(a-1,b+1,ans+"B") if a>b else (a+1,b-1,ans+"A")
        if min(a,b)==-1:
            print("No")
            break
    elif s1=="A":
        if i!=n-1:
            if s_[i+1]=="AB" and b==0 and a==1:
                a,c,ans=(a+1,c-1,ans+"A")
            elif s_[i+1]=="BC" and b==0 and c==1:
                a,c,ans=(a-1,c+1,ans+"C")
            else:
                a,c,ans=(a-1,c+1,ans+"C") if a>c else (a+1,c-1,ans+"A")
        else:
            a,c,ans=(a-1,c+1,ans+"C") if a>c else (a+1,c-1,ans+"A")
        if min(a,c)==-1:
            print("No")
            break
    else:
        if i!=n-1:
            if s_[i+1]=="AB" and a==0 and b==1:
                b,c,ans=(b+1,c-1,ans+"B")
            elif s_[i+1]=="AC" and a==0 and c==1:
                b,c,ans=(b-1,c+1,ans+"C")
            else:
                b,c,ans=(b-1,c+1,ans+"C") if b>c else (b+1,c-1,ans+"B")
        else:
            b,c,ans=(b-1,c+1,ans+"C") if b>c else (b+1,c-1,ans+"B")
        if min(b,c)==-1:
            print("No")
            break
else:
    print("Yes")
    for j in range(n):
        print(ans[j])
```
