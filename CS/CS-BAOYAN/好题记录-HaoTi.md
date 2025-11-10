### 货币系统 [P5020 [NOIP 2018 提高组] 货币系统 - 洛谷](https://www.luogu.com.cn/problem/solution/P5020)

这道题起源于学弟问我的一道题：
![4382833951acffc44bb8c0ddb5af0837.jpg|400](https://kold.oss-cn-shanghai.aliyuncs.com/4382833951acffc44bb8c0ddb5af0837.jpg)
```cpp
int solve_binary(int n)
{
    // 计算用二进制表示法需要多少种面值的硬币
    int value = 1;  // 当前位的面值：1, 2, 4, 8...
   	int ans=0; 
    while(n) {
        if(n & 1) {  // 如果当前位是1
			ans += value *
        }
        n = n >> 1;  // 右移一位
        value <<= 1;  // 面值翻倍
    }
    return count;
}
```

感觉还是挺有趣的. 洛谷这道题其实是这道题的强化
#TODO 


这里最重要一个 **insight** 就是： 你有一个货币系统，面值用数组给出 `int a[]`
怎么证明可以去掉其中的某些金额，从而化简呢？


- **首先**，可以粗略想一下：我们应该尽可能和第一个货币系统 $(n,a)$ 保持一致。勿增实体。我们猜测 $B=(m,b)$ 是 $A=(n,a)$ 货币系统的一个子集，即 $B\subset A$ 或者 $B=A$ 
- 引理 1：即 $B\subset A$ 或者 $B=A$ 
	- 证明：
		- **反证法**。如果 $x \in B,x\not\in A$ （这里指的是数组中的元素），则一定有 $A$ 中的某些元素 $a_{1},a_{2},\dots ,a_{i}$ 可以组成 $x$（因为 $A$ 和 $B$ 货币系统等价的假设 ）
		- 则这些元素，$B$ 一定都可以表示（同样是货币系统等价的假设）
		- 则去掉 $x$，$B$ 能表示的货币一定仍然和 $A$ 等价，去掉 $x$ 将使得 $B$ 元素更少，故原来的 $B$ 不是最优的，矛盾。
- 引理 2：最优的 $B$ 中的元素不可以互相表示。
	- **显然**，可以用其他元素表示的元素必然可以去掉而不影响张成的**货币集合**
- **最终解**：
	- 既然 $B$ 是 $A$ 的一个子集，且 $B$ 中的元素不可以互相表示，那么我们只需要删除 $A$ 中的可以互相表示的元素，剩下的元素个数就是我们要的结果了。
	- **互相表示**：
		- **可以看做一个完全背包问题**
		- 金额 $j$ 能被表示记录为 $f[j]$，则 $f[j]=true,\text{if } f[j-a[i]],\exists i$ 
		- 真正更新的时候，我们可以从小到大遍历金额。（因为大金额能被小金额组成，这需要一个排序），然后每轮更新，更新金额 $f[j]$ 能否被表示的信息。
		- 记得对 $ans$ 做更新。

#### 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
#define MAX_A 105
#define MAX_AI 25005
int solve(int n, int a[]);
int main()
{
	int T;
	scanf("%d", &T);
	for(int i=0;i<T;i++){
		int n=0;
		scanf("%d", &n);
		int a[MAX_A];
		memset(a,0,sizeof(a));
		for(int j=1;j<=n;j++){
			scanf("%d", &a[j]);
		}

		int ans=solve(n,a);
		printf("%d\n",ans);
	}
}

int solve(int n, int a[])
{
	int f[MAX_AI];
	memset(f,0,sizeof(f));

	// not set, f[0] could be 1
	f[0]=1;
	// sort the a, from low to high. because one can only be composited by smaller ones
	sort(a+1, a+n+1);
	
	int ans=n; // we found the one who can be compose by lower ones, and delete it;	
	for(int i=1; i<=n; i++){
		if(f[a[i]])	{
			ans--;
			// already know, jump
			continue;
		}
		// update the f[a[i]],
		// f[a[i]] could be composed by smaller coins + a[i]coin, itdepends on f[j-a[i]];
		for(int j=a[i]; j<= a[n]; j++){
			// if f[j]==0, but some f[j-a[i]] could be composed, then it can be composed
			// OR is needed, for if f[j]=1, we don't want to change that;
			f[j]=f[j] || f[j-a[i]];
		}
	}
	return ans;
}
```



### 数学题：学数电学魔怔之后的卡诺图和格雷码闲谈
[1611. 使整数变为 0 的最少操作次数](https://leetcode.cn/problems/minimum-one-bit-operations-to-make-integers-zero/)

给你一个整数 `n`，你需要重复执行多次下述操作将其转换为 `0` ：
- 翻转 `n` 的二进制表示中最右侧位（第 `0` 位）。
- 如果第 `(i-1)` 位为 `1` 且从第 `(i-2)` 位到第 `0` 位都为 `0`，则翻转 `n` 的二进制表示中的第 `i` 位。
返回将 `n` 转换为 `0` 的最小操作次数。
---
**逆向思维**：
从 `0` 出发，我们的操作在干什么？
- `0000 -> 0001`
	- 反转无意义，用第二类操作
- `0001 -> 0011`
- `0011 -> 0010`
- `0010 -> 0110`
- `0110 -> 0111`
- `0101 -> 0100`
- `0100 -> 1100`
- ....


- look familiar? **这就是格雷码序列**
- 阅读[[格雷码]]即可解决这道题，代码简单到难以置信

```cpp
class Solution {
public:
    int minimumOneBitOperations(int n) {
        // to get grey code
        // reverse thinking
        int ans=0;
        while(n){
           ans^=n; 
           n>>=1;
        } 
        return ans;
    }
};
```


## 单调栈
###  [3542. 将所有元素变为 0 的最少操作次数](https://leetcode.cn/problems/minimum-operations-to-convert-all-elements-to-zero/)
#### 分析
> [!Note] 将所有元素变为 `0` 的最少操作次数
> 给你一个大小为 `n` 的 **非负** 整数数组 `nums` 。你的任务是对该数组执行若干次（可能为 0 次）操作，使得 **所有** 元素都变为 0。
> 在一次操作中，你可以选择一个子数组 `[i, j]`（其中 `0 <= i <= j < n`），将该子数组中所有 **最小的非负整数** 的设为 0。
> 返回使整个数组变为 0 所需的**最少**操作次数。
> 一个 **子数组** 是数组中的一段连续元素。

**这里需要有一个洞察**：
- 每次我们应该能尽可能把一段子序列的所有最小的非负整数都设为 0，即先把所有最小变为 0，再把所有次小变为 0
	- 推论：单调增长的序列中，该序列中任意一个元素，不和其他元素在**同一次**操作被变为 `0`, 都要算一次
	- **推论**：如果两个数之间有一个**更小的数**隔开，则需要额外的一次操作。

**因此我们可以维护一个单调栈**。如果进栈数 `a` 其 `<` 栈顶，则说明其把栈顶和其他元素隔开。那我们需要弹出栈顶并把次数 `++`。

遍历结束后，栈中就是单调的，每个元素需要一个次数，所以把结果加上栈的大小。

> [!Note] 如果还不清楚
> 有另一个思路：假如你构成了这样一个序列 `12345 2`, 这个 `2` 的加入意味着前面序列的 `2345` 可可以和 `2` 构成一个闭的子序列，对 `23452` 做分治的操作，合并到全序列一定是最优的。
> **在分治的意义**上，我们 `pop` 了 `2345` 并记录次数，实际是完成了这个子序列的计算（新加入的 `2` 先不算，我们可以留到最后一次算）
> 所以，我们维护单调栈的过程，就是不停的完成分治计算每一小块的过程。

#### 代码

```cpp
class Solution {
public:
    int minOperations(vector<int>& nums) {
        vector<int> st;
        int ans = 0;
        for (int a : nums) {
            while (!st.empty() && st.back() > a) {
                ans++;
                st.pop_back();
            }
            if (a == 0) continue;
            if (st.empty() || st.back() < a) {
                st.push_back(a);
            }
        }
        return ans+st.size();
    }
};

```