# 感想

A問題のみの一完に留まり、B問題に一時間以上費やしたにも関わらずできませんでした。勉強不足で解答を見ても全くわかりませんでした。

一完に留まりましたがなぜか水色パフォが出ました。もう少しで緑です。

![](/AtCoder/AGC_038/AGC_038_1.png)


# [A問題](https://atcoder.jp/contests/agc038/tasks/agc038_a)


パズル的に解ける問題であると感じました。思考のフローは下記のようになります。

問題文から言い換えるのが少し難しく、全ての行列がこの形になるかの考察に少し時間がかかりました。

行と列のそれぞれに注目して考えると、条件を満たすものであればそれぞれを入れ替えても条件を満たす(自分の都合のいいように行列を作ることができる)

↓

また、条件を満たすものであれば0と1を入れ替えても条件を満たす(0の個数にまずは注目して考える)

↓

入れ替えていってまずは行列を最終的には下のような形にすることができる([公式の解説](https://img.atcoder.jp/agc038/editorial.pdf)を参照)

![](/AtCoder/AGC_038/AGC_038_2.png)


```python:answerA.py
h,w,a,b=map(int,input().split())
for i in range(h):
    if i<b:
        print("1"*a+"0"*(w-a))
    else:
        print("0"*a+"1"*(w-a))
```

# [B問題](https://atcoder.jp/contests/agc038/tasks/agc038_b)

普通に実装するとTLEになることは明らかなので考察をしたのですが、グラフに帰着させて解く問題のようで現時点で解くのは難しいと感じました。
