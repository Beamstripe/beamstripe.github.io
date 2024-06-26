---
title: '2022 Summer ACM training-Newcoder Vol.7'
date: 2022-08-08
permalink: /posts/2022/08/2022SummerTraining13-Newcoder7/
tags:
  - Chinese post
  - ACM
  - Algorithm
---

![image-20220808170134086](https://cdn.jsdelivr.net/gh/Beamstripe/img/img/2022/image-20220808170134086.png)

# C [Constructive Problems Never Die](https://ac.nowcoder.com/acm/contest/33192/C)

签到模拟题

好写一点的方法：

```c++
#include<bits/stdc++.h>
using namespace std;
int main()
{
	ios_base::sync_with_stdio(false);
	int T;
	cin>>T;
	while(T--)
	{
		int n;
		cin>>n;
		int ok=0;
		vector<int> a(n+5);
		vector<int> p(n+5);
		for(int i=1;i<=n;i++)
		{
			cin>>a[i];
			if(a[i]!=a[1])
				ok=1;
		}
		if(not ok)
		{
			cout<<"NO"<<endl;
			continue;
		}
		int l=1,r=n;
		for(int i=1;i<n;i++)
		{
			if(a[i]!=l)
				p[i]=l,l++;
			else
				p[i]=r,r--;
		}
		p[n]=l;
		if(p[n]==a[n])
		{
			for(int i=1;i<=n-1;i++)
			{
				if(p[n]!=a[i])
				{
					swap(p[n],p[i]);
					break;
				}
			}
		}
		cout<<"YES"<<endl;
		for(int i=1;i<=n;i++)
		{
			cout<<p[i]<<" \n"[i==n];
		}
	}
	return 0;
}
```

# F [Candies](https://ac.nowcoder.com/acm/contest/33192/F)

题目大意：将数组中连续两个相等或和为 $x$ 的两个数消去，求最多消除对数。

可以用栈模拟消去，之后从两侧消去栈中剩余的

```c++
#include <iostream>
#include <vector>
using namespace std;
vector<int> st;

int main(){
	int n,x,ans=0;
	cin>>n>>x;
	int tp,top;
	for(int i=0;i<n;i++){
		cin>>tp;
		if(st.empty())
		st.push_back(tp);
		else{
			top=st[st.size()-1];
			if(top==tp||top+tp==x){
				st.pop_back();
				++ans;
			}else st.push_back(tp);
		}
	}
	for(int i=0;i<st.size()-i-1;i++){
		if(st[i]==st[st.size()-i-1]||st[i]+st[st.size()-i-1]==x)ans++;
		else break;
	}
	cout<<ans<<endl;
	return 0;
}
```

# G [Regular Expression](https://ac.nowcoder.com/acm/contest/33192/G)

签到题：求匹配一个串所需正则表达式的最小长度（字符集为小写字母）

由于模式 $.*$ 可以匹配任意串，最小长度最多为2，特判就行

一个字符时“a”和“.”可以匹配，长度为1，方案数为2

两个字符以上的方案数则分别考z虑“ab”，“a+”，“a.”，“a\*”，“.a”，“.+”，“..”，“.\*”这几种能否匹配即可

```c++
#include<bits/stdc++.h>
#define ll long long
using namespace std;

//a . *
string S;
int sn;

int main()
{
	ios_base::sync_with_stdio(false);
	
	int Tcase; cin>>Tcase;
	while(Tcase--)
	{
		cin>>S;
		sn=S.size();
		int ans1,ans2;
		if(sn==1)
		{
			ans1=1, ans2=2;
			//a
			//.
		}
		else //sn>=2
		{
			ans1=2;
			ans2=0;
			if(sn==2) ans2++; //ab
			if(sn==2) ans2++; //a.
			int ok=1;
			for(int i=1;i<sn;i++) if(S[i]!=S[i-1]) ok=0;
			ans2+=ok; //a*
			ans2+=ok; //a+
			if(sn==2) ans2++; //.a
			if(sn==2) ans2++; //..
			ans2++; //.*
			ans2++;//.+
		}
		cout<<ans1<<' '<<ans2<<'\n';
	}
	
	return 0;
}
```

# J [Melborp Elcissalc](https://ac.nowcoder.com/acm/contest/33192/J)

题目大意：长为 $n$ 的数组 $a$ 仅含数字 $0$ 到 $k-1$  ，求模 $k$ 意义下区间和为 $0$ 的数量为 $t$ 时所有满足要求的数组个数

首先求出给出数组时模 $k$ 区间和为 $0$ 的数量，由于所有区间可以看做前缀和相减，处理前缀和后相同前缀和的对应下标就是满足要求的数组，设前缀和为 $i$ 的数量为 $x_i$ ，则总数为
$$
\sum_{i=0}^k\mathbb{C}_{x_i}^2=\sum_{i=0}^k\frac{x_i(x_i-1)}2
$$
然后对于长度为 $n$ ( $n\le64$ ) 的数组直接用DP转移数组个数

记 $f(i,j,k)$ 代表考虑完了 $0$~$i$ 的数，放进了 $j$ 个位置，对数量为 $k$ 的方案数，

则 $f(k-1,n,t)$ 即为答案。

std:

```c++
#include<bits/stdc++.h>
using namespace std;
const int MOD=998244353;
long long poww(long long x,int y)
{
	long long ret=1;
	while(y)
	{
		if(y&1)ret*=x,ret%=MOD;
		x*=x,x%=MOD;
		y>>=1;
	}
	return ret;
}
struct modint
{
	int v;
	modint(){v=0;}
	modint(int x){v=x%MOD;}
	modint(long long x){v=x%MOD;}
	modint operator+(const modint &a)const{return modint((v+a.v)%MOD);}
	modint operator+=(const modint &a){return *this=*this+a;}
	modint operator-(const modint &a)const{return modint((v-a.v+MOD)%MOD);}
	modint operator-=(const modint &a){return *this=*this-a;}
	modint operator*(const modint &a)const{return modint(1ll*v*a.v);}
	modint operator*=(const modint &a){return *this=*this*a;}
	modint operator/(const modint &a)const{return modint(1ll*v*poww(a.v,MOD-2));}
	modint operator/=(const modint &a){return *this=*this/a;}
	bool operator==(const modint &a)const{return v==a.v;}
}one(1);
modint fac[11111],ifac[11111];
modint C(int n,int m)
{
	return fac[n]*ifac[m]*ifac[n-m];
}
modint dp[77][77][3333];
int main()
{
	ios_base::sync_with_stdio(false);
	fac[0]=one;
	for(int i=1;i<=11000;i++)fac[i]=fac[i-1]*i;
	ifac[11000]=one/fac[11000];
	for(int i=11000;i>=1;i--)ifac[i-1]=ifac[i]*i;
	int n,k,t;
	cin>>n>>k>>t;
	for(int i=0;i<=n;i++)
	{
		dp[0][i][i*(i+1)/2]=C(n,i);
	}
	for(int now=1;now<=k-1;now++)
	{
		for(int i=0;i<=n;i++)
		{
			for(int cnt=0;cnt<=i*(i+1)/2;cnt++)
			{
				for(int j=0;j<=n-i;j++)
				{
					dp[now][i+j][cnt+j*(j-1)/2]+=dp[now-1][i][cnt]*C(n-i,j);
				}
			}
		}
	}
	cout<<dp[k-1][n][t].v<<endl;
	return 0;
}
```

# K [Great Party](https://ac.nowcoder.com/acm/contest/33192/K)

博弈论——Nim游戏变式

> Nim游戏：两名玩家，有若干堆石子，每堆石子的数量都是有限的，合法的移动是“选择一堆石子并拿走若干颗（不能不拿，可以全拿）”，如果轮到某个人时所有的石子堆都已经被拿空了，则判负

**Nim游戏结论**：所有堆异或和为0时先手必败，否则必胜

现在需要求区间内的游戏结果，有

> 一个区间如果长度是奇数，那么先手必胜。否则如果石子个数减一的异或和不为0，先手必胜；否则后手必胜。

如果区间长度是1，显然先手必胜。否则如果区间长度为偶数，先后手必定不能把某一堆石子取完，不然区间长度奇偶性会发生改变，导致该玩家必败。那么每堆石子有一个是不能取的，除此之外是一个nim模板（取走一堆的所有石子）。

如果区间长度为奇数，先手显然可以对最大那堆石子操作，使其变成一个区间长度为偶数且石子个数减一的异或和为0的局面。

对于所有子区间的询问，可以通过前缀异或和与分块完成，时间复杂度为$O(n\sqrt m)$

```c++
#include<bits/stdc++.h>
#define rep(i,x,y) for(int i=x; i<=y; ++i)

using namespace std;
const int N=100005,M=2000005,lim=350;
typedef long long LL;
int n,m,a[N],id[N],s[2][M];
struct D{int id,l,r;} dat[N];
LL ans[N],nw;

int getint()
{
	char ch;
	while(!isdigit(ch=getchar()));
	int x=ch-48;
	while(isdigit(ch=getchar())) x=x*10+ch-48;
	return x;
}

bool cmp(D a,D b)
{
	return id[a.l]==id[b.l]?a.r<b.r:a.l<b.l;
}

void add(int x)
{
	nw+=s[x&1][a[x]];
	++s[x&1][a[x]];
}

void del(int x)
{
	--s[x&1][a[x]];
	nw-=s[x&1][a[x]];
}

LL get(LL len)
{
	return len*(len+1)/2;
}

int main()
{
//	freopen("g.in","r",stdin);
//	freopen("g.out","w",stdout);
	n=getint(),m=getint();
	rep(i,1,n) a[i]=(getint()-1)^a[i-1],id[i]=(i-1)/lim+1;
	rep(i,1,m) dat[i].l=getint(),dat[i].r=getint(),dat[i].id=i,ans[i]=get(dat[i].r-dat[i].l+1);
	sort(dat+1,dat+1+m,cmp);
	int l=1,r=0;
	s[0][0]=1;
	rep(i,1,m)
	{
		while(r<dat[i].r) add(++r);
		while(r>dat[i].r) del(r--);
		while(l>dat[i].l) --l,add(l-1);
		while(l<dat[i].l) del(l-1),++l;
		ans[dat[i].id]-=nw;
	}
	rep(i,1,m) printf("%lld\n",ans[i]);
	return 0;
}
```