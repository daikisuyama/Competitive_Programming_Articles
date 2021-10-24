# 感想

D問題まではスムーズに通せましたが、Eが簡単にもかかわらずD問題までで集中力が切れてしまいました。反省です。

また、F問題で嘘解法にハマってしまいました。

# [A問題](https://atcoder.jp/contests/abc177/tasks/abc177_a)

$t \times s \geqq d$を満たすかを考えます。

```python:A.py
n,x,t=map(int,input().split())
y=-((-n)//x)
print(y*t)
```

# [B問題](https://atcoder.jp/contests/abc177/tasks/abc177_b)

$len(s) \times len(t)$は$10^6$程度なので、$t$と一致する$s$の部分列の候補($len(s)-len(t)+1$通り)を全探索します。

```python:B.py
s,t=input(),input()
ls=len(s)
lt=len(t)
ans=1000000000000
for i in range(ls-lt+1):
    ans_sub=0
    for j in range(lt):
        if t[j]!=s[i+j]:
            ans_sub+=1
    ans=min(ans,ans_sub)
print(ans)
```

# [C問題](https://atcoder.jp/contests/abc176/tasks/abc176_c)

$\frac{(A\_1+A\_2+…+A\_N)^2-(A\_1^2+A\_2^2+…+A\_N^2)}{2}$が解となります。二つの数字の掛け合わせを全て求めたいので、この式を思いつくことができます。

```python:C.py
n=int(input())
a=list(map(int,input().split()))
x=sum(a)
y=sum([(a[i])**2 for i in range(n)])
print(((x**2-y)//2)%(10**9+7))
```

# [D問題](https://atcoder.jp/contests/abc177/tasks/abc177_d)

友達関係により**連結されていく**ので、まず**UnionFind**を疑います。つまり、与えられた友達関係によって結合された集合に含まれる人同士が友達です。

ここで、グループ内に友達がいないように$N$人をできるだけ少ないグループ数にわけます。このグループ数は素集合系のそれぞれの集合の中で最大の要素数以上であれば良いため、これを出力します。

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

//以下、素集合と木は同じものを表す
class UnionFind {
public:
    vector<ll> parent; //parent[i]はiの親
    vector<ll> siz; //素集合のサイズを表す配列(1で初期化)
    map<ll,vector<ll>> group;//素集合ごとに管理する連想配列(keyはそれぞれの素集合の親、valueはその素集合の要素の配列)

