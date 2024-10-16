# 「DP」背包问题

## 前言

&emsp;&emsp;笔者的 DP 实在是太菜了，所以利用国庆来复习下各种背包问题。题目参考 AcWing。

## 01 背包问题

### 基本思路

&emsp;&emsp;01 背包是背包中最简单的模型。我们设计状态 $f[i, j]$ 表示在前 $i$ 个物品中选出若干个物品，并为其分配 $j$ 的空间时，所获得的最大价值。不难得到状态转移方程：

$$
f[i, j] = \max \lbrace f[i - 1, j - v[i]] + w[i], f[i - 1][j] \rbrace
$$

&emsp;&emsp;于是写出代码：

```cpp
#include <cstdio>
using namespace std;

const int& max(const int &a, const int &b) { return a > b ? a : b; }

const int MAXN = 1010;

int f[MAXN][MAXN];
int v[MAXN], w[MAXN];

int main()
{
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
#endif
    int n, m;
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) {
        scanf("%d%d", &v[i], &w[i]);
    }

    for (int i = 1; i <= n; i++) {
        for (int j = 0; j <= m; j++) {
            f[i][j] = max(f[i][j], f[i - 1][j]);
            if (j >= v[i]) {
                f[i][j] = max(f[i][j], f[i - 1][j - v[i]] + w[i]);
            }
        }
    }

    printf("%d\n", f[n][m]);

    return 0;
}
```

&emsp;&emsp;时间复杂度为 $\operatorname{O}(nm)$，足以通过此题。

### 滚动数组

&emsp;&emsp;不过我们能注意到，这道题的状态转移方程中，每一个状态只会由前一个阶段的状态转移过来。利用这个性质，我们可以采用**滚动数组优化**来降低空间复杂度。

&emsp;&emsp;在 01 背包模型中，我们可以尝试将数组缩到只有一维。每次在状态转移时，数组中保存的是上一个阶段的结果，而我们**在更新本阶段状态时，只可能用到上一阶段中空间小于当前状态的状态**。具体地讲，状态 $f[i, j]$ 只可能由 $f[i - 1, k] (k \leq j)$ 转移来。因此**为了保证当前状态用到的是上一阶段的状态，我们在更新当前状态时采取倒序循环**，从后到前地依次更新。这里给出核心代码：

```cpp
    for (int i = 1; i <= n; i++) {
        for (int j = m; j >= v[i]; j--) {
            f[j] = max(f[j], f[j - v[i]] + w[i]);
        }
    }
```

&emsp;&emsp;这样，尽管我们没有降低时间复杂度，却成功将空间复杂度降低至 $\operatorname{O}(m)$。

## 完全背包问题

### 基本思路

&emsp;&emsp;完全背包问题在 01 背包问题的基础上，将物品改为可选无限件。仍旧采取上一题的策略，得到状态转移方程：

$$
f[i, j] = \max \lbrace f[i - 1][j - v[i]] + w[i], f[i][j - v[i]] + w[i], f[i - 1][j] \rbrace
$$

&emsp;&emsp;写出代码：

```cpp
#include <cstdio>
using namespace std;

const int& max(const int &a, const int &b) { return a > b ? a : b; }

const int MAXN = 1010;

int f[MAXN][MAXN];
int v[MAXN], w[MAXN];

int main()
{
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
#endif
    int n, m;
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) {
        scanf("%d%d", &v[i], &w[i]);
    }

    for (int i = 1; i <= n; i++) {
        for (int j = 0; j <= m; j++) {
            f[i][j] = f[i - 1][j];
            if (j >= v[i]) {
                f[i][j] = max(f[i][j], f[i - 1][j - v[i]] + w[i]);
                f[i][j] = max(f[i][j], f[i][j - v[i]] + w[i]);
            }
        }
    }

    printf("%d\n", f[n][m]);

    return 0;
}
```

&emsp;&emsp;时间复杂度与空间复杂度都为 $\operatorname{O}(nm)$。

### 滚动数组

&emsp;&emsp;考虑到完全背包与 01 背包状态转移方程如此神似（只是加了个可以从当前阶段转移），我们再次思考滚动数组优化。

&emsp;&emsp;仔细思考，我们在解决 01 背包的时候，控制循环倒序执行是由于要限制当前阶段的状态只被上一阶段状态更新。那么我们**只需要将循环改为正序，就可以让当前状态也被本阶段状态更新了**。

&emsp;&emsp;核心代码：

```cpp
for (int i = 1; i <= n; i++) {
    for (int j = v[i]; j <= m; j++) {
        f[j] = max(f[j], f[j - v[i]] + w[i]);
    }
}
```

## 多重背包问题

### 基本思路

&emsp;&emsp;多重背包问题限制了物品可选的个数，容易想到一种思路——将每个物品拆分成 $s_i$ 个物品，跑一遍 01 背包即可。

