# 感想

3週間ぶりのコンテストで緊張して実力を十分に発揮できずに終わってしまいました。

具体的には、C問題までは合計約7分で終わらせたにも関わらず、D問題以降が過度な緊張によって頭がバグって解けませんでした。

# [A問題](https://atcoder.jp/contests/abc160/tasks/abc160_a)

A問題は三項演算子を使うと早いです。

```python:A.py
s=input()
print("Yes" if s[2]==s[3] and s[4]==s[5] else "No")
```

# [B問題](https://atcoder.jp/contests/abc160/tasks/abc160_b)

500→5の順に優先度が高いため、そのように計算する。

```python:B.py
x=int(input())
y=x//500
z=(x-y*500)//5
print(y*1000+z*5)
```

# [C問題](https://atcoder.jp/contests/abc160/tasks/abc160_c)

それぞれの家の間を一回だけ通って全ての家を通るのが最短距離です。ここで、1番目の家からN番目の家まで順に辿ると仮定すると、一周Kメートルの湖の周りよりも1番目の家とN番目の家の間の距離の分だけ短くなることがわかります。したがって、Kメートル-(隣り合う家の距離のうち最大のもの)が求める答えとなります。

```python:C.py
k,n=map(int,input().split())
a=list(map(int,input().split()))
ma=0
for i in range(n-1):
    ma=max(ma,a[i+1]-a[i])
print(k-max(ma,a[0]+k-a[n-1]))
```

# [D問題](https://atcoder.jp/contests/abc160/tasks/abc160_d)

$n=10^3$なので$O(n^2)$でもACできるという基本的なことに気づかず時間を溶かしました。

まず、この問題で特徴的なのはXとYが繋がれている点です。さらに、最短距離なのでX-Yは一回しか通らないことに注意すると(i,j)を定めた時に**X-Yを通るか通らないか**で場合分けすれば良いのではないかと考えることができます。

上記の発想に至れば、X-Yを通らない場合の最短距離はj-iで、通る場合の最短距離は$abs(X-i)+abs(j-Y)+1$とわかります(1はX-Yの最短距離)。これを任意の(i,j)に試しても$O(n^2)$であり、最短距離1~n-1のどれになるかを別に用意した配列に順に記録していけば答えを求めることができます。

```python:answerD.py
n,x,y=map(int,input().split())
ans=[[1000000]*n for i in range(n)]
for i in range(n-1):
    for j in range(i+1,n):
        ans[i][j]=min(j-i,abs(x-1-i)+1+abs(y-1-j))
_ans=[0]*(n-1)
for i in range(n-1):
    for j in range(i+1,n):
        _ans[ans[i][j]-1]+=1
for i in range(n-1):
    print(_ans[i])
```

# [E問題](https://atcoder.jp/contests/abc160/tasks/abc160_e)

**問題文の誤読**によって**無色のりんごを一個以上選ばなければならない**と勘違いしていました…。また、赤のりんごと緑のりんごを選びながら同時に無色のりんごを選んでいたため実装がかえって複雑になっていました。

同時に選んだ場合、赤のりんごと緑のりんごを**どちらから先に選んだか**で対称性が崩れるため、数え忘れるケースが出てきてしまいます。そこで、その対称性を保つために、赤のりんごをx個と緑のりんごをy個だけ美味しさが大きいものから順に**無色のりんごの美味しさによらずに**選びます(それぞれx個より多くy個より多く選んでも食べるx+y個の候補に含まれないことに注意する必要があります。)。

この元で、無色のりんごをいくつ選ぶか考えればよく、美味しさの大きい順に選んでいきます。また、選んだx個の赤のりんごとy個の緑のりんごのどちらとも無色のりんごは交換可能なので、**赤のりんごと緑のりんごをまとめて**美味しさの小さい順に並べ、小さいものから順に無色のりんごと入れ替えていけば良いです。

また、自分は見た瞬間に類題を思い出してheapqを使ったのですが、きちんと方針を立てていればheapqも使わずに解けるようでした([Writer解](https://img.atcoder.jp/abc160/editorial.pdf))。1つ目がheapqを使った解法のコード、2つ目がheapqを使わない解法のコードになります。

```python:answerE.py
import heapq
x,y,a,b,c=map(int,input().split())
def _int(x):
    return -int(x)
p=list(map(_int,input().split()))
q=list(map(_int,input().split()))
r=list(map(_int,input().split()))
heapq.heapify(p)
heapq.heapify(q)
heapq.heapify(r)

ans=[]
heapq.heapify(ans)
for i in range(x):
    _p=heapq.heappop(p)
    heapq.heappush(ans,-_p)
for i in range(y):
    _q=heapq.heappop(q)
    heapq.heappush(ans,-_q)
#ansは小さい順、rは大きい順
for i in range(x+y):
    if len(r)==0:break
    _ans=heapq.heappop(ans)
    _r=-heapq.heappop(r)
    if _r>=_ans:
        heapq.heappush(ans,_r)
    else:
        heapq.heappush(ans,_ans)
        break
print(sum(ans))
```

```python:answerE_better.py
x,y,a,b,c=map(int,input().split())

p=sorted(list(map(int,input().split())),reverse=True)[:x]
q=sorted(list(map(int,input().split())),reverse=True)[:y]
r=sorted(list(map(int,input().split())),reverse=True)

p.extend(q)
ans=sorted(p)

#ansは小さい順、rは大きい順
for i in range(x+y):
    if len(r)==i or r[i]<ans[i]:
        break
    else:
        ans[i]=r[i]
print(sum(ans))
```
