# 感想

（不備があったので、省略します。）

# [A問題](https://atcoder.jp/contests/abc168/tasks/abc168_a)

hon,bon,ponで分けて出力します。  
リストの中に入っているかを考えれば良いです。

```python:A.py
n=input()
if int(n[-1]) in [2,4,5,7,9]:
    print("hon")
elif int(n[-1]) in [3]:
    print("bon")
else:
    print("pon")
```

# [B問題](https://atcoder.jp/contests/abc168/tasks/abc168_b)

長さがk以下で処理を分けます。

```python:B.py
k=int(input())
s=input()
if len(s)<=k:
    print(s)
else:
    print(s[:k]+"...")
```

# [C問題](https://atcoder.jp/contests/abc168/tasks/abc168_c)

下図を書いて12時の方向からの角度を考えると、なす角は$|\frac{h+\frac{m}{60}}{12}-\frac{m}{60}|$になり、求めたいのは分針の先と磁針の先の角度になります。

![](/AtCoder/ABC_168/ABC168_1.png)

```python:C.py
import math
a,b,h,m=map(int,input().split())
q=abs((h+m/60)/12-m/60)*360
print(math.sqrt(a**2+b**2-2*a*b*math.cos(math.radians(q))))
```

#[D問題](https://atcoder.jp/contests/abc168/tasks/abc168_d)

まず、問題を見て制約が厳しそうなのでダイクストラ法と考えましたが、この問題ではそれぞれの経路の長さが等しいので、BFSやDFSで十分に実装可能です。また、辿る頂点を保存するタイプのダイクストラ法の問題を解いたことがなかったので、体感として非常に難しく感じました。

## ダイクストラ法による方法

ダイクストラ法を用いて**単一始点の最短経路**を求められますが、全ての辺を逆向きにすると**単一終点の最短経路**を求めることができます。この問題では、全ての辺が双方向で全ての辺を逆向きにしても辺の組み合わせは同じになるので、それぞれの頂点について**頂点1からの最短経路**を考えることでダイクストラ法を用いることができます。また、**それぞれの頂点から頂点1に向かって最短経路で向かう際に次に進むべき頂点の番号**を求めたいので、頂点1からの最短経路とみなす場合は、**頂点1からそれぞれの頂点に最短経路で向かう際に直前にいた頂点の番号**と言い換えることができます。

ここで、ダイクストラ法においてPriorityqueueで管理する**最短経路でたどりつくと思われる頂点の情報**に**その前にいた頂点がどこか**を付け加えることで、上記の言い換えを実装するすることができます。よって、以下のようなコードになります。

