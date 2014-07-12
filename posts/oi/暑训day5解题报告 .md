#暑训day5解题报告

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