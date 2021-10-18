# 感想

（不備があったので、省略します。）


# [A問題](https://atcoder.jp/contests/abc167/tasks/abc167_a)

sがtの末尾を除いた文字列と等しいかの判定をします。

```python:A.py
s=input()
t=input()
print("Yes" if s==t[:-1] else "No")
```

# [B問題](https://atcoder.jp/contests/abc167/tasks/abc167_b)

大きい数のカードから順番にとっていくことを考えます。

```python:B.py
a,b,c,k=map(int,input().split())
if a>=k:
    print(k)
elif a+b>=k:
    print(a)
else:
    print(a-(k-a-b))
```

# [C問題](https://atcoder.jp/contests/abc167/tasks/abc167_c)

まず、どの参考書を選ぶかの計算量は$O(n \times 2^n)$です。ここで、ある参考書を選んだ時にそれぞれのアルゴリズムの理解度がどれくらい上昇するかは$O(m)$で求められます。そして、どの参考書を選ぶか決めた後に全てのアルゴリズムの理解度が$X$以上かの判定を$O(m)$で行えば良いので、$O(2^n \times (n \times m + m))=O(2^n \times n \times m)$で求められます。以上より、制限時間に間に合うようなプログラムを書くことができます。

また、全ての参考書を買っても全てのアルゴリズムの理解度をX以上にできない場合があるので、infによる初期化をしました。

```python:C.py
n,m,x=map(int,input().split())
ca=[list(map(int,input().split())) for i in range(n)]
ans=100000000000000000
for i in range(2**n):
    check=[0]*m
    ans_=0
    for j in range(n):
        if ((i>>j)&1):
            ans_+=ca[j][0]#(✳︎)
            for k in range(m):
                check[k]+=ca[j][k+1]
    if all([l>=x for l in check]):
        ans=min(ans,ans_)
if ans==100000000000000000:
    print(-1)
else:
    print(ans)
```


# [D問題](https://atcoder.jp/contests/abc167/tasks/abc167_d)

一番簡単な方法はk回の移動を全て実行することですが、kの制約から不可能です。ここで、n回の移動を行うと、**二回以上訪れる町iが存在**します(この証明は省略します。)。さらに、二回以上訪れる町iから後に訪れた町は町iから必ず辿ることができるので、町i以降で二回目に町iを訪れるまでの町で**ループが形成**されます(入力例1であれば1→3→4のループが形成されます。)。

ここで、すでに訪れた町を保存することで、二度訪れる町があった時にループを発見することができます。また、訪れる町があるかどうかの判定は**O(1)で行いたいので集合として保存**します。ただ、**訪れる町の順番も保存**しなければならないので、配列で訪れる町の順番も保存しておきます。

以上でループが判定できれば、ループに入る前と後での場合分けおよびループ内でのどこがKに対応するか(mod)によって判定します。この判定等は下記のコードのコメントアウトを参照してください。

```python:answerD.py
#訪れる町の順番を配列で保存する
visited=[0]
#訪れた町を集合で保存する
visited_set=set()
visited_set.add(0)
n,k=map(int,input().split())
a=list(map(int,input().split()))
#next1が今訪れている町、next2が次に訪れる町
#next2がすでに訪れている町ならwhileループを抜ける
next1=0
next2=-1
while True:
    #次の町へ
    next2=a[next1]-1
    #今訪れている町にする
    next1=next2
    if next2 in visited_set:
        #すでに訪れた町なので
        break
    else:
        #順番通りに配列へ挿入
        visited.append(next2)
        #訪れたかどうかを記録する集合へ挿入
        visited_set.add(next2)

#訪れた町の総数よりkが小さい場合はそのままk番目の町を出力
if k<len(visited):
    print(visited[k]+1)
else:
    #ループの長さを求める(ここ間違えがち、図を書いた)
    loop_len=(len(visited)-visited.index(next2))
    #ループにたどり着くまで
    k-=(len(visited)-loop_len)
    #ループの長さの剰余を考えれば、ループ内でどこにいるかは簡単にわかる
    print(visited[k%loop_len+visited.index(next2)]+1)
```

# [E問題](https://atcoder.jp/contests/abc167/tasks/abc167_e)

n個のブロックが横に並んでおり、m色での塗り方はO(1)で求められそうなので、隣り合うブロックの組が$l$個あると仮定して色を塗ることにしました。

まず、一番左側からブロックの色を決めていくと、隣合うブロックの色が異ならない場合は$m \times (m-1) \times (m-1) \times (m-1) \times …$となります。それに対し、二番目のブロックが同じだとすれば、$m \times (m-1) \times 1 \times (m-1)  \times …$となることがわかります。したがって、この隣合うブロックで色が同じ部分は無視しても良いと考えられるため、**そのブロック同士を繋げて一個のブロックとみなす**ことにしました。

したがって、隣合うブロックの組が$l$個ある場合、**全て色が異なるような長さ$n-l$のブロック列を考えれば良い**ことになります。また、どのブロックの色が同じになるかで$_{n-1}C _{l}$通りあるので、隣り合うブロックの組が$l$個ある時の色の塗り方は$m \times (m-1)^{n-l-1} \times _{n-1}C _{l}$通りとなり、これを$l=0~k$で計算していけば良いです。ただし、組み合わせ計算は逆元を用いたものを利用することがあり、本問題ではmod(=998244353)の逆元を用います。