```c++:D.cc
//インクルード(アルファベット順,bits/stdc++.hは使わない派閥)
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
#define MOD 10000007 //10^9+7:合同式の法
#define MAXR 100000 //10^5:配列の最大のrange(素数列挙などで使用)
//略記
#define PB push_back //vectorヘの挿入
#define MP make_pair //pairのコンストラクタ
#define F first //pairの一つ目の要素
#define S second //pairの二つ目の要素

//ここから上はテンプレート

#define PLL pair<ll,ll>
#define VLL vector<ll>
//昇順の場合はgreater
//pairの比較は一つ目の要素→二つ目の要素の優先順位
#define PQ priority_queue<VLL,vector<VLL>,greater<VLL>>


//fは始点のインデックス
//nは頂点の総数
//edgeはそれぞれの頂点から伸びる辺についてその辺の先の頂点のインデックスおよび距離を持つ配列
//1からの最小距離でたどり着いたところにその前のやつ保存
pair<vector<ll>,vector<ll>> dijkstra(ll f,ll n,vector<vector<PLL>>& edge){
    //最短経路としてどの頂点が確定済みかをチェックする配列
    vector<ll> confirm(n,false);
    //それぞれの頂点への最短距離を保存する配列
    //始点は0,始点以外はINFで最短距離を初期化する
    vector<ll> mincost(n,INF);mincost[f]=0;
    //どの頂点に行けば良いかの道標(signpost)を保存する
    vector<ll> signpost(n,-1);
    //確定済みの頂点の集合から伸びる辺を伝ってたどり着く頂点の始点からの距離を短い順に保存するPriority queue
    //どの頂点からきたかも保存する(最短経路の保存)
    vector<ll> data={mincost[f],f,0};
    //要素が三つあるのでpairでなくvectorを用いる
    PQ mincand;mincand.push(data);signpost[f]=0;

    //mincandの要素がゼロの時、最短距離を更新できる頂点がないことを示す
    while(!mincand.empty()){
        //最短距離でたどり着くと思われる頂点を取り出す
        VLL next=mincand.top();mincand.pop();
        //すでにその頂点への最短距離が確定済みの場合は飛ばす
        if(confirm[next[1]]) continue;
        //確定済みではない場合は確定済みにする
        confirm[next[1]]=true;
        //確定済みのとき道標も決定する
        signpost[next[1]]=next[2];
        //その確定済みの頂点から伸びる辺の情報をとってくる(参照の方が速い)、lは辺の本数
        vector<PLL>& v=edge[next[1]];ll l=SIZE(v);
        REP(i,l){
            //辺の先が確定済みなら更新する必要がない((✳︎2)があれば十分なので(✳︎1)は実はいらない)
            if(confirm[v[i].F]) continue; //(✳︎1)
            //辺の先のmincost以上の場合は更新する必要がない(辺の先が確定済みの時は満たす)
            if(mincost[v[i].F]<=next[0]+v[i].S) continue; //(✳︎2)
            //更新
            mincost[v[i].F]=next[0]+v[i].S;
            //更新した場合はその頂点が(確定済みでない頂点の中で)最短距離になる可能性があるのでmincandに挿入
            //どの頂点から次の頂点にたどり着くかも保存する
            vector<ll> data_sub={mincost[v[i].F],v[i].F,next[1]};
            mincand.push(data_sub);
        }
    }
    return MP(mincost,signpost);
}

signed main(){
    ll n,m;cin >> n >> m;

    vector<vector<PLL>> edge(n);
    REP(i,m){
        ll a,b;cin >> a >> b;
        edge[a-1].PB(MP(b-1,1));
        edge[b-1].PB(MP(a-1,1));//逆向きの辺も挿入
    }

    pair<vector<ll>,vector<ll>>vv=dijkstra(0,n,edge);

    ll ans=0;
    if(find(ALL(vv.S),-1)==(vv.S).end()){
        cout << "Yes" << endl;
        REP(i,n-1) cout << (vv.S)[i+1]+1 << endl;
    }else{
        cout << "No" << endl;
    }
}
```

## BFSによる方法

任意の辺の長さが等しいので、BFSやDFSなどのグラフ探索アルゴリズムでも探索可能です。また、今回は最短経路を考えることから、**常に同じ深さ(距離)で辿り着ける頂点を保存するBFS**の方がDFSよりも効率よく最短経路を発見することができます。

基本的な実装はダイクストラ法と同じで、辿り着けるもので今まで訪れてないものを順に辿り、その際に直前にいた頂点も保存することを繰り返せば実装することができます。

また、Pythonでは、**再帰の上限回数の制限を再帰しうる回数以上にする**こと、保存するのにはO(1)で挿入および削除が行えるキューとして**collectionsモジュールのdequeを使用する**こと、にそれぞれ注意が必要です。

```python:answerD.py
import sys
sys.setrecursionlimit(10**6)
from collections import deque
n,m=map(int,input().split())
path=[[] for i in range(n)]
for i in range(m):
    a,b=map(int,input().split())
    path[a-1].append(b-1)
    path[b-1].append(a-1)
nan=deque([0])
check=[1 if i==0 else -1 for i in range(n)]
def bfs():
    global n,m,path,nan
    l=len(nan)
    if not l:
        return
    for _ in range(l):
        x=nan.popleft()
        for j in path[x]:
            if check[j]==-1:
                check[j]=x
                nan.append(j)
    bfs()
bfs()
if any([i==-1 for i in check]):
    print("No")
else:
    print("Yes")
    for i in range(1,n):
        print(check[i]+1)
```


#[E問題](https://atcoder.jp/contests/abc168/tasks/abc168_e)

