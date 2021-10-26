# 感想

A,B問題しか解けず、B問題は後から嘘解法であったことが判明し、C問題を愚直な解法により解こうとしたら一部のケースのみWAとなり、散々な目に合いました。また、C問題は解法の通り、最長経路を用いることで解くことができました。

# [A問題](https://atcoder.jp/contests/arc105/tasks/arc105_a)

二つに分けて一致するかを考えます。この時、ある選び方をして選んだ数の合計が全体の合計のちょうど$\frac{1}{2}$になれば良いと考えます。

よって、$a+b,b+c,c+d,d+a,a+c,b+d,a,b,c,d$のいずれかがちょうど1/2になるかを判定すれば良いです。また、数を三つ選ぶ時は考える必要がありません。

```python:A.py
a,b,c,d=map(int,input().split())
cand=[a+b,b+c,c+d,d+a,a+c,b+d,a,b,c,d]
s=sum([a,b,c,d])
if s%2==1:
    print("No")
elif s//2 in cand:
    print("Yes")
else:
    print("No")
```

# [B問題](https://atcoder.jp/contests/arc105/tasks/arc105_b)

愚直にシミュレーションを行います。また、最終的に残る数は一つだけで同じ数には同じ操作を行うので、setで数を保存しておきます。最大の数と最小の数を選ぶので整列されているset（C++の場合）は好都合です。

コンテスト中にはテストケースが弱く通せましたが、この解法は嘘解法です。例えば、$1 \ 10^9$となる時に$10^9$回のsetへの挿入を行うからです。したがって、この操作がユークリッドの互除法であることに気づいて全体のgcdを取る解法が正答となります。また、シミュレーションの際に(最大値)-$x \times$(最小値)\>0となる中で最大の$x$の分だけまとめてシミュレートして良いので、これを行えばおそらく高速に動作するはずです。

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

signed main(){
    //入力の高速化用のコード
    //ios::sync_with_stdio(false);
    //cin.tie(nullptr);
    ll n;cin>>n;
    set<ll> ans;
    REP(i,n){
        ll a;cin>>a;
        ans.insert(a);
    }
    while(SIZE(ans)!=1){
        ll l,r;l=*ans.begin();r=*--ans.end();
        ans.erase(r);
        ans.insert(r-l);
    }
    cout<<*ans.begin()<<endl;
}
```

# [C問題](https://atcoder.jp/contests/arc105/tasks/arc105_c)

C問題は貪欲法でのACしようとしてできなかったので解説の最長経路を取り入れてプログラムを書きました。また、解説より計算量的には重いですが、枝刈りを入れて高速化しました。

まず、任意のラクダの並び順を試して$N!$通りです。さらに、それぞれの並び順について、それぞれのパーツを渡れる際に必要なラクダの間の距離を求めるのに、$O(M \times N^2)$です。したがって、計算量は$O(N! \times M \times N^2)$となり間に合わないので、**うまく枝刈りなどを行うことにより高速化**します。

上記の方針は、

①ラクダの並び順を定める
②それぞれのパーツ($M$通り)のあるラクダから始めてどのラクダ($N-1$通り)から載せることができないかを求める。

ことで実装できます。

また、②で**貪欲法を使うとコーナーケースを踏んでWA**になります。ここでは、あるラクダから始めてどのラクダから載せることができないか(そのパーツの長さだけ離れていなければならない)という情報を**辺とみなす**ことにより、**最初のラクダから最後のラクダまでの最長経路を求めるという問題に帰着**できれば良いです。

よって、$dp[i]:=$(最初のラクダから$i$番目のラクダまでの最長距離)としたDPを②において行えば良いです。あるパーツを選んだ時を遷移とみなしますが、ここでは詳細な説明は省略します。

---

(以下で、定数倍高速化を行います。)

(長さ,重さ)でパーツを管理して$a \geqq c$かつ$b \leqq d$となる$(a,b),(c,d)$が存在した時、$(a,b)$を満たしていれば$(c,d)$も成り立つので、任意のパーツ間で長さの大小と重さの大小が一致するようにパーツを減らすことができます。したがって、長さの昇順にパーツを並べて長さの大きいものから小さいものを見て上記が成り立つ場合は小さい方を使わないものとしてチェックをします。これを任意のパーツで行うので、全体の計算量は$O(M^2)$となります。また、**一度チェックされたパーツについては探索を行う必要がないので枝刈り**することができます。

次に、**重さが最大のラクダが端にない場合は端に置いたほうが良い**ので(証明はしていません)、最後のラクダを重さの最大のラクダで固定してよく$(N-1)!$通りのラクダの並び順のみ考えれば良いです。

また、パーツは重さの昇順($\leftrightarrow$長さの昇順)で並んでいるので、あるパーツがあるラクダから載せることができない時、次のパーツが載せられないラクダもそのラクダ以降となるので、**尺取法のような計算量の節約**をすることができます。

以上の定数倍高速化により、$O(N! \times M \times N\^2)$を$O\^2+(N-1)! \times N \times (M+N))$に落とすことができます。最悪ケースの$N=10,M=100000$を考えれば大体$10\^{11}$が$10\^{10}$になります。

```c++:C_fastest.cc
#pragma GCC optimize("Ofast")

#include<bits/stdc++.h>
using namespace std;

#define REP(i,n) for(int i=0;i<int(n);i++)
#define REPD(i,n) for(int i=n-1;i>=0;i--)
#define SIZE(x) int(x.size())
#define INF 2000000000
#define PB push_back
#define F first 
#define S second

int w[8];
pair<int,int> partsx[100000];
bool info[100000];
int n,m;
vector<pair<int,int>> parts;

inline int read(){
	int x=0,w=0;char ch=0;
	while(!isdigit(ch)){w|=ch=='-';ch=getchar();}
	while(isdigit(ch)){x=(x<<1)+(x<<3)+(ch^48);ch=getchar();}
	return w?-x:x;
}

signed main(){
    n=read();m=read();
    REP(i,n)w[i]=read();
    sort(w,w+n);
    REP(i,m){partsx[i].F=read();partsx[i].S=read();}
    sort(partsx,partsx+m);
    int ma=INF;
    REP(i,m)ma=min(ma,partsx[i].S);
    if(ma<w[n-1]){
        cout<<-1<<endl;
        return 0;
    }
    REP(i,m)info[i]=true;
    REPD(i,m){
        if(!info[i])continue;
        REP(j,i){
            if(partsx[i].S<=partsx[j].S){
                info[j]=false;
            }
        }
    }
    REP(i,m)if(info[i])parts.PB(partsx[i]);
    m=SIZE(parts);
    int ans=INF;
    do{
        vector<int> dp(n,0);
        REP(j,n-1){
            int k=j;int check=w[k];
            REP(i,m){
                while(k!=n){
                    if(parts[i].S<check){
                        dp[k]=max(dp[j]+parts[i].F,dp[k]);
                        break;
                    }
                    k++;
                    check+=w[k];
                }
                if(k==n){
                    dp[n-1]=max(dp[j],dp[n-1]);
                    break;
                }
            }
        }
        ans=min(ans,dp[n-1]);
    }while(next_permutation(w,w+n-1));
    cout<<ans<<endl;
}
```
