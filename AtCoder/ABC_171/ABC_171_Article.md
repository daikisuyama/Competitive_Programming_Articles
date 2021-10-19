# 感想

コンテストの復習の成果が出てきて気がするので、今後は考察の丁寧さを意識していきたいです。

# [A問題](https://atcoder.jp/contests/abc171/tasks/abc171_a)

大文字かどうかはisupper関数で判定でき、以下のようになります。

```python:A.py
a=input()
print("A" if a.isupper() else "a")
```

# [B問題](https://atcoder.jp/contests/abc171/tasks/abc171_b)

このパターンの問題はいくらでもAtCoderで出てきている気がします。  
金額の小さい順にK個選ぶので昇順ソートして配列の1~K番目の和を考えれば良いです。

```python:B.py
n,k=map(int,input().split())
p=list(map(int,input().split()))
p.sort()
print(sum(p[:k]))
```


# [C問題](https://atcoder.jp/contests/abc171/tasks/abc171_c)

まず、犬の名前の長さを$l$とすればその犬の名前の命名の仕方は$26^l$通りあることが明らかです。また、ここから26進数として考えれば都合が良いことがわかります。**名前の長さを決めればあとはn進数の桁を決めていく処理**と考えることができるので、まずは名前の長さをn_len関数で求めることにしました。

ここで、与えられたNから$26,26^2,…$を0以下にならないように引いていくことで名前の長さは求めることができるので、この処理をした下では**名前の長さ$l$**と**その長さの名前の中で(辞書順で)何番目か$N$**という情報を持っています。

26進数の処理は下側の桁から決めていくと考えれば、`n%26`でその桁のアルファベットが求まって`n//=26`でその桁を捨てることができるので、これを名前の長さ$l$について全て行えば良いです。

```python:C.py
n=int(input())-1
def n_len():
    global n
    i=1
    while n>=26**i:
        n-=26**i
        i+=1
    return i
l=n_len()
alp=[chr(i) for i in range(97, 97+26)]
ans=""
for i in range(l):
    k=n%26
    ans=alp[k]+ans
    n//=26
print(ans)
```

# [D問題](https://atcoder.jp/contests/abc171/tasks/abc171_d)

数列を配列のまま保存してクエリの計算を行うと、全ての配列の要素をチェックする必要があり、計算量が$O(QN)$となり間に合いません。

クエリの処理は値が$B_i$の数を全て$C_i$に書き換えるというもので、辞書で数列内にそれぞれの数がいくつあるかを保存しておけばこの処理は$O(1)$で行うことができます。

したがって、要素の和を保存する用の変数を用意して、$B_i,C_i$が数列内に存在しない可能性を考慮すれば、計算量が$O(Q)$のプログラムを実装することができます。

```python:D.py
n=int(input())
a=dict()
ans=0
for i in map(int,input().split()):
    ans+=i
    if i in a:
        a[i]+=1
    else:
        a[i]=1
q=int(input())
for i in range(q):
    b,c=map(int,input().split())
    if c not in a:
        a[c]=0
    if b not in a:
        a[b]=0
    ans=ans-b*a[b]+c*a[b]
    a[c]+=a[b]
    a[b]=0
    print(ans)
```


# [E問題](https://atcoder.jp/contests/abc171/tasks/abc171_e)

