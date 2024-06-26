---
title: '2022 Summer ACM training-Newcoder Vol.6'
date: 2022-08-07
permalink: /posts/2022/08/2022SummerTraining12-Newcoder6/
tags:
  - Chinese post
  - ACM
  - Algorithm
---

![image-20220806205445878](https://cdn.jsdelivr.net/gh/Beamstripe/img/img/2022/image-20220806205445878.png)

# A [Array](https://ac.nowcoder.com/acm/contest/33191/A)

题目大意：对于一个数组 $a$ 满足 $\sum_{i=1}^k a_i\le\frac12(1\le k\le n)$ 构造一个数组 $b$ 使数组 $b$ 中每个连续的区间 $j\in[i,i+a_k]$ 存在 $b_j=k$ ( $a$ 的长度 $n\le 2\times10^5$ ,  $b$ 长度任意)

思维题：将数组 $a$ 的每个数 $x$ 化为相应最小的2次幂 $2^{\lfloor{\log_2x}\rfloor}$ 后发现和仍然小于 $\frac12$ ，填充后满足题目条件

其次填充答案时可以从较小数开始周期填充

std: （另：__builtin_clz()可用来计算前置0的个数，表达式意义即 $2^{\lfloor{\log_2x}\rfloor}$ ）

```c++
#include <iostream>
#include <algorithm>
#include <ctime>
using namespace std;
bool Mbe;
constexpr int N = 1 << 18;
int n, a[N], b[N];
pair<int, int> p[N];
bool Med;
int main() {
  //fprintf(stderr, "%.4lf MB\n", (&Mbe - &Med) / 1048576.0);
    //freopen("test.txt", "r", stdin);

  ios::sync_with_stdio(0);
  cin >> n;
  for(int i = 1; i <= n; i++) cin >> a[i], p[i] = make_pair(1 << 31 - __builtin_clz(a[i]), i);
  sort(p + 1, p + n + 1);
  int f = 0;
  for(int i = 1; i <= n; i++) {
    while(b[f]) f++;
    for(int P = f; P < N; P += p[i].first) b[P] = p[i].second;
  }
  cout << N << "\n";
  for(int i = 0; i < N; i++) cout << max(1, b[i]) << " ";
  //cerr << 1e3 * clock() / CLOCKS_PER_SEC << " ms\n";
  return 0;
}
/*
2022/8/1

start coding at
finish debugging at
*/
```



# B [Eezie and Pie](https://ac.nowcoder.com/acm/contest/33191/B)

题目大意：对每个点找 $k$ 祖先区间加1，求各点点权

在dfs栈中直接查询k祖先，直接用树上差分处理

当然 $O(nlogn)$ 的树链剖分做法也是可行的

比赛时用重链剖分找k祖先，大量MLE和TLE

std:

```c++
#include<bits/stdc++.h>
#define pb emplace_back
using namespace std;
typedef long long ll;

const int N=2e6+5;
int n,m,d[N],st[N],top,c[N];
vector<int> ve[N];

void dfs(int x,int fa)
{
	st[++top]=x;
	c[x]++,c[st[top-min(top,d[x]+1)]]--;
	for(int v:ve[x]) if(v!=fa) dfs(v,x);
	top--;
}

void dfss(int x,int fa)
{
	for(int v:ve[x]) if(v!=fa) 
	{
		dfss(v,x),c[x]+=c[v];
	}
}

int main()
{
	cin>>n;assert(1<=n&&n<=2000000);
	for(int i=1,x,y;i<n;i++) 
	{
		cin>>x>>y;assert(1<=x&&x<=n);assert(1<=y&&y<=n);
		ve[x].pb(y),ve[y].pb(x);
	}
	for(int i=1;i<=n;i++){cin>>d[i];assert(0<=d[i]&&d[i]<=n);}
	dfs(1,0);
	dfss(1,0);
	for(int i=1;i<=n;i++) cout<<c[i]<<' ';
}

```

# G [Icon Design](https://ac.nowcoder.com/acm/contest/33191/G)

签到题：模拟打表均可

# J [Number Game](https://ac.nowcoder.com/acm/contest/33191/J)

题目大意：三个数 $A,B,C$ 可以作如下转换：$B = A - B$ 和 $C=B-C$ 求能否将 $C$ 变换为 $x$ 

签到题：

观察得操作一或操作二连续操作2次是无效操作

可得到如下方程
$$
x=\begin{cases}
k\times(2B-A)+B-C&k\in Z,k\mod2=1(操作次数不同)\\
k\times(2B-A)+C&k\in Z,k\mod2=0(操作次数相同)
\end{cases}
$$
代码：

```python
T=int(input())
for i in range(T):
    A,B,C,x=map(int,input().split())
    if A==2*B:
        print('Yes' if C==x or x+C==B else 'No')
    else:
        print('Yes' if (x-C)%(2*B-A)==0 or (x+C-B)%(2*B-A)==0 else 'No')
```

# M [Z-Game on grid](https://ac.nowcoder.com/acm/contest/33191/M)

题目大意：棋盘上有若干格点，走到某些点A获胜，另一些点B获胜，两人A,B轮流操纵棋子向右或向下移动，走至右下角则两人平局，问A能否有把握必胜、必输还是平局

由于一个局面下，对最终胜负有影响的只有棋子的位置和目前是谁的回合。棋子只能向右或向下移动，所以状态是没有后效性的。因此我们可以考虑 dp

用一个布尔数  $dp[i][j]$ 表示移动到 $(i,j)$ ，先手是否必胜。因为每次只能移一格，所以位置确定后也能知道当前是先手还是后手。对于先手和后手，需要分开转移
$$
dp[i][j]=\begin{cases}
dp[i+1][j]\quad|\quad dp[i][j+1],(i+j)\mod2=0\\
dp[i+1][j]\quad\&\quad dp[i][j+1],(i+j)\mod2=1
\end{cases}
$$
对于其他情况，修改初始值即可

std:

```c++
#include<bits/stdc++.h>
#define M 505
using namespace std;
char str[M][M];
int n,m;
char dp[M][M][2]; 
int dfs(int x,int y,int f){
    if(str[x][y]=='A')return 1; //? 
    if(str[x][y]=='B') return 4;//?  
    if(x==n&&y==m)return dp[x][y][f]=2;//?? 
    char &t=dp[x][y][f]; 
    if(t==-1){ 
        if(f==0){
            t=0;
            if(x+1<=n)t|=dfs(x+1,y,f^1);
            if(y+1<=m)t|=dfs(x,y+1,f^1);
        }else{
            t=7;
            if(x+1<=n)t&=dfs(x+1,y,f^1);
            if(y+1<=m)t&=dfs(x,y+1,f^1);
        }
    }
    return t;
}
void solve(){
    cin>>n>>m;
    for(int i=1;i<=n;i++)scanf("%s",str[i]+1);  
    memset(dp,-1,sizeof(dp));
    int t=dfs(1,1,0);  
    printf("%s", t&1?"yes":"no");
    printf(" %s",t&2?"yes":"no");
    printf(" %s\n",t&4?"yes":"no"); 
}
int main(){
    int T;
    cin>>T;
    while(T--) solve(); 
    return 0;
}
```