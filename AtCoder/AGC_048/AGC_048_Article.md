# 感想

コンテスト中に諦めてTwitterをしていました。考察の体力がないので、今後鍛えていきたいです…。

# [A問題](https://atcoder.jp/contests/agc048/tasks/agc048_a)

まず、$a$しかない場合は自明に-1です。同様に与えられた文字列がすでに"atcoder"という文字列より大きければ自明に0です。

また、$a$以外の文字があれば最初にその文字を持ってくることで必ず辞書順で大きくすることができます。ここで、前から数えて初めて$a$ではない文字があるインデックスを$x$とすれば、最初にその文字を持ってくるのに必要な最小のswapの回数は$x$となります。

上記を答えとしたいのですが、**$x$に存在する文字が"t"より大きい場合は二番目の文字に持ってくれば辞書順で大きくすることができる**ので、1回ぶん少ない回数で辞書順にすることができます。逆にこの2パターン以外の場合は"aa…"という文字列となり"atcoder"より辞書順で大きくなることはありません。

```python:A.py
for _ in range(int(input())):
    s=input()
    if all(i=="a" for i in s):
        print(-1)
    else:
        n=len(s)
        if "atcoder"<s:
            print(0)
            continue
        for i in range(n):
            if s[i]!="a":
                if s[i]>"t":
                    print(i-1)
                else:
                    print(i)
                break
```

# [B問題](https://atcoder.jp/contests/agc048/tasks/agc048_b)

DPや愚直な方法を当てはめられそうであると初めは思いましたが、一通り考察したものの難しいことがわかりました。したがって、何らかの法則性を見つけるために実験を行うことが必要です。

また、下記は[解説]((https://atcoder.jp/contests/agc048/editorial/198))の略記なので、詳しくはそちらを参照してください。

---

まず、$A,B$になるインデックスを選んだ時に(と)及び[と]は自由に選べます。つまり、インデックスの位置関係が重要です。また、括弧を配置する問題であるため、インデックスの偶奇に着目して実験を行います。すると、$A$となるインデックスと$B$となるインデックスのいずれも**全体の偶数のインデックスと奇数のインデックスの個数が等しければ**、( )と[ ]を上手く配置することで必ず題意を満たす括弧列を作ることができることに気付きます。

```python:B.py
n=int(input())
a=list(map(int,input().split()))
b=list(map(int,input().split()))
s=sum(a)
e,o=[b[i]-a[i] for i in range(n) if i%2==0],[b[i]-a[i] for i in range(n) if i%2==1]
e.sort(reverse=True)
o.sort(reverse=True)
for i in range(n//2):
    if e[i]+o[i]>0:
        s+=e[i]+o[i]
    else:
        break
print(s)
```
