# 感想

日曜日にあった[ABC144](https://atcoder.jp/contests/abc144)の復習をします。
今までで一番ひどかったので学びがある気がします。<br>
初めてレートが下がったので悔しいです…。

# A,B問題

復習する内容もないのでコードだけ貼ります。

```python:answerA.py
a,b=map(int,input().split())
if(1<=a<=9 and 1<=b<=9):
    print(a*b)
else:
    print(-1)
```

```python:answerB.py
N=int(input())
f=0
for i in range(1,10):
    for j in range(1,10):
        if N==i*j:
            f=1
            break
    if f==1:
        break
if f==0:
    print("No")
else:
    print("Yes")
```

# C問題

1からnまで調べようとすると、$10^12$なので間に合いません。<br>
ここで、1~nまで調べると(n//i-1)+(i-1)について同じ値を二回数えてしまうため、1から$\sqrt{n}$までのみを調べればその全ての値を全てチェックすることができます。

```python:answerC.py
import math

n=int(input())
n2=int(math.sqrt(n))

minimum=n-1
for i in range(1,n2+1):
    if(n%i==0):
        minimum=min(minimum,(n//i-1)+(i-1))

print(minimum)
```

# D問題

下図のように考察すると、$\theta$は逆三角関数を使って求めれば良いだけです。

このような図形の問題が出るとは思っておらず、焦ってしまいました…。

![](/AtCoder/ABC_144/ABC_144_1.png)

```python:answerD.py
import math

a,b,x=map(int,input().split())

if 2*x<=a*a*b:
    p1=a*b*b
    p2=2*x
else:
    p1=2*(a*a*b-x)
    p2=a*a*a
    
sit=math.atan(p1/p2)
print(math.degrees(sit))

```

# E問題

上側がPythonのコードで、下側がC++のコードです。いずれの場合も一部のケースは通りましたが、ほとんどのテストケースはTLEになりました。<br>
原因は明らかで、$x$を答えとなる値として計算量が$O(N\times X)$になっているためです。<br>
この計算量を減らすためには二分探索を使えば良いようです。この問題では下限と上限がわかっているので、二分探索は容易に書けそうです。

```python:answerE.py
import math

n,k=map(int,input().split())
a=sorted([int(i) for i in input().split()],reverse=True)
f=sorted([int(i) for i in input().split()])
c=[[a[i],f[i]] for i in range(n)]
c.sort(key=lambda x:x[0]*x[1],reverse=True)

if n==1:
    print(max(0,(c[0][0]-k)*c[0][1]))
else:
    while k!=0 and c[0][0]!=0:
        x=c[0][0]*c[0][1]-c[1][0]*c[1][1]
        if c[0][0]*c[0][1]-c[1][0]*c[1][1]<c[0][1]:
            k-=1
            c[0][0]-=1
        else:
            l=math.ceil(x/c[0][1])
            m=min(l,c[0][0])
            k-=m
            c[0][0]-=m
        for i in range(n-1):
            if c[i][0]*c[i][1]<c[i+1][0]*c[i+1][1]:
                c[i],c[i+1]=c[i+1],c[i]
            else:
                break

    print(c[0][0]*c[0][1])
```

```c++:answerE.cc
#include<iostream>
#include<vector>
#include<utility>
#include<algorithm>
#include<cmath>
using namespace std;

typedef vector<pair<long long,long long> > food;

bool comp(const pair<long long,long long> &a,const pair<long long,long long> &b){
  return (a.first)*(a.second) > (b.first)*(b.second);
}

long long max(long long a,long long b){
  if(a>b){return a;}
  else{return b;}
}

long long min(long long a,long long b){
  if(a<b){return a;}
  else{return b;}
}

int main(){
  long long n,k;
  cin >> n >> k;
  vector<long long> v1(n,0);for(int i=0;i<n;i++){cin >> v1[i];}
  vector<long long> v2(n,0);for(int i=0;i<n;i++){cin >> v2[i];}
  food c(n,make_pair(0,0));
  sort(v1.begin(),v1.end());sort(v2.begin(),v2.end());
  reverse(v1.begin(),v1.end());
  for(int i=0;i<n;i++){
    c[i].first=v1[i];c[i].second=v2[i];
  }
  sort(c.begin(),c.end(),comp);
  if(n==1){
    cout << max(0,(c[0].first-k)*(c[0].second)) << endl;
  }else{
    while(k!=0 && c[0].first!=0){
      long long x=(c[0].first)*(c[0].second)-(c[1].first)*(c[1].second);
      if(x<c[0].second){
        k-=1;c[0].first-=1;
      }else{
        long long l=ceil(x/c[0].second);
        long long m=min(l,c[0].first);
        k-=m;c[0].first-=m;
      }
      for(food::iterator i=c.begin();i<c.end()-1;i++){
        if((i->first)*(i->second)<((i+1)->first)*((i+1)->second)){
          iter_swap(i,i+1);
        }else{
          break;
        }
      }
    }
    cout << (c[0].first)*(c[0].second) << endl;
  }
}
```
