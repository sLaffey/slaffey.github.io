# CF2067F Bitwise Slides

## 题目大意

你现在手上有三个数 $P, Q, R$，给一长度为 $n$ 的正整数数列 $\{a_n\}$，从前到后考虑每一个数，对于当前数字 $a_i$，你可以在 $P, Q, R$ 中任选一个数，并将其赋值为与 $a_i$ 按位异或后的结果。在任意时刻，$P, Q, R$ 都应**不是**两两不同的。你需要统计所有合法的操作数。显然，所有有可能的操作有 $3^n$ 中，因此你只需要输出答案对 $10^9+7$ 取模后的结果。

数据范围为 $1 \leq n \leq 2 \times 10^5, 1 \leq a_i \leq 10^9$。

## 解题思路

不难注意到，如果记 $p_i$ 为前缀异或和，那么始终有 $P \oplus Q \oplus R = p_i$。题目要求至少有一对相同的数，那么这两个数的异或和为 $0$，也就是说最后一个数一定是 $p_i$。由此，状态可以用一个三元组 $(x, x, p_i)$ 表示。于是我们可以设计状态数组 $dp[i, x]$ 表示在对前 $i$ 个数进行操作后，三个数的状态为 $(x, x, p_i)$（不考虑顺序）时的合法操作数。

乍一看，这个状态转移的数量级大得离谱，但是先别急，我们先推导状态转移方程。

考虑当前状态 $(x, x, p_i)$ 能从哪些状态转移，得出两个前置状态 $(x \oplus a_i, x, p_i)$ 和 $(x, x, p_i \oplus a_i)$。

对于 $(x \oplus a_i, x, p_i)$，枚举该状态中哪两个数相等。由于 $a_i > 0$，可以确定 $x \oplus a_i \neq x$。若 $x \oplus a_i = p_i$，则有 $x = p_i \oplus a_i = p_{i - 1}$，于是该状态为 $(p_i, p_{i - 1}, p_i)$，此时有两种选择可以转移到当前状态，对应 $2 \times dp[i - 1][p_i]$。若 $x = p_i$，则该状态为 $(p_{i- 1}, p_i, p_i)$，此时只有一种选择可以转移到当前状态，对应 $dp[i - 1][p_i]$。

对于 $(x, x, p_i \oplus a_i)$，当 $x \neq p_{i - 1}$ 时，只有一种情况可以转移，也就是说 $dp[i,x] = dp[i - 1, x]$；当 $x = p_{i - 1}$ 时，由于此时三个数都一样，有三种情况可以转移。

综上所述，当 $x = p_{i - 1}$ 时，状态 $(p_{i - 1}, p_{i - 1}, p_i)$ 可以由 $(p_i, p_i, p_{i - 1})$ 和 $(p_{i - 1}, p_{i - 1}, p_{i - 1})$ 共五种情况转移；当 $x = p_i$ 时，虽然我们在上面讨论了两次，但是实际上这两次都是同一种状态 $(p_i, p_i, p_{i - 1})$，只有一种转移情况；其他情况可行方案数也都不变。

也就是说，我们每次只需要转移一个数，理论可以做到线性，但是实际上由于需要离散化存储，总复杂度为 $\operatorname{O}(n\log n)$。实现时顺理成章地可以用上 `map` 与滚动数组优化。

## 参考代码

```cpp
#include <iostream>
#include <map>
using namespace std;

const int MAXN = 2e5 + 10, MOD = 1e9 + 7;
int p[MAXN];
map<int, int> dp;

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
        for (int i = 1; i <= n; i++) {
            int a;
            cin >> a;
            p[i] = p[i - 1] ^ a;
        }
        dp.clear();
        dp[0] = 1;
        for (int i = 1; i <= n; i++) {
            dp[p[i - 1]] = ((long long) dp[p[i]] * 2 + (long long) dp[p[i - 1]] * 3) % MOD;
        }
        int ans = 0;
        for (auto a : dp) {
            ans = (ans + a.second) % MOD;
        }
        cout << ans << endl;
    }
    return 0;
}
```