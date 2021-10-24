# 感想

今回も悔しい結果に終わりました。E問題は自分の方針にこだわれず時間がかかってしまいました。

また、F問題はsetを使った嘘貪欲をやろうとして実装に時間がかったので反省しています。さらに、この問題はまさかこの解法ではないだろと思っていたものが正解だったので、悔しいです…。

今回も悔しい結果でしたが、徐々に地力がついてきた気もするので、諦めずに頑張ります。おそらく黄パフォくらいを出せると波に乗れると思うので精進します。

# [A問題](https://atcoder.jp/contests/abc178/tasks/abc178_a)

簡単に書こうと思って失敗しました。

```python:A.py
print(1 if int(input())==0 else 0)
```

# [B問題](https://atcoder.jp/contests/abc178/tasks/abc178_b)

**端でない部分を選ぶよりも端を選んだ方が値が大きくなる**ことを示せるので、端の掛け算の4通りの最大値を求めます。

```python:B.py
a,b,c,d=map(int,input().split())
print(max(a*c,a*d,b*c,b*d))
```

# [C問題](https://atcoder.jp/contests/abc178/tasks/abc178_c)

**存在条件は否定を考えて補集合を考えれば良い**です。つまり、全体($10^n$)から1~9のみを選んだ時($9^n$)と0~8を選んだ時($9^n$)を除きます。また、1~8を選んだ時($8^n$)が重複するので除きます。よって、$10^n-9^n-9^n+8^n$が答えです。また、$mod \ 10^9+7$で答える必要がありますが、Pythonでは**組み込みの**pow関数を使う必要があります。

```python:C.py
n=int(input())
mod=10**9+7
print((pow(10,n,mod)-2*pow(9,n,mod)+pow(8,n,mod)+2*mod)%mod)
```

# [D問題](https://atcoder.jp/contests/abc178/tasks/abc178_d)

**数列の長さえわかれば重複組み合わせで決められそう**です。まず、全ての項が3以上という条件から**数列の長さは高々$[\frac{n}{3}]$**です。また、**重複組み合わせは0以上にしたい**ので、全ての項から3を引きます。したがって、数列の長さを$x$とすれば総和が$s-3x$で全ての数が0以上である場合を考えます。この時、$s-3x$個の球と$x-1$の仕切りを並べる問題と言い換えることができるため、組み合わせの総数は$\_{s-3x+x-1} C \_{s-3x}$となります。

```C++:D.cc
//デバッグ用オプション：-fsanitize=undefined,address

//コンパイラ最適化
#pragma GCC optimize("Ofast")

//インクルードなど
#include<bits/stdc++.h>
using namespace std;
typedef long long ll;

//マクロ
//forループ
//引数は、(ループ内変数,動く範囲)か(ループ内変数,始めの数,終わりの数)、のどちらか
//Dがついてないものはループ変数は1ずつインクリメントされ、Dがついてるものはループ変数は1ずつデクリメントされる
//FORAは範囲for文(使いにくかったら消す)
#define REP(i,n) for(ll i=0;i<ll(n);i++)
#define REPD(i,n) for(ll i=n-1;i>=0;i--)
#define FOR(i,a,b) for(ll i=a;i<=ll(b);i++)
#define FORD(i,a,b) for(ll i=a;i>=ll(b);i--)
#define FORA(i,I) for(const auto& i:I)
//xにはvectorなどのコンテナ
#define ALL(x) x.begin(),x.end() 
#define SIZE(x) ll(x.size()) 
//定数
#define INF 1000000000000 //10^12:∞
#define MOD 1000000007 //10^9+7:合同式の法
#define MAXR 100000 //10^5:配列の最大のrange
//略記
#define PB push_back //挿入
#define MP make_pair //pairのコンストラクタ
#define F first //pairの一つ目の要素
#define S second //pairの二つ目の要素

ll fac[MAXR+1],finv[MAXR+1],inv[MAXR+1];

//テーブルの作成
void COMinit() {
    fac[0]=fac[1]=1;
    finv[0]=finv[1]=1;
    inv[1]=1;
    FOR(i,2,MAXR){
        fac[i]=fac[i-1]*i%MOD;
        inv[i]=MOD-inv[MOD%i]*(MOD/i)%MOD;
        finv[i]=finv[i-1]*inv[i]%MOD;
    }
}

// 二項係数の計算
ll COM(ll n,ll k){
    if(n<k)return 0;
    if(n<0 || k<0)return 0;
    return fac[n]*(finv[k]*finv[n-k]%MOD)%MOD;
}

signed main(){
    //入力の高速化用のコード
    //ios::sync_with_stdio(false);
    //cin.tie(nullptr);
    COMinit();
    ll s;cin>>s;
    ll ans=0;
    FOR(x,1,ll(s/3)){
        ll k=s-3*x;
        ans+=COM(k+x-1,k);
        ans%=MOD;
    }
    cout<<ans<<endl;
}
```


