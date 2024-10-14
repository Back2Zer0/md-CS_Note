# DP前言

下面一系列的线性dp问题更依赖闫氏dp分析法的发挥。

这里记录几条经验：

1. 当状态计算方程中，在遍历时出现了 f[i-1] 的字样，那么数组下标就要从1开始，来防止负数下标和越界。

2. 状态表示，我们划分集合的原则为 “不重不漏”：==不重复，不遗漏==。

   > 当状态属性 要求取 sum 集合的总和时，不重复的原则很重要。
   >
   > 但状态属性取 max/min 集合的最值时，集合间元素发生重复不会影响结果。
   >
   > 举个例子：  1,2,3
   >
   > - 求最值：max = max(max (1,2) , max (2,3) )。 这里2发生了重复，但不影响最大值3的求取。
   >
   > - 求总和：sum = sum(sum (1,2) , sum(2,3) )。 这里2发生了重复，影响了总和的求取（2被加了两次）

3. 状态表示分为三部分：
   - 集合的维度：一维数组 f[i] 能求出答案吗？不能的话就用二维数组f[i] [j] ，还不行就三维。
   - 集合的表示：每个集合，它的意义是什么？这个意义要能概括题目的所有情况。
   - 集合的属性：求sum还是max？属性有时会影响状态计算方程的展开。

4. 一个简单的数学模型：

   > 集合A：a1,a2,a3 ... an

   > 集合B：a1+w,a2+2, ... an+w

   则有：``min(A)+w = min(B) ``

   或 ：``min(A) = min(B) - w``
   
5. C++语言，一秒处理数据大小为 10^8^左右，尽量把复杂度控制在10^7^，最多10^8^

# 线性DP

## 数字三角形



![1680398003594](C:\Users\fancyzzz\AppData\Roaming\Typora\typora-user-images\1680398003594.png)

> DP分析
>
> 状态表示：
>
> 1
>
> 状态转移：

### 朴素做法

```cpp
#include<iostream>
using namespace std ;
const int N = 510 , INF = 1e9;

int f[N][N];
int a[N][N];
int n;

int main(void)
{
    //初始化数据
    cin>>n;
    for(int i=1;i<=n;i++){
        for(int j=1;j<=i;j++){
            scanf("%d",&a[i][j]);
        }
    }
    //dp数组赋初值 (下标从1开始，不会产生越界问题。但下标为0的数据要提前赋初值，也就是INF)
    for(int i=0;i<=n;i++){
        for(int j=0;j<=i+1;j++){ //初值范围是[0,i+1] ，左右端点 0和i+1 都会用到。
            f[i][j] = -INF;
        }
    }
    f[1][1] = a[1][1];  //转移方程的初值。
    //状态转移
    for(int i=2;i<=n;i++){
        for(int j=1;j<=i;j++){
            f[i][j] = max(f[i-1][j]+a[i][j],f[i-1][j-1]+a[i][j]);
        }
    }
    //输出
    int res = -INF;
    for(int i=1;i<=n;i++) res = max(res,f[n][i]);
    cout<<res;
    
}
```

### 更优雅的写法

```cpp
#include <bits/stdc++.h>

using namespace std;

const int N = 510;

int f[N][N];

int n;

int main()
{
    cin >> n;
    for (int i = 1; i <= n; i++)
        for (int j = 1; j <= i; j++)
            cin >> f[i][j];

    for (int i = n - 1; i >= 1; i--)
        for (int j = 1; j <= i; j++)
            f[i][j] = max(f[i + 1][j + 1], f[i + 1][j]) + f[i][j];

    cout << f[1][1] << endl;
}
```

> 该做法是从最下方，向上推。
>
> 朴素做法是从第二层，向下推。

### 一维优化(朴素版本)

```cpp
 #include <iostream>
#include <algorithm>

using namespace std;

const int N = 510, INF .....................................................................
    = -1e9;

int a[N][N];
int f[N];
int n, m;

int main()
{
    cin >> n;

    for (int i = 1; i <= n; i ++)
        for (int j = 1; j <= i; j ++)
		cin >> a[i][j];

    for (int i = 0; i <= n + 1; i ++ ) f[i] = INF;

    f[1] = a[1][1];

    for (int i = 2; i <= n; i ++)
        for (int j = i; j >= 1; j --)
            f[j] = max(f[j] + a[i][j], f[j - 1] + a[i][j]);

    int res = INF;

    for (int i = 1; i <= n; i ++) res = max(res, f[i]);

    cout << res << endl;

    return 0;

}
```



## 最长上升子序列

![1680429535606](C:\Users\fancyzzz\AppData\Roaming\Typora\typora-user-images\1680429535606.png)

### 朴素做法

闫氏dp：	

状态表示： f[i] ,  1.集合：所有以第i个数字结尾的上升子序列

​	  					2.属性：max

状态计算：（注意，本题dp未用到 i-1 的递增公式，故 i 的下标从0开始）

​				分析集合：  （ 0 | 1 | 2 | 3 |...| i-1）

