# 感想
簡単だと思っていたC問題をバグらせ続け、きつかったです。

# [A問題](https://yukicoder.me/problems/no/1168)

そこまで大きな数にはならないので愚直に100回行います。

```python:A.py
def calc(n):
    ret=0
    while n!=0:
        ret+=(n%10)
        n//=10
    return ret
N=int(input())
for i in range(100):
    N=calc(N)
print(N)
```

# [B問題](https://yukicoder.me/problems/no/1169)

こういう構築の場合は闇雲に決めると難しいので、まずは自分でルールを決めることを考えます。今回はそれぞれの行で**数が連続するように**決めます。また、$N=5$場合は以下の二つのパターンを考えましたが、一つ目のパターンでは行と対角成分は条件を満たしますが、列が条件を満たさないので二つ目のパターンが正解です。

```
(左→右で連続)
12345
12345
12345
12345
12345

(右→左で連続)
15432
32154
54321
21543
43215
```

また、上記の行列について、自分は一番最後の要素が+2ずつされていることに注目して実装を行いました。

```python:B.py
n=int(input())
ans=[]
st=2
if n==1:exit(print(1))
for i in range(n):
    now=[(st+i)%n if st+i!=n else n for i in range(n)][::-1]
    st+=2
    st%=n
    if st==0:st=n
    print(" ".join(map(str,now)))
```

# [C問題](https://yukicoder.me/problems/no/1170)

やることは比較的見えやすいのですが、**探索済みのところを複数回探索しないように気をつける**必要があります。また、以下では駅を頂点と言い換えています。

まずは初めの以下のサンプルで検証します。

```
5 4 6
0 2 5 7 8
```

![](/yukicoder/Contest_261/Contest_261_1.png)

上図のように、辺で繋がる部分集合を考え、ある部分集合$X$に含まれるインデックスの場合は$X$のサイズを求めます。また、**辺で繋がっていない互いに素な部分集合の集まりを素集合族と見なす**ことができ、**UnionFind木**を使うことで簡単に管理することができます。

ここで、UnionFind木でuniteする頂点を座標の小さい頂点から順に選んでいきますが、距離がA以上B以下の頂点はつなげられるので、今選んだ頂点の座標を$C$とすれば$C-B$以上$C-A$以下または$C+A$以上$C+B$以下の頂点を選べば良いことになります。また、**座標の小さい頂点から**繋ぐ辺を選びその辺は双方向性なので、$C+A$以上$C+B$以下の頂点を選べば良いです。

また、この工夫のみでは計算量的にまずいです。なぜなら、$C+A$以上$C+B$以下の頂点を選ぶことによる最悪計算量は$O(N^2 \alpha(n))$であるからです。$\alpha(n)$はアッカーマン関数$A(n,n)$の逆関数で$\log{n}$より小さいです。

ここで、探索済みの頂点を考えるために、今見ている頂点の座標を$C$,一つ手前で見た頂点の座標を$C'$とすれば、下図のようになります。

![](/yukicoder/Contest_261/Contest_261_2.png)


上図より、$C+B$以下で最大の頂点から順に座標が小さくなる方向で頂点をたどって辺を繋いでいき、同じ素集合にある頂点にぶつかった場合は**それ以後の探索を打ち切る**という方法により探索を効率化することができます。これにより探索範囲が被らないので、$O(N \alpha(n))$となって高速なプログラムを書くことができます。また、最終的に求める素集合族に含まれる集合のサイズはsize関数により$O(1)$で求めることができます。

```c++:C.cc
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

    //コンストラクタの:の後ろでメンバ変数を初期化
    UnionFind(ll n):parent(n),siz(n,1){ 
        //最初は全ての要素の根はそれ自身であるとして初期化
        for(ll i=0;i<n;i++){parent[i]=i;}
    }

    ll root(ll x){ //データxの属する木の根を取得
        if(parent[x]==x) return x;
        return parent[x]=root(parent[x]);
        //代入式の値は代入した変数の値になる
        //経路圧縮(根に直接要素を繋ぐことで計算を効率化する)を行っている
    }

    void unite(ll x,ll y){ //xとyの木を併合
        ll rx=root(x);//xの根をrx
        ll ry=root(y);//yの根をry
        if(rx==ry) return; //同じ木にある時
        //小さい集合を大きい集合へと併合(ry→rxへ併合)
        if(siz[rx]<siz[ry]) swap(rx,ry);
        siz[rx]+=siz[ry];
        parent[ry]=rx; //xとyが同じ木にない時はyの根ryをxの根rxにつける
    }

    bool same(ll x,ll y){//xとyが属する木が同じかを判定
        ll rx=root(x);
        ll ry=root(y);
        return rx==ry;
    }

    ll size(ll x){ //xの素集合のサイズを取得
        return siz[root(x)];
    }

    //↓他の人のライブラリにはないけど便利だと思う関数
    //O(n)であることに注意
    void grouping(ll n){//素集合どうしをグループ化
        REP(i,n){
            if(group.find(parent[i])==group.end()){
                group[parent[i]]={i};
            }else{
                group[parent[i]].PB(i);
            }
        }
    }
};

signed main(){
    //入力の高速化用のコード
    //ios::sync_with_stdio(false);
    //cin.tie(nullptr);
    ll n,a,b;cin>>n>>a>>b;
    vector<ll> x(n);
    REP(i,n)cin>>x[i];
    UnionFind u(n);
    //チェックしすぎない
    REP(i,n){
        ll now=upper_bound(ALL(x),x[i]+b)-x.begin();now--;
        //ここかな
        //上からか
        while(now>=0){
            if(x[now]<x[i]+a)break;
            if(u.same(i,now))break;
            u.unite(i,now);
            now--;
        }
    }
    REP(i,n)cout<<u.size(i)<<endl;
}
```