```c++:answerE.cc
//参考：http://ehafib.hatenablog.com/entry/2015/12/23/164517
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
#define MOD 998244353 //10^9+7:合同式の法
#define MAXR 300000 //10^5:配列の最大のrange(素数列挙などで使用)
//略記
#define PB push_back //vectorヘの挿入
#define MP make_pair //pairのコンストラクタ
#define F first //pairの一つ目の要素
#define S second //pairの二つ目の要素

ll fac[MAXR+1],finv[MAXR+1],inv[MAXR+1];

//テーブルの作成
void COMinit() {
    fac[0]=fac[1]=1;
    finv[0]=finv[1]=1;
    inv[1]=1;
    FOR(i,2,MAXR){
        fac[i]=fac[i-1]*i%MOD;
        inv[i]=MOD-inv[MOD%i]*(MOD/i)%MOD;
        finv[i]=finv[i-1]*inv[i]%MOD;
    }
}

// 二項係数の計算
ll COM(ll n,ll k){
    if(n<k)return 0;
    if(n<0 || k<0)return 0;
    return fac[n]*(finv[k]*finv[n-k]%MOD)%MOD;
}

ll modpow(ll a,ll n,ll mod){
    ll res = 1;
    while(n > 0){
        if(n & 1) res = res * a % mod;
        a = a * a % mod;
        n >>= 1;
    }
    return res;
}

signed main(){
    COMinit();
    ll n,m,k;cin >> n >> m >> k;
    ll ans=0;
    REP(l,k+1){
        ll ansx=1;
        ansx*=COM(n-1,l);
        ansx*=modpow(m-1,n-l-1,MOD);
        ansx%=MOD;
        ansx*=modpow(m,1,MOD);
        ansx%=MOD;
        ans+=ansx;
        ans%=MOD;
    }
    cout << ans << endl;
}
```


# [F問題](https://atcoder.jp/contests/abc167/tasks/abc167_f)

まず、括弧列が存在する時、(を+1,)を-1として前からその累積和を考えていった時に全要素について0以上かつ一番最後まで足した時の合計が0であれば必要十分です。また、この問題では、**文字列の中でも**この条件が成り立つ必要があり、難しいです。

ここで、それぞれの文字列の累積和が大きいものから順に左側から連結することを考えましたが、それぞれの文字列内での累積和が常に正であるようにしなければならないです。例えば、ある文字列の累積和の最小値が0以上であるときはどの順序で繋げても全要素について累積和は常に0以上になります。したがって、問題になるのは**文字列の最小の累積和が0より小さいもの**についてです。

この元で、**ある文字列の累積和が0以上**の場合に注目すると、その文字列を付け足しても累積和を小さくしません。また、付け加えた際の最小値が0以上であれば続けて繋げていくことができます。つまり、**文字列の累積和の最小値**が大きいものから順に貪欲に連結していくのが最適の繋げ方になります。一方、**ある文字列の累積和が0より小さい場合**、単純に貪欲に決めていくのは難しいです。この時は、発想の転換が必要です。0以上の場合は左側から順に繋げていましたが**、逆に右側から順に繋げていく**と考えることで、同じように貪欲に考えることができます。

最後に、この問題で重要なことをまとめます。

1. 括弧列は(を+1、)を-1と見て計算する。
2. 括弧列が成り立つことと、その全要素について累積和が0以上かつ最終値が0、は同値。
3. 括弧のみの文字列(括弧列を含む)内で累積和を考えた時の最小値および最終値が重要。
4. 累積和の最小値は0以上であるのでそれを下回らないように考える。累積和を増やす方向であれば貪欲に考えられるが、累積和を減らす方向の場合は括弧のみの文字列を逆転させて考えると良い。


```python:answerF.py
import sys
n=int(input())
s=[input() for i in range(n)]
t=[2*(i.count("("))-len(i) for i in s]
if sum(t)!=0:
    print("No")
    sys.exit()
st=[[t[i]] for i in range(n)]

for i in range(n):
    now,mi=0,0
    for j in s[i]:
        if j=="(":
            now+=1
        else:
            now-=1
        mi=min(mi,now)
    st[i].append(mi)
#st.sort(reverse=True,key=lambda z:z[1])
u,v,w=list(filter(lambda x:x[1]>=0,st)),list(filter(lambda x:x[1]<0 and x[0]>=0,st)),list(filter(lambda x:x[1]<0 and x[0]<0,st))
v.sort(reverse=True)
v.sort(reverse=True,key=lambda z:z[1])
w.sort(key=lambda z:z[0]-z[1],reverse=True)
lu=len(u)
lv=len(v)
now2=0
for i in range(n):
    if i<lu:
        now2+=u[i][0]
    elif i<lu+lv:
        if now2+v[i-lu][1]<0:
            print("No")
            break
        now2+=v[i-lu][0]
    else:
        if now2+w[i-lu-lv][1]<0:
            print("No")
            break
        now2+=w[i-lu-lv][0]
else:
    print("Yes")
```
