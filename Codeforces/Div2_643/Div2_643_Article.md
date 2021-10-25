# 感想

約一年前にAtCoderを始めたのですが、コンテストが多いと聞いていたコドフォにも出ることにしました。

# [A問題](https://codeforces.com/contest/1355/problem/A)

10進数に直した時の桁の中に0が一つでも混じっている場合は$minDigit(a\_n)=0$であり、$k \geqq n$で$a\_k$の値は変化しません。また、$minDigit(a\_n)=0$にはそのうちなるという予測はつく($\because$任意のnで$minDigit(a\_n)\neq0$となるパターンはなさそう)ので、$minDigit(a\_n)=0$になったらその時の$a\_k$を出力してbreakしました。

```python:A.py
t=int(input())
for i in range(t):
    a,k=map(int,input().split())
    for i in range(k-1):
        ax=str(a)
        l,r=int(min(ax)),int(max(ax))
        if l==0:
            print(a)
            break
        else:
            a+=(l*r)
    else:
        print(a)
```

# [B問題](https://codeforces.com/contest/1355/problem/B)

全員をグループに含めなければならないと勘違いをしていました…。

全員をグループに含める必要がないので、「グループの必要人数」が少ない人から順に選んでいきます。この際に、グループとして選んだ人の人数はそれぞれの人の「グループの必要人数」以上である必要があるので、この「グループの必要人数」以上でない場合は必要な人数だけグループに人を追加します。そしてグループが条件を満たす場合は題意のグループ数に追加することができます。

Pythonではpretest通ったので油断しましたがTLEになりました。こういうギリギリの計算量の場合はC++で提出した方が確実かもしれません。

```python:B.py
t=int(input())
for i in range(t):
    n=int(input())
    e=sorted(list(map(int,input().split())))
    ans,now=0,0
    while True:
        nowx=now+e[now]
        while nowx<n:
            if nowx-now<e[nowx-1]:
                nowx=now+e[nowx-1]
            else:
                break
        
        if nowx>=n:
            if nowx==n:
                if nowx-now>=e[nowx-1]:
                    ans+=1
            break
        else:
            ans+=1
            now=nowx
    print(ans)
```

```c++:answerB.cc
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

signed main(){
    ll t;cin >> t;
    vector<ll> answers(t);
    REP(i,t){
        ll n;cin >> n;
        vector<ll> e(n);REP(j,n)cin >> e[j];
        sort(ALL(e));
        ll ans=0;ll now=0;
        while(true){
            ll nowx=now+e[now];
            while(nowx<n){
                if(nowx-now<e[nowx-1]){
                    nowx=now+e[nowx-1];
                }else{
                    break;
                }
            }

            if(nowx>=n){
                if(nowx==n){
                    if(nowx-now>=e[nowx-1]){
                        ans++;
                    }
                }
                break;
            }else{
                ans++;
                now=nowx;
            }
        }
        answers[i]=ans;
    }
    REP(i,t){
        cout << answers[i] << endl;
    }
}
```

# [C問題](https://codeforces.com/contest/1355/problem/C)

この問題はちゃんと考察をして通せたので良かったと思います。
まず、三角形として成り立つには、(最小の二辺の和)$\>$(最大の一辺)が成り立つ必要があります。

また、三角形の三辺$x,y,z$の間には$A \leqq x \leqq B \leqq y \leqq C \leqq z \leqq D$が成り立つので、$x+y\>z$であれば良いです。さらに、$A+B \leqq x+y \leqq B+C$が成り立ち、このx+yの候補は$10^5$程度のオーダーであることは明らかなので、x+yを決めた元でのzの候補を$O(1)$で求めれば良いと考えました。

ここまで考えれば、zのうちx+yよりも小さいものがいくつあるか求めれば良いですが、ここに一つ落とし穴があります。x+yの値の候補は$10\^5$程度のオーダーですが、x+yの値に対してx,yの組が複数あるという点です。この複数あるものの処理は下図を書くことでうまく考えることができます。

![](/Codeforces/Div2_643/Div2_643_1.png)

よって、$x+y=k$のときの(x,y)の組の候補がわかるので、この時のzの候補を考えると下の画像のようになります。

![](/Codeforces/Div2_643/Div2_643_2.png)

よって、$x+y=k$の時の(x,y)の組の数とzの候補の数の積を足し合わせたものが答えになります。

```python:C.py
a,b,c,d=map(int,input().split())
xy=dict()
for i in range(a+b,b+c+1):
    f=min(i-a-b+1,b+c-i+1)
    f=min(f,b-a+1,c-b+1)
    xy[i]=f
ans=0
for i in range(a+b,b+c+1):
    if i>d:
        ans+=(xy[i]*(d-c+1))
    elif i<=c:
        continue
    else:
        ans+=(xy[i]*(i-c))
print(ans)
```

# [D問題](https://codeforces.com/contest/1355/problem/D)

しかし、Petyaが**配列を都合よく作ることができる**ことから、Petyaが勝つために**都合の良い配列を作りたい**という発想に至ります。

また、部分配列についてその合計としてk,S-kのいずれかになるうなものが存在しなければ良いので、その対称性にも注目すれば、**合計が0に近い部分配列と合計がSに近い部分配列のみが存在すような配列**を想定すれば良いと考えました。

そのような配列は、配列の全要素が1以上S-1以下であり全要素の合計がSであることにも注目すれば以下の図のようになります。

![](/Codeforces/Div2_643/Div2_643_3.png)

上図のような配列を想定すれば、部分配列の合計としてありうるのは$1$〜$n-1,S-(n-1)$〜$S$であり、**対称性も考慮できている**ので都合が良いのではないかと考えました。また、この都合の良い配列を作る(Petyaが勝つ)には、$n-1 \< k \< S-(n-1)$を満たすようなkが存在すれば良く、このようなkが存在するための条件は$n-1 \< S-(n-1)$であり、これを実装すると以下のようになります($\because$kがこれを満たす時、S-kもこれを満たすのも明らか)。

```python:answerD.py
n,s=map(int,input().split())
if n<s-(n-1):
    print("YES")
    print(" ".join(map(str,[1 if i!=n-1 else s-(n-1) for i in range(n)])))
    print(n)
else:
    print("NO")
```