# [E問題](https://atcoder.jp/contests/abc178/tasks/abc178_e)

まず、二点の一方の点を固定したり$x$座標や$y$座標を固定したりしましたが埒が明きませんでした。次に考えるべきは**絶対値を外すこと**です。ここで、絶対値を外す際の典型パターンとして$|x|=max(x,-x)$があります。これに従うと、二点間$(x\_1,y\_1),(x\_2,y\_2)$のマンハッタン距離は$max((x\_1+y_1)-(x\_2+y\_2),(x\_1-y_1)-(x\_2-y\_2),(-x\_1+y_1)-(-x\_2+y\_2),(-x\_1-y_1)-(-x\_2-y\_2))$となります。したがって、**任意の二点間で**考えれば、それぞれの点$(x,y)$について$x+y,x-y,-x+y,-x-y$ごとに(最大値)-(最小値)を求めてその最大値が答えとなります。

```python:E.py
n=int(input())
xy=[list(map(int,input().split())) for i in range(n)]
ans=[]
cand=[xy[i][0]+xy[i][1] for i in range(n)]
ans.append(max(cand)-min(cand))
cand=[-xy[i][0]+xy[i][1]for i in range(n)]
ans.append(max(cand)-min(cand))
cand=[-xy[i][0]-xy[i][1] for i in range(n)]
ans.append(max(cand)-min(cand))
cand=[xy[i][0]-xy[i][1] for i in range(n)]
ans.append(max(cand)-min(cand))
print(max(ans))
```

# [F問題](https://atcoder.jp/contests/abc178/tasks/abc178_f)

まず、前の要素から一致しないように順にswapをする貪欲法を考えましたが、正当性が示せないため却下しました。次に、いずれもソート済みで一致しないようにすれば良いことから、逆にすることで一致するのは真ん中のみではと推測できます(✳︎)。また、この方法は簡単すぎると思って却下しましたが、(✳︎)の考察を進めると下記のようになります。

まず、$A$と$B$で1~$n$が合計でそれぞれ何個ずつあるかを考え、$n$より多くある要素がある場合は題意を満たす数列が作成できないのは鳩の巣原理より明らかなのでNoを出力します。

このもとで、先ほどの方針にしたがってBをまずは逆順にします。この時、$(\forall x \in [l,r])(A\_x=B\_x=p)$となる$p$が一つのみ存在します。逆に存在しない時はそのまま出力すれば題意を満たします。また、$(\forall x \notin [l,r])(A\_x \neq B\_x)$もこの時成り立ちます。

ここで、$x \in [l,r]$の任意の要素についてスワップを行います。このスワップは$x(\in [l,r])$と$y(\notin [l,r])$の間で行います。また、このスワップは$y(\notin [l,r])$について$A\_y \neq p$かつ$B\_y \neq p$であれば行うことができ、スワップで選んだ二つの要素はそれぞれのインデックスで一致することはありません。さらに、$A$と$B$に出てくる$p$は$n$個以下なので、任意の$x(\in [l,r])$に対してスワップを行うことが可能です。

以上より、逆順にして重なったところのみ重なってないところとのスワップを繰り返す方法の正当性が示されたので、これを実装して以下のようになります。

```python:F.py
n=int(input())
a=list(map(int,input().split()))
b=list(map(int,input().split()))
from collections import Counter,deque
x,y=Counter(a),Counter(b)
for i in range(1,n+1):
    if x[i]+y[i]>n:
        print("No")
        exit()
ans=b[::-1]
change=deque()
num=-1
now=deque()
for i in range(n):
    if a[i]==ans[i]:
        change.append(i)
        num=ans[i]
    else:
        now.append(i)
if change==[]:
    print("Yes")
    print(" ".join(map(str,ans)))
    exit()
while len(change):
    q=now.popleft()
    p=change.popleft()
    if a[q]!=num and ans[q]!=num:
        ans[q],ans[p]=ans[p],ans[q]
    else:
        change.appendleft(p)

print("Yes")
print(" ".join(map(str,ans)))
```
