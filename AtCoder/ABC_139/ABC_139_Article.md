# 結果と感想

4回目にして茶色になることができました。

Fは境界部分の場合分けさえできればできそうで、今回は全体的に感触が良かったので、もう少しパフォーマンスをあげたかったです。次のコンテストも頑張って、今月中に緑になる目標を達成したいです。

![](https://github.com/daikisuyama/Old_Competitive_Programming_Articles/blob/main/AtCoder/ABC_139/ABC_139_1.png)


# [A問題](https://atcoder.jp/contests/abc139/tasks/abc139_a)

1日目から順に、同じ天気であるならcをプラス1し、合計のcを出力します。

```python:answerA.py
s=input()
t=input()
c=0
for i in range(3):
    if s[i]==t[i]:
        c+=1

print(c)
```

# [B問題](https://atcoder.jp/contests/abc139/tasks/abc139_b)

どの未使用の差込口を使ってもA口の電源タップによって差込口はA-1口だけ増えます。また、**B口以上**にするにはB-1口(元々の差込口のぶんを除いた)ぶん以上の差込口を増やせばいいので、電源タップを増やしていった時に差込口がちょうどB口になる場合(`(b-1)%(a-1)==0`)に気をつければceil関数を使って簡単に書けます。

## ceil関数について

引数で与えられた数を切り上げて返す関数です。ちなみに、切り捨てはfloor関数です。

```python:answerB.py
import math
a,b=map(int,input().split())
print(math.ceil((b-1)/(a-1)))
```

調べていたら、実行速度の観点から以下の方がいいようです。

```python:answerB.py
a,b=map(int,input().split())
print(-((-b+1)//(a-1)))
```

# [C問題](https://atcoder.jp/contests/abc139/tasks/abc139_c)

hに前から順にアクセスし、右隣の要素が現在の要素以下ならcを+1、そうでないならその時点でのcとdの大きい方をdとして更新し、cは再び0にします。そして、最後の要素まで比較したら最後に再び更新したdを出力すれば良いです。

```python:answerC.py
n=int(input())
h=[int(i) for i in input().split()]
c=0
d=c
for i in range(n-1):
    if h[i]>=h[i+1]:
        c+=1
    else:
        d=max([c,d])
        c=0

print(max([c,d]))

```

# [D問題](https://atcoder.jp/contests/abc139/tasks/abc139_d)

今回のセットでおそらく最も早くできた問題でした。
1~Nである数を割ったときの余りの最大値は0~N-1で、ここでは2~Nで1~N-1をそれぞれ割って1でNを割ればその最大値を満たすということがわかるので、これは上記のように簡単に求めることができます。

```python:answerD.py
n=int(input())
print((n-1)*n//2)
```

# [E問題](https://atcoder.jp/contests/abc139/tasks/abc139_e)

色々考えてはみたものの明らかにTLEになりそうなので飛ばしました。解説見ても有向グラフとか深さ優先探索とか書いてあったのでとりあえず飛ばします。

# [F問題](https://atcoder.jp/contests/abc139/tasks/abc139_f)

あるベクトルAに対してエンジンベクトルの角度が90度以下であれば、ベクトルAの方向に進むのに適したベクトルとみなすことができるというのはわかったのですが([解説](https://img.atcoder.jp/abc139/editorial.pdf)を参照)、ありうるベクトルAを必要十分に取る方法が時間内にはわからず、昨日も考えていたのですが、わかりませんでした。

嘘解法だとは思うのですが、以下のコードはACになりました。わかる方がいれば、嘘解法ではない方法でACを取れる方法を教えてください。

```python:answerF.py
import cmath
import math
n=int(input())
a=[]
b=[]
#1
for i in range(n):
    x,y=map(int,input().split())
    a.append(complex(x,y))
    b.append(cmath.phase(a[i]))
a.sort(key=cmath.phase)
#2
ma=0
for i in range(n):
    z=a[i]
    c=b[i]-math.pi/2
    d=b[i]+math.pi/2
    l=max([c,-math.pi])
    r=min([d,math.pi])
    if a[i]!=complex(0,0):
        for j in range(n):
            if j!=i:
                if l<=b[j]<=r or (c<=-math.pi and c+2*math.pi<=b[j]<=math.pi) or (d>=math.pi and -math.pi<=b[j]<=d-2*math.pi):
                    z+=a[j]
    ma=max([abs(z),ma])
#3
m=0
for i in range(n):
    if a[i]!=complex(0,0):
        za,zb=complex(0,0),complex(0,0)
        for j in range(n):
            p=cmath.phase(a[j]/a[i])
            if -math.pi<=p<=0:
                za+=a[j]
            if 0<=p<=math.pi:
                zb+=a[j]
        m=max([m,abs(za),abs(zb)])
        if i!=n-1 and a[i]+a[i+1]!=0:
            zc,zd=complex(0,0),complex(0,0)
            for j in range(n):
                p=cmath.phase(a[j]/((a[i]+a[i+1])/2))
                if -math.pi<=p<=0:
                    zc+=a[j]
                if 0<=p<=math.pi:
                    zd+=a[j]
            m=max([m,abs(zc),abs(zd)])

print(max([m,ma]))
```

まず、1において入力を受け取ります。

配列aには入力を複素数オブジェクトにして格納し、配列bにはその複素数オブジェクトの偏角を格納しています。(複素数ではありますが、ベクトル的に用いています。)
次に、2においては、それぞれのエンジンを進みたい方向のベクトルと捉え、そのベクトルと90度以下の角度をなすようなエンジンだけを用いるようにしています。

その次に、3においては、それぞれのエンジンを進みたい方向ではなく進みたいベクトルから見た境界部分のベクトル(進みたい方向から±90度のベクトル)として捉えました。さらに、これでは境界条件が不十分だったようなので、配列aで隣り合ってるもの同士(1において偏角でソートしているので、偏角の近いもの同士)の中間のベクトルも境界部分のベクトルとして考えました。

2の方法だけだと7割くらいしかACにならず、明らかに不十分だと思ったので3の方法に転換しました。しかし、3の方法であっても一つだけWAになってしまったので、2と3の両方をまとめて無理やり全てACにしました。おそらく、進みたい方向を全列挙できていないために境界部分の場合分けがうまくいってないと考えられます。
