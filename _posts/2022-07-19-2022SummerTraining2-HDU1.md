---
title: '2022 Summer ACM training-HDU Vol.1'
date: 2022-07-19
permalink: /posts/2022/07/2022SummerTraining2-HDU1/
tags:
  - Chinese post
  - ACM
  - Algorithm
---

# 1002：[Dragon slayer](http://acm.hdu.edu.cn/showproblem.php?pid=7139)

题目大意：地图上有k面墙（水平/竖直），求从起点到终点最少需要打破几面墙

对以下的样例示意图
![image-20220719224742313](https://cdn.jsdelivr.net/gh/Beamstripe/img/img/2022/image-20220719224742313.png)

考虑如下转化：
$$
00000\\
0s000\\
11100\\
00000\\
00111\\
000t0
$$
实现如下：

```c++
#include <iostream>
#include <vector>
#include <cstring>
#include <queue>
using namespace std;
typedef pair<int,int> pii;
typedef pair<pii,pii> PII;
vector<PII> walls; 
int map[50][50];
int n,m,k;
int xs,ys,xt,yt;
const int dpos1[][2]={0,1,1,0,0,-1,-1,0};
const int dpos2[][2]={0,2,2,0,0,-2,-2,0};
bool checkpos(int x,int y){
	return x>=0&&x<n&&y>=0&&y<m;
}
bool bfs(int st){
	memset(map,0,sizeof(map));
	// set walls according to state
	for(int i=0;i<walls.size();i++){
		if(st&(1<<i)){
			int wsx,wsy,wtx,wty;
			wsx=walls[i].first.first;
			wsy=walls[i].first.second;
			wtx=walls[i].second.first;
			wty=walls[i].second.second;
			if(wsx==wtx){
				if(wsy>wty)swap(wsy,wty);
				for(int j=wsy;j<=wty;j++){
					map[wsx][j]=-1;
				}
			}else{
				if(wsx>wtx)swap(wsx,wtx);
				for(int j=wsx;j<=wtx;j++){
					map[j][wsy]=-1;
				}
			}
		}
	}
	// Set Boundaries
	for(int i=0;i<=m;i++)map[n][i]=-1;
	for(int i=0;i<=n;i++)map[i][m]=-1;
	// bfs for path
	queue<pii> q;
	q.push({xs,ys});
	while(q.size()){
		int x=q.front().first,y=q.front().second;
		q.pop();
		if(x==xt&&y==yt)return true;
		for(int i=0;i<4;i++){
			int dx1=x+dpos1[i][0],dy1=y+dpos1[i][1],dx2=x+dpos2[i][0],dy2=y+dpos2[i][1];
			if(checkpos(dx1,dy1)&&map[dx1][dy1]!=-1&&checkpos(dx2,dy2)&&map[dx2][dy2]==0){
				map[dx2][dy2]=1;
				q.push({dx2,dy2});
			}
		}
	}
	return false;
}
void solve(){
	walls.clear();
	cin>>n>>m>>k;
	cin>>xs>>ys>>xt>>yt;
	// Transfrom the starting and ending coordinates
	xs=xs*2+1;xt=xt*2+1;ys=ys*2+1;yt=yt*2+1;
	// Set up borders: with walls around the area
	n=n*2+1;m=m*2+1;
	for(int i=0;i<k;i++){
		int x1,y1,x2,y2;
		cin>>x1>>y1>>x2>>y2;
		walls.push_back({ {x1*2,y1*2},{x2*2,y2*2} });
	}
	int ans=k;
	// Traverse through all subsets of WALLS
	for(int wallstatus=0;wallstatus<(1<<k);wallstatus++){
		if(bfs(wallstatus)){
			ans=min(ans,k-__builtin_popcount(wallstatus)); 
		}
	}
	cout<<ans<<endl;
}
int main(){
	int t;
	cin>>t;
	while(t--)solve();
	return 0;
}
```

# 1003：[Backpack](http://acm.hdu.edu.cn/showproblem.php?pid=7140)

题目大意：01背包求装满情况下抑或和最大值

转移方程：
$$
f(i,j,k)=f(i-1,j,k)\vee f(i-1,j\oplus v_i,k-w_i)
$$
考虑bitset压位（$w_{max}=1024$)，将 $k-w_i$ 的操作改成将对应bitset左移 $w_i$ 位

考虑状态压缩，去掉维度 $i$ ，得实际转移方程
$$
f_i(j,k)=f_{i-1}(j,2^{k-1})\vee f_{i-1}(j\oplus v_i,2^{k-1-w_i})
$$
标程：

