# 感想

解けるはずの問題でコーナーケースを忘れたり実装でバグらせたことで焦ってしまいました…。

# [A問題](https://yukicoder.me/problems/no/1198)

$$A^2 -B^2 =N \leftrightarrow (A-B)(A+B)=N$$

上記のように式変形でき、**$A-B$と$A+B$の偶奇が一致する**ことから、$(A-B,A+B)=$(偶数,偶数),(奇数,奇数)となれば良いです。また、この時、前者は4の倍数,後者は奇数であることが必要です。また、$A-B\<A+B$なので、$N=4,1$の場合はそれぞれ成り立たないことに気をつければ以下のようになります。

```python:A.py
n=int(input())
if n%2==1:
    if n==1:
        print(-1)
    else:
        print(1)
elif n%4==0:
    if n==4:
        print(-1)
    else:
        print(1)
else:
    print(-1)
```

# [B問題](https://yukicoder.me/problems/no/1199)

この問題では**順番に**お菓子を見て取っていけば良いことがわかります。ただし、**貪欲法で行うことができない**ので、以下のようなDPをおきます。

$dp[i][j]:=$($i$個目のお菓子まで取った時にお菓子を奇数個(or偶数個)選んでいる際の最大の幸せ度)

また、$j=0$の時は奇数個で$j=1$の時は奇数個とします。

この時、遷移は難しくなく、以下のようになります。

(1)$dp[i+1][0]$の更新

$a[i+1]$を選ばない時は$dp[i][0]$
$a[i+1]$を選ぶ時は$dp[i][1]+a[i+1]$

(2)$dp[i+1][0]$の更新

$a[i+1]$を選ばない時は$dp[i][1]$
$a[i+1]$を選ぶ時は$dp[i][0]-a[i+1]$

```python:B.py
n,m=map(int,input().split())
a=[sum(list(map(int,input().split()))) for i in range(n)]
dp=[[0,0] for i in range(n)]
dp[0][0]=a[0]
for i in range(n-1):
    #食べるか食べないか
    dp[i+1][0]=max(dp[i][0],dp[i][1]+a[i+1])
    dp[i+1][1]=max(dp[i][1],dp[i][0]-a[i+1])
print(max(dp[n-1]))
```

# [C問題](https://yukicoder.me/problems/no/1200)

まず、問題を数式に起こせば、$A \times B=X-C,A \times C=Y-B$となります。これを変形すると$(A-1)(B-C)=X-Y,(A+1)(B+C)=X+Y$となります。また、実装を楽にするために$X \geqq Y$が成り立つとし、成り立たない場合はswapします。

ここで、$(A-1)(B-C)=X-Y$に注目すると、$A-1$は$X-Y$の約数なので、**$O(\sqrt{X})$で$A$の候補を列挙**することができます。また、$A$を固定した時、$B-C=\frac{X-Y}{A-1},B+C=\frac{X+Y}{A+1}$が成り立ちます。したがって、$v=\frac{X-Y}{A-1},w=\frac{X+Y}{A+1}$とおけば、$B=\frac{v+w}{2},C=\frac{w-v}{2}$が成り立ちます。この時、$B,C$は整数にならない可能性がありますが、整数型へと型変換して$B,C$を求めておきます。この$B,C$について$(A-1)(B-C)=X-Y,(A+1)(B+C)=X+Y$が成り立つかどうかを確かめ、その個数を数えていけば良いです。

また、上記では$X \neq Y$を仮定して考察を行っているので、**$X=Y$の場合の考察を別で行う**必要があります。$X=Y$の時、$(A-1)(B-C)=0 \leftrightarrow A=1 \ or \ B=C$より、以下のように場合分けを行えば良いです。

(1)$A=1$の時
$(A+1)(B+C)=X+Y \leftrightarrow B+C=X$より、$(B,C)=(1,X-1),(2,X-2),…,(X-1,1)$の$X-1$通りがあります。

(2)$B=C$の時
$(A+1)(B+C)=X+Y \leftrightarrow (A+1)B=X$より、$A+1$は$X$の約数で先ほどの議論が用いることができます。
ただし、**$\frac{X}{A+1}$が1または2になる時は注意深く場合分けを行う必要**がありますが、**コーナーケース**として分けて処理を行いました。

```c++:C.cc
//デバッグ用オプション：-fsanitize=undefined,address

//コンパイラ最適化
#pragma GCC optimize("Ofast")

//インクルードなど
#include<bits/stdc++.h>
using namespace std;
typedef int ll;

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


ll check_divisors(ll x,ll y){
    ll ret=0;
    ll n=x-y;
    vector<ll> divisors;//約数を格納する配列
    FOR(i,1,sqrt(n)){
        if(n%i==0){
            divisors.PB(i);
            if(i!=n/i){
                divisors.PB(n/i);
            }
        }
    }
    //a-1の候補
    FORA(i,divisors){
        ll a,b,c;a=i+1;
        ll v,w;v=n/(a-1);w=(x+y)/(a+1);
        b=(v+w)/2;c=(w-v)/2;
        if(b>0 and c>0 and (a-1)*(b-c)==(x-y) and (a+1)*(b+c)==(x+y)){
            ret++;        
        }
    }
    return ret;
}

ll check_divisors2(ll x,ll y){
    ll ret=x-1;
    FOR(i,3,sqrt(x)){
        if(x%i==0){
            if(i!=ll(x/i)){
                ret+=2;
            }else{
                ret++;
            }
        }
    }
    if(x%2==0){
        if(2!=ll(x/2) and x!=2){
            ret++;
        }
    }
    if(x!=1 and x!=2)ret++;
    return ret;
}

signed main(){
    //入力の高速化用のコード
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    ll s;cin>>s;
    REP(i,s){
        ll x,y;cin>>x>>y;
        if(x<y)swap(x,y);
        if(x==y){
            cout<<check_divisors2(x,y)<<endl;
        }else{
            cout<<check_divisors(x,y)<<endl;
        }
    }
}
```
