# 感想

精進を全くしてない割にはパフォーマンスが4桁キープできていて地力はついていると感じました。ただ、もっと精進しないとこの先には行けないので、努力したいです。

また、今回はDまで解けましたが、方針を立ててから解けるまでにだいぶ時間がかかりました。加えて、E問題もそこまで難しくなかったので、悔しさがさらに募りました。

# [A問題](https://atcoder.jp/contests/abc156/tasks/abc156_a)

内部レーティングと表示レーティングが違うことに注意して計算します。

```python:A.py
n,r=map(int,input().split())
if n>=10:
    print(r)
else:
    print(r+100*(10-n))
```

# [B問題](https://atcoder.jp/contests/abc156/tasks/abc156_b)

kで何回割れるかをwhile文を回して考え、nが0になったら終了します。

```python:B.py
ans=0
n,k=map(int,input().split())
while True:
    n=n//k
    ans+=1
    if n==0:
        break
print(ans)
```

# [C問題](https://atcoder.jp/contests/abc156/tasks/abc156_c)

一番小さいx座標から一番大きいx座標までを順にPとして考えていき、その中で最小の体力の総和を求めれば良いです。

```python:C.py
n=int(input())
x=list(map(int,input().split()))
x.sort()
y=[]
for i in range(x[0],x[-1]+1):
    su=0
    for j in range(n):
        su+=((x[j]-i)**2)
    y.append(su)
print(min(y))
```

#[D問題](https://atcoder.jp/contests/abc156/tasks/abc156_d)

この問題は方針自体は容易に考えることができます。なぜなら、下式が二項定理より明らかだからです。

$$\_n C \_1 + \_n C \_2 + … + \_n C \_{a-1} + \_n C \_{a+1} + … + \_n C \_{b-1} + \_n C \_{b+1} + … + \_n C \_n=2^n-1 - \_n C \_a - \_n C \_b$$

しかし、コンビネーションの計算をする際にnが10^9程度になることから、初めにコンビネーションの計算のための表を作る方法ではO(n)となり、TLEになってしまいます。

ここで、基本に立ち返ると、$ \_n C \_k = \frac{n}{1} \times \frac{n-1}{2} \times … \times \frac{n-k+1}{k}$となるため、前から順にかけ算を行うことで**常にその結果をintで保ったまま**計算を行うことで、$O(k)$で値を求めることができます。また、このかけ算の際にMOD($=10^9+7$)で割った余りを考えるのが面倒であると感じたので、modintのライブラリを利用しました。

```c++:answerD.cc
#include<iostream>
#include<vector>
#include<algorithm>
#include<utility>
#include<cmath>
using namespace std;
typedef long long ll;

const ll MAX = 200000;
const ll MOD=1000000007;
//マクロ
#define REP(i,n) for(ll i=0;i<(ll)(n);i++)
#define REPD(i,n) for(ll i=(ll)(n)-1;i>=0;i--)
#define FOR(i,a,b) for(ll i=(a);i<=(b);i++)
#define FORD(i,a,b) for(ll i=(a);i>=(b);i--)
#define ALL(x) (x).begin(),(x).end() //sortなどの引数を省略したい
#define SIZE(x) ((ll)(x).size()) //sizeをsize_tからllに直しておく
#define INF 1000000000000
#define PB push_back
#define MP make_pair
#define F first
#define S second

template<ll mod> class modint{
public:
    ll val=0;
    //コンストラクタ
    constexpr modint(ll x=0):val(x%mod){while(x<0)x+=mod;}
    //算術演算子
    constexpr modint operator +(const modint &r){return modint(*this)+=r;}
    constexpr modint operator -(const modint &r){return modint(*this)-=r;}
    constexpr modint operator *(const modint &r){return modint(*this)*=r;}
    constexpr modint operator /(const modint &r){return modint(*this)/=r;}
    //代入演算子
    constexpr modint &operator +=(const modint &r){
        val+=r.val;
        if(val>=mod)val-=mod;
        return *this;
    }
    constexpr modint &operator -=(const modint &r){
        if(val<r.val)val+=mod;
        val-=r.val;
        return *this;
    }
    constexpr modint &operator *=(const modint &r){
        val=val*r.val%mod;
        return *this;
    }
    constexpr modint &operator /=(const modint &r){
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
    constexpr bool operator ==(const modint& r){return this->val==r.val;}
    constexpr bool operator <(const modint& r){return this->val<r.val;}
    constexpr bool operator !=(const modint& r){return this->val!=r.val;}
    //入出力ストリーム
    friend constexpr istream &operator >>(istream &is,const modint<mod>& x){
        ll t;is >> t;
        x=modint<mod>(t);
        return (is);
    }
    friend constexpr ostream &operator <<(ostream &os,const modint<mod>& x){
        return os<<x.val;
    }
    //累乗
    friend constexpr modint<mod> modpow(const modint<mod> &a,ll n){
        if(n==0)return 1;
        modint<mod> t=modpow(a,n/2);
        t=t*t;
        if(n&1)t=t*a;
        return t;
    }
};

using mint = modint<MOD>;

signed main(){
    ll n,a,b;cin >> n >> a >> b;
    const mint x=2;
    mint ans=modpow(x,n);
    ans-=1;
    vector<mint> com(b+1);
    FOR(i,1,b){
        if(i==1){
            com[i]=n;
        }else{
            com[i]=com[i-1];
            com[i]*=(n-i+1);
            com[i]/=i;
        }
    }
    ans=ans-com[a]-com[b];
    cout << ans << endl;
}
```