    //コンストラクタ
    UnionFind(ll n):parent(n),siz(n,1){ 
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
    void grouping(ll n){
        //経路圧縮を先に行う
        REP(i,n)root(i);
        //mapで管理する(デフォルト構築を利用)
        REP(i,n)group[parent[i]].PB(i);
    }
};

signed main(){
    //入力の高速化用のコード
    //ios::sync_with_stdio(false);
    //cin.tie(nullptr);
    ll n,m;cin>>n>>m;
    UnionFind uf(n);
    REP(i,m){
        ll a,b;cin>>a>>b;
        uf.unite(a-1,b-1);
    }
    uf.grouping(n);
    ll ans=0;
    FORA(i,uf.group){
        ans=max(ans,SIZE(i.S));
    }
    cout<<ans<<endl;
}
```

# [E問題](https://atcoder.jp/contests/abc177/tasks/abc177_e)

まず、二つの条件を処理する必要があります。一つ目は任意の$1 \leqq i \< j \leqq N$で$GCD(A\_i,A\_j)=1$が成り立つことです。また、二つ目は$GCD(A\_1,A\_2,…,A\_N)=1$が成り立つことです。二つ目の条件は$O(N \log{A_{max}})$を行うだけで良いですが、一つ目の条件については安直にやると$O(N^2 \log{A_{max}})$となります。

したがって、一つ目の条件をさらに言い換えます。まず、**任意の**というのが扱いにくかったので、否定を考えて$GCD(A\_i,A\_j) \neq 1$となる$i,j$の組が**少なくとも一つ存在する場合**を考えることにしました。この時、$GCD(A\_i,A\_j) \neq 1$となる時は(1でない)**共通する約数が存在する**ことを示せれば良いです。したがって、それぞれの$A\_i$の約数を列挙して同じ約数が存在するかをチェックすることにしました。しかし、この方法では計算量が$O(N\sqrt{A\_{max}})$なので、難しいと判断しました(C++ではこの方法でも通るようです。)。

ここで、約数を全て列挙するのではなく素因数分解に含まれる素数のみを列挙すれば良いことに気付きました。つまり、$12=2 \times 2 \times 3$であれば$2,3$を列挙します。この時、エラトステネスの篩により$O(\log{A_{i}})$での素因数分解を行うことができるため、全体の計算量は$O(N\log{A_{i}})$となります。

以上より以下の順番で実装を行います。

1. 素因数分解で用いる配列`PN_chk`,約数が何回でてくるかをチェックする配列`check`,それぞれの$A\_i$の素因数分解で出てくる数をチェックする辞書(or集合)`prime`を用意する。
2. エラトステネスの篩を利用して`PN_chk`の初期化を行う。
3. それぞれの$A\_i$について素因数分解を行い`prime`に格納する。格納した素数は`check`に記録する。
4. `check`に記録されたそれぞれの素数の個数で2以上のものがあるかをチェックする。一つでも存在する場合は全体のgcdが1の時に`setwise coprime`、そうでない時に`not coprime`となる。また、一つも存在しない場合は`pairwise coprime`となる。

```c++:E.cc
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
#define MAXR 1000000 //10^6:配列の最大のrange
//略記
#define PB push_back //挿入
#define MP make_pair //pairのコンストラクタ
#define F first //pairの一つ目の要素
#define S second //pairの二つ目の要素

#define PN 1 //素数のマークは1

//MAXR(=10^6)までの整数が素数であると仮定する
vector<ll> PN_chk(MAXR+1,PN);//0indexでi番目が整数iに対応(0~MAXR)


//O(MAXR)
void se(){
    ll MAXRR=sqrt(MAXR)+1;
    //2以上の数の素因数分解をするので無視して良い
    PN_chk[0]=0;PN_chk[1]=0;
    FOR(i,2,MAXRR){
        //たどり着いた時に素数と仮定されているなら素数
        //ある素数の倍数はその素数で割り切れるのでマークづけ
        if(PN_chk[i]==1) FOR(j,i,ll(MAXR/i)){PN_chk[j*i]=i;}
    }
}

vector<ll> check(MAXR+1,0);

//O(logn)
//primeはこの場合は整列されてないので注意！！！
//一回のみのcheck
map<ll,ll> prime;

void prime_factorize(ll n){
    if(n<=1) return;
    //1の場合はnは素数
    if(PN_chk[n]==1){prime[n]+=1;return;}
    //マークされている数は素数
    prime[PN_chk[n]]+=1;
    //マークされている数でnを割った数を考える
    prime_factorize(ll(n/PN_chk[n]));
}

signed main(){
    se();
    //入力の高速化用のコード
    //ios::sync_with_stdio(false);
    //cin.tie(nullptr);
    ll n;cin>>n;
    vector<ll> a(n);REP(i,n)cin>>a[i];
    //check二回つくか
    REP(i,n){
        prime_factorize(a[i]);
        FORA(j,prime){
            check[j.F]++;
        }
        prime.clear();
    }
    ll g=gcd(a[0],a[1]);
    FOR(i,2,n-1){
        g=gcd(g,a[i]);
    }
    REP(i,MAXR+1){
        if(check[i]>=2){
            if(g==1){
                cout<<"setwise coprime"<<endl;
            }else{
                cout<<"not coprime"<<endl;
            }
            return 0;
        }
    }
    cout<<"pairwise coprime"<<endl;
}

```

# [F問題](https://atcoder.jp/contests/abc177/tasks/abc177_f)

まず、下に移動するか右に移動するかを考えますが、$k+1$行目に移動する時に下に移動する回数は必ず$k$回です。したがって、移動の最小回数は**右に移動する最小回数のみ**を考えれば良いです(**操作の分離**)。

また、ある始点から移動する際に右に移動する回数を最小にしたいので、下に行ける時は下に行けば良いです(**貪欲な移動**)。したがって、**始点が決まれば終点は一意に決まる**と言えます。また、右にいかなければならないのは今いる点が$A\_i$から$B\_i$に含まれる時のみです。

よって、(終点,始点)の候補を管理しながら上の行から順に調べていきます。また、上記の考察から、終点が$A\_i$から$B\_i$に含まれる場合のみ終点を$B\_{i}+1$へと移動させますが、含まれない場合は終点を移動させる必要はありません。さらに、終点を移動させる時に複数の始点が存在する場合はその中で**最大の始点のみを残せば良い**です。また、$B\_{i}+1$が$W$になる場合は、(終点,始点)の候補の更新は必要ありません。

さらに、終点が$A\_i$から$B\_i$に含まれる場合を求めるには(終点,始点)の組を`set`に格納して`lower_bound`と`upper_bound`を利用して求めれば良いです。また、含まれる場合はその要素数を減らしますが、**全体として減らす回数は$W$を超えない**ので、全体の計算量は$O((H+W) \log{(H+W)})$となります。

また、それぞれの行での**移動回数の最小値を高速に求める**ために、**`multiset`で終点-始点の値を保存**しておき、先ほどの`set`と同様に削除や追加を行います。

よって、実装をまとめて以下のようになります。

1. (終点,始点)を管理する`set`のtofrom,移動回数(終点-始点)を保存する`multiset`のcostを用意する。
2. tofromとcostを初期化する。
3. それぞれの行での操作を行う。
    1. $[A\_i,B\_i]$に終点が入るものについては終点を$B\_i+1$に移動させ、tofromとcostのいずれも変化させる。また、移動させる必要のない場合がありその際の実装に注意する。そして、$B\_{i}+1$が$W$になる場合はtofromには挿入を行わずにcostにのみ挿入を行う。
    2. costを見て、最小の要素がINFでない場合はその要素を出力し、INFの場合は-1を出力する。

```c++:F.cc
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
#define MOD 1000000 //10^9+7:合同式の法
#define MAXR 100000 //10^5:配列の最大のrange
//略記
#define PB push_back //挿入
#define MP make_pair //pairのコンストラクタ
#define F first //pairの一つ目の要素
#define S second //pairの二つ目の要素

signed main(){
    //入力の高速化用のコード
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    ll h,w;cin>>h>>w;
    //終点,始点を管理
    set<ll> tofrom;
    //最小を管理
    multiset<ll> cost;
    REP(i,w){
        tofrom.insert(i*MOD+i);
        cost.insert(0);
    }
    REP(i,h){
        ll a,b;cin>>a>>b;a--;b--;
        auto j=tofrom.lower_bound(a*MOD);
        if(j!=tofrom.end()){
            ll ne=-1;
            while(j!=tofrom.end() or *j<(b+2)*MOD){
                cost.erase(cost.lower_bound(ll(*j/MOD)-*j%MOD));
                ne=*j%MOD;
                j=tofrom.erase(j);
            }
            if(b==w-1){
                cost.insert(INF);
            }else{
                cost.insert(b+1-ne);
                tofrom.insert((b+1)*MOD+ne);
            }
        }
        if(*cost.begin()!=INF)cout<<*cost.begin()+i+1<<"\n";
        else cout<<-1<<"\n";
    }
}
```
