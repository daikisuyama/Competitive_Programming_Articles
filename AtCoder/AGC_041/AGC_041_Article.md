# 今回の成績

![](/AtCoder/AGC_041/AGC_041_1.png)

初めて二完できたので及第点ですが、C問題は発想を少し転換すればいいだけだったので、三完したかったです。

# [A問題](https://atcoder.jp/contests/agc041/tasks/agc041_a)


比較的すぐ解法を思いついたのですが、早とちりして明らかに間違っている解法を提出し、C++のコードとしてPythonのコードを提出したので2WAを吐きました。  
まず、一番わかりやすいのは卓Aと卓Bの距離が偶数である場合です。この場合はそれぞれの卓にいる卓球選手がお互いに卓の距離を2ずつ縮めていけばよく、距離を2で割ったものを出力します。  
次に、卓Aと卓Bの距離が奇数である場合を考えます。この場合、端(卓1or卓N)まで行ってそこに一つのラウンドとどまることで二人のいる卓の距離を偶数にすることができます。この際、端の近い方を考えれば良く、以下のコードのc,dの二つでminの方を出力します。  
この問題で重要なのは、**遷移の際の状態を分類することです**。この問題では卓1で勝利する場合と卓Nで敗北する場合で**のみ**他の卓に移らないことがわかります。  
また、両者が共に近づく場合が明らかに最小になり、そのような場合はその距離が偶数になるため、端にいる場合は偶奇が変化すると考えれば容易に考察することができます。

```python:answerA.py
x=[]
n,a,b=map(int,input().split())
if (b-a)%2==0:
    print((b-a)//2)
else:
    c=(a-1)+(b-a+1)//2
    d=(n-b)+(b-a+1)//2
    print(min(c,d))
```

# [B問題](https://atcoder.jp/contests/agc041/tasks/agc041_b)

とりあえず降順に人を並べます。この時、$A\_1~A\_p$の問題はP問がコンテストに採用されることから無条件に採用することができます。  
ここで、そのP個の問題のうちの一番点の低い$A\_p$の問題の代わりにそれよりも小さい問題を選ぶことができればその問題をコンテストに採用することができます。  

まず、$v\leqq p$の場合は$A\_1\~A\_{p-1}$と$A\_j(j\geqq p+1)$に投票し続け、$A\_j$が$A\_p$を超えることができれば良いです。  
それに対し、$v\>p$の場合は$A\_j(j\geqq p+1)$について$A\_1\~A\_{p-1}$及び$A\_j$に全員のジャッジが投票してもまだ票が余っているため、**$A\_j$が$A\_p$を超えることができれば良いだけでは判断できない**ことがわかります。  
また、ジャッジの残りの票の選び方を考えた時、$A\_k(k\>j+1)$に投票しても$A\_j$を超えることにも気づかねばなりません。  

つまり、$A\_1\~A\_{p-1}$、$A\_j\~A\_n$に投票し、投票した問題の点数を+mし、ジャッジの残りの票を$A\_p\~A\_{j-1}$に入れても、$A\_j$が任意のlについて$A\_l(p \leqq l \leqq j-1)$以上であれば$A\_j$をコンテストに採用することができると言えます。    
また、**ジャッジの残りの票を$A\_p$がA$\_j$を超えないよう最大限投票**した時に、$A\_p\~A\_{j-1}$が$A\_j$に等しくなることがわかります(この時、$A\_p\~A\_{j-1}$は元の$A\_j$よりも大きいのでそれぞれに最大限投票してもmを超えないことが保証されます。)。  
また、降順に並んでおり、**ある$A\_j$を境にコンテストに採用できなくなるので、そのような$A\_j$は二分探索で探すことができます**。

```python:answerB.py
n,m,v,p=map(int,input().split())
a=[int(i) for i in input().split()]
a.sort(reverse=True)
a=a[p-1:]
le=len(a)
l,r=0,le-1

def adopt(x):
    global a,n,m,v,p
    b=a[x]+m
    #(1)
    if b<a[0]:
        return False
    al=v*m
    al-=(n-x)*m
    #(2)
    if al<=0:
        return True
    s=x*b-sum(a[:x])
    #(3)
    if al>s:
        return False
    else:
        return True

while l+1<r:
    k=(l+r)//2
    if adopt(k):
        l=k
    else:
        r=k

if adopt(r):
    print(p+r)
else:
    print(p+l)
```


```python:コンテスト後にshortestとfastest両方狙いに行ったコード
n,m,v,p=map(int,input().split())
a=sorted(list(map(int, input().split())),reverse=True)[p-1:]
l,r=0,n-p
while l+1<r:
    k=(l+r)//2
    b=a[k]+m
    if b<a[0] or (v-(n-k))*m>(k*b-sum(a[:k])): r=k
    else: l=k
b=a[r]+m
print(p+l if(b<a[0] or (v-(n-r))*m>(r*b-sum(a[:r]))) else p+r)
```

# [C問題](https://atcoder.jp/contests/agc041/tasks/agc041_c)

