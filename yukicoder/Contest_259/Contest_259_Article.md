# 感想

一見すると難しいA問題から逃げてしまい、反省しています。また、B問題も定数を間違えてしまって時間をロスしてしまいました。ただ、C問題はやったことのない類の問題でしたが粘って解くことができました。

# [A問題](https://yukicoder.me/problems/no/1139)

1. $t$の範囲は$D$に依存し$D$は**極めて大きく**その$t$の**最小値**$T$を求める必要がある
2. $t<T$では成り立たず$t>T$では成り立つという**単調性がある**

上記の2条件から、$T$は二分探索により求められます。また、ある$t$におけるそれぞれのスライムの合計の走った距離を$O(N)$で求められれば、計算量が$O(N \log{D})$となって実装可能です。

ここで、$t$における全てのスライムの合計の進んだ距離を考えますが、二つのスライム(速さは$v_1$,$v_2$)が合体すると一つのスライムになり、残ったスライムの速さは$v_1+v_2$になります。したがって、**合計の進む距離に注目**すればそれぞれのスライムが合体しないものとしても変わらないので、全てのスライムが(合体しない時に)$t$まででどれだけ進むかを考えれば$O(N)$で求めることができます。

```python:A.py
n,d=map(int,input().split())
x=list(map(int,input().split()))
v=list(map(int,input().split()))
s=sum(v)

def f(k):
    return s*k

#lは偽,rは真
l,r=0,1000000000000
while l+1<r:
    k=l+(r-l)//2
    if f(k)>=d:
        r=k
    else:
        l=k
print(r)
```

# [B問題](https://yukicoder.me/problems/no/1140)

まず、素数判定をクエリで行うので、エラトステネスの篩によって判定する可能性のある素数を全てチェックします。ここで$p$が素数ではないと判定されれば$-1$を出力すれば良いので、以下では$p$を素数と仮定して議論をすすめます。

この元で$A\^{P!}\  (mod \ P)$を単純に$((A\^1)\^2)\^3…$と求めていくと$mod \ P$の条件下でも間に合いません。ここで思い出すべきは**フェルマーの小定理**です。この定理では、素数$p$に対して$p$の倍数でない$A$について**$A\^{p-1}=1 \ (mod \ p)$**が成り立ちます。ここで、$A\^{p!}=(A\^{p-1})\^x \ (x=1,2,…,p-2,p)$が成り立つので($\because \ p$は素数より$2$以上)、$A$が$p$の倍数の場合は$A\^{p!}=0\  (mod \ P)$、そうでない場合は$A\^{p!}=1\  (mod \ P)$となります。

以上をまとめれば、$p$が素数でない時は$-1$、$p$が素数で$A\%p=0$の時は$0$、$p$が素数で$A\%p \neq 0$の時は$1$を出力すれば良いです。

```c++:B.cc
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
#define MAXR 6000000 //10^5:配列の最大のrange
//略記
#define PB push_back //挿入
#define MP make_pair //pairのコンストラクタ
#define F first //pairの一つ目の要素
#define S second //pairの二つ目の要素

#define PN true //素数
#define NPN false //素数ではない
//非本質
#define MAXRR 3000 //√MAXR以上の数を設定する

//MAXRまでの整数は素数であると仮定する(ここから削る)
vector<bool> PN_chk(MAXR+1,PN);//0indexでi番目が整数iに対応(0~MAXR)
//素数を格納する配列を用意しておく
vector<ll> PNs;

void se(){
    //0と1は素数ではない
    PN_chk[0]=NPN;PN_chk[1]=NPN;

    FOR(i,2,MAXRR){
        //たどり着いた時に素数と仮定されているなら素数
        if(PN_chk[i]) FOR(j,i,ll(MAXR/i)){PN_chk[j*i]=NPN;}
    }
    FOR(i,2,MAXR){
        if(PN_chk[i]) PNs.PB(i);
    }
}

ll modpow(ll a,ll n,ll p){
    if(n==0)return 1;
    ll t=modpow(a,n/2,p);
    t=t*t%p;
    if(n&1)t=t*a%p;
    return t;
}

signed main(){
    //入力の高速化用のコード
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    se();
    ll t;cin>>t;
    REP(_,t){
        ll a,p;cin>>a>>p;
        if(!PN_chk[p]){
            cout<<-1<<endl;
            continue;
        }
        if(a%p==0){
            cout<<0<<endl;
            continue;
        }
        cout<<1<<endl;
    }
}
```

