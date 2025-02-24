# CF2066B White Magic

## 题目大意

给定序列 $\{a_n\}$，找出其最长的子序列 $\{b_m\}$ 满足对任意 $1 \leq i \leq n - 1$，$\min\{b_1, b_2, \ldots, b_i\} \geq \operatorname{mex}\{b_{i + 1}, b_{i + 2}, \ldots, b_m\}$ 成立，并输出其长度。特别地，我们认为所有长度为 $1$ 的数列都是合法答案。

> $\operatorname{mex}$ 运算定义为未出现在数列中的最小自然数。

多组数据评测。每个测试点有 $t$ 组测试数据，每组测试数据给出 $n$ 与 $\{a_n\}$。数据范围为 $1 \leq t \leq 10^4, 1 \leq n \leq 2 \times 10^5, 0 \leq a_i \leq 10^9$。

## 解题思路

不难发现一个事实，若所有 $a_i$ 都不为 $0$，那么对任意的 $i$ 有 $\min\{a_1, a_2, \ldots, a_i\} > 0$ 且 $\operatorname{mex}\{a_{i + 1}, a_{i + 2}, \ldots, a_{n}\} = 0$，则序列 $\{a_n\}$ 满足题意，可以确定答案是 $n$。

如果有一个 $0$ 存在，不妨设 $a_k = 0$，则对任意 $i \geq k$ 有 $\min\{a_1, a_2, \ldots, a_i\} = 0, \operatorname{mex}\{a_{i + 1}, a_{i + 2}, \ldots, a_n\} = 0$，因此题中条件也成立。

如果有两个 $0$ 存在呢？不妨设 $a_l, a_r = 0$，不难发现，对于 $l \leq i < r$，有 $\min\{a_1, a_2, \ldots, a_i\} = 0$，且 $\operatorname{mex}\{a_{i + 1}, a_{i + 2}, \ldots, a_n\} > 0$，必然不满足条件。对于有两个以上的 $0$ 的情况，也可以推导出相同的结论。

于是可以发现，答案中的最长子序列只含最多一个 $0$，因此长度只可能为 $n - c_0$ 或 $n - c_0 + 1$，其中 $c_0$ 为 $0$ 的个数。

序列 $\{a_n\}$ 中不含 $0$ 时，我们可以立即得到答案。当含 $0$ 时，我们只需取最靠左的 $0$ 然后暴力验证即可。不妨取所有 $0$ 中下标最小的一个为 $a_i$，任取一个下标不是最小的为 $a_j$，若将 $a_j$ 算进子序列中时满足性质，那么也会有 $\min\{a_1, a_2, \ldots, a_{i - 1}\} \geq \operatorname{mex}\{a_i, a_{i + 1}, \ldots, a_j, \ldots, a_n\} \geq \operatorname{mex}\{a_i, \ldots, a_n\}$，所以取下标最小的总是最优。

统计后缀 $\operatorname{mex}$ 时，可以利用序列 $\operatorname{mex}$ 的值不会超过长度加一的性质进行优化。

## 参考代码

```cpp
#include <iostream>
#include <cstring>
using namespace std;

const int MAXN = 2e5 + 10;
int a[MAXN];
bool v[MAXN];
int pre[MAXN];

int main()
{
#ifndef ONLINE_JUDGE
    freopen("./in", "r", stdin);
#endif
    ios_base::sync_with_stdio(false);
    int t;
    cin >> t;
    while (t--) {
        int n;
        cin >> n;
        int cnt = 0, l = 0;
        for (int i = 1; i <= n; i++) {
            cin >> a[i];
            if (a[i] == 0) {
                cnt++;
                if (l == 0) {
                    l = i;
                }
            }
        }
        if (cnt == 0) {
            cout << n << endl;
        }
        else {
            memset(v, 0, sizeof(bool) * (n + 2));
            pre[1] = a[1];
            for (int i = 2; i < l; i++) {
                pre[i] = min(pre[i - 1], a[i]);
            }
            int p = 0;
            bool flag = true;
            for (int i = n; i >= 1; i--) {
                if (i < l && pre[i] < p) {
                    flag = false;
                    break;
                }
                if (a[i] > n + 1) continue;
                v[a[i]] = true;
                while (v[p] == true) p++;
            }
            cout << (n - cnt + flag) << endl;
        }
    }
    return 0;
}
```