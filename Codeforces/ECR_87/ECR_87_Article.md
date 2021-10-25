# 感想

Codeforcesの通称エデュフォと呼ばれるコンテストに参加しました。解けない原因を精査しながら復習します。

# [A問題](https://codeforces.com/contest/1354/problem/A)

問題文の解読が難しくて解くのにだいぶ時間がかかりました。$a,b,c,d$の四つの値が与えられますが、この問題は下図のように考察できます。

![](/Codeforces/ECR_87/ECR_87_1.png)

寝る時間が合計で$a$以上になればよく、初めに$b$だけ寝るとアラームが鳴り、それ以降は$c$だけアラームが鳴ってそのうち$d$の時間は**寝ようとする**ので、実際に寝ることができるのは$c-d$だけでこれが無限に繰り返されます。

まず、$b \geqq a$の時は起きるのは$b$だけ時間が経った時です。この時$a$ではなく$b$であることに注意が必要です。そして、$c\leqq d$の時は初めの$b$以外で寝ることができないので、a>bの時は-1を出力する必要があります。

以上を実装して以下のようになります。

```python:A.py
t=int(input())
for i in range(t):
    a,b,c,d=map(int,input().split())
    if d>=c:
        if b>=a:
            print(b)
        else:
            print(-1)
    else:
        if b>=a:
            print(b)
        else:
            a-=b
            num=-((-a)//(c-d))
            print(b+num*c)
```

# [B問題](https://codeforces.com/contest/1354/problem/B)

1,2,3の全てを含むような部分文字列のうち最も短いものの長さを出力します。ただし、そのような部分文字列が存在しない場合は0を出力する必要があります。

この問題は区間が**連続的に移動して数えることができる**ので、**尺取り法**を使えば求めることができます。尺取法の実装では、「**右端を右へと順に進める**→**左端を右へと順に進める**」の順でクエリ処理を行えば良いです。また、この際に**右端がその調べたい配列の範囲を超えない**ことおよび**左端が右端を超えない**ことに注意が必要です。

```python:B.py
t=int(input())
for i in range(t):
    s=input()
    le=len(s)
    l,r=0,0
    d=[0,0,0]
    d[int(s[0])-1]=1
    ans=0
    while True:
        #rの決定
        while not all(d) and r!=le-1:
            r+=1
            d[int(s[r])-1]+=1
            if r==le-1:
                break
        #lの決定
        while l!=le-1:
            if all(d):
                if ans==0:
                    ans=r-l+1
                else:
                    ans=min(ans,r-l+1)
                d[int(s[l])-1]-=1
                l+=1
            else:
                break
        if r==le-1 or l==le-1:
            break
    print(ans)
```

# [C1問題](https://codeforces.com/contest/1354/problem/C1)

多角形の中心から一番離れているのはその多角形の頂点であり、**対称性が良い配置**を考える必要があるため、。

偶数の場合は、以下のような図形の配置にすると最小の正方形で多角形を囲むことができます。

![](/Codeforces/ECR_87/ECR_87_2.png)

多角形の中心と多角形の頂点を結んで三角形を作って三角比(sin,cos,tan)を用いて計算すれば$\frac{1}{\tan{\frac{90}{n}}}$求めることができます。

```python:C1.py
import math
t=int(input())
for i in range(t):
    n=int(input())
    print(1/math.tan(math.radians(90/n)))
```

# [C2問題](https://codeforces.com/contest/1354/problem/C2)

奇数の場合は一番左側の図を汚く書いて上下の辺が正方形に接すると勘違いしていました。

![](/Codeforces/ECR_87/ECR_87_3.png)

こちらも先ほどと同様に**対称性を考えて描くと上記の3パターンしかない**ですが、左端と右端のパターンは**正方形を回すと同じパターン**とみなせます。したがって、左端と真ん中のパターンのどちらが面積が大きいかを考えますがこれは明らかに真ん中のパターンになります(計算すればわかります。)。

この真ん中のパターンは右端と左端のパターンの間であり、左端から右端で$\frac{180}{n}$だけ動かすので$\frac{90}{n}$だけ左端のパターンから動かすと考えれば、先ほどと同様に幾何的な考察を行うことで$\frac{\cos{\frac{45}{n}}}{\sin{\frac{90}{n}}}$と求めることができます。

