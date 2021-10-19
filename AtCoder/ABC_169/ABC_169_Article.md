# 感想

C問題を30分もバグらせてしまい、悔しいです。

# [A問題](https://atcoder.jp/contests/abc169/tasks/abc169_a)

掛け算をするだけです。

```python:A.py
a,b=map(int,input().split())
print(a*b)
```

入力で受け取った空白を\*に置き換えることでも可能であるようです。

```python:A_shortest.py
print(eval(input().replace(" ","*")))
```

# [B問題](https://atcoder.jp/contests/abc169/tasks/abc169_b)

計算が遅くなるのを避けるために、$10^{18}$を超えたらbreakします。  
Pythonは**多倍長整数で極めて大きい**数の計算もできますが、**大きい数字の計算はそれだけ遅くなる**ことにも気を配る必要があります。

```python:B.py
n=int(input())
a=sorted(list(map(int,input().split())))
ans=1
for i in range(n):
    ans*=a[i]
    if ans>10**18:
        print(-1)
        break
else:
    print(ans)
```

# [C問題](https://atcoder.jp/contests/abc169/tasks/abc169_c)

この問題では**10進数の小数の精度**が問題なので、整数に直して計算したのが下記のコードです。整数に直すために、小数点を取るようにしました。

```python:C.py
a,b=input().split()
a=int(a)
b=int("".join(b.split(".")))
#b=int(b.strip("."))
print((a*b)//100)
```

また、この問題の10進数の小数の精度の演算を厳密にするためにはPythonでは**Decimalモジュール**を使えば良いです。

```python:C_decimal.py
from decimal import Decimal
a=int(a)
b=Decimal(b)
print(int(a*b))
```


他にも、有理数を誤差なしで計算する**fractionsモジュール**を使うと下記のコードのように計算することができます。

```python:C_fractions.py
from math import floor
from fractions import Fraction
a,b=input().split()
a=int(a)
b=Fraction(b)
print(int(a*b))
```

# [D問題](https://atcoder.jp/contests/abc169/tasks/abc169_d)

$z$が素数の累乗で表されるのでまずは$n$の素因数分解を行います。今回は試し割り法を用いました。

この時、素因数分解により$n=p_1^{q_1}\times p_2^{q_2}\times …\times p_k^{q_k}$($p_1,p_2,…p_k$は素数で$q_1,q_2,…q_k$は$1$以上の整数)という式が得られます。この元で、それぞれの素数について**最大何回の操作をできるか**を考えます。素因数分解に$q_i$だけ含まれる素数$p_i$を考えると、操作では$p_i^1,p_i^2,…$と**累乗の肩が小さいものから順に**$z$として選べば良いです。また、累乗の肩の合計は$q_i$なので、**$1+2+…+x \leqq q_i$となる$x$のうち最大の$x$を探せば良い**と言い換えることができます。

したがって、**$l$番目の要素が$1+2+…+l$であるような配列を用意すれば**、このような配列の$q_i$以下で一番最大の要素のインデックスを求めるという言い換えられます。このような要素はupper_boundの一つ隣の要素であるため、これをそれぞれの素数について計算して足していけば求めることができます。

```c++:D.cc
//インクルード(アルファベット順)
#include<algorithm>//sort,二分探索,など
#include<bitset>//固定長bit集合
#include<cmath>//pow,logなど
#include<complex>//複素数
#include<deque>//両端アクセスのキュー
#include<functional>//sortのgreater
#include<iomanip>//setprecision(浮動小数点の出力の誤差)
#include<iostream>//入出力
#include<iterator>//集合演算(積集合,和集合,差集合など)
#include<map>//map(辞書)
#include<numeric>//iota(整数列の生成),gcdとlcm(c++17)
#include<queue>//キュー
#include<set>//集合
#include<stack>//スタック
#include<string>//文字列
#include<unordered_map>//イテレータあるけど順序保持しないmap
#include<unordered_set>//イテレータあるけど順序保持しないset
#include<utility>//pair
#include<vector>//可変長配列

using namespace std;
typedef long long ll;

//マクロ
//forループ関係
//引数は、(ループ内変数,動く範囲)か(ループ内変数,始めの数,終わりの数)、のどちらか
//Dがついてないものはループ変数は1ずつインクリメントされ、Dがついてるものはループ変数は1ずつデクリメントされる
#define REP(i,n) for(ll i=0;i<(ll)(n);i++)
#define REPD(i,n) for(ll i=n-1;i>=0;i--)
#define FOR(i,a,b) for(ll i=a;i<=(ll)(b);i++)
#define FORD(i,a,b) for(ll i=a;i>=(ll)(b);i--)
//xにはvectorなどのコンテナ
#define ALL(x) (x).begin(),(x).end() //sortなどの引数を省略したい
#define SIZE(x) ((ll)(x).size()) //sizeをsize_tからllに直しておく
#define MAX(x) *max_element(ALL(x)) //最大値を求める
#define MIN(x) *min_element(ALL(x)) //最小値を求める
//定数
#define INF 1000000000000 //10^12:極めて大きい値,∞
#define MOD 1000000007 //10^9+7:合同式の法
#define MAXR 100000 //10^5:配列の最大のrange(素数列挙などで使用)
//略記
#define PB push_back //vectorヘの挿入
#define MP make_pair //pairのコンストラクタ
#define F first //pairの一つ目の要素
#define S second //pairの二つ目の要素

map<ll,ll> prime;//素因数分解でそれぞれの素数がいくつ出てきたかを保存するmap

//O(√n)
//整列済み(mapはkeyで自動で整列される)
void prime_factorize(ll n){
    if(n<=1) return;
    ll l=sqrt(n);
    FOR(i,2,l){
        if(n%i==0){
        prime_factorize(i);prime_factorize(ll(n/i));return;
        }
    }
    //mapでは存在しないkeyの場合も自動で構築される
    prime[n]++;return;
}

signed main(){
    //入力の高速化用のコード
    //ios::sync_with_stdio(false);
    //cin.tie(nullptr);
    ll n;cin >> n;
    prime_factorize(n);
    vector<ll> num(100);
    REP(i,100){
        num[i]=i+1;
        if(i>0)num[i]+=num[i-1];
    }
    ll ans=0;
    for(auto i=prime.begin();i!=prime.end();i++){
        //cout << i->F << " " << i->S << endl;
        ans+=(ll(upper_bound(ALL(num),i->S)-num.begin()));
    }
    cout << ans << endl;
}
```