```c++
#include<bits/stdc++.h>
using namespace std;
int n,m,cnt;
bitset<1050> f[1050],g[1050];
void solve(){
	cin>>n>>m;
	for(int j=0;j<1024;++j){
		f[j].reset();
	}
	f[0][0]=1;
	int x,y;
	int ans=-1;
	for(int i=1;i<=n;++i){
		cin>>x>>y;
		for(int j=0;j<1024;++j){
			g[j]=f[j];
			g[j]<<=x;
		}
		for(int j=0;j<1024;++j){
			f[j]|=g[j^y];
		}
	}
	for(int j=0;j<1024;++j){
		if(f[j][m]){
			ans=j;
		}
	}
	cout<<ans<<endl;
	return;
}

int main(){
	std::ios::sync_with_stdio(false);
	cin.tie(0);
	int t=1;cin>>t;
	for(int i=1;i<=t;++i){
		solve();
	}
	return 0;
} 
```



# 1009：[Laser](http://acm.hdu.edu.cn/showproblem.php?pid=7146)

题目大意：判断是否存在一个点使所有给出点在过该点的倾斜角为$0\degree,45\degree,90\degree,135\degree$的直线上

若所有点均在一条直线上，则一定满足条件；

否则对任意给出点 $(x,y)$，必须存在点 $(x_s,y_s)$ ， ，$x_s\ne x$ 且 $y_s\ne y$ 时有 $|y_s-y|=|x_s-x|$ 

Approach 1.

随机算法：

```c++
#include<bits/stdc++.h>
using namespace std;
typedef pair<int, int> pii;
typedef long long ll;
const double eps = 1e-8;
const double PI = acos(-1.0);
int sgn(double x) {
    if (fabs(x) < eps) {
        return 0;
    }
    return (x < 0) ? -1 : 1;
}
struct Point {
    ll x, y;
    Point(ll x = 0.0, ll y = 0.0): x(x), y(y) {}
};
typedef Point Vector;
Vector operator + (Vector A, Vector B) {
    return Vector(A.x + B.x, A.y + B.y);
}
Vector operator - (Point A, Point B) {
    return Vector(A.x - B.x, A.y - B.y);
}
Vector operator * (Vector A, double p) {
    return Vector(A.x * p, A.y * p);
}
Vector operator / (Vector A, double p) {
    return Vector(A.x / p, A.y / p);
}
bool operator < (const Point A, const Point B) {
    return A.x < B.x || (sgn(A.x - B.x) == 0 && A.y < B.y);
}
bool operator == (const Point A, const Point B) {
    return sgn(A.x - B.x) == 0 && sgn(A.y - B.y) == 0;
}
double Dot(Vector A, Vector B) {
    return A.x * B.x + A.y * B.y;
}
double Cross(Vector A, Vector B) {
    return A.x * B.y - A.y * B.x;
}
double Length(Vector A) {
    return sqrt(Dot(A, A));
}

Point GetLineIntersection(Point P, Vector v, Point Q, Vector w) {
    Vector u = P - Q;
    double t = Cross(w, u) / Cross(v, w);
    return P + v * t;
}

int parallel(Point ua, Point ub,Point va, Point vb)//等于0说明向量平行
{
    return ((ua.x-ub.x)*(va.y-vb.y)-(va.x-vb.x)*(ua.y-ub.y));
}

mt19937 rng(chrono::steady_clock::now().time_since_epoch().count());

inline int randint(const int &l,const int &r){
    return uniform_int_distribution<int>(l, r)(rng);
}

const ll mod = 1e9+7;

bool check(vector<Point> &vp, Point s, int n) {
    for (int i = 0; i < n; i++) {
        auto it = vp[i];
        if (!sgn(it.x - s.x) || !sgn(it.y - s.y)) continue;
        if (sgn(fabs(it.x - s.x) - fabs(it.y - s.y))) return false;
    }
    return true;
}

void solve() {
    int n;
    cin >> n;
    vector<Point> v(n);
    bool flag = true;
    for (int i = 0; i < n; i++) {
        cin >> v[i].x >> v[i].y;
        if (i >= 2) {
            flag = flag && parallel(v[i], v[i-1], v[i-1], v[i-2]) == 0;
        }
    }
    if (flag) {
        cout << "YES\n";
        return;
    } 
    int SUIJI_CNT = 1000;
    for (int i = 0; i < SUIJI_CNT; i++) {
        int a1 = randint(0, n - 1);
        int a2 = randint(0, n - 1);
        if (a1 == a2) continue;
        int b1 = randint(0, n - 1);
        int b2 = randint(0, n - 1);
        if (b1 == b2) continue;
        if (parallel(v[a1], v[a2], v[b1], v[b2]) == 0) continue;
        Point p = GetLineIntersection(v[a1], v[a2] - v[a1], v[b1], v[b2] - v[b1]);
        if (check(v, p, n)) {
            cout << "YES\n";
            return;
        }
    }
    cout << "NO\n";
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(0); cout.tie(0);
    int T = 1;
    cin >> T;
    while (T--) solve();
    return 0;
}
```

