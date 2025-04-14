# 吉吉餐厅

## 题目大意

有 $N$ 个厨师和 $N/2$ 个灶台（$N$ 为偶数），每个灶台需要分配一个厨师长和一个副厨师长。每个厨师当厨师长时所需要的薪资 $a_i$ 和当副厨师长时所需要的薪资 $b_i$ 不同，且前者总大于后者。一个灶台的厨师长年龄必须大于副厨师长年龄。

现按照年龄从小到大的顺序给出每个厨师要求的薪资，最小化总薪资并输出它的值。

数据范围：$2 \leq N \leq 10^4, 1 \leq b_i < a_i \leq 10^5$。

## 解题思路

考虑 DP。令 $w_i = a_i - b_i$，则原问题等同于在这 $N$ 个数中按上述规则选出 $N/2$ 个并最小化其总和。

考虑按年龄从大到小依次决定是否选择。建立 DP 数组 $f_{i, j}$ 表示第 $i$ 个数及以后的数中选的数和没选的数的差值为 $j$ 时总和的最小值。如此考虑是因为我们实际上不关心选了多少个数没选多少个数，只要差值大于 $0$，当前的数就可以不选，反之则必须选。

考虑当前的数选或不选，得到状态转移方程：

$$
\begin{aligned}
&f_{i, j} = \min\{ f_{i + 1, j + 1}, f_{i + 1, j - 1} + w_i\} \ (j > 0) \\
&f_{i, 0} = f_{i + 1, j + 1}
\end{aligned}
$$

初始化均为正无穷。

由于空间复杂度 $O(N^2)$ 难以接受，采用滚动数组优化。时间复杂度虽然也为 $O(N^2)$，但由于常数较小可以通过。

## 参考代码

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
using namespace std;

const int MAXN = 1e4 + 10;
int w[MAXN];
int f[2][MAXN >> 1];

int main()
{
#ifndef ONLINE_JUDGE
    freopen("./in", "r", stdin);
#endif
    ios_base::sync_with_stdio(false);
    int n;
    cin >> n;
    int ans = 0;
    for (int i = 1; i <= n; i++) {
        int a, b;
        cin >> a >> b;
        ans += b;
        w[i] = a - b;
    }
    memset(f, 0x3f, sizeof f);
    int p = 0;
    f[0][0] = 0;
    for (int i = n; i >= 1; i--) {
        p ^= 1;
        f[p][0] = f[p ^ 1][1];
        for (int j = 1; j <= min(n - i + 1, n >> 1); j++) {
            f[p][j] = min(f[p ^ 1][j - 1] + w[i], f[p ^ 1][j + 1]);
        }
    }
    cout << ans + f[p][0] << endl;
    return 0;
}
```