# Segments Covering

## 题目大意

一条有限长直线被划分为 $m$ 格，给你 $n$ 条线段，每条线段可以覆盖 $[l, r]$ 的格子，其存在的概率为 $p/q$。计算直线上任意一格都恰好被一条线段覆盖的概率。

## 解题思路

考虑 dp。记 $f[m]$ 表示直线上 $1 \sim m$ 的格子都恰好被一条线段覆盖的概率。

初始状态 $f[0] = \prod_{i=1}^{n} (1 - p_i/q_i)$ 表示所有线段都不存在。考虑将某条线段从不存在改为存在，有状态转移：

$$
f[i] = \sum_{r_j = i} f[l_j] \times \frac{q_j}{q_j - p_j} \times \frac{p_j}{q_j}
$$

将线段按照右端点从小到大排序后，逐条线段遍历转移即可，时间复杂度 $\operatorname{O}(n(\log n + \log p))$，瓶颈在排序和求逆元。

## 参考代码

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int MAXN = 2e5 + 10, MOD = 998244353;

inline long long power(long long a, long long b)
{
    long long ans = 1;
    while (b)
    {
        if (b & 1) ans = ans * a % MOD;
        a = a * a % MOD;
        b >>= 1;
    }
    return ans;
}

struct Seg {
    int l, r;
    int p, q;
} a[MAXN];
int n, m;
long long f[MAXN];
long long pr[MAXN], qr[MAXN];

int main()
{
#ifndef ONLINE_JUDGE
    freopen("./in", "r", stdin);
#endif
    ios_base::sync_with_stdio(false);
    cin.tie(nullptr), cout.tie(nullptr);

    cin >> n >> m;
    for (int i = 1; i <= n; i++) {
        cin >> a[i].l >> a[i].r >> a[i].p >> a[i].q;
    }

    sort(a + 1, a + n + 1, [](const Seg &a, const Seg &b) { return (a.r == b.r) ? (a.l < b.l) : (a.r < b.r); });

    f[0] = 1;
    for (int i = 1; i <= n; i++) {
        pr[i] = a[i].p * power(a[i].q, MOD - 2) % MOD;
        qr[i] = (a[i].q - a[i].p) * power(a[i].q, MOD - 2) % MOD;
        f[0] = f[0] * qr[i] % MOD;
    }
    for (int i = 1; i <= n; i++) {
        f[a[i].r] = (f[a[i].r] + (f[a[i].l - 1] * power(qr[i], MOD - 2) % MOD) * pr[i] % MOD) % MOD;
    }

    cout << f[m] << endl;
    return 0;
}
```