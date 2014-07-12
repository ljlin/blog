#暑训day2解题报告

##A - Ginkgo Numbers
签到题
```
#include <iostream>
#include <cmath>
#define sqr(x) ((x)*(x))
using namespace std;

bool calc(int m,int n,int p,int q,int t)
{
    if(sqr(m)+sqr(n)==t){
        if(((m*p+n*q)%t==0)&&
           ((m*q-n*p)%t==0))
           return 1;
    }
    return 0;
}

bool check(int p,int q)
{
    bool findt=0,findr=0;
    int t,s;
    s=sqr(p)+sqr(q);
    for(int t=2;t<s;t++)
        if(s%t==0){
            int mid=round(sqrt(t));
            for(int m=0;m<=mid;m++){
                int n=round(sqrt(t-sqr(m)));
                if(calc( m, n,p,q,t)||calc( m,-n,p,q,t)||
                   calc(-m, n,p,q,t)||calc(-m,-n,p,q,t)){
                    findt=1;
                    break;
                }
            }
            int r=s/t;
            if(findt && r>1){
                int mid=round(r);
                for(int x=1;x<=mid;x++){
                    int y=round(sqrt(r-sqr(x)));
                    if(sqr(x)+sqr(y)==r){
                        findr=1;
                        break;
                    }
                }
                if(findr)return 1;
            }
        }
    return 0;
}

int main()
{
    int n;
    cin>>n;
    for(;n>0;n--){
        int p,q;
        cin>>p>>q;
        cout<<(check(p,q)?'C':'P')<<endl;
    }
    return 0;
}
```
***
##C - One-Dimensional Cellular Automaton
构造矩阵快速幂加速模拟对数列的操作
```
#include <cstdio>
#include <cstring>
#define M 60 
int n,m,A,B,C,T ; 
struct Matrix {
	int a[M][M] ,r,c; 
	Matrix(int _r=0,int _c=0): r(_r),c(_c) 
	{
		memset(a,0,sizeof(a)); 
	}
	void clear()
	{
		memset(a,0,sizeof(a));
	}
	void set()
	{
		for (int i=0; i<c; ++i) a[i][i] = 1; 
	}
	Matrix operator* (Matrix & p) 
	{
		Matrix res(r,p.c) ;
		for (int i=0; i<r; ++i)
			for(int j=0; j<p.c; ++j) 
				for (int k=0; k<c; ++k) 
					res.a[i][j] = (res.a[i][j] + a[i][k]*p.a[k][j] % m) % m ; 
		return res; 
	}
};
Matrix S0(1,60) ,b(60,60) ; 
Matrix pow(Matrix& b, int T) 
{
	Matrix res(b.r,b.r); 
	res.set(); 
	while (T) 
	{
		if (T&1) res= res* b;
		b = b*b ;
		T>>=1 ; 
	}
	return res; 
}
int main()
{
	while (scanf("%d%d%d%d%d%d",&n,&m,&A,&B,&C,&T), m) {
		for (int i=0;i<n;++i ) scanf("%d",&S0.a[0][i]);
		S0.c= n ; 
        b.r= b.c= n ;
        b.clear() ; 
    	for (int i=0; i<n; ++i ) {
    		if (i-1>=0) b.a[i-1][i]= A ;
    		b.a[i][i] = B ;
    		if (i+1<n) b.a[i+1][i] = C ; 
    	}
    	b = pow(b,T) ; 
    	S0 = S0 * b ;
    	for (int i=0; i<n; ++i) printf("%d%c",S0.a[0][i]," \n"[i==n-1]) ;
	}
}
```

***

##D - Find the Outlier
高斯消元，枚举一个，剩下的进行消元。如果枚举的那个是有矛盾的，那消元能得到一个零行，出于精度考虑，认为最后那个系数最小的才是零行。队友写的。模板题。
```C++
#include<iostream>
#include<cstring>
#include<cstdio>
#define MAXN 100
#define fabs(x) ((x)>0?(x):(-x))
#define eps 1e-10
using namespace std;
double a[100][100], b[100] ,f[100],map[10][100] ;
int n,m,d ;
inline double pow(int x,int p)
{
    if(p==0) return 1 ;
    if (x==0) return 0 ;
    double ans = 1 ;
    for (int i=1; i<=p; ++i) ans = ans* x ;
    return ans ;
}
void exchange(int x,int y,int m){
    double t ;
    for (int i=0; i<m ; ++i) {
        t= a[x][i]; a[x][i]=a[y][i] ; a[y][i]  =t ;
    }

}
double gauss(int n, int m )
{
    int p ;
    for (int k=0; k<m ; ++k) {
        p = k ;
       for (int j = k+1; j<n ; ++j )
       if (fabs(a[j][k]) > fabs(a[p][k]))
         p= j ;

        exchange(k,p,m+1) ;

       if (fabs(a[k][k])<eps) return 0 ;
       for (int j=k+1; j<n ; ++j) {
          double r = a[j][k] / a[k][k] ;
          for (int i=k; i<=m; ++i)
            a[j][i]-= r*a[k][i];
       }
    }
    return fabs(a[n-1][m]);
}
int main()
{
    while (cin>>n , n ){
        for (int i=0 ;i<=n+2; ++i) {
            cin>>f[i] ;
            for (int j=0; j<=n; ++j) map[i][j] = pow(i,j) ;
        }
        double max = 2341234123413 ; int ans ;
        for (int k=0; k<=n+2; ++k) {
            int m =0 ;
            for (int i=0; i<=n+2; ++i) if (k!=i) {
                for (int j=0; j<=n; ++j) a[m][j] = map[i][j];
                a[m][n+1] = f[i] ;
                ++m ;
            }
            double pp = gauss(n+2,n+1);
            if (max>pp ){  max = pp ; ans = k ;}
        }
        cout <<ans<< endl ;
    }
}

```

***

##F - Never Wait for Weights

带权并查集模板题，队友写的。
```C++
#include <cstdio>
#include <cstring>
struct node {
    int father, def ; // fa- son ;
} ;
const int maxn = 200000 ;
node a[maxn] ;
int n , m ;
int find (int x)
{
   if (a[x].father!=x) {
      int r = a[x].father ;
      a[x].father = find(a[x].father) ;
      a[x].def = a[r].def + a[x].def ;
   }
   return a[x].father ;
}
void Union(int x,int y,int d) // x- y =d
{
    int fx = find(x) ,fy= find(y) ;
    a[fy].father = fx;
    a[fy].def = a[x].def + d - a[y].def ;
}
int main()
{
    while (scanf("%d%d",&n,&m)!=EOF , n) {
        for (int i=1; i<=n; ++i) {
            a[i] .father = i ;
            a[i] .def = 0 ;
        }

        for (int i=1; i<=m; ++i) {
            getchar();
            char c = getchar () ;
            int x,y ,d;
            if (c=='!') {
                scanf("%d%d%d",&x,&y,&d) ;
                Union(y,x,d);
            }else {
                scanf("%d%d",&x,&y) ;
                int fx = find(x), fy = find(y) ;
                if (fx!=fy) puts("UNKNOWN");
                else printf("%d\n",a[x].def-a[y].def);
            }
        }
    }
}
```