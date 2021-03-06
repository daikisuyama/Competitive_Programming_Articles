# 感想

NoSub撤退はしないと決めてコンテストに臨みましたが、結果的に0完でした。悔しいです。

# [A問題](https://atcoder.jp/contests/agc045/tasks/agc045_a)

## 概要

$x$を$x \oplus A_i$で置き換える(or置き換えない)と言う操作を$i=1$~$n$で繰り返し、最終的に0にできるかを考える問題です。

## 前提

二人の対戦型のゲームで勝ち負けが決まるので、**最終状態がどうなるか**を考えます(①)。また、XORは**それぞれのbitを独立に計算できる**ことにも注目する必要があります(②)。

## 方針1

まず、全てのbitが0になるので、$x=0$になるのが特殊なケースであることに気づきました。したがって、**人0の目標を達成するのはかなり難しいのでは**と考えその場合を探ることにしました。しかし、前提②よりそれぞれのbitが独立に動くので、**発見的に探すのは難しい**と考え、この方針は却下しました。

## 方針2

$x=0$を$x$の値の状態と見なせばそれぞれのラウンド$i$での$x$の状態は高々$10^{18}$通り程度なので、$m=10^{18}$とすれば$O(m  n)$で求めることができるのではと考えました。また、前提①と合わせれば、**最終的に$x=0$になるようなラウンド$i$の時点での$x$の値の候補**を番号0の人は考えて、番号1の人は**その候補から外れるような$x$の値を作れるか**と考えます。

→それぞれのラウンドでの値の候補数$m$が大きいので計算を間に合わせることができず、$m$を小さくしようとして実験を行いましたがうまく考察できませんでした。

## 方針3

方針2の考察を深めることで正解の解法に近づくことができます。つまり、先ほどの方針2は**状態の遷移なのでDP**を考えればよく、以下のように状態を保存する配列を用意すれば良いです。

$DP[i]=$($i$番目のラウンドの前に$x=t$でそこからゲームを続けて$x=0$となるような$t$の集合)

このもとで$DP[i]$の遷移を考えると、以下のようになります。

### (1)$S\_i=0$のとき

番号0の人は**できるだけ多くの$x$の候補**を考えたいので、そのラウンド$i$の時点でありうる$x$の候補を全列挙すれば良いです。よって、$DP$の遷移は以下のようになります。

$DP[i]=DP[i+1] \cup \\{v\oplus A\_i|v \in DP[i+1]\\}$

### (2)$S_i=1$のとき

番号1の人は次のラウンド$i+1$の時点でありうる$x$の候補($DP[i+1]$)に**含まれないような$x$**を作りたいと考えると場合分けがしやすいです。

また、$x \notin DP[i+1]$となる$x(\in DP[i])$を考えると、ラウンド$i$で番号1の人は何もしないことで最終的な$x$を0でなくすことができるので、$x \in DP[i+1]$であることが必要です。

さらに、$x \in DP[i+1]$でも$x \oplus A\_i \notin DP[i+1]$の可能性があるので、この時の$DP$の遷移は以下のようになります。

```math
\begin{cases}
DP[i]=DP[i+1] \ \ \ \  ( ^∀ x\in DP[i+1])(x \oplus A\_i \in DP[i+1])\\
DP[i]=0 \ \ \ \  (^∃ x \in DP[i+1])(x \oplus A\_i \notin DP[i+1])\\
\end{cases}
```


## 方針4

方針3から状態数を落とすことを考えます。ここでは、**$DP[i]$の集合の全要素を保存せずに$DP[i]$を表現する**ことを考えます。

まず、**数式で表現する**と$DP[i]$が以下のように書けるのは自明だと思います。

$$DP[i]=\\{a\_iA\_i \oplus a\_{i+1}A\_{i+1} \oplus … \oplus a\_nA\_n\\}(a\_{i},a\_{i+1},…,a\_n \in \\{0,1 \\})$$

上記の表現では、$a\_{i},a\_{i+1},…,a\_n$を定数倍として$A\_{i},A\_{i+1},…,A\_n$をbitごとに分け、$\\{0,1\\}$の直積集合の元とすれば、$A\_{i},A\_{i+1},…,A\_n$はそれぞれベクトルとみなせます。

よって、$DP[i]$はベクトル$A\_{i},A\_{i+1},…,A\_n$の**XORによる線形結合で表現できるベクトル空間**と考えることができます。さらに、$1 \leqq A\_i \leqq 10^{18}<2^{60}$よりMSB(最上位ビット)のは高々59なので、このベクトル空間の次元は高々60です。

したがって、このベクトル空間を表現するものとして**基底があれば十分**で高々60個の整数を保存しておけば良いです。したがって、DPの遷移では基底を保存することを考えます。ここで、通常であれば**掃き出し法を使って基底を考えれば良い**のですが、掃き出し法では計算量的に間に合いません。

実は、XORの特徴を使うと基底を求めるのはあまり難しくありません。XORを線形結合とするベクトル空間で基底が存在する時、**それぞれの基底は異なるMSBの整数として定義できる**ことを用います($\because$MSBが同じ基底についてそれらのXORをとれば一方のMSBを小さくすることができる)。…(✳︎)

これに従えば、$DP[i]=$($A_i$~$A_n$で表されるベクトル空間の基底の集合)としてDPの遷移は以下のように考えることができます。

### (1)$S_i=0$のとき

任意の$v(\in DP[i])$について$A_i$が線型独立の場合、$DP[i]=DP[i+1]\cup \\{A_i\\}$
任意の$v(\in DP[i])$について$A_i$が線型従属の場合、$DP[i]=DP[i+1]$

### (2)$S_i=1$のとき

任意の$v(\in DP[i])$について$A_i$が線型独立の場合、$DP[i]=0$
任意の$v(\in DP[i])$について$A_i$が線型従属の場合、$DP[i]=DP[i+1]$

また、線形独立か線形従属かの判定をするには、(✳︎)の性質を用いて**MSBが異なる基底を$DP[i]$に保存する**と実装が簡単です。すなわち、$DP[i][j]=$($A_i$~$A_n$で表されるベクトル空間のMSBが$j$の基底)として定義すれば良いです。

よって、$A\_i$がそれらの基底と線型独立かの判定をするには、$A\_i$と同じMSBの基底が存在する場合は$Ai$とその基底のXORで$A\_i$を置き換える($\because $ (✳︎))ことを繰り返し、最終的に$A\_i=0$になる場合は$A\_i$は線形従属であり、$A\_i$とMSBが等しい基底が存在しない場合は$A\_i$は線型独立となります。

以上の操作で線形独立か線形従属かの判定をすることで(1)と(2)でのDPの遷移を表現することができ、$O(t \times n \times (\log{A_{max}})^2)$の計算量でのプログラムを書くことができます。

```python:A.py
import numpy as np
#0-indexed,MSBがma以下の時
def msb(x,ma=59):
    for i in range(ma,-1,-1):
        if (x>>i)&1:
            return i
    return len()
t=int(input())
for _ in range(t):
    n=int(input())
    a=list(map(int,input().split()))
    s=input()
    #独立、msb同じものはない
    #逆に異なれば独立
    msbs=[0 for i in range(60)]
    for i in range(n-1,-1,-1):
        now=a[i]
        m=59
        #独立のときfはTrue、独立でない時fはFalse
        while True:
            m=msb(now,m)
            if msbs[m]==0:
                f=True
                break
            now=msbs[m]^now
            if now==0:
                f=False
                break
        if s[i]=="0":
            if f:
                msbs[m]=now
        else:
            if f:
                print(1)
                break
    else:
        print(0)
```