コンテスト中には解けませんでした。コンテスト中の考察により縦長にドミノを置いた場合と横長にドミノを置いた場合の数を均等にする必要があり、均等にした場合はクオリティを3にすることで条件を満たすドミノの配置が作れることがわかりました(n=2,3の場合は除く)。  
ここで対称性を考えてドミノを配置していったところnが3の倍数では対角線にそって順に縦横にドミノを配置することでクオリティ1のドミノの配置を見つけることができました。しかし、nが3の倍数ではないときは対称性の高い配置を見つけることができませんでした(**ここで別の方法を考えるべき。**)。  
そこで、**大きい数のものは分割するとうまくいくことが多いので**、この問題でもそのようにして考えます。i行目からj行目かつi列目からj列目の部分正方行列に注目して考えると、この正方行列以外のi行目からj行目とi列目からj列目の部分が0であれば部分正方行列のクオリティがi行目からj行目およびi列目からj列目になることがわかります。したがって、そのような部分正方行列で構成すれば良いことがわかります。先ほどのクオリティが3である行列を考えると、4次以上の正方行列では見つかりそうな雰囲気を感じます。ここからは**頑張ってそのような正方行列を探していくことになります**。  
[解答](https://img.atcoder.jp/agc041/editorial.pdf)の方法を見ると、3,4,5,6,7次の正方行列を見つけ、mod4で場合分けをして正方行列の対角線に順に正方行列を配置していけば良いです。(上から数えると4次,4次,4次,…,(4or5or6or7)次と並びます。)僕は、nが3の倍数の時は簡単な配置があることを利用してmod6で場合分けをしましたが、3,4,5,6,7,8次の正方行列を見つける必要があるので解答の方法がおそらく一番簡単な方法になると思われます。

```python:answerC.py
n=int(input())

if n==2:
    print(-1)
else:
    x=[]
    if n%3==0:
        for i in range(n//3):
            x.append("."*(3*i)+"a"+"."*(n-3*i-1))
            x.append("."*(3*i)+"a"+"."*(n-3*i-1))
            x.append("."*(3*i)+".aa"+"."*(n-3*i-3))
    elif n%6==1:
        for i in range(n//6-1):
            x.append("."*(6*i)+".a.b.c"+"."*(n-6*i-6))
            x.append("."*(6*i)+".a.b.c"+"."*(n-6*i-6))
            x.append("."*(6*i)+"ddg.ll"+"."*(n-6*i-6))
            x.append("."*(6*i)+"e.g.kk"+"."*(n-6*i-6))
            x.append("."*(6*i)+"e.hhj."+"."*(n-6*i-6))
            x.append("."*(6*i)+"ffiij."+"."*(n-6*i-6))
        x.append("."*(n-7)+".aab.c.")
        x.append("."*(n-7)+"d..b.c.")
        x.append("."*(n-7)+"d..eeff")
        x.append("."*(n-7)+"g..mm.l")
        x.append("."*(n-7)+"gnn...l")
        x.append("."*(n-7)+"h...kkj")
        x.append("."*(n-7)+"hii...j")
    elif n%6==2:
        for i in range(n//6-1):
            x.append("."*(6*i)+".a.b.c"+"."*(n-6*i-6))
            x.append("."*(6*i)+".a.b.c"+"."*(n-6*i-6))
            x.append("."*(6*i)+"ddg.ll"+"."*(n-6*i-6))
            x.append("."*(6*i)+"e.g.kk"+"."*(n-6*i-6))
            x.append("."*(6*i)+"e.hhj."+"."*(n-6*i-6))
            x.append("."*(6*i)+"ffiij."+"."*(n-6*i-6))
        x.append("."*(n-8)+".a.bb.cc")
        x.append("."*(n-8)+".a...m.j")
        x.append("."*(n-8)+"..pp.m.j")
        x.append("."*(n-8)+"hh..i.o.")
        x.append("."*(n-8)+"gg..i.o.")
        x.append("."*(n-8)+"..n.ll.k")
        x.append("."*(n-8)+"f.n....k")
        x.append("."*(n-8)+"f.dd.ee.")
    elif n%6==4:
        for i in range(n//6):
            x.append("."*(6*i)+".a.b.c"+"."*(n-6*i-6))
            x.append("."*(6*i)+".a.b.c"+"."*(n-6*i-6))
            x.append("."*(6*i)+"ddg.ll"+"."*(n-6*i-6))
            x.append("."*(6*i)+"e.g.kk"+"."*(n-6*i-6))
            x.append("."*(6*i)+"e.hhj."+"."*(n-6*i-6))
            x.append("."*(6*i)+"ffiij."+"."*(n-6*i-6))
        x.append("."*(n-4)+"aacb")
        x.append("."*(n-4)+"ffcb")
        x.append("."*(n-4)+"hgdd")
        x.append("."*(n-4)+"hgee")
    else:
        for i in range(n//6):
            x.append("."*(6*i)+".a.b.c"+"."*(n-6*i-6))
            x.append("."*(6*i)+".a.b.c"+"."*(n-6*i-6))
            x.append("."*(6*i)+"ddg.ll"+"."*(n-6*i-6))
            x.append("."*(6*i)+"e.g.kk"+"."*(n-6*i-6))
            x.append("."*(6*i)+"e.hhj."+"."*(n-6*i-6))
            x.append("."*(6*i)+"ffiij."+"."*(n-6*i-6))
        x.append("."*(n-5)+"aabbc")
        x.append("."*(n-5)+"g.h.c")
        x.append("."*(n-5)+"gjh..")
        x.append("."*(n-5)+"dj.ii")
        x.append("."*(n-5)+"deeff")
    for i in range(n):
        print("".join(x[i]))
```