# [D問題](https://yukicoder.me/problems/no/1171)

runのでき方に注目すると、部分列を選んだ時に**前後の文字が異なれば**runの数は一つ増えます。したがって、前から文字列$S$を走査する際に**最後の文字が何か**を保存しておけば良さそうであると考えました。また、最後の文字の種類(**状態**)は高々26通りで、走査する際の遷移も簡単に定義できそうなのでDPを用いて解くことにしました。

したがって、初めは以下のようにDPを定義しました。

$dp[i][j]:=$($i$文字目まで決めた時に最後のアルファベットが$j$(=0~25)になる場合のrunの数)

しかし、この定義だと$dp[i-1]$から$dp[i]$への遷移を考える時の加算がうまくいきません。ここで必要な情報を精査すると、**$i$文字目まで決めた時の文字列の個数**がrun数の計算で必要であることがわかります。つまり、DPを以下のように定義すれば良いです。

$dp[i][j]:=$($i$文字目まで決めた時に最後のアルファベットが$j$(=0~25)になる場合の(文字列の個数,runの数))

この定義をすれば、遷移を下記の三通りに場合分けすることで実装することができます。また、$i$文字目のアルファベットを$d$とおき、dpの配列は0初期化されているとします。

(1)$i$文字目を部分列の最初の文字として選ぶ場合
$dp[i][d][0]+=1$
$dp[i][d][1]+=1$

(2)$i$文字目を部分列に含まない場合
$dp[i][j][0]+=dp[i-1][j][0]$
$dp[i][j][1]+=dp[i-1][j][1]$

(3)$i$文字目を部分列に含む場合
[1]$j=d$の時
$dp[i][d][0]+=dp[i-1][j][0]$
$dp[i][d][1]+=dp[i-1][j][1]$
[2]$j \neq d$の時(runは文字列の分だけ増える)
$dp[i][d][0]+=dp[i-1][j][0]$
$dp[i][d][1]+=(dp[i-1][j][0]+dp[i-1][j][1])$

また、これで$dp[n-1]$のrunの和を求めますが、このままだとMLEになります。ここで、遷移は$i-1$→$i$のみなので、この二つのみを保存することで空間計算量を減らします。

```python:D_MLE.py
s=input()
n=len(s)
#dp[i][j]:=i文字目まで決めた時にアルファベットjになるような(文字列の個数,Runの個数)
#ペアにしてもつの珍しいけど、どちらもわからないと決まらないので
#MLEになった
dp=[[[0,0] for i in range(26)] for i in range(n)]
mod=10**9+7
for i in range(n):
    d=ord(s[i])-97
    #新しく追加
    dp[i][d]=[1,1]
    if i==0:continue
    #選ばない場合(増えはしない)
    for j in range(26):
        dp[i][j][0]+=dp[i-1][j][0]
        dp[i][j][0]%=mod
        dp[i][j][1]+=dp[i-1][j][1]
        dp[i][j][1]%=mod
    #選ぶ場合(異なればその文字列数ぶんだけ増える)
    for j in range(26):
        if j==d:
            dp[i][j][0]+=dp[i-1][j][0]
            dp[i][j][0]%=mod
            dp[i][j][1]+=dp[i-1][j][1]
            dp[i][j][1]%=mod
        else:
            dp[i][d][0]+=dp[i-1][j][0]
            dp[i][d][0]%=mod
            dp[i][d][1]+=(dp[i-1][j][0]+dp[i-1][j][1])
            dp[i][d][1]%=mod
ans=0
for j in range(26):
    ans+=dp[n-1][j][1]
    ans%=mod
print(ans)
```

```python:D_AC.py
s=input()
n=len(s)
#dp[i][j]:=i文字目まで決めた時にアルファベットjになるような(文字列の個数,Runの個数)
#ペアにしてもつの珍しいけど、どちらもわからないと決まらないので
#MLEになった
#dpを配列にしない
x,y=[[0,0] for i in range(26)],[[0,0] for i in range(26)]
mod=10**9+7
for i in range(n):
    d=ord(s[i])-97
    #新しく追加
    if i==0:
        x[d]=[1,1]
        continue
    #選ばない場合(増えはしない)
    for j in range(26):
        y[j][0]=x[j][0]
        y[j][0]%=mod
        y[j][1]=x[j][1]
        y[j][1]%=mod
    y[d][0]+=1
    y[d][1]+=1
    #選ぶ場合(異なればその文字列数ぶんだけ増える)
    for j in range(26):
        if j==d:
            y[j][0]+=x[j][0]
            y[j][0]%=mod
            y[j][1]+=x[j][1]
            y[j][1]%=mod
        else:
            y[d][0]+=x[j][0]
            y[d][0]%=mod
            y[d][1]+=(x[j][0]+x[j][1])
            y[d][1]%=mod
    x,y=y,x
ans=0
for j in range(26):
    ans+=x[j][1]
    ans%=mod
print(ans)
```
