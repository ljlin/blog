#暑训day3解题报告


比赛|94 - Andrew Stankevich's Contest, Warmup
---|------
地址|http://acm.zju.edu.cn/onlinejudge/showContestProblems.do?contestId=315
题解|http://blog.watashi.ws/1322/andrew-stankevich-11-solution/

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

考察想象力的题目，比较幸运思考的方向是对的。

首先一个矩形区域的黑点和白点个数都是比较容易通过坐标求得。

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
***
##I - Radio Waves
感觉跟noip关押罪犯一样，就是变成完全图了，对所有边排个序，先尽量满足小的边两端发射不同信号，记录每个点敌对点（发射不同信号的点），以后每次合并时对边（u,v)，合并u和v的敌对点，v和u的敌对点。

这有点个过程实际上是在维护一个二分图的不同分量，对每个X部或者Y部分别是一团并查集，然后对每个X部或Y部的点又能由敌对数组找到对方。所以类似于我在维护许多对儿两坨东西，然后这两坨也不知道谁是X谁是Y，但是能通过敌对数组联系，每次处理一条边就是把两对儿“两坨”变成一对儿“两坨”。

```Cpp
#include <iostream>
#include <algorithm>
#include <cmath>
#include <cstring>
#include <cstdio>
#include <queue>
#define sqr(x) ((x)*(x))
#define eps 1e-15
using namespace std;
const int maxn = 1200+100;

struct edge
{
    edge(int _u=0,int _v=0,double _w=0):u(_u),v(_v),w(_w){};
    int u,v;
    double w;
    bool operator < (const edge& rig) const {
        return w<rig.w;
    }
};

int n,X[maxn],Y[maxn],oppsite[maxn],f[maxn],color[maxn];
edge E[maxn*maxn];
double ans;
int Find(int x){return x==f[x] ? x : f[x] = Find(f[x]);}
void Union(int x,int y){
    int fx=Find(x),fy=Find(y);
    if(fx!=fy)
        f[fx]=f[fy];
}
double dis(int a,int b){return sqrt(sqr(X[a]-X[b])+sqr(Y[a]-Y[b]));}

void dfs(int u,int c){
    //if(color[u]) return;
    color[u]=c;
    for(int i=1;i<=n;i++)
        if(i!=u && dis(i,u)<ans+eps)
          if (!color[i])  dfs(i,3-c);
}

bool BFS(double limit)
{
    memset(color,0,sizeof(color));
    queue<int> Q;
    for(int i=1;i<=n;i++){
        if(color[i]==0){
            color[i]=1;
            Q.push(i);
            while(!Q.empty()){
                int u=Q.front();
                Q.pop();
                for(int v=1;v<=n;v++){
                    if(v==u || dis(u,v)>limit-eps)continue;
                    if(color[u]==color[v])return false;
                    if(color[v]==0){
                        color[v]=3-color[u];
                        Q.push(v);
                    }
                }
            }
        }
    }
    return true;
}
int main()
{
    while(cin>>n){
        int m=0;
        for (int i = 1; i <= n; ++i)
             scanf("%d%d",&X[i],&Y[i]);
        for (int i = 1; i <= n; ++i)
            for (int j = i+1; j <= n; ++j)
                    E[m++]=edge(i,j,dis(i,j));
        sort(E,E+m);
        /*
        for (int i = 0; i < m; ++i)
        {
            cout<<E[i].u<<' '<<E[i].v<<' '<<sqrt(E[i].w)<<endl;
        }
        */

        for(int i=1;i<=n;i++) f[i]=i;
        memset(oppsite,0,sizeof(oppsite));

        ans = E[m-1].w;
        for (int i = 0; i < m; ++i)
        {
            if(Find(E[i].u)==Find(E[i].v)){
                ans=E[i].w;
                break;
            }
            if(oppsite[E[i].u]==0) oppsite[E[i].u]=E[i].v;
            if(oppsite[E[i].v]==0) oppsite[E[i].v]=E[i].u;

            Union(E[i].u,oppsite[E[i].v]);
            Union(E[i].v,oppsite[E[i].u]);

        }

        printf("%.10f\n",ans/2);
        for(int i=1;i<=n;i++){
            if (f[i]==i)
            {
                cout
            }
            printf("%d%c",," \n"[i==n]);
        }
    }
    return 0;
}
```