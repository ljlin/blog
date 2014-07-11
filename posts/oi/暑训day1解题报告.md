#暑训day1解题报告#
===
*A - Painting the sticks*

因为不能覆盖涂/涂两次，所以就数数有几个三个一块儿就行了。

```
#include<cstdio>
int a[100],ans ;
int main()
{
    int n , t = 0 ;
    while (scanf("%d",&n)!=EOF) {
       for (int i=1; i<=n; ++i) scanf("%d",a+i);
       ans = 0 ; 
       for (int i=1; i<=n ; ++i) if (a[i]!=a[i-1]) ++ans ; 
       ans = ans /3 + ( ans % 3>0)  ; 
       ++t ; 
       printf("Case %d: %d\n",t,ans) ; 
    }
}
```
 
- - -
*B - "Ray, Pass me the dishes!"*

线段树，没过，不知道是想错了还是写呲毛了。


```
#include <iostream>
#include <cstring>
#include <cstdio>

using namespace std;

/**
 * 待处理序列A从1到N
 * min对应的序列是前缀和序列右移一位
 */
const int  MaxN = 500000;
#define left  first
#define right second
long long A[MaxN];
struct Range:pair<int,int>{
    Range(int a=0,int b=0){
        left  = a;
        right = b;
    }
    long long sum(){
        return A[right]-A[left-1];
    }
};
struct node {
    int l,r,maxPos,minPos;
    node(int x=0){l=r=maxPos=minPos=x;}
    long long ans(){return A[l]-A[r-1];}
    long max(){return A[maxPos];}
    long min(){return A[minPos-1];}
};
struct SegmentTree
{
    #define MaxSegmentLength 500000
    const int root;
    SegmentTree():root(1){}
    node data[4*MaxSegmentLength];
    void newNode(int p,int x){data[p]=node(x);}
    void maintain(int p){
        int lch= p*2,rch= p*2+1,t=0;

        t=(data[lch].ans()>data[rch].ans())?lch:rch;

        data[p].l=data[t].l;
        data[p].r=data[t].r;

        if(data[p].ans()<(data[rch].max()-data[lch].min())){
            data[p].l=data[lch].minPos;
            data[p].r=data[rch].maxPos;
        }

        data[p].maxPos = (data[lch].max()>data[rch].max())?
                            data[lch].maxPos:
                            data[rch].maxPos;
        data[p].minPos = (data[lch].min()<data[rch].min())?
                            data[lch].minPos:
                            data[rch].minPos;
    }
    void build(int x,int y,int p){
        if (x==y)
            newNode(p,x);
        else {
            int mid=(x+y)/2;
            build(x    ,mid,p*2  );
            build(mid+1,y  ,p*2+1);
            maintain(p);
        }
    }
    int queryMax(int x,int y,int l,int r,int p)
    {
        //cout<<"QueryMax  ("<<x<<','<<y<<")  ["<<l<<','<<r<<"] @"<<p<<endl;
        if (x==l && y==r) return data[p].maxPos;
        int mid = (l+r)/2,LPos,RPos;
        //cout<<"        "<<l<<','<<mid<<'|'<<mid+1<<','<<r<<endl;
        if (y<=mid) return queryMax(x,y,l    ,mid,p*2  );
        if (x >mid) return queryMax(x,y,mid+1,r  ,p*2+1);
            LPos=queryMax(x    ,mid,l    ,mid,p*2);
            RPos=queryMax(mid+1,y  ,mid+1,r  ,p*2+1);
            //cout<<"     max(L,R)"<<LPos<<' '<<RPos<<endl;
            return A[LPos]>A[RPos] ? LPos : RPos;
    }
    int queryMin(int x,int y,int l,int r,int p)
    {
        //cout<<"QueryMin  ("<<x<<','<<y<<")  ["<<l<<','<<r<<"] @"<<p<<endl;
        if (x==l && y==r) return data[p].minPos;
        int mid = (l+r)/2,LPos,RPos;
        //cout<<"        "<<l<<','<<mid<<'|'<<mid+1<<','<<r<<endl;
        if (y<=mid) return queryMin(x,y,l    ,mid,p*2  );
        if (x >mid) return queryMin(x,y,mid+1,r  ,p*2+1);
            LPos=queryMin(x    ,mid,l    ,mid,p*2);
            RPos=queryMin(mid+1,y  ,mid+1,r  ,p*2+1);
            //cout<<"     min(L,R)"<<LPos<<' '<<RPos<<endl;
            return A[LPos-1]<A[RPos-1] ? LPos : RPos;
    }
    Range query(int x,int y,int l,int r,int p)
    {
        //cout<<"Query  ("<<x<<','<<y<<")  ["<<l<<','<<r<<"] @"<<p<<endl;
        if (x==l && y==r) return Range(data[p].l,data[p].r);
        int mid = (l+r)/2 ;
        //cout<<"        "<<l<<','<<mid<<'|'<<mid+1<<','<<r<<endl;
        if (y<=mid) return query(x,y,l    ,mid,p*2  );
        if (x >mid) return query(x,y,mid+1,r  ,p*2+1);
            Range LRange,RRange,ans;
            LRange=query(x    ,mid,l    ,mid,p*2  );
            RRange=query(mid+1,y  ,mid+1,r  ,p*2+1);
            ans = LRange.sum()>RRange.sum() ? LRange : RRange;
            //cout<<"     max(Query)"<<ans.sum()<<endl;
            int MaxPos=queryMax(mid+1,y  ,mid+1,r  ,p*2+1),
                MinPos=queryMin(x    ,mid,l    ,mid,p*2  );
            //cout<<"          MaxPos:"<<MaxPos<<"  "<<"MinPos:"<<MinPos<<endl;
            if(A[MaxPos]-A[MinPos-1]>ans.sum()){
                ans.left  = MinPos;
                ans.right = MaxPos;
            }
            //cout<<"      max(max-min)"<<ans.sum()<<endl;
            return ans;
    }
    #undef MaxSegmentLength
};

SegmentTree Tree;

int main()
{
    int n,m,CaseNum=0;A[0]=0;
    while(scanf("%d%d",&n,&m)!=EOF){
        for(int i=1;i<=n;i++) scanf("%lld",A+i);
        for(int i=2;i<=n;i++) A[i]+=A[i-1];

        Tree.build(1,n,Tree.root);
        printf("Case %d:\n",++CaseNum);

        for(int i=0;i<m;i++){
            int a,b;
            scanf("%d%d",&a,&b);
            Range ans = Tree.query(a,b,1,n,Tree.root);
            printf("%d %d\n",ans.left,ans.right);
        }
    }
    /*
    int N,delta;
    cin>>N;
    for(int i=1;i<=N;i++) cin>>A[i];
    //delta=A[1];
    //A[1]=A[0]=0;
    for(int i=2;i<=N;i++)
        A[i]+=A[i-1];//-delta;
    //for(int i=1;i<=N;i++)cout<<A[i]<<' ';
    //cout<<endl;
    Tree.build(1,N,Tree.root);
    int a,b;
    while(cin>>a>>b){
        Range ans = Tree.query(a,b,1,N,Tree.root);
        cout<<ans.left<<' '<<ans.right<<endl;
        //cout<<ans.left<<','<<ans.right<<endl<<endl;
    }
    */
    return 0;
}
//10
// 1  2  3  4  5  6  7  8  9  10
// 4  5  6  2  9  0 -1 13  9 -10
// 4  9 15 17 26 26 25 38 47  37
// 0  1  2 -2  5 -4 -5  9  5 -14
```
 
