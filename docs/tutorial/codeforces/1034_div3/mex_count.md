# MEX Count

## 题目大意

给一个整数序列 $a_1, a_2, \ldots, a_n$，对所有 $0 \leq k \leq n$，问在原序列中去掉 $k$ 个数后，$\operatorname{mex}\{a\}$ 有多少可能的值。

数据范围：$1\leq n \leq 2 \times 10^5, 0 \leq a_i \leq n$。

## 解题思路

记 $c(i)$ 表示数字 $i$ 在数列 $a$ 中出现的次数，$p$ 为原数列 $\operatorname{mex}$ 值。

考虑任意的 $k$。对所有小于 $p$ 的 $p$ 个数，若有 $0 \leq i \leq p$ 且 $c(i) \leq k$，则我们可以通过去掉所有的 $i$ 来改变 $\operatorname{mex}$ 的值为 $i$。若此时 $k$ 个要去掉的数没有消耗完，则我们可以去掉大于 $p$ 的数和所有 $c(i) > 1$ 的数中多出的部分，这些改变不会影响结果。

由此我们可以计算所有大于 $p$ 的数和所有 $c(i) > 1$ 的数中多出的部分，记为 $s$，那么答案将由 $k$ 与 $s, c(i)$ 之间的大小关系决定。

当 $k \leq s$ 时，所有 $0\leq i \leq p$ 且 $c(i) \leq k$ 的数都可以成为答案。除此之外还可以选择只去掉 $s$ 中的数，从而原数列的 $\operatorname{mex}$ 也可以成为答案。

当 $k > s$ 时，我们先去掉所有 $s$ 中的数，再考虑剩下的数，显然此时有 $p - (k - s)$ 种答案。

## 参考代码

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
using namespace std;

const int MAXN = 2e5 + 10;
int v[MAXN];
struct pp {
    int a, c;
} p[MAXN];
int u[MAXN];

int main()
{
#ifndef ONLINE_JUDGE
    freopen("./in", "r", stdin);
#endif
    ios_base::sync_with_stdio(false);
    cin.tie(nullptr), cout.tie(nullptr);
    int t;
    cin >> t;
    while (t--) {
        int n;
        cin >> n;
        memset(v, 0, sizeof(int) * (n + 1));
        memset(u, 0, sizeof(int) * (n + 1));
        for (int i = 1; i <= n; i++) {
            int t;
            cin >> t;
            v[t]++;
        }
        int p = n + 1, s = 0;
        for (int i = 0; i <= n; i++) {
            if (v[i] && i < p) u[v[i]]++;
            s += max(v[i] - 1 + (i > p), 0);
            if (v[i] == 0 && p == n + 1) p = i; 
        }
        cout << 1 << ' ';
        for (int i = 1, t = 0; i <= n; i++) {
            t += u[i];
            if (i <= s) cout << t + 1 << ' ';
            else cout << max(p + 1 - i + s, 1) << ' ';
        }
        cout << '\n';
    }
    return 0;
}
```