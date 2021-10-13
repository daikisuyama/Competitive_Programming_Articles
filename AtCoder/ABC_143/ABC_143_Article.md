# 感想　

昨日のABC143の復習をします。
予定が合わず前回のABCに出られなかったので一ヶ月ぶりのABCでした。相変わらずDまでしか解けませんが、DからEの壁を乗り越えたいです。

# [A問題](https://atcoder.jp/contests/abc143/tasks/abc143_a)

若干問題文が難しかったので戸惑いましたがif文を書くだけでした。

```Python:answerA.py
a,b=map(int,input().split())
if 2*b>=a:
    print(0)
else:
    print(a-2*b)
#if文の部分は以下のようにできる
#print(max(0,a-2*b))
#print(max([0,a-2*b]))
#前者の書き方は、比較するのが二つのみの時に用いることができる。
```

# [B問題](https://atcoder.jp/contests/abc143/tasks/abc143_b)

Bはループを回すだけでした。

```Python:answerB.py
n=int(input())
d=[int(i) for i in input().split()]

c=0
for i in range(n-1):
    for j in range(i+1,n):
        c+=d[i]*d[j]

print(c)
```

# [C問題](https://atcoder.jp/contests/abc143/tasks/abc143_c)

文字列を前から順に見ていって前の文字と異なるところだけカウントすれば良いです。

```Python:answeC.py
n=int(input())
s=input()

c=1
for i in range(1,n):
    if s[i-1]!=s[i]:
        c+=1

print(c)
```

# [D問題](https://atcoder.jp/contests/abc143/tasks/abc143_c)

まず、三つの不等式をそれぞれ比較するのは遅いので逆順でソートして一つの不等式のみを比較すれば良いと考えます。<br>
そして、三辺の長さのうち短い二辺A,Bが小さいとC<(A+B)が成り立たないので、Aが小さくなりすぎたらbreak、Bが小さすぎてAの一番大きいものを考えてもC<(A+B)が成り立たない場合もbreakということを繰り返せば良いです。<br>
PythonではTLEになったので、C++でACしました。

```Python:answerD.py
n=int(input())
l=sorted([int(i) for i in input().split()],reverse=True)

#print(l)
co=0
for i in range(n-2):
    c=l[i]
    x=0
    for j in range(i+1,n-1):
        b=l[j]
        for k in range(j+1,n):
            a=l[k]
            if c<(a+b):
                co+=1
            else:
                if k==j+1:
                    x=1
                break
        if x==1:
            break

print(co)
```

```c++:answerD.cc
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;

int main(){
  int n;
  int lx;
  int a,b,c;
  vector<int> l;
  int x;
  int co=0;
  cin>>n;
  for(int i=0;i<n;i++){
    cin >> lx;
    l.push_back(lx);
  }
  sort(l.begin(),l.end());
  reverse(l.begin(),l.end());
  for(int i=0;i<n-2;i++){
    c=l[i];
    x=0;
    for(int j=i+1;j<n-1;j++){
      b=l[j];
      for(int k=j+1;k<n;k++){
        a=l[k];
        if(c<(a+b)){
          co+=1;
        }else{
          if(k==j+1){x=1;}
          break;
        }
      }
      if(x==1){break;}
    }
  }
  cout << co << endl;
}
```
