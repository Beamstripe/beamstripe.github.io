---
title: '2022 Summer ACM training-Newcoder Vol.9.1(Bonus contest)'
date: 2022-08-17
permalink: /posts/2022/08/2022SummerTraining19-NewcoderBonus/
tags:
  - Chinese post
  - ACM
  - Algorithm
---

![image-20220817192824198](https://cdn.jsdelivr.net/gh/Beamstripe/img/img/2022/image-20220817192824198.png)

# E [Everyone is bot](https://ac.nowcoder.com/acm/contest/38727/E)

对于第 $i$ 个人，如果他是所有人中第 $j$ 个进行复读的，他会获得 $a_{i,j}$ 瓶
冰红茶。
但是如果他是所有进行了复读的人当中倒数第 $p$ 个进行复读的人，那么
他不会获得任何冰红茶，并且需要交给咩噗特雷格博 bot 154 瓶冰红茶
（即获得 −154 杯冰红茶）。
每个人都想最大化自己获得的冰红茶数量，求所有人一共会拿到多少冰
红茶。

思维题：每个人必然在第一轮复读

设轮到某人时仅剩1人就到倒数第 $p$ 个，此人必会复读。同理，倒数 p-2 人也会选择复读，推知第一轮前 $n\%p$ 人必抢先复读

可以考虑如果有人不复读，后面本来没有机会的人必然会抓住机会，那么前面有人就会失去机会。

代码略

# G [Good red-string](https://ac.nowcoder.com/acm/contest/38727/G)



# H [Here is an Easy Problem of Zero-chan](https://ac.nowcoder.com/acm/contest/38727/H)

有一颗 $n$ 个节点且以 $1$ 为根的有根树。
第 $i$ 个点的点权为 $i$。
多次查询编号为 $x$ 的点, $∏^n_{i=1} lca(i, x)$ 的末尾有多少个零。

由题得求2的因子与5的因子的数量较小值。

所以对于查询 $x$ 而言，我们可以枚举 $x$ 到根节点的所有节点，对于每个
点计算有多少个点与 $x$ 不在同一个儿子子树内。
对于上述解法的时间复杂度上界为 $n_2$，此时我们可以发现，对于点 $x$，
他的父亲节点 $fa_x$ 对于祖先路径上的权值计算与 $x$ 相同，所以我们可以
采用树形 dp 来减少重复的计算。

转移方程：
$$
dp_x=dp_{fa_x}+size_x\times(cnt_{x2|5}-cnt_{fa_{x}2|5})
$$

```c++
#include <iostream>
#include <algorithm>
#include <vector>
#include <queue>
#include <map>
#include <functional>
#include <cstring>
#include <string>
#include <cmath>
#include <bitset>
#include <iomanip>
#include <set>
#include <ctime>
#include <cassert>
#include <complex>
#include <cstdio>
#define rep(_it,_lb,_ub) for(int _it=(_lb);_it<=(_ub);_it++)
#define rep2(_it,_ub,_lb) for(int _it=(_ub);_it>=(_lb);_it--)
using namespace std;
typedef long long ll;
typedef long double ld;
//#define double ld
//#define int ll
inline char gc(){static char buf[100000],*p1=buf,*p2=buf;return p1==p2&&(p2=(p1=buf)+fread(buf,1,100000,stdin),p1==p2)?EOF:*p1++;}
#define gc getchar
template <typename T>inline void read(T &s){s=0;T w = 1;char c=gc();while(c<'0'||c>'9'){if(c=='-')w=-1;c=gc();}while(c>='0'&&c<='9'){s=(s<<1)+(s<<3)+c-'0';c=gc();}s*=w;}
const ll mod=998244353;
const ll maxn=100005;
const ll inf=0x3f3f3f3f;
const double pi=acos(-1);
typedef pair<ll,ll> pii;
struct node{
	int l,r,val,lztag;
};
node twos[maxn<<2];
node fives[maxn<<2];
int tws[maxn],fvs[maxn];
int ct[maxn],cf[maxn];
int d[maxn];
int f[maxn];
int sz[maxn];
int son[maxn];
int dfn[maxn];
int ori[maxn];
int top[maxn];
int cnt=0;
vector<int> g[maxn];
void dfs(int x){
	sz[x]=1;
	d[x]=d[f[x]]+1;
	int tmp=x,cnt2=0,cnt5=0;
	for(int i=0;i<g[x].size();i++){
		int v=g[x][i];
		if(v!=f[x]){
			f[v]=x;
			dfs(v);
			sz[x]+=sz[v];
			ct[v]=sz[v]*(tws[v]-tws[x]);
			cf[v]=sz[v]*(fvs[v]-fvs[x]);
			if(sz[v]>sz[son[x]]){
				son[x]=v;
			}
		}
	}
	
}
void dfs2(int x,int tp){
	top[x]=tp;
	dfn[x]=++cnt;
	ori[cnt]=x;
	if(son[x]){
		dfs2(son[x],tp);
	}
	for(int i=0;i<g[x].size();i++){
		int v=g[x][i];
		if(v!=f[x]&&v!=son[x]){
			dfs2(v,v);
		}
	}
}
void pushUp(node* segTree,int root){
	segTree[root].val=segTree[root<<1].val+segTree[root<<1|1].val;
}
void build(int* wgt,node* segTree,int root,int l,int r){
	if(l==r){
        segTree[root].l=segTree[root].r=l;
        segTree[root].val=wgt[ori[l]];
        return;
    }
    int mid=(l+r)>>1;
    build(wgt,segTree,root<<1,l,mid);
    build(wgt,segTree,root<<1|1,mid+1,r);
    segTree[root].l=segTree[root<<1].l;
    segTree[root].r=segTree[root<<1|1].r;
    pushUp(segTree,root);
}
void pushDownLztag(node* segTree,int root){
	if(segTree[root].lztag){
		int lc=(root<<1),rc=(root<<1|1),lztag=segTree[root].lztag;
		segTree[lc].val+=lztag*(segTree[lc].r-segTree[lc].l+1);
		segTree[rc].val+=lztag*(segTree[rc].r-segTree[rc].l+1);
		segTree[lc].lztag+=lztag;
		segTree[rc].lztag+=lztag;
		segTree[root].lztag=0;
	}
}
int getsum(node* segTree,int root,int l,int r){
	int ans=0;
	if(l<=segTree[root].l&&segTree[root].r<=r){
		return segTree[root].val;
	}
	pushDownLztag(segTree,root);
	int mid=(segTree[root].l+segTree[root].r)>>1;
	if(l<=mid)
		ans+=getsum(segTree,(root<<1),l,r);
	if(r>mid)
		ans+=getsum(segTree,(root<<1|1),l,r);
	return ans;
}

int queryChain(node* segTree,int u,int v){
    int ans=0;
    while(top[u]!=top[v]){
        if(d[top[u]]>d[top[v]])swap(u,v);
        ans+=getsum(segTree,1,dfn[top[v]],dfn[v]);
        v=f[top[v]];
    }
    if(dfn[u]>dfn[v])swap(u,v);
    ans+=getsum(segTree,1,dfn[u],dfn[v]);
    return ans;
}
int querySubtree(node* segTree,int x){
	return getsum(segTree,1,dfn[x],dfn[x]+sz[x]-1);
}
void solve(){
	int n,q;
	cin>>n>>q;
	for(int i=2;i<=n;i+=2){
		tws[i]=tws[i/2]+1;
	}
	for(int i=5;i<=n;i+=5){
		fvs[i]=fvs[i/5]+1;
	}
	for(int i=0;i<n-1;i++){
		int u,v;
		cin>>u>>v;
		g[u].push_back(v);
		g[v].push_back(u);
	}
	dfs(1);
	dfs2(1,0);
	build(ct,twos,1,1,n);
	build(cf,fives,1,1,n);
	for(int i=0;i<q;i++){
		int x;
		cin>>x;
		int five,two;
		two=queryChain(twos,1,x);
		five=queryChain(fives,1,x);
		cout<<min(five,two)<<endl;
	}
}
signed main(){
	int TT;
	TT=1;
//	read(TT);
	while(TT--){
		solve();
	}
	return 0;
}


```

# J [ Jellyfish and its dream](https://ac.nowcoder.com/acm/contest/38727/J)

给一个序列，值域为 $0 ∼ 2$。
如果 $(a_i + 1)\mod 3 = a_{(i+1) \mod n}$，就可以将 $a_i$ 赋值为 $(a_i + 1)\mod 3$。
问有限次操作后是否能使所有元素均相等

求差分数组

一次操作可以让差分数组相邻的 $(2, 1)$ 变成 $(0, 0)$， $(1, 1)$ 变成 $(2, 0)$， $(0, 1)$ 变成
$(1, 0)$。
不难发现 $0$ 没有意义。可以通过 $(0, 1)$ 变成 $(1, 0)$这个操作来移动 1的位
置。
只要 $1$ 的数量不少于 $2$ 的数量即可。

```c++
#include<cstdio>
int T;
int n;
int a[2000009];
int vc[10];
int main()
{
	scanf("%d",&T);
	while(T--)
	{
		scanf("%d",&n);
		for(int i(1);i<=n;++i)scanf("%d",a+i);
		a[0]=a[n];
		vc[1]=vc[2]=0;
		for(int i(1);i<=n;++i)
		{
			++vc[(a[i]-a[i-1]+3)%3];
		}
		puts((vc[1]>=vc[2])?"Yes":"No");
	}
}
```

# M [Maimai DX 2077](https://ac.nowcoder.com/acm/contest/38727/M)

签到模拟，略