Approach 2.

假设我们已经确定一个点在水平方向上(米字的一横)，那么随便找一个不在这条直线上的点，用竖线和两条斜线可以交在这条线的三个点上，只需要暴力判断这三个点作为中心点是否合法即可。

对于更普遍的情况，我们只需要每次将坐标旋转45度，将上面的过程重复4次即可，这样可以讨论到所有情况。或者可以直接找到两个不共线的点，讨论两条线的相交情况，做12次判定也是可以的。

标程：

```c++
#include<bits/stdc++.h>
#define N 1000009
using namespace std;
typedef long long ll;
int x[N],y[N],n,a[N],b[N]; 
inline ll rd(){
	ll x=0;char c=getchar();bool f=0;
	while(!isdigit(c)){if(c=='-')f=1;c=getchar();}
	while(isdigit(c)){x=(x<<1)+(x<<3)+(c^48);c=getchar();}
	return f?-x:x;
}
inline bool pd(int x,int y){
	if(x==0||y==0||x==y||x+y==0)return 1;
	return 0;
}
inline bool check(int xx,int yy){
	for(int i=1;i<=n;++i)if(!pd(xx-x[i],yy-y[i]))return 0;
	return 1;
}
inline bool ch(){
	int tag=1;
	for(int i=2;i<=n;++i){
		if(x[i]==x[1])continue;
		tag=0;
		if(check(x[1],y[i]))return 1;
		if(check(x[1],y[i]+(x[i]-x[1])))return 1;
		if(check(x[1],y[i]-(x[i]-x[1])))return 1;
		break;
    }
    if(tag)return 1;
    return 0;
}
int ps=0;
inline bool check(){
	scanf("%d",&n);
	for(int i=1;i<=n;++i)scanf("%d%d",&a[i],&b[i]);
	for(int i=1;i<=n;++i)x[i]=a[i],y[i]=b[i];
	if(ch())return 1;
	for(int i=1;i<=n;++i)x[i]=b[i],y[i]=a[i];
	if(ch())return 1;
	for(int i=1;i<=n;++i)x[i]=a[i]+b[i],y[i]=a[i]-b[i];
	if(ch())return 1;
	for(int i=1;i<=n;++i)x[i]=a[i]-b[i],y[i]=a[i]+b[i];
	if(ch())return 1; 
    return 0;
}
int main(){
	int T;
	scanf("%d",&T);
	while(T--){
		if(check())puts("YES");
		else puts("NO"); 
	} 
    return 0;
}

```

# 1011：[Random](http://acm.hdu.edu.cn/showproblem.php?pid=7148)

题目大意：$N$ 个 $[0,1]$ 随机数， $M$次操作删除最大值或最小值（概率相同），求剩余数和的期望

在题目情景下可以忽略去除的数是否最大或最小，因为无论如何去除数，剩下的每个数的期望均为定值$\frac12$。因此所求为$\frac{n-m}2\mod p$.

```c++
#include<cmath>
#include<iostream>
#include<algorithm>
#include<string>
#include<cstdio>
using namespace std;
long long p=1e9+7;
int main(){
    //cout<<qmi(2,p-2,p)<<endl; ---500000004
    int t;cin>>t;
    for(int i=0;i<t;i++){
        long long n,m;
        scanf("%lld%lld",&n,&m);
        long long tt=(n-m);
        if(tt==0) cout<<0;
        else if(tt%2==0) cout<<tt/2;
        else 
		cout<<500000004+tt/2;
        cout<<endl;
    } 
}
```

# 1012：[Alice and Bob](http://acm.hdu.edu.cn/showproblem.php?pid=7149)

题目大意：博弈论

Alice先划分两个集合后，Bob删去其中一个集合，剩下集合所有数减1.

Alice胜利条件：集合中出现0

Bob胜利条件：删去所有数

做法：

**$A[i-1]+=A[i]/2$ 判断 $A[0]$是否大于0**

```c++
#include<bits/stdc++.h>
using namespace std;
typedef pair<int, int> pii;
typedef long long ll;
const ll mod = 1e9+7;

void solve() {
    int n;
    cin >> n;
    vector<int> v(n+2);
    for (int i = 0; i <= n; i++) cin >> v[i];
    for (int j = n; j > 0; j--) v[j-1] += v[j] / 2;
    if (v[0]) cout << "Alice\n";
    else cout << "Bob\n";
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(0); cout.tie(0);
    int T = 1;
    cin >> T;
    while (T--) solve();
    return 0;
}
```