まず、$A\_i A\_j + B\_i B\_j=0$の式を変形してこれを満たす$(i,j)$の組(仲の悪いサンマの組)を見つけたいですが、このような等式は$\frac{A\_i}{B\_i}=-\frac{B\_j}{A\_j}$として変形すると、**左辺は$i$のみに依存し右辺は$j$のみに依存する**ので、このような$(i,j)$の組がないようにすれば良いことがわかります。また、この際に0除算が発生する可能性があるため、**$A\_i,A\_j,B\_i,B\_j$に0が含まれる場合を別にして考えた方が良い**ことがわかります。

まず、$A\_i,A\_j,B\_i,B\_j$に0が含まれない場合で考えます。ここで、$\frac{A\_i}{B\_i}=-\frac{B\_j}{A\_j}$となる$(i,j)$の組について**分数どうしが等しいかを評価する**必要があるので、**分子と分母の組どうしが等しいかを評価すれば良い**と考えました。

また、$\frac{A\_i}{B\_i}=-\frac{B\_j}{A\_j}$は分子と分母の組が等しくなくても成り立ちます。例えば、$(A\_i,B\_i)=(2,3),(-2,-3),(4,6)$は全て$(A\_j,B\_j)=(3,-2)$に対して仲が悪いです。したがって、$(A\_i,B\_i)=(2,3),(-2,-3),(4,6)$は全て同じ組とみなすことができます。ここで、**分数が等しいかは既約分数にした時に等しい**かで決まるので、$|A\_i|$と$|B\_i|$の最大公約数で$A\_i$と$B\_i$をそれぞれ割って$A_i$が正になるように調整すれば、これらを同じ組とみなすことができます。(**傾きが等しいか**と捉えるとわかりやすいです。)

また、$A\_i,B\_i$に0が含まれるような$i$匹目のイワシを考えると、$A\_i,B\_i=0,0$の場合はどのイワシと一緒でも$A\_i A\_j + B\_i B\_j=0$が常に成り立つので、ありうるのはそのイワシ一匹のみを選ぶ場合のみになります。$A\_i=0,B\_i \neq0$の場合は$B\_i=0,A\_i \neq0$のイワシと仲が悪いので、前者は$A\_i,B\_i=0,1$で$A\_i,B\_i=1,0$として処理をすれば先ほどの$A\_i,A\_j,B\_i,B\_j$に0が含まれない場合と同様の扱いをすることができます。

ここで、$(A_i,B_i)$を持つm匹のサンマと$(A_j,B_j)$を持つn匹のサンマの間で$A_i A_j + B_i B_j=0$が成り立つとします。この時のサンマの選び方は$2^m+2^n-1$通りになります。なぜなら、m匹のサンマのうちの一匹でも選んだらn匹のサンマは一匹も選べず逆もまた然りだからです。

また、それぞれのサンマを選べるかは**仲の悪いサンマが選ばれたかのみ**に依存するので、それ以外のサンマの選び方とは**独立に**決まります。したがって、仲の悪いサンマの間での組み合わせの数($2^m+2^n-1$通り)が求まれば、**他の仲の悪いサンマの間での組み合わせの数との積**によりその組み合わせを求めることができます。また、仲の悪いサンマがない場合は通常通り$2^k$として組み合わせの数を求めることができます。

```python:answerE.py
#10**9+7のあまりを忘れていた
import math 
n=int(input())
d=dict()
for i in range(n):
    a,b=map(int,input().split())
    if a==0 and b==0:
        pass
    elif a==0:
        a,b=0,1
    elif b==0:
        a,b=1,0
    else:
        x=math.gcd(abs(a),abs(b))
        a,b=a//x,b//x
        if a<0:
            a,b=-a,-b
    if (a,b) in d:
        d[(a,b)]+=1
    else:
        d[(a,b)]=1
#こっからは数え上げ
#print(d)
ans1,ans2=1,0
for i in d:
    if d[i]==0:
        continue
    if i==(0,0):
        ans2=d[i]
        d[i]=0
        continue
    #独立…
    a,b=i
    if (-b,a) in d:
        ans1*=(2**(d[i])+2**(d[(-b,a)])-1)
        d[(-b,a)]=0
        d[i]=0
    elif (b,-a) in d:
        ans1*=(2**(d[i])+2**(d[(b,-a)])-1)
        d[(b,-a)]=0
        d[i]=0
    else:
        ans1*=(2**(d[i]))
        d[i]=0
print((ans1+ans2-1)%(10**9+7))
```
