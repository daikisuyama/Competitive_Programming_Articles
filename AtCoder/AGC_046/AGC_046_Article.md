# 感想

A問題が予想外に簡単で驚きました。  
B問題も的外れな考察ではなかったので、引き続き努力したいです。

# [A問題](https://atcoder.jp/contests/agc046/tasks/agc046_a)

下図のように、始点を複素数平面の原点Oとすると、偏角が$+x$されることがわかります。また、操作後に常に同一円周上にあるのは明らかなので偏角が操作開始時と等しければよく、$k$回の操作後に$l$周して時に原点Oにたどり着くような$k$のうち最も小さいものを考えればよいです。

![](/AtCoder/AGC_046/AGC_046_1.png)


したがって、$kx=360  l \leftrightarrow k=\frac{360l}{x}$が成り立ちます。ここで、$k$は整数なので、$360l$は$x$かつ$360$の倍数であり最小の$l$を考えれば良いので最小公倍数になります。以上より、$360$と$x$の最小公倍数を$x$で割ったものが答えです。

```python:A.py
from math import gcd
def lcm(a,b):
    return a//gcd(a,b)*b
x=int(input())
print(lcm(360,x)//x)
```

# [B問題](https://atcoder.jp/contests/agc046/tasks/agc046_b)

まず、誤読して、都合の良いパターンを探そうとしました。また、順番には関係なく塗られ方を直接考えれば良いのでは？、行の方と列の方のどちらかを塗ったのかで場合分けすれば良いのでは？、などと考えましたが、考察の結果難しいとわかりました。

上記の方針から、次に動的計画法による方針に移りました。この方針に移ったのは、行または列を追加するだけなので状態を管理すれば良いこと、遷移が(D+C)-(B+A)回のみであることが理由になります。

ここで、DPの配列はdp[i][x,y]=(i回の操作で行がxで列がyとなるような塗られ方の総数)としました。また、DPの遷移は**行を追加するのか列を追加するのか**で変わってきます。前者の場合はdp[i][x,y]→dp[i+1][x+1,y]で後者の場合はdp[i][x,y]→dp[i+1][x,y+1]になります。

この時、遷移の際に被るパターンの塗り方が存在するので、dp[i+1][x,y+1]からとdp[i+1][x+1,y]からのdp[i+2][x+1,y+1]への遷移を下図で考えます。

![](/AtCoder/AGC_046/AGC_046_2.png)

ここで、$x+1$行目と$y+1$列目において被るパターンが発生することが上図よりわかるので、**共通部分である$x \times y$の行列**を考えると考察を進めることができそうです。

さらに、$(x+1)\times y$の行列と$x \times (y+1)$行列の二つから$(x+1)\times (y+1)$行列へと遷移させる際に被るパターンは$x\times y$の行列から$(x+1)\times (y+1)$行列を生成する際に$(x+1)\times y$の行列と$x \times (y+1)$行列のどちらを経由しても作り出せるようなパターンと言い換えることができ、下図のようなパターンになります。

![](/AtCoder/AGC_046/AGC_046_3.png)

よって、上図の被る部分を消せば良いですが、"$x+1$がAになる場合と$y+1$がBになる場合は被りが発生しないこ"と及び"$x+1$がCを超える場合と$y+1$がDを超える場合は存在しないこと"、の二つに注意して実装すると2つ目のコードのような実装となります

ただ、2つ目のコードでは、dpの結果を保存するコンテナでmapを使っており、アクセスに対数時間かかるので計算量的にギリギリです。ここでは、mapを使わず通常の配列を使うことで5~10倍程度の高速化を行うことができ、dpを保存するコンテナの要素の定義は以下のように変化します。

**dp[i][x,y]=(i回の操作で行がxで列がyとなるような塗られ方の総数)**
↓
**dp[i][j]=(i回の操作で行がjだけ増えた場合の塗られ方の総数)**

よって、x=j+Aでy=i-j+Bであり、インデックスに注意して実装を行えば先ほどと同様の議論をすることができ、1つ目のコードのように実装することができます。

```c++:B_AC.cc
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
//定数
#define INF 1000000000000 //10^12:極めて大きい値,∞
#define MOD 998244353 //10^9+7:合同式の法
#define MAXR 100000 //10^5:配列の最大のrange(素数列挙などで使用)
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

mint dp[6000][3000]={0};

signed main(){
    ll A,B,C,D;cin>>A>>B>>C>>D;
    //i番目はi個の列or行を選んでる
    //a=Aの時が配列のインデックス0に対応
    //dp[i][j]として要素の合計はA+B+i,aの要素の合計はj+A,bの要素の合計はB+i-j
    //j+AはA以上C以下,B+i-jはB以上D以下
    //jは0以上C-A以下、かつ、B+i-D以上i以下
    dp[0][0]=1;
    REP(i,(D+C)-(B+A)){
        FOR(j,max(ll(0),B+i-D),min(C-A,i)){
            if(B+i-j!=D)dp[i+1][j]+=(dp[i][j]*mint(j+A));
            if(A+j!=C)dp[i+1][j+1]+=(dp[i][j]*mint(B+i-j));
        }
        if(i==0)continue;
        FOR(j,max(ll(0),B+i+1-D),min(C-A,i+1)){
            if(j!=0 and i+1!=j){
                dp[i+1][j]-=(dp[i-1][j-1]*mint(j+A-1)*mint(B+i-j));
            }
        }
    }
    cout<<dp[(D+C)-(B+A)][C-A]<<endl; 
}
```

```c++:B_TLE.cc
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
//定数
#define INF 1000000000000 //10^12:極めて大きい値,∞
#define MOD 998244353 //10^9+7:合同式の法
#define MAXR 100000 //10^5:配列の最大のrange(素数列挙などで使用)
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


signed main(){
    ll A,B,C,D;cin>>A>>B>>C>>D;
    //i番目はi個の列or行を選んでる
    vector<map<pair<ll,ll>,mint>> dp((D+C)-(B+A)+1);
    dp[0][MP(A,B)]=1;
    REP(i,(D+C)-(B+A)){
        for(auto j=dp[i].begin();j!=dp[i].end();j++){
            ll a,b;a=j->F.F;b=j->F.S;
            if(b!=D){
                dp[i+1][MP(a,b+1)]+=(dp[i][MP(a,b)]*mint(a));
            }
            if(a!=C){
                dp[i+1][MP(a+1,b)]+=(dp[i][MP(a,b)]*mint(b));
            }
        }
        if(i==0)continue;
        for(auto j=dp[i+1].begin();j!=dp[i+1].end();j++){
            ll a,b;a=j->F.F;b=j->F.S;
            if(a!=A and b!=B){
                dp[i+1][MP(a,b)]-=(dp[i-1][MP(a-1,b-1)]*mint(a-1)*mint(b-1));
            }
        }
    }
    cout<<dp[(D+C)-(B+A)][MP(C,D)]<<endl; 
}
```