[hamayanhamayanさんの記事](https://www.hamayanhamayan.com/entry/2017/05/20/145021)を参照すればわかるように、XORのよく使う性質には以下の3つがあります($a$は0以上の整数)。

1. 交換則及び結合則
2. $a \oplus a=0$
3. $2a \oplus (2a+1)=1 \leftrightarrow 4a \oplus (4a+1) \oplus (4a+2) \oplus (4a+3)=0$

この問題では主に上記の性質2の**被りが生じると0なのでいい感じに消せる**ことを利用して考察すると、下図のようになります。(暗黙的に性質1も用いています。)

![](/AtCoder/ABC_171/ABC_171_1.png)

まず、$a\_1,a\_2,…,a\_n$のXORを考えると$b\_1,b\_2,…,b\_n$は$n-1$回ずつ現れるのは明らかです。さらに、**$b_i$を求めるために$a\_i$に注目する**と、$a\_i$をもう一回XORすることで$b\_1,…,b\_{i-1},b\_{i+1},…,b\_n$が$n$回ずつ,$b\_i$は$n-1$回ずつ出現することがわかります。したがって、問題の制約より$n$は偶数なので、$b\_1,…,b\_{i-1},b\_{i+1},…,b\_n$によるXORは全て0で、$b\_i$のみが残ることがわかります。

以上より、前計算として$a\_1,a\_2,…,a\_n$のXORを求め、答えとしてはその数と$a_1,a_2,…,a_n$それぞれのXORを求めればよく以下のようになります。

```python:E.py
n=int(input())
a=list(map(int,input().split()))
b=0
for i in range(n):
    b^=a[i]
ans=[0]*n
for i in range(n):
    ans[i]=a[i]^b
print(" ".join(map(str,ans)))
```


# [F問題](https://atcoder.jp/contests/abc171/tasks/abc171_f)

(以下では、元々の文字列を$S$、その長さを$N$、$i$文字目を$S_i$、最終的に生成する文字列を$S\^{'}$と表します。)

まず、**文字列かつ順番が関係しそう**なのでDPを疑いましたが、計算量的に間に合いません。ここで、サンプルを利用した実験と問題の制約から多くの文字列が表せそうなので、**組み合わせ計算によって$O(K+N)$程度に収めよう**と考えました。また、後から追加する文字の自由度が高いので、**$S$に含まれる文字の場所を決めてから後から追加する文字の組み合わせの総数を考える**という方法を採りました。

しかし、$S$に含まれる文字を決めた後に他の文字がそれぞれ26通りずつあるとして組み合わせを求めると、**$S\^{'}$に重複する文字列が生じる可能性**があります。したがって、先ほどの$S$に含まれる文字の位置を決める方針と合わせれば、**$S$に含まれる文字の位置を決めたもとで$S\^{'}$の作り方が一意に決まるような方法**を求めれば良いです。また、この方法は**$S\^{'}$に対し$S$に含まれる文字の位置が一意に決まるような方法**(✳︎)を求めれば良いと言い換えることができます。

ここで、(✳︎)についてですが、**前から$S\^{'}$を眺めた時に$S$に含まれる文字が最初に出てくる位置**とすれば一意に決めることができます。また、このもとでは、$S\^{'}$の$S \_{i-1} $の位置から$S \_{i} $の位置までは**$S\_{i} $以外の全てのアルファベット**を使うことができます。したがって、$S\_{N} $の位置より前で$S$に含まれない文字は全て25(=26-1)通りとなります。また、$S\_{N} $の位置より後の文字列についてはどのアルファベットでも良いので26通りとなります。

以上より、26通りある場合と25通りある場合の境目である$S\_{N} $の位置を決めれば、残りの$S$に含まれる文字の位置が何通りあるかと26と25が何乗ずつあるかを求めることで以下のような実装になります。また、以下では整数を常にmodをとって管理する[modint構造体](https://qiita.com/DaikiSuyama/items/2dcc49932f2489292e44)を使っています。

```c++:F.cc
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
#define SIZE(x) ll(x.size()) //sizeをsize_tからllに直しておく
//定数
#define INF 1000000000000 //10^12:極めて大きい値,∞
#define MOD 1000000007 //10^9+7:合同式の法
#define MAXR 3000000 //10^5:配列の最大のrange(素数列挙などで使用)
//略記
#define PB push_back //vectorヘの挿入
#define MP make_pair //pairのコンストラクタ
#define F first //pairの一つ目の要素
#define S second //pairの二つ目の要素

template<ll mod> class modint{
public:
    ll val=0;
    //コンストラクタ
    modint(ll x=0){while(x<0)x+=mod;val=x%mod;}
    //コピーコンストラクタ
    modint(const modint &r){val=r.val;}
    //算術演算子
    modint operator -(){return modint(-val);} //単項
    modint operator +(const modint &r){return modint(*this)+=r;}
    modint operator -(const modint &r){return modint(*this)-=r;}
    modint operator *(const modint &r){return modint(*this)*=r;}
    modint operator /(const modint &r){return modint(*this)/=r;}
    //代入演算子
    modint &operator +=(const modint &r){
        val+=r.val;
        if(val>=mod)val-=mod;
        return *this;
    }
    modint &operator -=(const modint &r){
        if(val<r.val)val+=mod;
        val-=r.val;
        return *this;
    }
    modint &operator *=(const modint &r){
        val=val*r.val%mod;
        return *this;
    }
    modint &operator /=(const modint &r){
        ll a=r.val,b=mod,u=1,v=0;
        while(b){
            ll t=a/b;
            a-=t*b;swap(a,b);
            u-=t*v;swap(u,v);
        }
        val=val*u%mod;
        if(val<0)val+=mod;
        return *this;
    }
    //等価比較演算子
    bool operator ==(const modint& r){return this->val==r.val;}
    bool operator <(const modint& r){return this->val<r.val;}
    bool operator !=(const modint& r){return this->val!=r.val;}
};

using mint = modint<MOD>;

//入出力ストリーム
istream &operator >>(istream &is,mint& x){//xにconst付けない
    ll t;is >> t;
    x=t;
    return (is);
}
ostream &operator <<(ostream &os,const mint& x){
    return os<<x.val;
}

//累乗
mint modpow(const mint &a,ll n){
    if(n==0)return 1;
    mint t=modpow(a,n/2);
    t=t*t;
    if(n&1)t=t*a;
    return t;
}

//二項係数の計算
mint fac[MAXR+1],finv[MAXR+1],inv[MAXR+1];
//テーブルの作成
void COMinit() {
    fac[0]=fac[1]=1;
    finv[0]=finv[1]=1;
    inv[1]=1;
    FOR(i,2,MAXR){
        fac[i]=fac[i-1]*mint(i);
        inv[i]=-inv[MOD%i]*mint(MOD/i);
        finv[i]=finv[i-1]*inv[i];
    }
}
//演算部分
mint COM(ll n,ll k){
    if(n<k)return 0;
    if(n<0 || k<0)return 0;
    return fac[n]*finv[k]*finv[n-k];
}

signed main(){
    //入力の高速化用のコード
    //ios::sync_with_stdio(false);
    //cin.tie(nullptr);
    COMinit();
    ll k,l;cin>>k;
    string s;cin>>s;n=SIZE(s);
    mint ans=0;
    FOR(i,n,k+n){
        ans+=(modpow(25,i-n)*modpow(26,k+n-i)*COM(i-1,n-1));
    }
    cout<<ans<<endl;
}
```
