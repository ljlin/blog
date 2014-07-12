#暑训day5解题报告

##B - Elevator Stopping Plan

学了一下网上找到了[贪心的题解](http://blog.csdn.net/zhanglizhe_cool/article/details/5639257)。二分答案以后贪心验证。第一，在lim时间内自己爬楼梯能到达目的地的话就完全不乘电梯。然后验证其他的楼层。首先为了方便，把楼层从0开始重新标号，用p表示电梯现在停靠的楼层，用cnt表示到达这层之前停的次数。

主要是抽象出三个不等式，来描述一些关系。

         不等式       |  意义  |用途
---------------------|-------|---
4p+10cnt<=lim        |电梯到达p层总共消耗时间不能比lim长       |判false
4h+10cnt+20(h-p)<=lim|对于目的地在p的乘客。电梯在h停，人再下几层|取h最大，计算新的停靠点
4p+10cnt+20(p-h)<=lim|目的地在p的旅客如果能在h下电梯，然后自己向上爬几层在lim之内到达就不必考虑|取p最大，得到新的需要考虑的目的地

```C++
#include <iostream>
#include <cstring>

using namespace std;
const int maxn = 40;

int m,n,A[maxn],ans[maxn],cnt;

bool ok(int lim)
{
    int h,p=lim/20+1;cnt=0;
    while(p<=m){
        while(p<=m && !A[p])p++;
        if(p*4+10*cnt>lim) return 0;
        h=(lim-10*cnt+20*p)/24;
        p=(lim-10*cnt+16*h)/20+1;
        ans[cnt++]=h;
    }
    return 1;
}

int main()
{
    while(cin>>n,n){
        memset(A,0,sizeof(A));
        m=0;
        for(int i=0;i<n;i++){
            int t;
            cin>>t;
            A[t-1]=1;
            m=max(t-1,m);
        }
        int l=0,r=14*m,mid=(l+r)>>1;
        for(;l<r-1;mid=(l+r)>>1)
            if(ok(mid))
                r=mid;
            else
                l=mid;
        ok(r);
        cout<<r<<endl<<cnt;
        for(int i=0;i<cnt;i++)
            cout<<' '<<ans[i]+1;
        cout<<endl;
    }
    return 0;
}

```

##F - Sum of Factorials

01背包只有11个物品

```C++
#include <iostream>
#include <cstring>
using namespace std;

bool f[2000000] ;
int n ;
int a[15];

int main()
{
    a[0]=1;
    for (int i=1;i<=10;++i ) a[i]=a[i-1]*i ;
    memset(f,0,sizeof(f));f[0]=1;

    for (int i=0; i<=10; ++i)
        for (int j=1000000; j>=a[i]; --j)
            f[j]|=f[j-a[i]];

    while(cin>>n,n!=-1)
        cout<<(f[n]?"YES":"NO")<<endl;
}
```