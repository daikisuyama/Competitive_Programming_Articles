# 感想

C問題で読み落としていた部分に気付きかずに時間を使いすぎたため、Dの解く時間が足りなくなりました。

# [A問題](https://atcoder.jp/contests/arc106/tasks/arc106_a)

TLにも余裕があるので適当に全探索します。A,Bのいずれもあまり大きな数になりません。

```python:A.py
n=int(input())
x3,x5=[3],[5]
while True:
    if x3[-1]*3<n:
        x3.append(x3[-1]*3)
    else:
        break
while True:
    if x5[-1]*5<n:
        x5.append(x5[-1]*5)
    else:
        break
x3,x5=[i for i in x3 if i<n],[i for i in x5 if i<n]
y3,y5=set(x3),set(x5)
#print(y3,y5)
for i in y3:
    if n-i in y5:
        a,b=i,n-i
        ans1,ans2=0,0
        while a!=1:
            a//=3
            ans1+=1
        while b!=1:
            b//=5
            ans2+=1
        print(ans1,ans2)
        exit()
print(-1)
```

# [B問題](https://atcoder.jp/contests/arc106/tasks/arc106_b)

操作回数を最小にするなども求められていないため、任意の頂点の$a\_i,b\_i$の和が等しければ良いのではと思ったのですが、グラフが非連結である可能性に気づきました。

いずれにせよ、連結な頂点同士の間では$a$の値をうまく調整できるので、**それぞれの連結成分に含まれる頂点の$a\_i,b\_i$の和が等しい**ことが条件になります。UnionFindを使えば実装も難しくありません。

```C++:B.cc
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

//以下、素集合と木は同じものを表す
class UnionFind{
public:
    vector<ll> parent; //parent[i]はiの親
    vector<ll> siz; //素集合のサイズを表す配列(1で初期化)
    map<ll,vector<ll>> group; //集合ごとに管理する(key:集合の代表元、value:集合の要素の配列)
    ll n; //要素数

    //コンストラクタ
    UnionFind(ll n_):n(n_),parent(n_),siz(n_,1){ 
        //全ての要素の根が自身であるとして初期化
        for(ll i=0;i<n;i++){parent[i]=i;}
    }

    //データxの属する木の根を取得(経路圧縮も行う)
    ll root(ll x){
        if(parent[x]==x) return x;
        return parent[x]=root(parent[x]);//代入式の値は代入した変数の値なので、経路圧縮できる
    }

    //xとyの木を併合
    void unite(ll x,ll y){
        ll rx=root(x);//xの根
        ll ry=root(y);//yの根
        if(rx==ry) return;//同じ木にある時
        //小さい集合を大きい集合へと併合(ry→rxへ併合)
        if(siz[rx]<siz[ry]) swap(rx,ry);
        siz[rx]+=siz[ry];
        parent[ry]=rx;//xとyが同じ木にない時はyの根ryをxの根rxにつける
    }

    //xとyが属する木が同じかを判定
    bool same(ll x,ll y){
        ll rx=root(x);
        ll ry=root(y);
        return rx==ry;
    }

    //xの素集合のサイズを取得
    ll size(ll x){
        return siz[root(x)];
    }

    //素集合をそれぞれグループ化
    void grouping(){
        //経路圧縮を先に行う
        REP(i,n)root(i);
        //mapで管理する(デフォルト構築を利用)
        REP(i,n)group[parent[i]].PB(i);
    }

    //素集合系を削除して初期化
    void clear(){
        REP(i,n){parent[i]=i;}
        siz=vector<ll>(n,1);
        group.clear();
    }
};

signed main(){
    //小数の桁数の出力指定
    //cout<<fixed<<setprecision(10);
    //入力の高速化用のコード
    //ios::sync_with_stdio(false);
    //cin.tie(nullptr);
    ll n,m;cin>>n>>m;
    vector<ll> a(n);REP(i,n)cin>>a[i];
    vector<ll> b(n);REP(i,n)cin>>b[i];
    UnionFind uf(n);
    REP(i,m){
        ll c,d;cin>>c>>d;
        uf.unite(c-1,d-1);
    }
    uf.grouping();
    FORA(i,uf.group){
        ll s=0;
        FORA(j,i.S){
            s+=(a[j]-b[j]);
        }
        if(s!=0){
            cout<<"No"<<endl;
            return 0;
        }
    }
    cout<<"Yes"<<endl;
}
```

# [C問題](https://atcoder.jp/contests/arc106/tasks/arc106_c)

成り立つ場合を満たす構築はすぐにできたのですが、問題文の**成り立たない場合は-1を出力する**という文章を読み落として無駄な時間を過ごしました。

まず、**成り立たない場合の条件を確認**します。ここで、題意の二人は区間スケジューリング問題を解いており前者は正しく解けているので最大の個数の区間を選ぶことができています。したがって、$M \geqq 0$が必ず成り立ち、$M\<0$の場合は-1を出力します。

よって、以降では$M \geqq 0$のときの構築を考えます。区間スケジューリング問題を解いたことのある人ならわかるように、$L\_i$の昇順で選ぶ場合は最小の$L\_i$の区間が長い区間のパターンで$R\_i$の昇順で選ぶ場合よりも明らかに少ない区間の個数しか選ぶことができません。図で表すなら下図です。

![](/AtCoder/ARC_106/ARC_106_1.png)