- - -
*C - Plucking fruits*

最小瓶颈树，跟kruskal类似


```
#include<cstdio>
#include<cstring>
#include<algorithm>
const int maxn= 1010 ,maxm = maxn*maxn;

struct edge{int u,v,w;};
struct query{int u,v,d,ord;};
int f[maxn],n,m,r;
query q[maxm];
edge e[maxm];
int ord[maxm];
bool ans[maxm];

bool compe(edge  a,edge  b){return a.w>b.w ;}
bool compq(query a,query b){return a.d>b.d ;}

int Find(int x){ return f[x]==x ? x : f[x]=Find(f[x]);}
int Union(int x,int y){
    int fx = Find(x),fy = Find(y);
    if(fx!=fy){
        f[fx]=fy;
    }
    return 0;
} 


int main()
{
    int caseNuym = 1;
    while(scanf("%d%d%d",&n,&m,&r)!=EOF) {
        for(int i=1;i<=n;i++)f[i]=i;
        memset(ans,0,sizeof(ans));
        
        for(int i=0;i<m;i++)
            scanf("%d%d%d",&e[i].u,&e[i].v,&e[i].w);
        for(int i=0;i<r;i++){
            scanf("%d%d%d",&q[i].u,&q[i].v,&q[i].d);
            q[i].ord = i;
        }
        
        std::sort(e,e+m,compe);
        std::sort(q,q+r,compq);
        
        int head = 0;

        for(int i=0;i<r;i++){
            for(;(e[head].w>=q[i].d)&&(head<m);head++)
                Union(e[head].u,e[head].v);
            if(Find(q[i].u)==Find(q[i].v))
                ans[q[i].ord] = 1;
            else 
                ans[q[i].ord] = 0;
        }
    printf("Case %d:\n",caseNuym ); 
    for (int i=0;i<r;++i)
    if (ans[i]) puts("yes"); else puts("no");
     ++caseNuym ; 
    }
}
```
- - -
*F - Remember the Word*

