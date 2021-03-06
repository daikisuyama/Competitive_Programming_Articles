# 感想

A問題の実装を迷っていたら時間切れになってしまいました。

# [A問題](https://atcoder.jp/contests/agc044/tasks/agc044_a)

まず、コンテスト中は+1や-1を調整する用と考え、2倍と3倍と5倍のうちどれかを**繰り返すことで一意に決まる**のではと考えました。したがって、$k$( $\in$ {$2,3,5$})進数として$n$を考えれば良いのではと思いましたが、サンプルを参照したら当然決まりませんでした。ここで、勇気を持って方針転換ができなかったのが、今回の敗因です。

ところで、この方針にこだわる前に、**$n$から0までの数で必要なコインの枚数をメモしたDP**を考えようとしてDPの要素数が$n$で無理だと判断していました。実は、**メモを配列ではなく辞書とする事でDPを高速化**する事ができ、この問題ではこの発想を利用することができます。

---

まず、**ある$l$で必要なコインの枚数がわかっている時、+1,-1,÷2,÷3,÷5を使って動く先の数を絞ります**(**状態の限定**がDPの基本)。例えば$l$÷5をしたい時を考えます。この答えが整数になるには$l$%5=0が必要なので、$l$に+1,-1を作用させて5で割り切れるようにすれば良いです。

ここで、$l$に+6以上もしくは-6以下を作用させると、その時点で$6 \times D$以上のコインを使うことになりますが、この場合、6以上もしくは-6以下を作用させてから割るよりも**先に割ってしまった方が効率が良い**のは明らかです。(e.g. 11→5→1("-1"×6→"÷5")よりも11→10→1("-1"×1→"÷5"→"-1"×1)の方が使うコインは少なくなります。)

同様に、ある$l$にするのに必要なコインの枚数と割りたい数$k$( $\in$ {$2,3,5$})が決まっている場合、最小のコイン枚数になるような操作では、**$l$に近い割りたい数(2,3,5)の倍数**(2通りあることに注意,コード内では**最寄り**と表記している)に移動して割り算を行って次の計算を行えば良いです。また、これを再帰的に繰り返すために、メモ化DFSと呼ばれるものを実装すれば良いです。ただし、割り算を使わずに-1のみで$l$から$0$へと$l$回かけて変化させるのが最適な場合もあるので、注意が必要です。

```python:A.py
def dfs(n):
    global b,check,d
    #ttfは235、chaはabc
    #2,3,5の場合それぞれ
    for ttf,cha in zip([2,3,5],b):
        #-1ずつしていく場合
        if check[0]>check[n]+n*d:
            check[0]=check[n]+n*d

        #最寄りまで-してく場合
        x1,x2=n//ttf,n%ttf
        if x1 in check:
            if check[x1]>check[n]+x2*d+cha:
                check[x1]=check[n]+x2*d+cha
                dfs(x1)
        else:
            check[x1]=check[n]+x2*d+cha
            dfs(x1)

        #最寄りまで+1してく場合
        x1,x2=n//ttf+1,ttf-n%ttf
        if x1 in check:
            if check[x1]>check[n]+x2*d+cha:
                check[x1]=check[n]+x2*d+cha
                dfs(x1)
        else:
            check[x1]=check[n]+x2*d+cha
            dfs(x1)

t=int(input())
for i in range(t):
    N,*b,d=map(int,input().split())
    check={N:0,0:N*d}
    dfs(N)
    print(check[0])

```