上図では先ほど述べた最小の$L\_i$で長い区間が$l$個の区間を内包するとしています。このとき、前者は$n-1$個の区間を選べ、後者は$n-l$個の区間しか選べません。よって、この時$m=l-1$となります。また、$0 \leqq l \leqq n-1$となりますが、$l=0$のときは$m=0$なので、$1 \leqq l \leqq n-1 \leftrightarrow 0 \leqq m \leqq n-2$の時は構築をすることができます。また、$m=n-1,n$のときは構築ができていませんが、両者とも一つ以上の区間を選択することができるので$m=n$は成り立たず、$m=n-1$のときは高橋くんが$n$個の区間,青木くんが$1$個の区間を選択する場合ですが、高橋くんが$n$個の区間を選択するときはいずれの区間も重ならないので青木くんも$n$個の区間を選ぶことができます。

また、上記では$n=1,m=0$の場合を考慮できていないので、注意が必要です。これは$m=0$の場合は重ならない区間を並べるだけでできるので、$m=0$を先に場合分けすることでも回避できます。

```python:C.py
n,m=map(int,input().split())
if n==1 and m==0:
    print(1,2)
    exit()
if m==n or m==n-1 or m<0:
    print(-1)
    exit()
ans=[[1,10**7]]
for i in range(m+1):
    ans.append([2*(i+1),2*(i+1)+1])
for i in range(n-m-2):
    ans.append([10**7+2*(i+1),10**7+2*(i+1)+1])
#print(ans)
for i in ans:
    print(i[0],i[1])
```

#[D問題](https://atcoder.jp/contests/arc106/tasks/arc106_d)

まず、総和の問題でただの多項式なので**それぞれの$A\_i$の寄与などに注目する**ことを考えました。すると、**$(A\_L+A\_R)^x$を展開**したくなるので、二項定理により以下のようになります。

$(A\_L+A\_ R)^x=\_xC\_0 A\_L^x+\_xC\_1 A\_L^{x-1} A\_R+…+\_xC\_x A\_R^x$

ここで、ある$A\_i$の寄与を考えると、**$A\_i$は自身以外の全ての数と$(A\_L,A\_R)$の組になる**ので、上式を利用すれば、二項定理で展開して出てくるそれぞれの項について以下のようになります。(✳︎)

$\_xC\_0:A\_i^{x}(A\_1^0+…+A\_{i-1}^0+A\_{i+1}^0+…+A\_{n}^0)$
$\_xC\_1:A\_i^{x-1}(A\_1^1+…+A\_{i-1}^1+A\_{i+1}^1+…+A\_{n}^1)$
↓
$\_xC\_{x-1}:A\_i^{1}(A\_1^{x-1}+…+A\_{i-1}^{x-1}+A\_{i+1}^{x-1}+…+A\_{n}^{x-1})$
$\_xC\_{x}:A\_i^{0}(A\_1^{x}+…+A\_{i-1}^{x}+A\_{i+1}^{x}+…+A\_{n}^{x})$

また、このまま実装すると、$\_xC\_i$を前計算しても$x$の候補が$k$通り,$A\_i$の候補が$n$通り,上記のそれぞれの二項係数で$x+1$通り,$(\ )$内の計算で$n$回などとなり、愚直にやっても間に合わなそうです。よって、さらに式をまとめることを考えます。

ここで、任意の$A\_i$についてそれぞれの二項係数$\_xC\_i$での和を取る**ことを考えます。すると、下図のようになります。

![](/AtCoder/ARC_106/ARC_106_2.png)

よって、この和をまとめると$(A\_1^{x-i}+A\_2^{x-i}+…+A\_n^{x-i}) \times (A\_1^{i}+A\_2^{i}+…+A\_n^{i})-(A\_1^{x}+A\_2^{x}+…+A\_n^{x})$になります。

したがって、二項係数の前計算を行うだけでなく、$y[i]:=(A\_1^i+A\_2^i+…+A\_n^i)$の前計算を$i=0$~$k$で行右ことで($O(nk)$)、$\_xC\_i$での和は$O(1)$で求めることができます。したがって、それぞれの$x$での答えを求めるのに全体の計算量は$O(k^2)$となります。

以上より、全体の計算量は$O(n+nk+k^2)=O(k(k+n))$となり、十分高速です。

(✳︎)…これ以降は$1 \leqq L \leqq n-1,L+1 \leqq R \leqq n$ではなく、$1 \leqq L,R \leqq N,L \neq R$という条件を考えていますが、対称性より最終的な答えを1/2するだけで良いです。

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
#define MOD 998244353 //10^9+7:合同式の法
#define MAXR 1000 //10^5:配列の最大のrange
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
    //小数の桁数の出力指定
    //cout<<fixed<<setprecision(10);
    //入力の高速化用のコード
    //ios::sync_with_stdio(false);
    //cin.tie(nullptr);
    COMinit();
    ll n,k;cin>>n>>k;
    vector<ll> a(n);REP(i,n)cin>>a[i];
    //po
    vector<vector<mint>> po(n,vector<mint>(k+1,0));
    REP(i,n){
        po[i][0]=1;
        REP(j,k){
            po[i][j+1]=po[i][j]*a[i];
        }
    }
    vector<mint> y(k+1,0);
    REP(i,k+1){
        REP(j,n){
            y[i]+=po[j][i];
        }
    }
    vector<mint> ans(k,0);
    FOR(x,1,k){
        mint ans_sub=0;
        REP(j,x+1){
            ans_sub+=(COM(x,j)*((y[x-j])*(y[j])-y[x]));
        }
        ans[x-1]+=ans_sub;
    }
    REP(i,k){
        cout<<ans[i]/2<<endl;
    }
}
```