```python:C2.py
import math
t=int(input())
for i in range(t):
    n=int(input())
    print(math.cos(math.radians(45/n))/math.sin(math.radians(90/n)))
```

# [D問題](https://codeforces.com/contest/1354/problem/D)

まず、挿入する位置や削除する位置は**ソートされているもとで決まる**ので、この多重集合をソートしてそのソートを保ったまま操作することを考えます。

ここで、ソートしたもとで**重複する数字**はまとめて考えても同じなので、それぞれの数字がいくつあるかを保存する配列x($1 \leqq$(xの要素)$ \leqq n$)を用意します。

また、挿入はO(1)で実行する($\because$配列の要素を+1するだけなので)ことができますが、削除の際に-1する配列の要素を決める際にO(n)かかります。

ここで、**それぞれの数字がいくつあるかを保存する配列x**をBITで管理すると、挿入はBITのadd関数を用いることでO($\log{n}$)になりますが、削除はBITのlower_bound関数で削除したい要素が何番目かを取得することができるため、O($(\log{n})^2$)で可能になります。よって、O($n(\log{n})^2$)で実装できます。

```c++:D.cc
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
#define MAX(x) *max_element(ALL(x)) //最大値を求める
#define MIN(x) *min_element(ALL(x)) //最小値を求める
//定数
#define INF 1000000000000 //10^12:極めて大きい値,∞
#define MOD 1000000007 //10^9+7:合同式の法
#define MAXR 100000 //10^5:配列の最大のrange(素数列挙などで使用)
//略記
#define PB push_back //vectorヘの挿入
#define MP make_pair //pairのコンストラクタ
#define F first //pairの一つ目の要素
#define S second //pairの二つ目の要素

//1-indexed
class BIT {
public:
    ll n; //データの長さ
    vector<ll> bit; //データの格納先
    BIT(ll n):n(n),bit(n+1,0){} //コンストラクタ

    //k & -kはLSB

    //bit_iにxをO(log(n))で加算する
    void add(ll i,ll x){
        if(i==0) return;
        for(ll k=i;k<=n;k+=(k & -k)){
            bit[k]+=x;
        }
    }

    //bit_1 + bit_2 + …  + bit_n をO(log(n))で求める
    ll sum(ll i){
        ll s=0;
        if(i==0) return s;
        for(ll k=i;k>0;k-=(k & -k)){
            s+=bit[k];
        }
        return s;
    }

    //a_1 + a_2 + … + a_i >= x となるような最小のiを求める(a_k >= 0)
    //xが0以下の場合は該当するものなし→0を返す
    ll lower_bound(ll x){
        if(x<=0){
            return 0;
        }else{
            ll i=0;ll r=1;
            //最大としてありうる区間の長さを取得する
            //n以下の最小の二乗のべき(BITで管理する数列の区間で最大のもの)を求める
            while(r<n) r=r<<1;
            //区間の長さは調べるごとに半分になる
            for(int len=r;len>0;len=len>>1) {
                //その区間を採用する場合
                if(i+len<n && bit[i+len]<x){
                    x-=bit[i+len];
                    i+=len;
                }
            }
            return i+1;
        }
    }
};



signed main(){
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    ll n,q;cin >> n >> q;
    BIT a(n);REP(i,n){ll a_sub;cin >> a_sub;a.add(a_sub,1);}
    vector<ll> k(q);REP(i,q) cin >> k[i];
    REP(i,q){
        if(k[i]>0){
            a.add(k[i],1);
        }else{
            //必ず数列内に含まれるものが指定される
            a.add(a.lower_bound(-k[i]),-1);
        }
    }
    vector<ll> answers(n);
    REP(i,n){answers[i]=a.sum(i+1);}

    if(answers[n-1]<=0){
        cout << 0 << endl;return 0;
    }
    REP(i,n){
        if(i>0){
            if(answers[i]-answers[i-1]>0){
                cout << i+1 << endl;return 0;
            }
        }else{
            if(answers[i]>0){
                cout << i+1 << endl;return 0;
            }
        }
    }
}
```
