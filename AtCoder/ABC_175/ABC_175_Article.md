# 感想

メンタルを崩さず久しぶりに良い成績を取れたので嬉しかったです。

# [A問題](https://atcoder.jp/contests/abc175/tasks/abc175_a)

3,2,1,0しか出力がないのでそれぞれの場合で場合分けしました。  
もっと綺麗な解法があるようなので、A問題から綺麗に解くことを意識します。

```python:A.py
s=input()
if s=="RRR":
    print(3)
elif s=="RRS" or s=="SRR":
    print(2)
elif s=="SSS":
    print(0)
else:
    print(1)
```

# [B問題](https://atcoder.jp/contests/abc175/tasks/abc175_b)

三角形の三辺を全探索します。また、重複なく数え上げるために`range(n)`→`range(i+1,n)`→`range(j+1,n)`とする必要があります。

```python:B.py
n=int(input())
l=list(map(int,input().split()))
l.sort()
ans=0
for i in range(n):
    for j in range(i+1,n):
        for k in range(j+1,n):
            if l[i]+l[j]>l[k] and l[i]!=l[j] and l[j]!=l[k]:
                ans+=1
print(ans)
```

# [C問題](https://atcoder.jp/contests/abc175/tasks/abc175_c)

[この問題](https://atcoder.jp/contests/abc161/tasks/abc161_c)とほとんど同じです。

絶対値は初めのうちは**単調減少させる**ことができその操作が最適ですが、座標の絶対値が$d$以下になる(この時を$e$とする)とそこからは**$e$と$d-e$を往復する**のが最適となります。

よって、この往復は**残りの移動回数の偶奇**によって決まり、$e$に初めてたどり着いた時の残りの移動回数が偶数の時は$e$で奇数の時は$d-e$が答えとなります。

```python:C.py
x,k,d=map(int,input().split())
x=abs(x)
if x>k*d:
    print(x-k*d)
else:
    k-=x//d
    x-=(d*(x//d))
    ans=[x,abs(d-x)]
    print(ans[k%2])
```

# [D問題](https://atcoder.jp/contests/abc175/tasks/abc175_d)

コメントアウトしながら丁寧な実装を心がけたので、コーナーケースをスムーズに発見できました。

まず、コマを書いてある数字にしたがって移動させますが、あるマスに二回たどり着いたとすると**そのあとの挙動は同じなのでサイクルが現れそう**であることがわかります。また、順列であるという制約から**必ずサイクルが存在する**ことがわかります。

この時、それぞれのマスから先のマスを辿っていきますが、$K$が極めて大きいのでサイクルに単位でまとめて計算する必要があります。したがって、まずはサイクルを検出します。いくつか方法がありますが、自分は**すでに辿ったマスを記録する配列**と**辿った順番を記録する配列**を用意して実装を行いました。具体的には、始点としてそれぞれのマスを選んで、すでに辿ったマスが現れるまでマスを辿ることを繰り返すことを試行します。また、**サイクルに含まれない部分については先に計算**しておき、それでも$K$に満たない場合はサイクルの計算に移ります。

ここで、サイクルを回り始めた際に**一周もできない場合**が存在するので先に場合分けします。これ以後はサイクルを回るべきか否かを考えます。まず、サイクルの一周で得られるスコアを計算します。この際に、**サイクルのスコアが負の場合**はそもそもサイクルを回る必要がありません。しかし、サイクルを回る必要はなくてもその**サイクルの途中まで回ると最大値が更新できるパターン**があるので、そのようなパターンをチェックする必要があります。次に、**サイクルのスコアが非負の場合**はサイクルを回った方が良いので、回れるだけ回ります。そして、最後に余った部分はマスの移動を試行すれば良いと考えたのですが、**実はこの方法ではWAのパターンが生じます**。つまり、サイクルの合計スコアが非負の場合でも**最後のサイクルは回さない方が良いパターンがある**ということです。したがって、**最後のサイクルのみ分けて考える**ことでこのようなパターンを考慮できます。

以上の場合分けにより、全てのパターンを網羅できるため、これを実装して下記のコードになります。

```python:D.py
n,k=map(int,input().split())
#0-indexed
p=[i-1 for i in list(map(int,input().split()))]
c=list(map(int,input().split()))
#1回以上K回以下
ansF=-1000000000000000000
for i in range(n):
    #すでに辿ったもの
    check=[False]*n
    #辿った順番
    turns=[]
    #今
    now=i
    while check[now]==False:
        check[now]=True
        turns.append(now)
        now=p[now]
    #答え(今の)
    ans=0
    #kのうち何回分
    #更新を毎回行う
    spare=turns.index(now)
    if spare>=k:
        for j in range(k):
            ans+=c[turns[j]]
            ansF=max(ans,ansF)
        continue
    for j in range(spare):
        ans+=c[turns[j]]
        ansF=max(ans,ansF)
    turns=turns[spare:]
    spare=k-spare
    #サイクルの長さ
    l=len(turns)
    #サイクルの和(あまりも)
    #サイクルどこで止めるか
    x=0
    for j in range(l):
        x+=c[turns[j]]
    #そもそもサイクル回らない場合
    if spare<=l:
        for j in range(spare):
            ans+=c[turns[j]]
            ansF=max(ans,ansF)
        continue
    #サイクル回した方が良い場合もある
    #最後のサイクルは回さない方が良いパターンも
    if x>=0:
        ans+=(x*(spare//l-1))
        ansF=max(ans,ansF)
        for j in range(l):
            ans+=c[turns[j]]
            ansF=max(ans,ansF)
        for j in range(spare%l):
            ans+=c[turns[j]]
            ansF=max(ans,ansF)
    #サイクル回さない方がいい場合
    else:
        for j in range(l):
            ans+=c[turns[j]]
            ansF=max(ans,ansF)
print(ansF)
```


# [E問題](https://atcoder.jp/contests/abc175/tasks/abc175_e)

**グリッド上での探索問題**かつ**その遷移は明らか**なので、DPを方針として立てました。具体的には以下のように状態を保存します。

$dp[i][j][k]:=$(マス($i,j$)にたどり着いた時にその行で選んだアイテムが$k$個の時のアイテムの価値の総和の最大値)

ここで、下への遷移と右への遷移がありますが、少しややこしいので**場合分け**が必要です。また、$dp[i][j][l]$の状態からの遷移を考えます。さらに、以下ではマス($i,j$)のアイテムの価値を$item[i][j]$としています。

(1)下への遷移の場合
マス($i,j$)からマス($i+1,j$)へ遷移します。また、$i+1$行目ではマス($i+1,j$)上にあるアイテムしか選びようがないので、**$l$の値に遷移は依存しません**。  
[1]マス($i+1,j$)上にあるアイテムを選ばない時  
$dp[i+1][j][0]=max(dp[i+1][j][0],dp[i][j][l])$  
[2]マス($i+1,j$)上にあるアイテムを選ぶ時  
$dp[i+1][j][1]=max(dp[i+1][j][1],dp[i][j][l]+item[i+1][j])$  

(2)右への遷移の場合  
マス($i,j$)からマス($i,j+1$)へ遷移します。また、$i$行目では$l$個のアイテムを選んでいるので、**$l=3$の時はそれ以上選ぶことができません**。  
[1]マス($i,j+1$)上にあるアイテムを選ばない時  
$dp[i][j+1][l]=max(dp[i][j+1][l],dp[i][j][l])$  
[2]マス($i,j+1$)上にあるアイテムを選ぶ時(ただし、$l<3$)
$dp[i][j+1][1]=max(dp[i][j+1][1],dp[i][j][l]+item[i][j+1])$  

以上の遷移がわかれば、DPを0初期化することと、マス($0,0$)にアイテムが存在する場合はdp[0][0][1]=$item[0][0]$で初期化することさえ忘れなければ三次元のDPを行うだけです。  

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
    ll r,c,k;cin>>r>>c>>k;
    vector<vector<ll>> item(r,vector<ll>(c,0));
    REP(i,k){
        ll x,y,z;cin>>x>>y>>z;
        item[x-1][y-1]=z;
    }
    vector<vector<vector<ll>>> dp(r,vector<vector<ll>>(c,vector<ll>(4,0)));
    if(item[0][0]>0)dp[0][0][1]=item[0][0];
    REP(i,r){
        REP(j,c){
            REP(l,4){
                if(l<3){
                    if(i!=r-1){
                        dp[i+1][j][0]=max(dp[i+1][j][0],dp[i][j][l]);
                        if(item[i+1][j]>0)dp[i+1][j][1]=max(dp[i+1][j][0],dp[i][j][l])+item[i+1][j];
                    }
                    if(j!=c-1){
                        dp[i][j+1][l]=max(dp[i][j+1][l],dp[i][j][l]);
                        if(item[i][j+1]>0)dp[i][j+1][l+1]=max(dp[i][j+1][l+1],dp[i][j][l]+item[i][j+1]);
                    }
                }else{
                    if(i!=r-1){
                        dp[i+1][j][0]=max(dp[i+1][j][0],dp[i][j][l]);
                        if(item[i+1][j]>0)dp[i+1][j][1]=max(dp[i+1][j][0],dp[i][j][l])+item[i+1][j];
                    }
                    if(j!=c-1){
                        dp[i][j+1][l]=max(dp[i][j+1][l],dp[i][j][l]);
                    }
                }
            }
        }
    }
    cout<<*max_element(ALL(dp[r-1][c-1]))<<endl;
}
```
