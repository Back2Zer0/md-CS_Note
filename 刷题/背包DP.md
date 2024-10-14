# B递归问题特性

>①问题有最优子结构：问题存在最优解，且与其子问题最优解重合
>
>②无后效：前后状态值只和值本身有关，和问题无关。
>
>解决思路：
>
>①将原问题分解为子问题
>
>②确定状态
>
>③确定初始状态值
>
>④确定状态转移方程（由最优子结构推到其父结构，直到目标状态）

# 四个背包问题

## 01背包

> 题目：
>
> 有 N 件物品和一个容量是 V 的背包。每件物品只能使用一次。
>
> 第 i 件物品的体积是 vi，价值是 wi。
>
> 求解将哪些物品装入背包，可使这些物品的总体积不超过背包容量，且总价值最大。
> 输出最大价值。
>
> #### 输入格式
>
> 第一行两个整数，N，V，用空格隔开，分别表示物品数量和背包容积。
>
> 接下来有 N 行，每行两个整数 vi,wi，用空格隔开，分别表示第 i 件物品的体积和价值。
>
> #### 输出格式
>
> 输出一个整数，表示最大价值。
>
> #### 数据范围
>
> 0<N,V≤1000
> 0<vi,wi≤1000

### **朴素做法**

既然是动态规划，我们就要找到 i 和 i-1 之间的关系嘛。观察会发现，

对于第 i 件物品，只有选或不选两种选择：

- 不选：``f[i][j] = f[i-1][j]``  即最优解和前 i-1 个物品的情况一样。
- 选：``f[i][j] = f[i-1][j-v[i]]+w[i]`` 即最优解包含了选择第 i 个物品的情况。

这样的话dp的关键：==状态转移方程== 就成型了：

``f[i][j] = max(f[i-1][j],f[i-1][j-v[i]]+w[i])``

暴力dp，因为有两个限定条件 ： 前N件物品和前M个重量，所以空间设为二维数组。

建立一个二维数组 N [ i ] [ j ]，含义为前i件物品和j重量时的最优解。

> 这里还需要两个一维数组 v [n] 和 w [n] 来分别存储第 i 个物品的 体积和价值。

```cpp
#include<iostream>
using namespace std ;

const int N = 1010;
int a[N][N];
int v[N],w[N]; 
int n,m;

int dp()
{
    for(int i=1;i<=n;i++){
        for(int j = 1;j<=m;j++){
            a[i][j]=a[i-1][j];
            if(j>=v[i])
                a[i][j] = max(a[i][j],a[i-1][j-v[i]]+w[i]);
        }
    }
    return a[n][m];
}
int main(void)
{
    cin>>n>>m;
    for(int i = 1;i<=n;i++){
        cin>>v[i]>>w[i];
    }
    cout << dp();
    return 0;
}
```



### **空间优化（一维数组）**

两个关键点：

1. 优化原因：朴素做法中，二维数组只用到了 ``a[i][j]`` 和 ``a[i-1][j]`` ,这里可以优化一下，利用一个一维数组实现状态更新。

   > 你可以理解为二维数组 ``a[N][N]`` 变成了 ``a[1][N]``
   
