# 感想

（不備があったため、省略します。）

# [A問題](https://atcoder.jp/contests/abc161/tasks/abc161_a)

（不備があったため、省略します。）

# [B問題](https://atcoder.jp/contests/abc161/tasks/abc161_b)

（不備があったため、省略します。）

# [C問題](https://atcoder.jp/contests/abc161/tasks/abc161_c)

nがk以上の場合はnがn-kに置き換えられ、nがk以下の場合はk-nに置き換えられます。このとき、1つ目の操作ではkで割った余りは変わらず、1つ目の操作ではn%kとk-n%kの間を行き来します。

よって、`min(n%k,k-n%k)`を出力すれば良いです。

```python:C.py
n,k=map(int,input().split())
n=n%k
print(min(n,k-n))
```

# [D問題](https://atcoder.jp/contests/abc161/tasks/abc161_d)

まず樹形図を書きました。樹形図を書いているうちに**i桁目とi+1桁目の間に関係があるのではないか**と気づきました。そこで**1桁目から2桁目でどうなるか実験**してみると以下のようになりました。

![](/AtCoder/ABC_161/ABC_161_1.png)

実験の結果から、ルンルン数を文字列として扱えばi桁のルンルン数の後ろ側に"i桁のルンルン数のi桁目との差の絶対値が1以下"の**数字を付け足せば**i+1桁目のルンルン数を作ることができることがわかります。また、上記の実験から、**i桁目が0または9の場合は後ろに付け足す数字は二通りでそれ以外の場合は三通りである**ことがわかります。(✳︎)

したがって、それぞれの桁について上記の操作によって配列に格納していき、**その配列に含まれる要素の合計がkを超えたら上記の操作を終わらせ**、その中のk番目を求めれば良いです。

(✳︎)…コンテスト中は前側にも付け足してしまっていたので重複をsetで除いていました（1つ目のコード）。二つ目のコードは後ろ側のみに付け足しているコードです。

```python:answerD.py
k=int(input())
ans=[[i for i in range(1,10)]]
d=9
while d<k:
    ans.append([])
    for i in ans[-2]:
        x=str(i)
        y=int(x[0])
        if y==1:
            ans[-1].append(str(y)+x)
            ans[-1].append(str(y+1)+x)
        elif 2<=y<=8:
            ans[-1].append(str(y-1)+x)
            ans[-1].append(str(y)+x)
            ans[-1].append(str(y+1)+x)
        else:
            ans[-1].append(str(y-1)+x)
            ans[-1].append(str(y)+x)
        z=int(x[-1])
        if z==0:
            ans[-1].append(x+str(z))
            ans[-1].append(x+str(z+1))
        elif 1<=z<=8:
            ans[-1].append(x+str(z-1))
            ans[-1].append(x+str(z))
            ans[-1].append(x+str(z+1))
        else:
            ans[-1].append(x+str(z-1))
            ans[-1].append(x+str(z))
    ans[-1]=list(set(ans[-1]))
    d+=len(ans[-1])
l=len(ans[-1])
v=sorted([int(i) for i in ans[-1]])

print(v[k-(d-l)-1])
```



# [E問題](https://atcoder.jp/contests/abc161/tasks/abc161_e)

区間→"imosか累積和"という短絡的な思考を当て嵌めようとしましたが、この問題はそのような思考では解くのが難しいです。

まず、全ての基本である貪欲法を用いて、**前から順に働ける日を考えていく**とします。この時、i番目に選んだ働ける日は、働ける日を前から選んだ時に**i番目に選び得るの中で最も早い日**ということを意味します。つまり、**i番目に選ぶ日はその日以降にしか現れない**ということです。逆に、**後ろから順に働ける日を考えていく**と、j番目に選んだ働ける日は**その日以前にしか現れません**。また、働くのはちょうどk日なので後ろから数えてj番目の日は前から数えるとk-j+1番目の日であることに注意が必要です。

以上より、前から数えてi番目に選ぶ日について、**x日以降かつy日以前でなければならない**という情報が得られました。ただ、x\<yの場合はi番目の日として複数の候補がありますが、**x＝yの場合はi番目の日としてx(=y)以外に候補がありません**。よって、前からと後ろからでそれぞれ順番に働ける日をk日数えて、前から数えてi番目に働く日が同じ場合のみ出力すれば良いです。

```python:answerE.py
n,k,c=map(int,input().split())
s=input()
l=[-1]*k
r=[-1]*k
nowl=0
indl=0
while nowl<n and indl<k:
    for i in range(nowl,n):
        if s[i]=="o":
            l[indl]=i
            nowl=i+c+1
            indl+=1
            break
nowr=n-1
indr=k-1
while nowr>=0 and indr>=0:
    for i in range(nowr,-1,-1):
        if s[i]=="o":
            r[indr]=i
            nowr=i-c-1
            indr-=1
            break
for i in range(k):
    if l[i]==r[i]:
        print(l[i]+1)
```



# [F問題](https://atcoder.jp/contests/abc161/tasks/abc161_f)

$k$が**nの約数である場合**、**nをkで割り切れなくなるまでkで割る**と、その後は**nをn-kで置き換える**ことになります。したがって、C問題と同様に考えれば、**n mod kが1であれば最終的にnは1になる**と言えます。**kがnの約数でない場合、nをn-kで置き換える操作しかできない**ので、n mod kが1になるかのみをチェックすれば良いです。

また、n mod kが1になる時、nを1にするまでn-kに入れ替える操作をm回行った時にm*k+1=n$\leftrightarrow$l*k=n-1になることから、**kがn-1の約数であれば良い**ことがわかります。(✳︎)

以上を実装しますが、下記のコードではmake_divisors関数で約数を全て求め、それぞれの約数をkとしてn mod kのチェックを行いました。

```python:answerF.py
def make_divisors(n):
    divisors=[]
    for i in range(1,int(n**0.5)+1):
        if n%i==0:
            divisors.append(i)
            if i!=n//i:
                divisors.append(n//i)
    #約数の小さい順にソートしたい場合
    #divisors.sort()
    #約数の大きい順にソートしたい場合
    #divisors.sort(reverse=True)
    return divisors

n=int(input())
x=make_divisors(n)
l=len(x)
ans=0
for i in range(l):
    k=n
    if x[i]==1:
        continue
    while k%x[i]==0:
        k//=x[i]
    if k%x[i]==1:
        ans+=1

y=make_divisors(n-1)
l2=len(y)
ans2=0
for i in range(l2):
    k=n
    if y[i]==1:
        continue
    while k%y[i]==0:
        k//=y[i]
    if k%y[i]==1:
        ans2+=1
print(ans+ans2)
```

```python:answerF_better.py
def make_divisors(n):
    divisors=[]
    for i in range(1,int(n**0.5)+1):
        if n%i==0:
            divisors.append(i)
            if i!=n//i:
                divisors.append(n//i)
    #約数の小さい順にソートしたい場合
    #divisors.sort()
    #約数の大きい順にソートしたい場合
    #divisors.sort(reverse=True)
    return divisors

def all_pattern(l):
    global n
    ans=0
    for ds in make_divisors(l)[1:]:
        k=n
        while k%ds==0:
            k//=ds
        ans+=(k%ds==1)
    return ans

n=int(input())
print(all_pattern(n)+all_pattern(n-1))
```
