#暑训day4解题报告

##A - Elevator
签到题
```C++
#include <iostream>

using namespace std;

int main()
{
	int T;cin>>T;
	for(;T>0;T--){
		long long N,M,A,sum=0;
		cin>>N>>M;
		for(int i=0;i<N;i++){
			cin>>A;
			sum+=A;
		}
		if(sum>M)
			cout<<"Warning"<<endl;
		else
			cout<<"Safe"<<endl;
	}
}
```

***

##B - Continuous Login
暴力打个表，发现不会超过三个数字。一开始错了好久，各种YY奇葩做法。
```C++
#include<iostream>
#include<cstring>
using namespace std;
const long long MaxN = 123456789;
const long long MaxI = 15713;

int f[MaxI+10];

int find(int x)
{
	if((x<f[1])||(f[MaxI]<x))return 0;
	if(f[MaxI]==x)return MaxI;
	int l=1,r=MaxI;
	while(l<r-1){
		int mid=(l+r)>>1;
		if(f[mid]<=x)
			l=mid;
		else
			r=mid;
	}
	if(f[l]==x)return l;
	if(f[r]==x)return r;
	return 0;
}

void work(int N)
{
	int ans[5];
	ans[0]=find(N);
	if(ans[0]){
		cout<<ans[0]<<endl;
		return ;
	}
	else{
		for(int i=1;i<=MaxI;i++){
			ans[0]=i;ans[1]=find(N-f[i]);
			if(ans[1]){
				cout<<ans[0]<<' '<<ans[1]<<endl;
				return;
			}
		}
		for(int i=1;i<=MaxI;i++){
			ans[0]=i;
			for(int j=i;j<=MaxI;j++){
				ans[1]=j;ans[2]=find(N-f[i]-f[j]);
				if(ans[2]){
					cout<<ans[0]<<' '<<ans[1]<<' '<<ans[2]<<endl;
					return;
				}
			}
		}
	}
}

int main()
{
    int T;cin>>T;
    f[0]=0 ;
    for(int i=1;i<=MaxI;++i) f[i]=f[i-1]+i;
    while (T--) {
    	int N;cin>>N;
    	work(N);
    }
}
```

***

##D - Ranking System
模拟队友写的，感觉比较直接。注意细节。
```C++
#include <iostream>
#include <algorithm>
#include <cmath>
#include <string>
using namespace std;
typedef struct {
    int id ;
    string time;
    int s ;
    int pos ;
} mem ;
const double percent[7]= {0,0,0,0.3, 0.2, 0.07, 0.03} ;
mem a[3000] ;
int ans[3000];
int n,m;
bool comp(mem a, mem b){
    if (a.s<b.s) return true ;
    if (a.s==b.s && a.time>b.time) return true ;
    if (a.s==b.s && a.time==b.time && a.id>b.id ) return true;
    return false;
}
int main()
{
    int T;
    cin>>T;
    while(T--) {
        cin>>n;
        int sum =0 ;
        for (int i=1; i<=n; ++i) {
            cin>>a[i].id>>a[i].time>>a[i].s ;
            a[i].pos = i ;
            if (a[i].s>0)  ++sum ;
        }
        sort(a+1,a+n+1,comp) ;
        int ST=0 ;
        for (int i=1; i<=n; ++i) if (a[i].s<=0) {
            ans[a[i].pos] = 1 ;
        } else if (a[i].s>0) { ST = i ; break ; }
        if (!ST) ST=n+1 ;
        int level=6 ,pre= n ;
        while (level>2) {
            int p= floor( percent[level] * sum ) ;
            bool flag =0 ;
            if (p>0) flag = 1;
            for (int i=pre; i>=ST&&p>0; --i,--p){
                ans[a[i].pos] = level;
                pre= i ;
            }
            if (flag)--pre;
            --level ;
        }
        for (int i=pre; i>=ST; --i) ans[a[i].pos] = 2;
        for (int i=1; i<=n; ++i ) cout<<"LV"<<ans[i]<<endl;
    }
}
```

***

##F - Calculate the Function
还是矩阵加速数列递推，不过这次系数不是常数，需要用线段树维护一段区间对应的矩阵相乘的结果。
```C++
#include<iostream>
#include<cstring>
#include<cstdio>
using namespace std;
const int sed = 1000000007 ;
#define LL long long
typedef struct {
    LL a,b,c,d ;
} node ;
node tree[800000] ;
int n ,m ; int b[200000];
node cheng(node a,node b) {
   node p ;
   p.a  = (a.a*b.a%sed + a.b*b.c %sed ) %sed ;
   p.b  = (a.a*b.b%sed + a.b*b.d %sed ) %sed ;
   p.c  = (a.c*b.a%sed + a.d*b.c %sed ) %sed ;
   p.d  = (a.c*b.b%sed + a.d*b.d %sed ) %sed ;
   return p ;
}
void build(int x,int y,int p) {
    if (x==y) {
        tree[p].a= 0 ;
        tree[p].b = b[x];
        tree[p].c= 1 ;
        tree[p].d= 1;
        return ;
    }
    int mid= (x+y) / 2;
    build (x,mid,p*2);
    build (mid+1,y,p*2+1) ;
    tree[p] = cheng(tree[p*2] ,tree[p*2+1]);
}
node find(int x,int y,int l,int r,int p)
{
    if (l==r) return tree[p];
    if (x==l && y==r) return tree[p];
    int mid = (l+r)/2 ;
    if (y<=mid) return  find(x,y,l,mid,p*2);
    if (x>mid) return find(x,y,mid+1,r,p*2+1) ;
    return cheng( find(x,mid,l,mid,p*2), find(mid+1,y,mid+1,r,p*2+1) ) ;
}
int main()
{
    int t;
    scanf("%d",&t);
    while (t--) {
        scanf("%d%d",&n,&m);
        for (int i=1; i<=n ; ++i) scanf("%d",b+i);
        build(1,n,1);
        for (int i=1; i<=m; ++i) {
            int l, r;
            scanf("%d%d",&l,&r) ;
            if (l==r) printf("%lld\n",b[l]);
            else if (l+1==r) printf("%lld\n",b[r]) ;
            else if (l+2<=r) {
                node p =find(l+2,r,1,n,1) ;
                LL ans = (b[l]*p.b % sed + b[l+1]*p.d %sed) %sed ;
                printf("%lld\n",ans);
            }
        }
    }
}
```

***

##I - ?(>_o)!
卖萌题目，就是根据题目要求模拟一下。
```C++
#include <iostream>
#include <string>
#include <cstring>
#include <cstdio>
using namespace std;
string s,sa ;
char c[500];
int main()
{
    int t ;
    cin>>t ;
    getchar();
    while(t--) {
        getline(cin,s);
        sa = "" ;
        for (int i=0;i<s.length() ; ++i ) {
            if (s[i]=='!') sa+="Hello, world!" ;
            if (s[i]=='_') sa+=s ;
        }
        if (sa==s)
          cout<<"Yes"<<endl;
        else
          cout<<"No"<<endl;
    }
}
```