2. 倒序更新：使用一维数组，则必然要在更新数据时使用倒序。

   > 1.正序为啥不行？
   >
   > 二维状态转移方程为 ``a[i][j] = max(a[i][j],a[i-1][j-v[i]+w[i])``
   >
   > 一维数组，正序更新则变为 ``a[i][j] = max(a[i][j],a[i][j-v[i]+w[i])``
   >
   > **因为没有二维数组记录，导致一维的数据由第i-1层，跳到了第i层。**
   >
   > (你也可以这样理解，正序会导致前i个数据里的某个数据由于满足条件而**重复使用**。！想到了完全背包没有？)
   >
   > 2.倒序为啥行？
   >
   > 计算 ``a[i][j]`` 需要 ``a[i-1][j-v[i]]``，而倒序的情况下，a[``j-v[i]``] 恰好是“未被污染的”，我们所需要的数据。

```cpp
#include<iostream>
using namespace std ;

const int N = 1010;
int a[N];
int v[N],w[N]; 
int n,m;

int dp()
{
    for(int i=1;i<=n;i++){
        for(int j = m;j>=v[i];j--){
            a[j] = max(a[j],a[j-v[i]]+w[i]);
        }
    }
    return a[m];
}
int main(void)
{
    cin>>n>>m;
    for(int i = 1;i<=n;i++){
        cin>>v[i]>>w[i];
    }
    cout << dp();
    return 0;
}
```

## 完全背包

> 问题同上，只多了一个条件： **每种物品都有无限件可用。** 



![1679574367514](https://img-blog.csdnimg.cn/de8f3cc7c1644add9ab91ccbb15e583f.png#pic_center)

作者：yxc  链接：https://www.acwing.com/video/945/

> - 请教一个问题，A式中的dp[i−1] [j]，和实际上B式替换后中的dp[i−1] [j−kv]+(k−1)w后面应该还有一项dp[i−1] [j−(k+1)v]+kw,在转换到C式时，这两项怎么处理的。视频中没有看明白。
>
>   答：
>
>   B式仅从数学上来看，好像末尾缺少了一项  dp[i-1] [j-v-kv] + kw 。
>
>   但**换元后要考虑取值范围的变化**。
>
>   因为两者的 j 是一样的，对应了 1⩽k⩽T，让 j 变成 j−v 就等同于让 k 的范围缩小为 0⩽k⩽T−1 了。
>
>   简而言之就是原来的背包容量 j 最多可以承受 T 个 v，而替换成 j−v 后，最多就只能承受 T−1 个 v 了，把式子最后一项化简消去，就成这样了。
>
> - B式最后一项少了一个 w 怎么理解？
>
>   B式的第一项和A式第二项对应，而 w 前的系数也就差 1，直到最后一项。所以B式 + w 才能替换A项。



代码示例：

### 朴素做法

```cpp
   for (int i = 1; i <= n; i++)
        for (int j = 1; j <= m; j++) {
            int t = j / v[i];
            for (int k = 0; k <= t; k++)
                dp[i][j] = max(dp[i][j], dp[i - 1][j - k * v[i]] + k * w[i]);
        }
————————————————
原文链接：https://blog.csdn.net/raelum/article/details/128996521
```

### 一维优化

```cpp
#include <iostream>
#include <algorithm>

using namespace std;

const int N = 1010;

int n, m;
int v[N], w[N];
int f[N];

int main()
{
    cin >> n >> m;
    for (int i = 1; i <= n; i ++ ) cin >> v[i] >> w[i];

    for (int i = 1; i <= n; i ++ )
        for (int j = v[i]; j <= m; j ++ )//正序了
            f[j] = max(f[j], f[j - v[i]] + w[i]);

    cout << f[m] << endl;

    return 0;
}
```

> 番外：
>
> 1.为什么一维背包的 j ，可以从``v[i]``开始，“跳过”``j<v[i]``的情况？
>
> ![img](https://img-blog.csdnimg.cn/62c4c146493f4b858ccf30d736ae60d9.png#pic_center)
> 关键在这里。二维不能从v[i]开始，因为有一个赋值语句为j<v[i]的数据赋初值。
>
> 2.一维的情况下，赋值语句从f[i] [j] = f[i-1] [j] 变成了 f[ j ] = f[ j ] ,两者等价，但右式可以消掉。  判断语句挪到循环里，结束。 
>
> 3.省略的``f[i] = f[i]`` 等价于 ``f[i][j] = f[i-1][j]``
>
> 因为赋值语句里的 f[i] = f[i] ,右值为上一层循环里的 f[i] ，即 i - 1  。
>
> 这个f[j]还没有在第 i 层的循环里被更新过。



## 多重背包

> 多重背包增加的条件： **第 i 种物品最多有 si 件，**(每件体积是 vi，价值是 wi)。 



### 朴素做法

这时需要增加一层循环，即枚举第 i 种物品的 【0,si】件时的最优解，再从最优解中找到更优解。

(注意：一个限制条件是，选择k个物品势必会引起背包体积问题，需要在该循环中增加体积限制条件)

```cpp
#include<iostream>
using namespace std; 

const int N = 110;

int f[N][N];
int v[N],w[N],s[N];

int main(void)
{
    int n,m;
    cin>>n>>m;
    for(int i=1;i<=n;i++){
        cin>>v[i]>>w[i]>>s[i];
    }
    for(int i=1;i<=n;i++){
        for(int j=0;j<=m;j++){      //
            for(int k=0;k<=s[i] && k*v[i]<=j;k++){ // k*v[i]<=j 防止下标为负数，即装不下的情况出现
            f[i][j] = max(f[i][j],f[i-1][j-k*v[i]]+k*w[i]);
            }   
        }
    }
    cout<<f[n][m];
    return 0;
}
```

> k从0开始：k=0 时，包含了不选择第 i 件物品的情况。



### 一维优化

```cpp
#include <bits/stdc++.h>

using namespace std;

const int N = 110;

int v[N],w[N],s[N];

int dp[N];

int main()

{
    int n, m;

    cin >> n >> m;

    for(int i = 1; i <= n; i++)  cin >> v[i] >> w[i] >> s[i];

    for(int i = 1 ;i <= n; i++)
    {
        for(int j = m; j >= v[i] ; j--)   //
        {
            for(int k = 1; k <= s[i] && k * v[i] <= j; k ++)
            {
                dp[j] = max(dp[j], dp[j- k * v[i]] + k * w[i]);
            }
        }
    }

    cout << dp[m] << endl;

    return 0;

}
```

> 这里有两个点：
>
> - 倒序遍历，直到 v[ i ] 。
>
>   同上
>
> - k=1开始
>
>   实际上，k=0开始和k=1开始，结果是一样的。把k=0带入，发现赋值语句变成了：
>
>   ``dp[j] = max(dp[j],dp[j])``
>
>   所以你是否想起了完全背包里被省略的那个 ``dp[j] = dp[j]``?

### 二进制优化

> 数据范围增强了：
>
> 0<N≤1000
> 0<V≤2000
> 0<vi,wi,si≤2000

思路：

这是0—1背包的，原来你解决多重背包是把n份物品分为n个1份，现在是分成2^0 2^1 2^2 2^3 2^4......2^n个，分好后把它们重新看做一个整体，这些整体相加可以等于[1,n]中任何物品==一份份总和==的集合，
简单讲就是 **n = 1+1+1+1 ...(执行n次) = 1 2 4 8 ...（执行log n 次）**

```cpp
#include<iostream>
#include<vector>

using namespace std ;
struct good
{
    int v;
    int w;
};
vector<good> goods;

const int N = 2023;
int f[N];


int main(void)
{
    int n,m;
    cin>>n>>m;
    for(int i = 1;i<=n;i++){
        int v,w,s; //不需要数组，而是转化成二进制数统一存到vector中去
        cin>>v>>w>>s;
        for(int k=1;k<=s;k*=2 )
        {
            s-=k;
            goods.push_back({v*k,w*k});
        }
        if(s>0) goods.push_back({v*s,w*s});
    }
    
    for(auto good:goods){   //不需要 i / n 了。传统笨蛋线性数目已经没用了
        for(int j=m;j>=good.v;j--){
            f[j] = max(f[j],f[j-good.v]+good.w);
        }
    }
    cout<<f[m]<<endl;
    
    return 0;
}
```

## 分组背包

> 有 N 组物品和一个容量是 V 的背包。
>
> 每组物品有若干个，同一组内的物品最多只能选一个。
> 每件物品的体积是 vij，价值是 wij，其中 i 是组号，j 是组内编号。

分组背包实际上是多重背包的变种。多重背包的解决思路是，将``k*v[i]``(k=0,1,2.....) 打包起来，形成v[ i ]集合中的一部分，我们要做的就是在v [ i ] 集合中选择一个最优解。

这不正是**分组择其一**的过程吗？

>结论：我们可以把分组背包看成多重背包
>理由：我们把每组看做一个“物品”，而这个物品我们可以选择0到si个；
>通过从后向前的遍历顺序来确保，我们对组的决策只有一种：要么选这个组，要么不选；
>然后在通过枚举组内的情况，来对组内进行决策：要么选0个，选1个.....；
>
>来自 acwing：@WZ

### 朴素做法

```cpp
#include <iostream>

using namespace std;

const int N = 110;

int n, m;
int dp[N][N];
int v[N][N], w[N][N], s[N];

int main()
{
    cin >> n >> m;
    for(int i = 1; i <= n; i++)
    {
        cin >> s[i];
        for(int j = 1; j <= s[i]; j++)
        {
            cin >> v[i][j] >> w[i][j];
        }
    }

    for(int i = 1; i <= n; i++)
    {
        for(int j = 0; j <= m; j++)
        {
            for(int k = 0; k <= s[i]; k++)
            {
                if(j >= v[i][k])
                    dp[i][j] = max(dp[i][j], dp[i - 1][j - v[i][k]] + w[i][k]);
            }
        }
    }

    cout <<  dp[n][m] << endl;

    return 0;
}

```

### 一维优化

```cpp
#include <iostream>
#include <algorithm>

using namespace std;

const int N = 110;

int n, m;
int v[N][N], w[N][N], s[N];
int f[N];

int main()
{
    cin >> n >> m;

    for (int i = 1; i <= n; i ++ )
    {
        cin >> s[i];
        for (int j = 0; j < s[i]; j ++ )
            cin >> v[i][j] >> w[i][j];
    }

    for (int i = 1; i <= n; i ++ )
        for (int j = m; j >= 0; j -- )
            for (int k = 0; k < s[i]; k ++ )
                if (v[i][k] <= j)
                    f[j] = max(f[j], f[j - v[i][k]] + w[i][k]);

    cout << f[m] << endl;

    return 0;
}

```

### 一维再优化

一个都不选的方案在状态优化成1维的时候就可以省略了，因为本层的f[j]就是上一层的f[j]。

```cpp
#include<iostream>
using namespace std ;
const int N = 110;
int n,m;
int f[N],v[N],w[N];

int main(void)
{
    cin>>n>>m;
    for(int i =0;i<n;i++)
    {
		int s;
        cin>>s;
        for(int j = 0;j<s;j++) cin>>v[j]>>w[j];
        for(int j = m;j>=0;j--)
    		for(int k=0;k<s;k++)
                if(j>=v[k])
                    f[j] = max(f[j],f[j-v[k]]+w[k]);
    }
    cout<<f[m]<<endl;
    return 0;
}
```



## 总结几个关键点

### 正序/逆序问题：

优化成一维数组的情况，只有完全背包会正序遍历 j 。

因为一维数组的情况下，正序遍历如果背包剩余容积足够（``j>v[i]``），会重复装入一个物品，这是完全背包的思路。

逆序遍历则不会有重复的现象，因为前面的数据都为 0 。

> 从左往右更新有脏数据。
>
> 从右往左更新都是 0 。

**小规律：**

-  四种背包的朴素做法，也就是二维状态都是正序，一维都是倒序。

下面是一个很有参考价值的例子和过程模拟。

---

> 例子：假设有3件物品，背包的总体积为10
>
>    物品    体积     价值 
>   i = 1     4       5  
>   i = 2     5       6  
>   i = 3     6       7  
> 因为 f[0][j] 总共0件物品，所以最大价值为 0， 即 f[0][j] == 0 成立

    如果 j 层循环是递增的： 
    for (int i = 1; i <= n; i++) {
        for (int j = v[i]; j <= m; j++) {
            f[j] = max(f[j], f[j - v[i]] + w[i]);
        }
    }
    当还未进入循环时:
    f[0] = 0;  f[1] = 0;  f[2] = 0;  f[3] = 0;  f[4] = 0;  
    f[5] = 0;  f[6] = 0;  f[7] = 0;  f[8] = 0;  f[9] = 0; f[10] = 0;
    
    当进入循环 i == 1 时：
    f[4] = max(f[4], f[0] + 5); 即max(0, 5) = 5; 即f[4] = 5;
    f[5] = max(f[5], f[1] + 5); 即max(0, 5) = 5; 即f[5] = 5;
    f[6] = max(f[6], f[2] + 5); 即max(0, 5) = 5; 即f[6] = 5;
    f[7] = max(f[7], f[3] + 5); 即max(0, 5) = 5; 即f[7] = 5;
    
    重点来了！！！
    f[8] = max(f[8], f[4] + 5); 即max(0, 5 + 5) = 10; 即f[8] = 10;
    这里就已经出错了
    因为此时处于 i == 1 这一层，即物品只有一件，不存在单件物品满足价值为10
    所以已经出错了。
    
    如果 j 层循环是逆序的：
    for (int i = 1; i <= n; i++) {
        for (int j = m; j >= v[i]; j--) {
            f[j] = max(f[j], f[j - v[i]] + w[i]);
        }
    }
模拟过程如下：

    当还未进入循环时:
    f[0] = 0;  f[1] = 0;  f[2] = 0;  f[3] = 0;  f[4] = 0;  
    f[5] = 0;  f[6] = 0;  f[7] = 0;  f[8] = 0;  f[9] = 0; f[10] = 0;
    
    当进入循环 i == 1 时：w[i] = 5; v[i] = 4;
    j = 10：f[10] = max(f[10], f[6] + 5); 即max(0, 5) = 5; 即f[10] = 5;
    j = 9 ：f[9] = max(f[9], f[5] + 5); 即max(0, 5) = 5; 即f[9] = 5;
    j = 8 ：f[8] = max(f[8], f[4] + 5); 即max(0, 5) = 5; 即f[8] = 5;
    j = 7 ：f[7] = max(f[7], f[3] + 5); 即max(0, 5) = 5; 即f[7] = 5;
    j = 6 ：f[6] = max(f[6], f[2] + 5); 即max(0, 5) = 5; 即f[6] = 5;
    j = 5 ：f[5] = max(f[5], f[1] + 5); 即max(0, 5) = 5; 即f[5] = 5;
    j = 4 ：f[6] = max(f[4], f[0] + 5); 即max(0, 5) = 5; 即f[4] = 5;
    
    当进入循环 i == 2 时：w[i] = 6; v[i] = 5; 
    j = 10：f[10] = max(f[10], f[5] + 6); 即max(5, 11) = 11; 即f[10] = 11;
    j = 9 ：f[9] = max(f[9], f[4] + 6); 即max(5, 11) = 5; 即f[9] = 11;
    j = 8 ：f[8] = max(f[8], f[3] + 6); 即max(5, 6) = 6; 即f[8] = 6;
    j = 7 ：f[7] = max(f[7], f[2] + 6); 即max(5, 6) = 6; 即f[7] = 6;
    j = 6 ：f[6] = max(f[6], f[1] + 6); 即max(5, 6) = 6; 即f[6] = 6;
    j = 5 ：f[5] = max(f[5], f[0] + 6); 即max(5, 6) = 6; 即f[5] = 6;
    
    当进入循环 i == 3 时: w[i] = 7; v[i] = 6; 
    j = 10：f[10] = max(f[10], f[4] + 7); 即max(11, 12) = 12; 即f[10] = 12;
    j = 9 ：f[9] = max(f[9], f[3] + 6); 即max(11, 6) = 11; 即f[9] = 11;
    j = 8 ：f[8] = max(f[8], f[2] + 6); 即max(6, 6) = 6; 即f[8] = 6;
    j = 7 ：f[7] = max(f[7], f[1] + 6); 即max(6, 6) = 6; 即f[7] = 6;
    j = 6 ：f[6] = max(f[6], f[0] + 6); 即max(6, 6) = 6; 即f[6] = 6;
就模拟一下发现没有错误，即逆序就可以解决这个优化的问题了。

### 从1开始/从v[i]开始

- 从1开始的都是朴素二维数组做法。因为二维数组在j<v[i]时也要有数据，供后续状态转移时使用。

- 一维数组不需要。因为一维的情况下，赋值语句从f[i] [j] = f[i-1] [j] 变成了 f[ j ] = f[ j ] ,两者等价，但右式可以消掉。  判断语句挪到循环里，结束。 

> 同理，省略的``f[i] = f[i]`` 等价于 ``f[i][j] = f[i-1][j]``
>
> 因为赋值语句里的 f[i] = f[i] ,右值为上一层循环里的 f[i] ，即 i - 1  。
>
> 这个f[j]还没有在第 i 层的循环里被更新过。

### k的出现

k在**多重背包、分组背包（特殊多重背包）、完全背包单独的朴素形式**出现。

01背包只有选或不选两种情况，不需要 k 的引入。

完全背包一维形式在循环中已经包含了“重复调用，向上更新”的目的，不需要再引入 k。

> 这一点可以参考 正序/逆序问题。



## 实战演练

### 最大数字（蓝桥杯2022国赛-D）

【问题描述】
给定一个正整数 N。你可以对 N 的任意一位数字执行任意次以下 2 种操作：
1.将该位数字加 1。如果该位数字已经是 9，加 1 之后变成 0。
2.将该位数字减 1。如果该位数字已经是 0，减 1 之后变成 9。
你现在总共可以执行 1 号操作不超过 A 次，2 号操作不超过 B 次。
请问你最大可以将 N 变成多少？
【输入格式】
第一行包含 3 个整数：N, A, B。
【输出格式】
一个整数代表答案。
【样例输入】
123 1 2
【样例输出】
933
【样例说明】
对百位数字执行 2 次 2 号操作，对十位数字执行 1 次 1 号操作。
【评测用例规模与约定】
对于 30% 的数据，1 ≤ N ≤ 100; 0 ≤ A, B ≤ 10
对于 100% 的数据，1 ≤ N ≤ 1017; 0 ≤ A, B ≤ 100

> 有A和B两种选择；
>
> 对于第 i 位数字，
>
> - 选 j 次 A ，相当于一件物品： 花费是 j ，价值是 10^n-1 * num (num:当前位的数字)
>
> - 选 k 次 B，相当于一件物品： 花费是 k，价值是  10^n-1 * num
>
> 数据结构：f[i] [j] [k] : j<=A  , k<=B   。含义为**第 i 位数字拥有 j  次 A 和 k 次 B的最大值。**

状态转移：

```cpp
f[i] [j] [k] = max(f[i] [j] [k] , f[i][j-x][k]);
f[i] [j] [k] = max(f[i] [j] [k] , f [i][j][k-y]);
```

```cpp
#include<iostream>
#include<cmath>
using namespace std;
const int N = 20,M = 110; //N 最大数字位数 ， M 选择最大次数
long long f[N][M][M]; //f(i,j,k)  :  第 n 位 拥有 j 次 选择A 和 k 次 选择B 的最大值

int A,B;
string s;  //char s[N] 也ok 

int getMod(int x,int mod)//取模 
{
	return (x%mod+mod)%mod;
}
int main(void)
{
	cin>>s;
	cin>>A>>B;
	int n = s.size();
	for(int i = 1;i<=n;i++)
	{
		for(int p = 0;p<=9;p++)
		{
			int x = s[i-1] - '0';    //Str 从0开始！ 
            //这里注意前后顺序，从x加到a /从x减到a
			int a = getMod(p-x,10); // 通过选择A从t取到p位的花费 
			int b = getMod(x-p,10); // 通过选择B从t取到p位的花费 
            //枚举当拥有价值 a，b 的情况 
			for(int j=0;j<=A;j++) 
			{
				for(int k=0;k<=B;k++)
				{		//pow:更新到当前值的价值 
					if(j>=a) f[i][j][k] = max(f[i][j][k],f[i-1][j-a][k]+ (long long)pow(10,n-i)*p);
					if(k>=b) f[i][j][k] = max(f[i][j][k],f[i-1][j][k-b]+ (long long)pow(10,n-i)*p);
				}
			}

		}

	}
	cout<<f[n][A][B];
	
	return 0; 
}
```

> 为什么背包里，
>
> ```cpp
> if(j >= x) f[i][j][k] = max(f[i][j][k] , f[i - 1][j - x][k] + f10[n - i] * p) ;
> if(k >= y) f[i][j][k] = max(f[i][j][k] , f[i - 1][j][k - y] + f10[n - i] * p) ;
> ```
>
> 不写成下面的形式？
>
> ```cpp
>  max(f[i-1][j][k] , f[i - 1][j - x][k] + f10[n - i] * p) ;
> //		↑
> ```
>
> 因为 j 和 k 从0开始遍历，已经把不选择A或B的情况包括了。
>
> 这就好比多重背包里，选择 0 个背包（k==0) 时，已经囊括了 ``f[i][j] = f[i-1][j]`` 的情况。
>
> ``f[i-1] [j-k* v[i]]+k*w[i]);`` 在 k = 0 时不正是那种情况？