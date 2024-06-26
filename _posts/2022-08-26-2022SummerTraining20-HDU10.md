---
title: '2022 Summer ACM training-HDU Vol.10'
date: 2022-08-26
permalink: /posts/2022/08/2022SummerTraining20-HDU10/
tags:
  - Chinese post
  - ACM
  - Algorithm
---

![image-20220826111049840](https://cdn.jsdelivr.net/gh/Beamstripe/img/img/2022/image-20220826111049840.png)

# 1001 [Winner Prediction](http://acm.hdu.edu.cn/showproblem.php?pid=7244)

网络流：

一个包含 $n$ 个参赛者的比赛（无平局），赢得最多场次比赛的参赛者（们）获胜；当前有些比赛已经结束，有些比赛还未开始，验证1号参赛者能否有可能获胜

先让 $1$ 号选手赢下所有和他有关的比赛，设此时选手 $i$ 赢了 $a_i$ 场比赛。如果存在某个 $a_i>a_1$ 则 $1$ 号选手不可能成为冠军。否则选手 $i$ 至多还能再赢 $b_i=a_1-a_i$ 场比赛。因此考虑建立一张网络流图：每场未进行的比赛在图中用一个点表示，源点向它连容量为 $1$ 的边，它向它的两个参赛选手的对应点各自连容量为 $1$ 的边；选手 的对应点向汇点连容量为 $b_i$ 的边。计算该图最大流，若**从源点出发的边**满流则答案为 YES ，否则为 NO 。

```c++
#include <cstdio>
#include <algorithm>
#include <queue>
#include <cstring>
using namespace std;
const int N = 505;
int n, T, m1, m2, s[N];
namespace Flow
{
    struct Edge
    {
        int To, Val, Nxt;
    } Ed[10005];
    int n, S, T, Head[2005], cur[2005], dis[2005], cnt;
    void AddEdge(int x, int y, int val)
    {
        // printf("AddEdge(%d,%d,%d)\n", x, y, val);
        Ed[++cnt] = (Edge){y, val, Head[x]};
        Head[x] = cnt;
        Ed[++cnt] = (Edge){x, 0, Head[y]};
        Head[y] = cnt;
    }
    bool bfs()
    {
        for (int i = 1; i <= n; i++)
            dis[i] = 1e9;
        queue<int> Q;
        Q.push(S);
        dis[S] = 0;
        while (!Q.empty())
        {
            int Now = Q.front();
            Q.pop();
            for (int i = Head[Now]; i != -1; i = Ed[i].Nxt)
            {
                if (dis[Ed[i].To] == 1e9 && Ed[i].Val)
                {
                    Q.push(Ed[i].To);
                    dis[Ed[i].To] = dis[Now] + 1;
                }
            }
        }
        return dis[T] != 1e9;
    }
    int dfs(int x, int val)
    {
        if (x == T || val == 0)
        {
            return val;
        }
        int Out = 0;
        for (int &i = cur[x]; i != -1; i = Ed[i].Nxt)
        {
            if (dis[Ed[i].To] != dis[x] + 1 || !Ed[i].Val)
                continue;
            int tmp = dfs(Ed[i].To, min(val, Ed[i].Val));
            val -= tmp;
            Out += tmp;
            Ed[i].Val -= tmp;
            Ed[i ^ 1].Val += tmp;
            if (val == 0)
                break;
        }
        return Out;
    }
    int Dinic()
    {
        int ans = 0;
        while (bfs())
        {
            memcpy(cur, Head, sizeof(cur));
            ans += dfs(S, 1e9);
        }
        return ans;
    }
}
int main()
{
    // freopen("1001.in", "r", stdin);
    // freopen("1001.out", "w", stdout);
    scanf("%d", &T);
    while (T--)
    {
        scanf("%d%d%d", &n, &m1, &m2);
        for (int i = 1; i <= n; i++)
            s[i] = 0;
        for (int i = 1; i <= m1; i++)
        {
            int x, y, z;
            scanf("%d%d%d", &x, &y, &z);
            if (z == 1)
                s[x]++;
            else
                s[y]++;
        }

        Flow::n = n + m2 + 2;
        Flow::S = n + m2 + 1;
        Flow::T = n + m2 + 2;
        Flow::cnt = -1;
        for (int i = 1; i <= Flow::n; i++)
        {
            Flow::Head[i] = -1;
        }

        int cnt = 0;
        for (int i = 1; i <= m2; i++)
        {
            int x, y;
            scanf("%d%d", &x, &y);
            if (x == 1 || y == 1)
            {
                cnt++;
                s[1]++;
                continue;
            }
            Flow::AddEdge(n + i, x, 1);
            Flow::AddEdge(n + i, y, 1);
            Flow::AddEdge(Flow::S, n + i, 1);
        }
        bool flag = true;
        for (int i = 2; i <= n; i++)
        {
            if (s[i] > s[1])
                flag = false;
            Flow::AddEdge(i, Flow::T, s[1] - s[i]);
        }
        if (!flag)
        {
            printf("NO\n");
            continue;
        }
        printf(Flow::Dinic() == m2 - cnt ? "YES\n" : "NO\n");
    }
}
```

# 1003 [Wavy Tree](http://acm.hdu.edu.cn/showproblem.php?pid=7246)

贪心：

给定数组，一次操作可以让其中一个元素加1或减1，求最终形成波动数组的最少操作次数（波动数组：$a_i>\max\{a_{i-1},a_{i+1}\}$ 与$a_i<\min\{a_{i-1},a_{i+1}\}$对于 $1< i< n$ 必有1个成立）

贪心，对于两种波动情况分开讨论

```c++
#include <bits/stdc++.h>
#define rep(_it,_lb,_ub) for(int _it=(_lb);_it<=(_ub);_it++)
#define rep2(_it,_ub,_lb) for(int _it=(_ub);_it>=(_lb);_it--)
using namespace std;
typedef long long ll;
typedef long double ld;
//#define double ld
//#define int ll
inline char gc(){static char buf[100000],*p1=buf,*p2=buf;return p1==p2&&(p2=(p1=buf)+fread(buf,1,100000,stdin),p1==p2)?EOF:*p1++;}
#define gc getchar
//inline ll read(){char c=gc();ll su=0,f=1;for (;c<'0'||c>'9';c=gc()) if (c=='-') f=-1;for (;c>='0'&&c<='9';c=gc()) su=su*10+c-'0';return su*f;}
template <typename T>inline void read(T &s){s=0;T w = 1;char c=gc();while(c<'0'||c>'9'){if(c=='-')w=-1;c=gc();}while(c>='0'&&c<='9'){s=(s<<1)+(s<<3)+c-'0';c=gc();}s*=w;}
const ll mod=998244353;
const ll maxn=100005;
const ll inf=0x3f3f3f3f;
const double pi=acos(-1);
typedef pair<int,int> pii;
vector<ll> a;
vector<ll> b;
void solve(){
	a.clear();
	b.clear();
	int n;
	read(n);
	a.push_back(0);
	rep(i,1,n){
		ll x;
		read(x);
		a.push_back(x);
	}
	b.assign(a.begin(),a.end());
	//udu
	ll cnt1=0,cnt2=0;
	rep(i,1,n-1){
		if(i&1){
			if(a[i+1]>a[i])continue;
			cnt1+=a[i]+1-a[i+1];
			a[i+1]=a[i]+1;
		}else{
			if(a[i+1]<a[i])continue;
			cnt1+=a[i+1]+1-a[i];
			a[i+1]=a[i]-1;
		}
	}
	
	//dud
	rep(i,1,n-1){
		if(i&1){
			if(b[i+1]<b[i])continue;
			cnt2+=b[i+1]+1-b[i];
			b[i+1]=b[i]-1;
		}else{
			if(b[i+1]>b[i])continue;
			cnt2+=b[i]+1-b[i+1];
			b[i+1]=b[i]+1;
		}
	}
	cout<<min(cnt1,cnt2)<<endl;
}
signed main(){
//	ios::sync_with_stdio(false),cin.tie(0);
//#ifndef ONLINE_JUDGE
//	freopen("test.txt","r",stdin);
//#endif
	int TT;
	TT=1;
	read(TT);
	while(TT--){
		solve();
	}
	return 0;
}

```

# 1004 [Average Replacement](http://acm.hdu.edu.cn/showproblem.php?pid=7247)

给出一个森林，每次迭代将点权更改成子结点与其本身的点权均值，求多次迭代后每个结点的点权

1.每个连通块内的数最后会趋于同一个值。

2.记 $deg(i)$ 表示 的朋友数量，则每个连通块内 $\sum a_i(deg(i)+1)$ 为一定值。

```c++
#include<cstdio>
using namespace std;
int n,m,T,a[100005],deg[100005];
long long Ans[100005],Sum[100005];

int F[100005];
int Find(int x)
{
    return F[x] == x ? x : F[x] = Find(F[x]);
}
int main()
{
    // freopen("1004.in","r",stdin);
    // freopen("1005.out","w",stdout);
    scanf("%d",&T);
    while(T--)
    {
        scanf("%d%d",&n,&m);
        // printf("-- T = %d (n,m) = %d %d\n",T,n,m);
        for(int i=1;i<=n;i++)
        {
            scanf("%d",&a[i]);
            deg[i] = 0;
            F[i] = i;
            Sum[i] = Ans[i] = 0;
        }
        for(int i=1;i<=m;i++)
        {
            int x,y;
            scanf("%d%d",&x,&y);
            deg[x]++;
            deg[y]++;
            if(Find(x) != Find(y))
                F[Find(x)] = Find(y);
        }
        for(int i=1;i<=n;i++)
        {
            Ans[Find(i)] += (long long)(deg[i]+1) * a[i];
            Sum[Find(i)] += (deg[i]+1);
        }
        for(int i=1;i<=n;i++)
        {
            printf("%.6lf\n",(double)((long double)Ans[Find(i)] / Sum[Find(i)]));
            // printf("%lld %lld\n",Ans[Find(i)],Sum[Find(i)]);
        }
    }
    
}
```

# 1005 [Apples](http://acm.hdu.edu.cn/showproblem.php?pid=7248)

一个含 $N$ 个结点的环，环上每条边有边权 $w$（花费），每个点有一个点权，每次以 $|s|\times w$ 的代价将点权转移 $|s|$ 至相邻结点，求最小花费

考虑枚举 之间传输了多少苹果，设为 $x$ 。则接下来每两人之间传输的苹果数是固定的，可知答案为 $\sum_{i=1}^nl_i|x-a_i|$ 。容易知道这个式子的最小值在 $x$ 取 $a_1,a_2,\dots,a_n$ 的加权中位数时取到。将 $a_i$ 先离散化、排序，用序列数据结构维护 $l$ 的前缀和即可。时间复杂度 $O(n\log n)$。

```c++
#include <bits/stdc++.h>
using namespace std;
#define fo(i,j,k) for(int i=(j),end_i=(k);i<=end_i;i++)
#define fd(i,j,k) for(int i=(j),end_i=(k);i>=end_i;i--)
#define ll long long
const int N=(1<<20)+1;
int n,m;
ll a[N],b[N];
int c[N],d[N];
ll s1[N<<2],s2[N<<2];
#define lc (u<<1)
#define rc (u<<1|1)
#define ls lc,l,mid
#define rs rc,mid+1,r
inline void add(int u,int l,int r,int x,int d1,ll d2)
{
    s1[u]+=d1; s2[u]+=d2;
    if(l==r) return;
    int mid=l+r>>1;
    return x<=mid?add(ls,x,d1,d2):add(rs,x,d1,d2);
}
ll t1,t2;
inline void ask(int u,int l,int r,int L,int R)
{
    if(L<=l&&r<=R) return (void)(t1+=s1[u],t2+=s2[u]);
    int mid=l+r>>1;
    if(L<=mid) ask(ls,L,R);
    if(mid<R)  ask(rs,L,R);
}
inline int work(int u,int l,int r,ll d)
{
    if(l==r) return l;
    int mid=l+r>>1;
    return d>s1[lc]?work(rs,d-s1[lc]):work(ls,d);
}
int u;
inline void solve()
{
    u=work(1,1,m,(s1[1]+1)>>1);
    t1=t2=0;
    ask(1,1,m,1,u);
    printf("%lld\n",(b[u]*t1-t2+(s2[1]-t2)-b[u]*(s1[1]-t1)));
}

inline void work()
{
    scanf("%d",&n);
    fo(i,1,n)
    {
        scanf("%d",&a[i]); scanf("%d",&c[i]);
        a[i]-=c[i];
        scanf("%d",&c[i]);
    }
    
    a[1]=0;
    fo(i,2,n) a[i]+=a[i-1],b[i]=a[i];
    sort(b+1,b+n+1);
    m=unique(b+1,b+n+1)-b-1;
    fo(i,1,n) d[i]=lower_bound(b+1,b+m+1,a[i])-b;
    fo(i,1,n) add(1,1,m,d[i],c[i],b[d[i]]*c[i]);
    solve();
    int x,y,q; scanf("%d",&q);
    for(;q--;)
    {
        scanf("%d%d",&x,&y);
        add(1,1,m,d[x],(y-c[x]),b[d[x]]*(y-c[x]));
        c[x]=y; solve();
    }
    fo(i,1,n) a[i]=b[i]=c[i]=d[i]=0;
    fo(i,0,n<<2) s1[i]=s2[i]=0;
}
int main()
{
    int T; scanf("%d",&T);
    while(T--) work();
    return 0;
}
```

# 1007 [Even Tree Split](http://acm.hdu.edu.cn/showproblem.php?pid=7250)

一棵含偶数个结点的树，可以删去任意条边使其成为偶数个结点树的森林，求删边方案数

求出子树为偶数所有结点，记录其到父结点的连边，设边数为 $e$ ，答案为 $2^e-1$ 

```c++
#include <bits/stdc++.h>
#define rep(_it,_lb,_ub) for(int _it=(_lb);_it<=(_ub);_it++)
#define rep2(_it,_ub,_lb) for(int _it=(_ub);_it>=(_lb);_it--)
using namespace std;
typedef long long ll;
typedef long double ld;
//#define double ld
//#define int ll
inline char gc(){static char buf[100000],*p1=buf,*p2=buf;return p1==p2&&(p2=(p1=buf)+fread(buf,1,100000,stdin),p1==p2)?EOF:*p1++;}
#define gc getchar
//inline ll read(){char c=gc();ll su=0,f=1;for (;c<'0'||c>'9';c=gc()) if (c=='-') f=-1;for (;c>='0'&&c<='9';c=gc()) su=su*10+c-'0';return su*f;}
template <typename T>inline void read(T &s){s=0;T w = 1;char c=gc();while(c<'0'||c>'9'){if(c=='-')w=-1;c=gc();}while(c>='0'&&c<='9'){s=(s<<1)+(s<<3)+c-'0';c=gc();}s*=w;}
const ll mod=998244353;
const ll maxn=100005;
const ll inf=0x3f3f3f3f;
const double pi=acos(-1);
typedef pair<int,int> pii;
int sz[maxn];
int fa[maxn];
int cnt=0;
vector<int> g[maxn];
void dfs(int u){
	sz[u]=1;
	for(int i=0;i<g[u].size();i++){
		int v=g[u][i];
		if(v!=fa[u]){
			fa[v]=u;
			dfs(v);
			sz[u]+=sz[v];
		}
	}
	if(fa[u]&&sz[u]>=2&&sz[u]%2==0)cnt++;
}
ll qpow(ll a,ll x){
	ll ans=1,tmp=a;
	while(x){
		if(x&1)ans=ans*tmp%mod;
		tmp=tmp*tmp%mod;
		x>>=1;
	}
	return ans;
}
void solve(){
	cnt=0;
	int n;
	read(n);
	rep(i,1,n){
		g[i].clear();
		fa[i]=0;
		sz[i]=0;
	}
	rep(i,1,n-1){
		int u,v;
		read(u);read(v);
		g[u].push_back(v);
		g[v].push_back(u);
	}
	dfs(1);
	if(cnt==0){
		cout<<0<<endl;
	}else{
		cout<<qpow(2,(ll)cnt)-1<<endl;
	}
}
signed main(){
//	ios::sync_with_stdio(false),cin.tie(0);
//#ifndef ONLINE_JUDGE
//	freopen("test.txt","r",stdin);
//#endif
	int TT;
//	TT=1;
	read(TT);
	while(TT--){
		solve();
	}
	return 0;
}
```

# 1009 [Painting Game](http://acm.hdu.edu.cn/showproblem.php?pid=7252)

Alice和Bob在一张格数为n的纸条上涂色，规定每次涂色不能与任何着色格相邻，Alice想要着色格数量尽可能少，而Bob希望尽可能多。求给出先手与n的情况下游戏的获胜者

赛时折腾很久才想到周期为7的情况

可以把涂黑操作想象成剪掉连续的三个格子。如果每个连续段长度都不超过2 ，那么结果事实上已经决定了。在此之前，可以发现，Alice 的一种最优策略是：选某个连续段的左数第二个格子涂黑；Bob 的一种最优策略是：选某个连续段的**左数第三个格子**涂黑。设 分别为 Alice、Bob 面对长度为的空纸带时的答案。则有$f(n)=g(n-3)+1,g(n)=f(n-4)+2$。注意到 $f(n)=f(n-7)+3,g(n)=g(n-7)+3$， 因而可以归并到不超过7的情况快速计算答案。时间复杂度 $O(1)$。

```c++
#include <bits/stdc++.h>
#define rep(_it,_lb,_ub) for(int _it=(_lb);_it<=(_ub);_it++)
#define rep2(_it,_ub,_lb) for(int _it=(_ub);_it>=(_lb);_it--)
using namespace std;
typedef long long ll;
typedef long double ld;
//#define double ld
//#define int ll
inline char gc(){static char buf[100000],*p1=buf,*p2=buf;return p1==p2&&(p2=(p1=buf)+fread(buf,1,100000,stdin),p1==p2)?EOF:*p1++;}
#define gc getchar
//inline ll read(){char c=gc();ll su=0,f=1;for (;c<'0'||c>'9';c=gc()) if (c=='-') f=-1;for (;c>='0'&&c<='9';c=gc()) su=su*10+c-'0';return su*f;}
template <typename T>inline void read(T &s){s=0;T w = 1;char c=gc();while(c<'0'||c>'9'){if(c=='-')w=-1;c=gc();}while(c>='0'&&c<='9'){s=(s<<1)+(s<<3)+c-'0';c=gc();}s*=w;}
const ll mod=998244353;
const ll maxn=100005;
const ll inf=0x3f3f3f3f;
const double pi=acos(-1);
typedef pair<int,int> pii;

void solve(){
	int n;
	read(n);
	string st;
	cin>>st;
	if(n==1){
		cout<<1<<endl;return;
	}
	if(n==2&&st[0]!='A'){
		cout<<1<<endl;return;
	}
	if(n==3&&st[0]!='A'){
		cout<<2<<endl;return;
	}
	if(n==4&&st[0]!='A'){
		cout<<2<<endl;return;
	}
	if(n==5&&st[0]!='A'){
		cout<<3<<endl;return;
	}
	if(n==6&&st[0]!='A'){
		cout<<3<<endl;return;
	}
	int ans=0;
	if(st[0]=='A'){
		n-=2;ans++;
		int num=n/7;
		int numm=n%7;
		ans+=num*3;
		if(numm==2) ans++;
		if(numm==3) ans++;
		if(numm==4) ans+=2;
		if(numm==5) ans+=2;
		if(numm==6) ans+=3;
	}else{
		n-=6;ans+=3;
		int num=n/7;
		int numm=n%7;
		ans+=num*3;
		if(numm==2) ans++;
		if(numm==3) ans++;
		if(numm==4) ans+=2;
		if(numm==5) ans+=2;
		if(numm==6) ans+=3;
	}
	cout<<ans<<endl;
}
signed main(){
//	ios::sync_with_stdio(false),cin.tie(0);
//#ifndef ONLINE_JUDGE
//	freopen("test.txt","r",stdin);
//#endif
	int TT;
	TT=1;
	read(TT);
	while(TT--){
		solve();
	}
	return 0;
}
```