# [E問題](https://atcoder.jp/contests/abc156/tasks/abc156_e)

一回の移動で複数人が動けると勘違いしていたらコンテストが終わっていました…。

まず、**それぞれの部屋に注目**した時、その部屋の住人が0人の時はその部屋にいた人が他の部屋に移動していることがわかります。したがって、k回の移動をした時は住人が0人の部屋がk個になると仮定することができます。さらに、0~k-1個の部屋が0人になるパターンも**余計な移動を付け加えれば良いだけ**なので、k回の移動により表現することができます(詳しくは[解説](https://img.atcoder.jp/abc156/editorial.pdf)を参照)。

したがって、**0人の部屋が0~k個ある場合のそれぞれについて数え上げを行えば良い**ことがわかります(ただし、$k \leqq n-1$)。

また、0人の部屋がi個あるとき、0人の部屋の選び方は$ \_n C \_i$通りであり、1人以上人がいる部屋がn-i個で合計の人数はn人なので、以下のように考えることで人数の組み合わせは$ \_{n-1} C \_{n-i-1}$通りとなり、0人の部屋がi個の時の部屋の人数の組み合わせは、$ \_n C \_i \times \_{n-1} C \_{n-i-1}$になります。

![](/AtCoder/ABC_156/ABC_156_1.png)

よって、MOD($=10^9+7$)のあまりを求めることに注意しつつ実装すると、以下のようになります。


```c++:answerE.cc
#include<algorithm>//sort,二分探索,など
#include<bitset>//固定長bit集合
#include<cmath>//pow,logなど
#include<complex>//複素数
#include<deque>//両端アクセスのキュー
#include<functional>//sortのgreater
#include<iomanip>//setprecision(浮動小数点の出力の誤差)
#include<iostream>//入出力
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
#define REP(i,n) for(ll i=0;i<(ll)(n);i++)
#define REPD(i,n) for(ll i=(ll)(n)-1;i>=0;i--)
#define FOR(i,a,b) for(ll i=(a);i<=(b);i++)
#define FORD(i,a,b) for(ll i=(a);i>=(b);i--)
#define ALL(x) (x).begin(),(x).end() //sortなどの引数を省略したい
#define SIZE(x) ((ll)(x).size()) //sizeをsize_tからllに直しておく
#define INF 1000000000000
#define MOD 1000000007
#define PB push_back
#define MP make_pair
#define F first
#define S second
const ll MAX = 200001;

ll fac[MAX], finv[MAX], inv[MAX];

// テーブルを作る前処理
void COMinit() {
    fac[0] = fac[1] = 1;
    finv[0] = finv[1] = 1;
    inv[1] = 1;
    for (ll i = 2; i < MAX; i++){
        fac[i] = fac[i - 1] * i % MOD;
        inv[i] = MOD - inv[MOD%i] * (MOD / i) % MOD;
        finv[i] = finv[i - 1] * inv[i] % MOD;
    }
}

// 二項係数計算
ll COM(ll n,ll k){
    if (n == 1) return 1;
    if (n < k) return 0;
    if (n < 0 || k < 0) return 0;
    return fac[n] * (finv[k] * finv[n - k] % MOD) % MOD;
}

signed main(){
    COMinit();
    ll n,k;cin >> n >> k;
    if(k>=n)k=n-1;
    ll ans=0;
    REP(i,k+1){
        ans+=(COM(n,i)*COM(n-1,n-i-1));
        ans%=MOD;
    }
    cout << ans << endl;
}
```