> 在该集合中，题目要求数字大小逐渐上升，即 An-1< An 。
>
> 如果出现An-1>=An的情况，我们不会把它包含在集合中去，换言之，**所求的都是满足题意的某个集合的最大值。**

对于形如 AiAj 的上升子序列，可知f[j]的最长上升子序列，即f[i]的最长上升子序列+1。

故计算方程：f[ i ] = max(  f[ j ] +1 ), j = 0,1,2,3... i-1  且 ( a[i]>a[j] ) 。

### 二分（数据加强版）

![1680429706279](C:\Users\fancyzzz\AppData\Roaming\Typora\typora-user-images\1680429706279.png)

思路：

在朴素做法的基础上，我们需要追加一个集合,再次分析，来达到优化剪枝的效果。

> 分析：以 3 1 4 8 5 为例。 如果一个子序列的长度为1，那该序列的末尾值可以取 1 ，也可以取 8。
>
> 如果要最长子序列，那末尾值就必须最小，也就是min(1,8) = 1 ，取1不取8。

为了实现该功能：

追加一个数组q[i] ,这个数组表示长度为i时，结尾最小的序列值为q[i]。所以该数组有效值的长度，就是最长上升子序列的长度。

>遍历a数组里面的数，然后在q（这个数组的长度就是答案）这个数组里面查找是否存在一个大于且最靠近他的数，
>若果不存在话，说明这个数是在q所有的数中是最大的
>（这时下标也已经扫描到q数组的最右边，r这个下标已经定位到q的最右边，将其+1，更新q长度（即：更新答案）和新元素的值），
>若存在（即：该点的值不是上升地），r+1也不会增加q的长度（因为不是上升的情况，所以这里答案不会更新），说明该点最长子序列一定是前面其中的某个答案，只需要将里面地最靠近的那个元素进行更新即可

```cpp
#include <iostream>
#include <algorithm>

using namespace std;

const int N = 100010;

int n;
int a[N] = {7,3,4,5,1,8,3};
int q[N];

int main()
{
    n=7;
    int len = 0;
    for (int i = 0; i < n; i ++ )
    {
        int l = 0, r = len;
        while (l < r)
        {
  //  q里有无大于当前a[i]:  存在原地，不存在的话更新 
  //                        不存在的话就是 r = 0
  //                         存在的话就是 r != 0 ，即大于1的线段，更新len 
  //                 （当然，如果有多个大于a[i]的数，则选择最靠近a[i]的。） 
            int mid = l + r + 1 >> 1;
            if (q[mid] < a[i]) l = mid;// check()  
            else r = mid - 1;   
        }
        len = max(len, r + 1);
        q[r + 1] = a[i];   
        printf("i = %d ,r = %d ,a[i]=%d , len = %d;  ",i,r,a[i],len);
        for(int i=0;i<n;i++) cout<<q[i]<<" ";
        cout<<endl;
    }    ///a[N] = {7,3,4,5,1,8,3};
	for(int i =0;i<n;i++) printf("q%d=%d\n",i,q[i]);
	cout<<endl; 
    printf("%d\n", len);

    return 0;
}
```

> 注：此代码并非最终题解，因为保留了调试部分的输出语句和注释。

结果展示：

![1680432840390](C:\Users\fancyzzz\AppData\Roaming\Typora\typora-user-images\1680432840390.png)

## 最长公共子序列

![1680432897126](C:\Users\fancyzzz\AppData\Roaming\Typora\typora-user-images\1680432897126.png)

闫氏dp：	

状态表示： f[i] [j] ,  1.集合：字符串A的前i个字符，和字符串B的前j个字符的公共子序列

​	  					    2.属性：max

状态计算：

> 以两个字符串的末位字符，即A[i] 和 B[j] 划分，有四种情况：
>
> - 00 ：最长公共子序列的末尾字符，即不是A[i],也不是B[j] 。所以很自然，有 f(i,j) = f(i-1,j-1)
>
> - 11 :  最长公共子序列的末尾字符,即是A[i],又是B[j]。所以必然有A[i] == B[j] 且f(i,j) = f(i-1,j-1)+1
>
> - 01 :  末位不是A[i] ，但却是B[j]。经下面的分析，此种情况为 f(i,j) = f(i-1,j)
>
>   这里我们会想当然认为，f(i,j) = f(i-1,j)。但不是这样的。**因为f(i-1,j) 同样包含了 不选择B[j]的情况**，这与我们的集合情况不符。但由于此题求的是最大值，根据前言原则2，所以重复不影响结果。
>
> - 10 : 与01同理，重复无所谓，全覆盖就行。此种情 况为 f(i,j) = f(i,j-1)

​				分析集合：  （ 00|01|10|11）

> ​			集合00是被包含在01和10的集合中的。所以在状态转移方程中省去00。

状态转移方程：

``            f[i][j] = max(f[i - 1][j], f[i][j - 1]);
if (a[i] == b[j]) f[i][j] = max(f[i][j], f[i - 1][j - 1] + 1);``