# [E問題](https://atcoder.jp/contests/abc169/tasks/abc169_e)

まず、この問題では数列の長さの偶奇で中央値の定義が異なるので、**偶奇で場合分けして考えます**。ここで、偶奇の場合のどちらにも共通することですが、$A\_i \leqq X\_i \leqq B\_i$が成り立つので、$A\_1,A\_2,…,A\_n$の中央値の場合が$X\_1,X\_2,…,X\_n$のありうる中央値で**最小**で$B\_1,B\_2,…,B\_n$の中央値の場合が$X\_1,X\_2,…,X\_n$のありうる中央値で**最大**になります。さらに、$X\_i$は1ずつ動かすことができるので、最小値と最大値の間でありうる中央値は**全て表現できる**ことがわかります。

このもとで、偶奇での場合分けを考えます。まず、偶数の場合は、**中央値が小数になる可能性がある**ことに注意が必要です。すなわち、1つ目のサンプルを見ればわかるように、中央値の最小が$\frac{3}{2}$で最大が$\frac{5}{2}$の時は、$\frac{3}{2},2,\frac{5}{2}$となるので、$\frac{1}{2}$刻みでいくつありうるかを数えれば良いです。奇数の場合は中央値は整数にしかならないので、$1$刻みでいくつありうるかを数えれば良いです。

よって、これを実装して以下のようになります。

```python:answerE.py
n=int(input())
a,b=[],[]
for i in range(n):
    c,d=map(int,input().split())
    a.append(c)
    b.append(d)
a.sort()
b.sort()
if n%2==0:
    x=[a[n//2-1],a[n//2]]
    y=[b[n//2-1],b[n//2]]
    print(sum(y)-sum(x)+1)
else:
    x=a[n//2]
    y=b[n//2]
    print(y-x+1)
```


# [F問題](https://atcoder.jp/contests/abc169/tasks/abc169_f)

まず、このような問題では、全ての集合のパターンを列挙してから数え上げるのではなく、それぞれの要素がいくつか選ばれるかに注目すると良いです。したがって、$A_1,A_2,…,A_N$を一つずつ順に選んでいくことを考えます。また、和が$S$になるような部分集合を考えたいので、$dp[i][j][k]=$($A_1,A_2,…,A_i$の部分集合で$k$個選んで総和が$j$となるようなものの個数)とします。

このもとで、出力する答えは「$dp[N][S][k]\times $($dp[N][S][k]$となる部分集合$U$が含まれるような部分集合$T$の候補の数…①)」を$k$について合計したものになります。また、①は$2^{N-k}$となるので、$dp[N][S][k] \times 2^{N-k}$を$k$について足し合わせたものが答えです。

しかし、この解法では$O(N^2S)$になるので、計算量を減らす必要があります。現段階では、$DP$の遷移で部分集合$U$に含まれているかを考慮した後に部分集合$T$の候補数との積の計算を考えていて効率が悪いので、部分集合$U$,$T$に含まれているかを**同時に**考えながら$DP$の遷移を考えます。

よって、$A_i$に注目すれば、部分集合$T$に含まず部分集合$U$にも含まれないパターン…(1),部分集合$T$に含まれて部分集合$U$にも含まれるパターン…(2),部分集合$T$に含まれて部分集合$U$に含まれないパターン…(3)の三つのパターンについて$DP$の遷移を考えれば良いです。この時、$dp[i][j]=$($A_1,A_2,…,A_i$の部分集合で総和が$j$となるような部分集合$U$を含む部分集合$T$の個数)として、DPの遷移が下図のようになります。

![](/AtCoder/ABC_169/ABC_169_1.png)

```python:answerF.py
mod=998244353
n,s=map(int,input().split())
a=list(map(int,input().split()))
dp=[[0]*(s+1) for i in range(n+1)]
dp[0][0]=1
for i in range(n):
    for j in range(s+1):
        if j-a[i]>=0:
            dp[i+1][j]=dp[i][j]*2+dp[i][j-a[i]]
        else:
            dp[i+1][j]=dp[i][j]*2
        dp[i+1][j]%=mod
print(dp[n][s])
```
