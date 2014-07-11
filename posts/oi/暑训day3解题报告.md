#暑训day3解题报告

##A - Beer Problem
最小费用最大流，从城市A到城市B每天运输啤酒上限为c，运输的价格p对应一条边从A到B容量为c费用为+p的边，源点就是城市1，添加一个汇点以后，对每个城市其啤酒售价为x就从该城市向汇点连接一条容量无穷大，费用为-x的边。
```
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <iostream>
#include <queue>

using namespace std ;

//最小费用最大流 有向图双向边要开两倍
//改成spfa找最长路可以变成最大费用最大流
const int maxn= 10000 , maxm=100000 ,inf = 0x3f3f3f3f ;
class mincostflow {
public:
      int ad[maxn],pre[maxm],v[maxm],c[maxm],w[maxm],p[maxn],d[maxn],n,s,t,tot,ans;
      bool vis[maxn];
      void init()
      {
          tot = 1; ans = 0 ;
          memset(ad,0,sizeof(ad)) ;
      }
      inline int min(int a ,int b)
      {
          return a<b ? a : b ;
      }
      int spfa()
      {
          queue<int> q ;
          memset(vis,false,sizeof(vis)) ;
          memset(d,0x3f,sizeof(d));//这里要走的都是小于零的路径，初始值要弄成正无穷
          memset(p,0,sizeof(p));
          q.push(s) ; vis[s] = true ; d[s] =0 ;
          while (!q.empty())
          {
              int x = q.front() ;
              for (int j=ad[x]; j ; j=pre[j])
              {
                  if (c[j]>0 && d[v[j]] > d[x] + w[j])//最短路
                  {
                      d[v[j]] = d[x] +w[j];
                      p[v[j]] = j ;
                      if (!vis[v[j]])
                      {
                          q.push(v[j]);
                          vis[v[j]] = true ;
                      }
                  }
              }
              vis[x] =false ;
              q.pop() ;
          }
          return d[t]<0;//小于零代表走一趟能盈利
      }
      void getans()
      {
          int val;
          while (spfa())
          {
              int minc = inf ;
              int i = p[t] ;
              while ( i )
              {
                  minc = min( c[i] ,minc) ;
                  i= p[v[i^1]] ;
              }
              i = p[t];
              while (i)
              {
                  ans += minc * w[i] ;
                  c[i]-=minc; c[i^1]+=minc;
                  i =  p[v[i^1]] ;
              }
              //printf("%d\n",ans);
          }
      }
      void add(int x ,int y,int ci, int wi)
      {
          pre[++tot]=ad[x];
          ad[x] = tot;
          v[tot]= y;
          c[tot] = ci;
          w[tot] = wi;
          pre[++tot]=ad[y];
          ad[y] = tot;
          v[tot] = x ;
          c[tot] = 0;
          w[tot] = -wi ;
      }
} g;
int n,m;
int main()
{
    while (cin>>n>>m) {
        g.init() ;
        g.s = 1; g.t= n+1; g.n= n+1;
        for (int i=2; i<=n; ++i) {
            int x ;
            cin>>x ;
            g.add(i,g.t,0x3f3f3f3f,-x) ;
        }
        for (int i=1; i<=m;++i) {
            int x,y,c,w;
            cin>>x>>y>>c>>w;
            g.add(x,y,c,w);
            g.add(y,x,c,w);
        }
        g.getans();
        cout<<-g.ans<<endl;
    }
}

```
***
##C - Black and White

考察想象力的题目，比较幸运思考的方向是对的，首先一个矩形区域的黑点和白点个数都是比较容易通过坐标求得。

1. 横纵坐标有一个为偶数则黑色白色相等，等于面积一半
2. 如果都是奇数则黑色多一个

看到题目说点是按照顺时针给的，而且边和坐标轴平行（不会有一个线段斜跨一个方格）所以考虑从一个点向下一个点移动时候，对答案加减下一个点和原点围成的矩形的信息，一开始以为是从左下角往右上角走是加，回来是负的，后来花了个正方形发现是横着走就是减，竖着走就是加，估计证明也比较好证。

实现的时候要考虑图形可能出现在第一象限之外，要选一个横纵坐标相等而且在图形左下角的点当做原点。

```
#include<iostream>
#define x first
#define y second
using namespace std;
typedef pair<long long,long long> Point;
Point a[50020] ;
int n ;
long long  mx,my ;
long long min(long long a,long long b) { return a<b? a: b ; }
long long sum(Point p,int color)
{
    long long x = p.x - mx ;
    long long y = p.y - my ;
    if (x%2==1 && y%2==1) {
        return x*y/2+color ;
    }
    else return x*y / 2;
}
int main()
{
    while(cin>>n){
        mx = my = 0 ;
        long long cnt[2]={0,0};// 0 for white 1 for black
        Point pre,p;
        for (int i=1; i<=n; ++i)
        {
           cin>>a[i].x>>a[i].y;
           if (mx>a[i].x) {
              mx = a[i].x ; my = a[i].y ;
           }
           else if (mx==a[i].x && my>a[i].y) my = a[i].y ;
        }
        mx = min(mx,my) ; my=mx ;
        pre=a[1];
        for(int i=2;i<=n+1;i++){
            long long flag = 0;
            if(i==n+1)
                p=a[1];
            else
               p = a[i] ;
            flag=(p.x==pre.x)? 1 : -1;
            for(int j=0;j<2;j++)
                cnt[j]+=flag*sum(p,j);
            pre=p;
           // cout<<i<<' '<<cnt[0]<<' '<<cnt[1]<<endl;
        }
        cout<<cnt[1]<<" "<<cnt[0]<<endl;
    }
}

```

***
##D - Integer Numbers
用f[i]表示处理到第i个数字不用修改的数字个数。
* 转移方程 f[i]=min{f[j]}+1 A[i]-A[j]=i-j
发现{A[i]-i}相同一组的是一个比好好的序列，然后拍个序把最多的那组A[i]-i找出来就是答案。

```
//zgs写的太丑了。。。
#include <iostream>
#include <algorithm>

using namespace std;

const int maxn = 50000+100;
int A[maxn],B[maxn];

int main(int argc, char const *argv[])
{
    int n;
    while(cin>>n){
        for(int i=0;i<n;i++){
            cin>>A[i];
            B[i]=A[i]-i-1;
        }
        sort(B,B+n);
        int ansVal=B[0],ansCnt=1,len=1;
        for (int i = 1; i < n; ++i)
        {
            len = B[i]==B[i-1] ? len+1 : 1;
            if(len>ansCnt){
                ansCnt=len;
                ansVal=B[i];
            }
        }

        int first = 0;
        for (int i = 0; i < n; ++i)
        {
            if(A[i]-i-1==ansVal){
                first = A[i]-i;
                break;
            }
        }

        cout<<n-ansCnt<<endl;
        for (int i = 0; i < n; ++i)
            cout<<first+i<<" \n"[i==n-1];

    }
    return 0;
}

```

