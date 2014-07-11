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