### 朴素做法

枚举i，j —— 状态转移

```cpp
#include<iostream>
using namespace std ;
const int N = 1010;
int n,m;
char a[N],b[N];
int f[N][N];
int main(void)
{
    cin>>n>>m>>(a+1)>>(b+1); //下标从1开始
    for(int i=1;i<=n;i++){
        for(int j=1;j<=m;j++){
            f[i][j] = max(f[i-1][j],f[i][j-1]);
            if(a[i]==b[j]) f[i][j] = max(f[i][j],f[i-1][j-1]+1);
        }
    }
    printf("%d",f[n][m]);
    
}
```

## 最短编辑距离

![1680487949436](C:\Users\fancyzzz\AppData\Roaming\Typora\typora-user-images\1680487949436.png)

### 朴素做法

闫氏dp：

状态表示：f(i,j) 

​	1.集合：将A[1~i] 操作后与B[1~j]相等的操作方式 的集合

​	2.属性：min

状态计算：

( 删 | 增 | 改  )

- 删：删除之后匹配。删除多余的A[i]，即前面的最小值+1即现在的最小值。则有``f(i,j ) = f(i-1,j)+1`` ;

- 增：增加之后匹配。即A增加的值恰好是B[j] ,则有``f(i,j) = f(i,j-1)+1`` ;

- 改：更改（末尾值A[i])之后匹配。即要保证A[1~i-1] 和 B[1~j-1]是匹配的，再去改末位。

  则有``f(i,j) = f(i-1,j-1)+1`` 。

  > 注意：这一步要判断。当A[1~i-1]与B[1~j-1]匹配时，如果A[i]==B[j] 那就不需要再改末位，所以此时情况是f(i,j) = f(i-1,j-1)

代码：

```cpp
#include<iostream>
using namespace std ;
const int N = 1010;
int f[N][N] ; //dp数组要建好
char A[N],B[N];

int main(void)
{
    int n,m;
    scanf("%d%s",&n,A+1);
    scanf("%d%s",&m,B+1);
    //dp前的初始化
    for(int i=0;i<=m;i++) f[0][i] = i; //当A的字符串是空，B的字符串长度为i，操作数即i（第i步增加B[i]），i<=m B的长度
    for(int j=0;j<=n;j++) f[j][0] = j; //当B的字符串是空，A的字符串长度为i，操作数即j (第j步删除A[i]) ，j<=n A的长度
    for(int i=1;i<=n;i++){
        for(int j=1;j<=m;j++){
            //增删
            f[i][j] = min(f[i-1][j]+1,f[i][j-1]+1);
            //改要判断
            if(A[i]==B[j]) f[i][j] = min(f[i][j],f[i-1][j-1]);
            else f[i][j] = min(f[i][j],f[i-1][j-1]+1);
        }
    }
    //输出结果：把A的前n个字母变成B的前m个字母
    cout<<f[n][m];
}
```

## 编辑距离

![1680490525577](C:\Users\fancyzzz\AppData\Roaming\Typora\typora-user-images\1680490525577.png)



> **时间复杂度** ：n个字符串 , 每个字符串有 m次询问 n<=1000,m<=1000
> 		             字符串长度为10，比对时是n^2 ,即10^2 = 100
>
> 所以复杂度为： 1000 * 1000 * 100 = 10^8 ,  有点紧。

给定 n个字符串，和当前字符串进行比较，求边界距离。

> 关键点在于求边界距离
>
> 边界距离的求法，可参考上一题“最短编辑距离“

### 朴素做法

```cpp
#include<iostream>
#include<cstring>
using namespace std ;
const int N = 1010;
const int M = 15; //10
int f[N][N]; //dp数组
char str[N][M];//字符串数组，存储询问

//这里是最短编辑距离的算法。
int edit_distance(char a[],char b[]){
    
    int la = strlen(a + 1), lb = strlen(b + 1);

    for (int i = 0; i <= lb; i ++ ) f[0][i] = i;
    for (int i = 0; i <= la; i ++ ) f[i][0] = i;

    for (int i = 1; i <= la; i ++ )
        for (int j = 1; j <= lb; j ++ )
        {
            f[i][j] = min(f[i - 1][j] + 1, f[i][j - 1] + 1);
            f[i][j] = min(f[i][j], f[i - 1][j - 1] + (a[i] != b[j]));
        }

    return f[la][lb];
}

int main(void)
{
    int n,m;
    cin>>n>>m;
    for(int i=0;i<n;i++){
        scanf("%s",str[i]+1);
    }
    while(m--){  //开始询问：求给定 n 个字符串中，谁可以在上限操作次数内变成询问给出的字符串。
        char s[M];
        int limit; //上限次数
        scanf("%s%d",(s+1),&limit);
        
        int res = 0;
        for(int i=0;i<n;i++)  //遍历“给定字符串”
            if(edit_distance(str[i],s) <= limit ) //在边界范围内
                res++;
        
        cout<<res<<endl;
    }
    
    return 0;
}
```