```cpp
#include <cstdio>
using namespace std;

const int& max(const int &a, const int &b) { return a > b ? a : b; }

const int MAXN = 1e4 + 10;

int f[MAXN];
int v[MAXN], w[MAXN];
int top;

int main()
{
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
#endif
    int n, m;
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) {
        int tv, tw, s;
        scanf("%d%d%d", &tv, &tw, &s);

        while (s--) {
            v[++top] = tv;
            w[top] = tw;
        }
    }

    for (int i = 1; i <= top; i++) {
        for (int j = m; j >= v[i]; j--) {
            f[j] = max(f[j], f[j - v[i]] + w[i]);
        }
    }

    printf("%d\n", f[m]);

    return 0;
}
```

&emsp;&emsp;该算法时间复杂度达到 $\operatorname{O}(\sum s_i \times n)$。

### 二进制优化

&emsp;&emsp;实际上我们只需要让拆分出的物品的个数能表示出所有 $0 \sim s_i$ 就可以了。容易想到一个非常类似的问题：[砝码称重](https://www.acwing.com/problem/content/4583/)。笔者在之前的文章[「AcWing 2022 CSP-J 模拟赛」砝码称重](https://laffey.fun/acwing-2022-csp-j)中也给出了可行性的证明。这里不再赘述。

&emsp;&emsp;于是我们优化拆分过程。

```cpp
int t = 1;
while (t <= s) {
    v[++top] = tv * t;
    w[top] = tw * t;
    s -= t;
    t <<= 1;
}

if (!s) continue;
v[++top] = tv * s;
w[top] = tw * s;
```

&emsp;&emsp;这样总数为 $s_i$ 的物品就被分成了 $\lfloor \log_2 s_i \rfloor + 1$ 个。总的复杂度是 $\operatorname{O}(\sum \log s_i \times n)$，较为高效。

### 单调队列优化

&emsp;&emsp;多重背包问题还可以使用单调队列优化到 $\operatorname{O}(nm)$ 的复杂度。笔者水平有限，在这里无法作出讲解。读者可以自行查阅网上资料学习。

## 多维费用背包问题

&emsp;&emsp;直接暴力升维即可。以 AcWing 的二维背包为例。不难写出状态转移方程：

$$
f[i, j, k] = \max \lbrace \max\_{1 \leq j \leq V, 1 \leq k \leq M} \lbrace f[i - 1, j - v[i], k - m[i]] + w[i] \rbrace , f[i  - 1, j, k] \rbrace
$$

&emsp;&emsp;加上滚动数组优化，得到代码：

```cpp
#include <cstdio>
using namespace std;

const int& max(const int &a, const int &b) { return a > b ? a : b; }

const int MAXN = 1e3 + 10, MAXV = 110, MAXM = 110;
int f[MAXM][MAXM];
int v[MAXN], w[MAXN], m[MAXN];

int main()
{
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
#endif
    int n, V, M;
    scanf("%d%d%d", &n, &V, &M);
    for (int i = 1; i <= n; i++) {
        scanf("%d%d%d", &v[i], &m[i], &w[i]);
    }

    for (int i = 1; i <= n; i++) {
        for (int j = V; j >= v[i]; j--) {
            for (int k = M; k >= m[i]; k--) {
                f[j][k] = max(f[j][k], f[j - v[i]][k - m[i]] + w[i]);
            }
        }
    }

    printf("%d\n", f[V][M]);

    return 0;
}
```

## 混合背包问题

&emsp;&emsp;顾名思义，混合背包问题就是一个缝合怪，对于不同的物品分别处理即可。

&emsp;&emsp;将多重背包部分进行二进制拆分转化为 01 背包，完全背包部分特判处理，得到代码：

```cpp
#include <cstdio>
using namespace std;

const int& max(const int &a, const int &b) { return a > b ? a : b; }

const int MAXN = 1e3 + 10;
int f[MAXN];
int w[MAXN << 1], v[MAXN << 1];
bool inf[MAXN << 1];
int top;

int main()
{
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
#endif
    int n, m;
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) {
        int tv, tw, s;
        scanf("%d%d%d", &tv, &tw, &s);

        if (!s) {
            inf[++top] = true;
            v[top] = tv;
            w[top] = tw;
        }
        else if (s == -1) {
            inf[++top] = false;
            v[top] = tv;
            w[top] = tw;
        }
        else {
            int t = 1;
            while (t <= s) {
                v[++top] = tv * t;
                w[top] = tw * t;
                s -= t;
                t <<= 1;
            }

            if (!s) continue;
            v[++top] = tv * s;
            w[top] = tw * s;
        }
    }

    for (int i = 1; i <= top; i++) {
        if (inf[i]) {
            for (int j = v[i]; j <= m; j++) {
                f[j] = max(f[j], f[j - v[i]] + w[i]);
            }
        }
        else {
            for (int j = m; j >= v[i]; j--) {
                f[j] = max(f[j], f[j - v[i]] + w[i]);
            }
        }
    }

    printf("%d\n", f[m]);

    return 0;
}
```

## 分组背包问题

&emsp;&emsp;分组背包对 01 背包做了一个简单变式，我们只需要枚举每组选什么即可。不难得出状态转移方程：

$$
f[i, j] = \max \lbrace \max_{1 \leq k \leq s[i]} \lbrace f[i - 1][j - v[i, k]] + w[i, k] \rbrace , f[i - 1][j] \rbrace
$$

```cpp
#include <cstdio>
using namespace std;

const int& max(const int &a, const int &b) { return a > b ? a : b; }

const int MAXN = 110;
int v[MAXN][MAXN], w[MAXN][MAXN], s[MAXN];
int f[MAXN][MAXN];

int main()
{
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
#endif
    int n, m;
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) {
        scanf("%d", &s[i]);
        for (int j = 1; j <= s[i]; j++) {
            scanf("%d%d", &v[i][j], &w[i][j]);
        }
    }
    
    for (int i = 1; i <= n; i++) {
        for (int j = 0; j <= m; j++) {
            f[i][j] = f[i - 1][j];
            for (int k = 1; k <= s[i]; k++) {
                if (j < v[i][k]) continue;
                f[i][j] = max(f[i][j], f[i - 1][j - v[i][k]] + w[i][k]);
            }
        }
    }
    
    printf("%d\n", f[n][m]);
    
    return 0;
}
```

&emsp;&emsp;时间复杂度为 $\operatorname{O}(\sum s_i \times m)$，也可以使用滚动数组优化。

## 有依赖的背包问题

&emsp;&emsp;有依赖的背包问题可以使用树形 dp 解决。在每个节点处相当于做了一遍完全背包。为了方便状态转移，我们枚举当前节点分配多少空间，以及给子树分配多少空间。得到状态转移方程：

$$
f[x, j] = \max \lbrace \max \_{y \in Son(x), 0 \leq k \leq j - v[x]} \lbrace  f[y, k] + f[x, j - k]  \rbrace , w[x] \rbrace
$$

```cpp
#include <cstdio>
using namespace std;

const int& max(const int &a, const int &b) { return a > b ? a : b; }

const int MAXN = 110;
int Head[MAXN], Next[MAXN << 1], to[MAXN << 1], tot;

inline void add(const int &x, const int &y)
{
    to[++tot] = y;
    Next[tot] = Head[x];
    Head[x] = tot;
}

int v[MAXN], w[MAXN], p[MAXN];
int f[MAXN][MAXN];
int n, m, root;

void dp(int x)
{
    for (int i = v[x]; i <= m; i++) {
        f[x][i] = w[x];
    }

    for (int i = Head[x]; i; i = Next[i]) {
        int y = to[i];

        dp(y);

        for (int j = m; j >= v[x]; j--) {
            for (int k = 0; k <= j - v[x]; k++) {
                f[x][j] = max(f[x][j], f[y][k] + f[x][j - k]);
            }
        }
    }
}

int main()
{
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
#endif
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) {
        int p;
        scanf("%d%d%d", &v[i], &w[i], &p);

        if (p == -1) {
            root = i;
            continue;
        }
        add(p, i);
    }
    
    dp(root);
    
    printf("%d\n", f[root][m]);
    
    return 0;
}
```

## 求方案数

&emsp;&emsp;再开一个数组同步进行 dp 即可。设 $g[j]$ 表示背包容积为 $j$ 时的最优方案数，当然这是经过滚动数组压缩的。于是在更新 $f[j] = f[j - v[i]] + w[i]$ 时，我们更新 $g[j]$。令 $f[j - v[i]] + w[i] = k$，得到状态转移方程：

$$
g[j] = \left \lbrace
\begin {aligned}
&g[j - v[i]] \ (f[j] <k) \\\
&g[j] + g[j - v[i]] \ (f[j] = k)
\end {aligned}
\right.
$$

```cpp
#include <cstdio>
using namespace std;

const int& max(const int &a, const int &b) { return a > b ? a : b; }

const int MAXN = 1e3 + 10;
const unsigned int MOD = 1e9 + 7;
int v[MAXN], w[MAXN];

int f[MAXN], g[MAXN];

int main()
{
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
#endif
    int n, m;
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) {
        scanf("%d%d\n", &v[i], &w[i]);
    }

    for (int i = 0; i <= m; i++) {
        g[i] = 1;
    }

    for (int i = 1; i <= n; i++) {
        for (int j = m; j >= v[i]; j--) {
            int upd = f[j - v[i]] + w[i];
            if (f[j] < upd) {
                f[j] = upd;
                g[j] = g[j - v[i]];
            }
            else if (f[j] == upd) {
                g[j] += g[j - v[i]];
                g[j] %= MOD;
            }
        }
    }

    printf("%d\n", g[m]);

    return 0;
}
```

## 求具体方案

&emsp;&emsp;在每次转移时记录下从哪个状态转移来的，最后递归输出即可。