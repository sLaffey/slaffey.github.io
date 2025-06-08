# [ABC274E] Booster

## 前言

&emsp;&emsp;当时比赛的时候看这道题可做，但是没打出来，现在过了，写篇题解补偿下（不是

## Solution

&emsp;&emsp;很明显是一道状压题目，相当于吃奶酪的加强版。

&emsp;&emsp;我们考虑将经过的路径压缩成为一个二进制数，在这里我们为了方便进行 dp，将小镇与宝箱都压到一个状态里。也就是说，我们用一个 $n + m$ 位二进制数表示当前所经过的路径。

&emsp;&emsp;这样，我们不难设计出状态 $f[state, i]$ 表示当前走的路径状态为 $state$，当前处于点 $i$ 时走过的最短时间。

&emsp;&emsp;在这里就有一个问题：题目中还有「速度」这个相关量，我们是否需要再开一维，将速度也记录在状态中进行转移呢？

&emsp;&emsp;答案是否定的。仔细挖掘速度的性质可以发现，速度只和当前走过的路径有关，和当前所处的点、当前的最短时间都没有关系。因此我们只需要预处理一个数组 $g[state]$ 表示当前路径状态为 $state$ 时的速度大小，在转移时直接使用即可。

&emsp;&emsp;综合以上讨论，我们不难得到状态转移方程。设一个二进制数 $state$ 将某一位 $1$ 变成 $0$ 后得到的数为 $prev(state)$，则状态转移方程如下：

$$
f[now, i] = \min_{pre \in prev(state)} \lbrace f[pre, j] + dist(i, j) / g[pre] \rbrace
$$

&emsp;&emsp;其中 $j$ 表示 $pre$ 是由 $now$ 将第 $j - 1$ 位变成 $0$ 得到的（这里设最低位为第 $0$ 位）。初始化 $f[0, 0] = 0$，其余为无穷大。

&emsp;&emsp;有了状态转移方程就不难 dp 了。最后一个要注意的地方是，答案应为所有**经过所有城镇但不一定经过所有宝箱**的状态的最小值。

## Code

&emsp;&emsp;在写代码的时候如果用 `cout` 输出，要注意控制精度。默认保留 $5$ 位会挂掉。

```cpp
#include <iostream>
#include <iomanip>
#include <cstring>
#include <cmath>
using namespace std;

typedef long long ll;

const int MAXN = 18, SIZE = 1 << (MAXN - 1);

double f[SIZE][MAXN];
double g[SIZE];
double dist[MAXN][MAXN];
int x[MAXN], y[MAXN];
int n, m, all;

inline int popcnt(int x)
{
    int ans = 0;
    x >>= n;
    while (x) {
        ans++;
        x -= x & -x;
    }
    return ans;
}

inline void dp()
{
    for (int i = 0; i <= all; i++) {
        for (int j = 0; j < i; j++) {
            dist[i][j] = dist[j][i] = sqrt(pow(x[i] - x[j], 2) + pow(y[i] - y[j], 2));
        }
    }

    for (int i = 0; i < (1 << all); i++) {
        g[i] = 1 << popcnt(i);
    }

    memset(f, 0x7f, sizeof f);
    f[0][0] = 0;
    for (int now = 1; now < (1 << all); now++) {
        for (int i = 1; i <= all; i++) {
            if (!((now >> (i - 1)) & 1)) continue;
            int pre = now & (~(1 << (i - 1)));
            for (int j = 0; j <= all; j++) {
                if (i == j || (j && !((pre >> (j - 1)) & 1))) continue;
                f[now][i] = min(f[now][i], f[pre][j] + dist[i][j] / g[pre]);
            }
        }
    }
}

int main()
{
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    // freopen("out", "w", stdout);
#endif
    ios_base::sync_with_stdio(false);
    cin.tie(nullptr);
    
    cin >> n >> m;
    all = n + m;
    for (int i = 1; i <= all; i++) {
        cin >> x[i] >> y[i];
    }

    dp();

    double ans = INFINITY;
    for (int end = 0; end < (1 << m); end++) {
        int st = ((1 << n) - 1) | (end << n);
        for (int i = 1; i <= all; i++) {
            if (!((st >> (i - 1)) & 1)) continue;
            ans = min(ans, f[st][i] + dist[i][0] / g[st]);
        }
    }

    cout << fixed << setprecision(10) << ans << endl;

    return 0;
}
```