动态规划f[i]表示单词的i前缀串划分为集合中字符的方案数
- 转移方程  f[i]=sigma(f[i-s[j].len]) s[j]是word[0..i]一个后缀
- 边界        f[0]=1

实际上写的时候考虑的是向后更新会比较方便。

- f[i+s[j].len]+=f[i] (s[j]是word[i..end]一个前缀）
利用字典树找到一个状态的所有后继。

```
#include <cstdio>
#include <cstring>
#include <queue>

const long long maxnode    = 4010*100;
const long long SIGMA_SIZE = 26;
const long long maxlen     = 100;
const long long maxn       = 300010;
const long long MOD        = 20071027;

using namespace std;

char s[maxn];
int f[maxn];

struct Trie
{
    int ch[maxnode][SIGMA_SIZE];
    int val[maxnode];
    int size;
    Trie(){clear();}
    void clear(){
        size=1;
        memset(ch[0],0,sizeof(ch[0]));
        memset(val,0,sizeof(val));
    }
    int index(char t){return t-'a';};
    int buildNewNode(int u){
        memset(ch[size],0,sizeof(ch[size]));
        val[size]=u;
        return size++;
    }
    void insert(char *s){
        int u = 0;
        for(;*s;s++){
            int c = index(*s);//忘记调用idx()WA了好几次
            if(!ch[u][c])
                ch[u][c]=buildNewNode(0);
            u=ch[u][c];
        }
        val[u]=1;//val[u]=v
    }
    void find(int i,char *s){
        int u=0,len=0;
        for(;*s;s++){
            int c = index(*s);//忘记调用idx()WA了好几次
            if(ch[u][c]){
                len++;
                u=ch[u][c];
                f[i+len]=(val[u]*f[i]+f[i+len])%MOD;
                //if(val[u]){find an end }
            }
            else
                break;
        }

    }
};

Trie trie;

int main()
{
    int CaseNum = 0;
    while(scanf("%s",s)!=EOF){
        int n,len=strlen(s);
        memset(f,0,sizeof(f));f[0]=1;
        trie.clear();

        scanf("%d",&n);
        for(int i=0;i<n;i++){
            char key[110];
            scanf("%s",key);
            trie.insert(key);
        }

        for(int i=1;i<=len;i++)
            trie.find(i-1,s+i-1);

        printf("Case %d: %d\n",++CaseNum,f[len]);

    }
    return 0;
}
```
- - -
*I - Cake slicing*

4D/1D的动态规划，就是枚举这一刀是横切还是竖切，记忆化搜索。
```C++
#include <iostream>
#include <cstring>
#include <cstdio>

using namespace std;

const int maxn   = 20+10;
const int maxInt = 0x3f3f3f3f;

int A[maxn][maxn],f[maxn][maxn][maxn][maxn];

int sum(int x,int y,int a,int b)
    {return A[a][b]-A[x-1][b]-A[a][y-1]+A[x-1][y-1];}

int dfs(int x,int y,int a,int b){
    int S = sum(x,y,a,b),&ans=f[x][y][a][b];
    if(S==0)return maxInt;
    if(S==1)return 0;
    if(ans) return ans;
    ans=maxInt;
    for (int i = x; i < a; ++i)
    {
        ans = min(dfs(x,y,i,b)+dfs(i+1,y,a,b)+b-y+1,ans);
    }
    for (int i = x; i < a; ++i)
    {
        ans = min(dfs(x,y,a,i)+dfs(x,i+1,a,b)+a-x+1,ans);
    }
    return ans;
}

int main(int argc, char const *argv[])
{
    int n,m,k,CaseNum=0;
    while(cin>>n>>m>>k){
        memset(A,0,sizeof(A));
        memset(f,0,sizeof(f));
        for(int i=0;i<k;i++){
            int x,y;
            cin>>x>>y;
            A[x][y]=1;
        }
        for (int i = 1; i <= n; ++i)
        {
            for (int j = 1; j <= m; ++j)
            {
                A[i][j]+=A[i-1][j]+A[i][j-1]-A[i-1][j-1];
            }
        }
        int ans = dfs(1,1,n,n);
        cout<<"Case "<<++CaseNum<<": "<<ans<<endl;
    }
    return 0;
}
```