# [C問題](https://yukicoder.me/problems/no/1141)

まず、以下のように$r_i,c_i$をおくと、下図の四つの部分に分けることができます。

![](/yukicoder/Contest_259/Contest_259_1.png)

この元で、求める答えは①,②,③,④のそれぞれの長方形に含まれる数の積の積を求めれば良いことがわかります。このような長方形に含まれる部分については**二次元累積和**の事前計算によりクエリ処理を$O(1)$で行うことができます。

事前計算による表の作成において、**二次元累積和の場合**は以下のようになります。また、それぞれのマス目を$a[x][y]$とし、以下は全て0-indexedです。

$s[x][y] := [0, x) × [0, y)$ の長方形区間の和
$s[x+1][y+1] = s[x][y+1] + s[x+1][y] - s[x][y] + a[x][y]$

したがって、**二次元累積積の場合**も同様にすれば以下のようになります。

$s[x][y] := [0, x) × [0, y)$ の長方形区間の積
$s[x+1][y+1] = s[x][y+1] \times  s[x+1][y] \div s[x][y] \times a[x][y]$

よって、この**二次元累積積を四方向に定義すればよく**残りの三つの長方形区間については以下のようになります。

$s[x][y] := [w-1, x) × [0, y)$ の長方形区間の積
$s[x-1][y+1] = s[x][y+1] \times  s[x-1][y] \div s[x][y] \times a[x][y]$

$s[x][y] := [0, x) × [h-1, y)$ の長方形区間の積
$s[x+1][y-1] = s[x][y-1] \times  s[x+1][y] \div s[x][y] \times a[x][y]$

$s[x][y] := [w-1, x) × [h-1, y)$ の長方形区間の積
$s[x+1][y+1] = s[x][y+1] \times  s[x+1][y] \div s[x][y] \times a[x][y]$

これを実装すれば、あとは**開区間であることに注意**すればプログラムを書くことができます。

```c++:C.cc
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



signed main(){
    //入力の高速化用のコード
    //ios::sync_with_stdio(false);
    //cin.tie(nullptr);
    ll h,w;cin>>h>>w;
    vector<vector<ll>> A(h,vector<ll>(w,0));
    REP(i,h)REP(j,w)cin>>A[i][j];
    vector<vector<vector<mint>>> s(4,vector<vector<mint>>(h+1,vector<mint>(w+1,1)));
    REP(j,h){
        REP(k,w){
            ll x,y;
            x=j;y=k;
            s[0][x+1][y+1]=s[0][x+1][y]*s[0][x][y+1]/s[0][x][y]*A[x][y];
        }
    }
    REP(j,h){
        REP(k,w-1){
            ll x,y;
            x=j;y=w-1-k;
            s[1][x+1][y-1]=s[1][x+1][y]*s[1][x][y-1]/s[1][x][y]*A[x][y];
        }
    }
    REP(j,h-1){
        REP(k,w){
            ll x,y;
            x=h-1-j;y=k;
            s[2][x-1][y+1]=s[2][x-1][y]*s[2][x][y+1]/s[2][x][y]*A[x][y];
        }
    }
    REP(j,h-1){
        REP(k,w-1){
            ll x,y;
            x=h-1-j;y=w-1-k;
            s[3][x-1][y-1]=s[3][x-1][y]*s[3][x][y-1]/s[3][x][y]*A[x][y];
        }
    }
    #if 0
    REP(i,4){
        REP(j,h){
            REP(k,w){
                cout<<s[i][j][k]<<" ";
            }
            cout<<endl;
        }
        cout<<endl;
    }
    #endif
    ll q;cin>>q;
    REP(_,q){
        ll r,c;cin>>r>>c;
        mint ans=1;
        ll x,y;
        x=r-1;y=c-1;
        ans*=s[0][x][y];
        //x=r;y=w-c;
        ans*=s[1][x][y];
        //x=h-r;y=c;
        ans*=s[2][x][y];
        //x=h-r;y=w-c;
        ans*=s[3][x][y];
        cout<<ans<<endl;
    }
}
```
