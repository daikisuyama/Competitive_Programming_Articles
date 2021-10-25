# 感想

2ヶ月ぶりくらいにcodeforcesに参加しました。

# [A問題](https://codeforces.com/contest/1379/problem/A)

$O(t \times n^2)$の計算量が許容されるのであまり難しくありません。まずは、"abacaba"(以降、文字列Sとします)が文字列のどの部分に出てくる可能性かをチェックします。チェックしたときにすでに二回以上文字列Sが出てくる場合は"No"を出力し、一回のみ文字列Sが出てくる場合は"?"を"d~z"に変えて出力すれば良いです。その他の場合は、"?"を変えることで文字列Sになりうるもののうち**文字列Sを一つだけ**作り出すようなものがあるかをチェックしていきます。

また、文字列Sになりうる部分文字列のチェックと**文字列Sが一つのみ**になるかを別々に分ければわかりやすい実装になります。

```python:A.py
#abacaのできる可能性のあるところにチェック
#おかしい
#インデックスミスなど…
a=["a","b","a","c","a","b","a"]
t=int(input())
for _ in range(t):
    n=int(input())
    s=list(input())
    check=[0]*(n-6)            
    for i in range(n-6):
        for j in range(i,i+7):
            if s[j]!=a[j-i] and s[j]!="?":
                break
        else:
            if s[i:i+7]==a:
                check[i]=2
            else:
                check[i]=1
    if check.count(2)>1:
        print("No")
        continue
    elif check.count(2)==1:
        print("Yes")
        print(("".join(s)).replace("?","z"))
        continue
    #?を変えて一個だけのパターン
    g=False
    for i in range(n-6):
        if check[i]==1:
            f=False
            cand=[a[j-i] if i<=j<=i+6 else s[j] if s[j]!="?" else "z" for j in range(n)]
            for j in range(n-6):
                if i!=j:
                    for k in range(j,j+7):
                        if cand[k]!=a[k-j]:
                            break
                    else:
                        f=True
            if not f:
                print("Yes")
                print("".join(cand))
                g=True
                break
    if not g:print("No")
```

#[B問題](https://codeforces.com/contest/1379/problem/B)

まず$a,b,c$で表現できる範囲を考えます。これは$l$以上$r$以下なので、与式にそれぞれ当てはめて考えます。すると、以下のようになります。

![](/Codeforces/Div2_657/Div2_657_1.png)

このもとで、①と②について片方を固定して考えます。②は連続的で考えやすいので、先に①を固定します。

すると、①はmに近づけることが最適なので、$\lceil \frac{m}{a} \rceil \times a$または$\lfloor \frac{m}{a} \rfloor \times a$になります。よって、この値と$m$との絶対値の差が$r-l$となれば$b,c$も存在し、答えとなります。

これを任意の$a$について探して答えとなるものを一つでも求めれば良いです。

また、計算量は$O(t \times (r-l))$となり、答えは必ず求まることが問題文中に書かれているので、コードを書いて以下のようになります。

```python:B.py
t=int(input())
for _ in range(t):
    l,r,m=map(int,input().split())
    f=False
    for a in range(l,r+1):
        n1,n2=m//a,-((-m)//a)
        if l-r<=m-n1*a<=r-l:
            for c in range(l,r+1):
                b=m-n1*a+c
                if l<=b<=r:
                    if m+c-b!=0:
                        print(a,b,c)
                        f=True
                        break
        if f:break
        if l-r<=m-n2*a<=r-l:
            for c in range(l,r+1):
                b=m-n2*a+c
                if l<=b<=r:
                    if m+c-b!=0:
                        print(a,b,c)
                        f=True
                        break
        if f:break
```

#[C問題](https://codeforces.com/contest/1379/problem/C)

$n$が大きいので全てを試行できません。しかし、ある程度の回数の試行を行った後は**ある$b\_i$のみを選べば良い**です。ここで、最大の$b\_i$を取るのが最適だと思ったのですがサンプルが合いませんでした。良く考えると、$b\_i$を選ぶ時$a\_i$も選ぶ必要があり、その値次第では最大の$b\_i$を取るのが最適となりません。したがって、どの$b\_i$を取り続けるか全通り($m$通り)試すことにしました。

また、**$b\_i$を取らない時**は$n \leqq m$が成り立ち、この時は$a$のうち大きい方から$n$個を選べば良いです。また、$a\_i$は昇順で並んでいる必要がありますが、**$b\_i$に対応する$a\_i$があとで必要になる**ので、$a\_i$が昇順で並ぶ配列$c$を用意します。

ここで、$b\_i$を選ぶ時、$a$の中で$b\_i$より大きいものも選分のが和を最大にするのに最適で、配列$c$をupper_boundで探した要素以上の要素を全て選べば良いです。この時、**それらの要素の和を毎回計算するのは非効率的**なので、**累積和**により求めておきます。

また、$b\_i$と同時に$a\_i$を選ぶ必要があり、**$a\_i$が先ほど選ばれているかどうかで場合分け**します。以上を繰り返して最大となるものを答えとして求めれば良いです。

```c++:C.cc
//コンパイラ最適化用
#pragma GCC optimize("Ofast")
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
#define REP(i,n) for(ll i=0;i<ll(n);i++)
#define REPD(i,n) for(ll i=n-1;i>=0;i--)
#define FOR(i,a,b) for(ll i=a;i<=ll(b);i++)
#define FORD(i,a,b) for(ll i=a;i>=ll(b);i--)
//xにはvectorなどのコンテナ
#define ALL(x) x.begin(),x.end() //sortなどの引数を省略したい
#define SIZE(x) ll(x.size()) //sizeをsize_tからllに直しておく
//定数
#define INF 1000000000000 //10^12:極めて大きい値,∞
#define MOD 1000000007 //10^9+7:合同式の法
#define MAXR 100000 //10^5:配列の最大のrange(素数列挙などで使用)
//略記
#define PB push_back //vectorヘの挿入
#define MP make_pair //pairのコンストラクタ
#define F first //pairの一つ目の要素
#define S second //pairの二つ目の要素
#define Umap unordered_map
#define Uset unordered_set

signed main(){
    //入力の高速化用のコード
    //ios::sync_with_stdio(false);
    //cin.tie(nullptr);
    ll t;cin>>t;
    while(t--){
        ll n,m;cin>>n>>m;
        vector<ll> a(m);
        vector<ll> b(m);
        vector<ll> c(m);
        REP(i,m){cin>>a[i]>>b[i];c[i]=a[i];}
        //aのソート済みのやつ
        sort(ALL(c));
        //cの累積和を求める(逆方向から)
        vector<ll> d(m+1);d[m]=0;
        REPD(i,m)d[i]=c[i]+d[i+1];
        ll ans=0;
        //aのみを選ぶ時
        if(n<=m)ans=max(ans,d[m-n]);
        //どのbの要素を選ぶか
        REP(i,m){
            auto as=upper_bound(ALL(c),b[i]);
            ll anum=distance(c.begin(),as);
            if(a[i]>b[i] and m-anum<=n)ans=max(d[anum]+b[i]*(n-(m-anum)),ans);
            if(a[i]<=b[i] and m-anum<=n-1)ans=max(d[anum]+b[i]*(n-(m-anum)-1)+a[i],ans);
        }
        cout<<ans<<endl;
    }
}
```
