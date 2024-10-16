# 「NOIP 2021」数列

## 前言

&emsp;&emsp;马上 CSP-S 了，来感受下 t2 的难度。

&emsp;&emsp;以笔者现在的水平，还是无法在考场上做出 t2 这种 dp，CSP-S 2021 的题目也一样。

&emsp;&emsp;不管怎样，希望能考出个好成绩罢。

## Solution

&emsp;&emsp;计数类题目，考虑 dp。

&emsp;&emsp;我们发现 $S$ 可以从低到高逐位考虑，无论当前这位怎么填都不会影响之前位的状态。满足无后效性。因此我们**在 $S$ 上从低到高进行 dp**。

&emsp;&emsp;设计出 dp 状态 $f[i, j, k, l]$ 表示当前考虑到第 $i$ 位，$a$ 填到第 $j$ 位，目前已经有 $k$ 个 1，要往下一位进位的数为 $l$ 时的权值和。

&emsp;&emsp;考虑当前状态能更新后续哪些状态。我们枚举下一位填 $t$ 个，然后可以发现下一位为 $t + l \bmod 2$，并且会再往下一位的下一位进位 $\lfloor \frac{t + l}{2} \rfloor$ 个 1。于是当前状态能更新的状态为：

$$
f[i + 1, j + t, k + (t + l) \bmod 2, \lfloor \frac{t + l}{2} \rfloor]
$$

&emsp;&emsp;再考虑下当前状态会对后续状态产生的贡献。由于乘法分配律我们可以直接累乘一个 $v\_i^t$，又因为在剩下的 $n - j$ 个位置中任意选 $t$ 个都是不同的方案，所以可以再乘一个 $\operatorname{C}^t_{n - j}$，得到最后的贡献为：

$$
f[i, j, k, l] \times v\_j^t \times \operatorname{C}^t_{n - j}
$$

&emsp;&emsp;预处理一下组合数即可。

&emsp;&emsp;在统计答案时，我们枚举最后两维状态，再加上特判进位的数量是否合法，即可通过本题。

## Code

```cpp
#include <cstdio>
using namespace std;

const int MAXN = 35, MAXM = 110;
const unsigned int MOD = 998244353;

typedef long long ll;

ll C[MAXN][MAXN];
ll f[MAXM][MAXN][MAXN][MAXN];
int v[MAXM];

inline int popcnt(unsigned int x)
{
    int ans = 0;
    while (x) {
        ans++;
        x -= x & -x;
    }
    return ans;
}

int main()
{
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    // freopen("out", "w", stdout);
#endif
    int n, m, maxk;
    scanf("%d%d%d", &n, &m, &maxk);
    for (int i = 0; i <= m; i++) {
        scanf("%d", &v[i]);
    }

    C[0][0] = 1;
    for (int i = 1; i <= n; i++) {
        C[i][0] = 1;
        for (int j = 1; j <= i; j++) {
            C[i][j] = C[i - 1][j - 1] + C[i - 1][j];
            C[i][j] %= MOD;
        }
    }

    f[0][0][0][0] = 1;
    for (int i = 0; i <= m; i++) {
        for (int j = 0; j <= n; j++) {
            for (int k = 0; k <= maxk; k++) {
                for (int l = 0; l <= n >> 1; l++) {
                    ll base = 1;
                    for (int t = 0; t <= n - j; t++) {
                        (f[i + 1][j + t][k + ((t + l) & 1)][(t + l) >> 1]
                            += f[i][j][k][l] * base % MOD * C[n - j][t] % MOD) %= MOD;
                        base = base * v[i] % MOD;
                    }
                }
            }
        }
    }

    ll ans = 0;
    for (int k = 0; k <= maxk; k++) {
        for (int l = 0; l <= n >> 1; l++) {
            if (k + popcnt(l) <= maxk) {
                (ans += f[m + 1][n][k][l]) %= MOD;
            }
        }
    }

    printf("%lld\n", ans);

    return 0;
}
```