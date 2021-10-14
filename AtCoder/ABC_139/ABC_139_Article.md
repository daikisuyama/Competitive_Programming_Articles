# 結果と感想

4回目にして茶色になることができました。

Fは境界部分の場合分けさえできればできそうで、今回は全体的に感触が良かったので、もう少しパフォーマンスをあげたかったです。次のコンテストも頑張って、今月中に緑になる目標を達成したいです。

![](/AtCoder/ABC_139/ABC_139_1.png)


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
