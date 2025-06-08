# 「牛客竞赛」赛前集训营 2022 第三场

## 前言

&emsp;&emsp;这次成绩 70 + 60 + 10 + 15 = 155，排名 29/368。按照 NOIP 算的话正好够到去年省一线。还是有点悬啊。

## 一般图最小匹配

### 一般做法

&emsp;&emsp;考虑排序。可以发现排序后只会选择相邻的点。

&emsp;&emsp;考虑进行一个 dp 求解。设计状态 $f[i, j]$，表示前 $i$ 个点中选择 $j$ 对点的最小值。于是有转移方程：

$$
f[i, j] = \min \lbrace f[i - 1, j], f[i - 1, j - 1] + a[i] - a[i - 1] \rbrace
$$

&emsp;&emsp;分别对应选与不选。初始化 $f[1, i] = f[0, i] = INF \ (1 \leq i \leq m)$，$f[i, 0] = 0 \ (0 \leq i \leq n)$。

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

const int MAXN = 5e3 + 10;
int a[MAXN];
int f[MAXN][MAXN];

int main()
{
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    // freopen("out", "w", stdout);
#endif
    int n, m;
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) {
        scanf("%d", &a[i]);
    }

    sort(a + 1, a + n + 1);

    for (int i = 0; i <= n; i++) {
        f[i][0] = 0;
    }
    for (int i = 1; i <= m; i++) {
        f[0][i] = f[1][i] = 0x3f3f3f3f;
    }
    for (int i = 2; i <= n; i++) {
        for (int j = 1; j <= m; j++) {
            f[i][j] = min(f[i - 1][j], f[i - 2][j - 1] + a[i] - a[i - 1]);
        }
    }

    printf("%d\n", f[n][m]);

    return 0;
}
```

&emsp;&emsp;这是出题人也是绝大多数做题人的做法，时间复杂度为 $\Theta(nm)$。但是有一个做题人发明出了 $\Theta(m \log n)$ 的做法。

### 更好的做法

&emsp;&emsp;考虑贪心。记 $d[i] = a[i + 1] - a[i]$。则如果选择了 $d[i]$，$d[i - 1]$ 与 $d[i + 1]$ 就都不能选了。此时显然有可能会有选择两边比选择中间更优，于是我们用 $d[i - 1] + d[i + 1] - d[i]$ 代替这个点。用堆动态维护最小值累加答案即可。因为最多“反悔”一次，所以整个程序复杂度为  $\Theta(m \log n)$。

&emsp;&emsp;需要用链表维护前驱后继，以免出现选重点的情况。

&emsp;&emsp;如果您没有使用，下面这组数据或许可以 hack 掉。

- Input

```plain
10 3 
1 4 6 7 12 12 12 13 14 15
```

- Output

```plain
2
```

```cpp
#include <cstdio>
#include <algorithm>
#include <queue>
using namespace std;

const int MAXN = 5e3 + 10;
int a[MAXN];

int Prev[MAXN], Next[MAXN];
int d[MAXN];

struct item_type {
    int p, val;

    const bool operator < (const item_type &a) const
    {
        return val > a.val;
    }
};
priority_queue<item_type> heap;

bool v[MAXN];

void update(const item_type &t)
{
    int l = Prev[t.p], r = Next[t.p];
    v[l] = v[r] = true;

    Prev[t.p] = Prev[l], Next[t.p] = Next[r];
    Prev[Next[r]] = t.p, Next[Prev[l]] = t.p;

    d[t.p] = (l && r) ? d[r] + d[l] - d[t.p] : 0x3f3f3f3f;
    heap.push({t.p, d[t.p]});
}

int main()
{
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    // freopen("out", "w", stdout);
#endif
    int n, m;
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) {
        scanf("%d", &a[i]);
    }

    sort(a + 1, a + n + 1);
    for (int i = 1; i < n; i++) {
        d[i] = a[i + 1] - a[i];
        heap.push({i, d[i]});
        Prev[i] = i - 1;
        Next[i] = i + 1;
    }
    Next[n - 1] = 0;

    int ans = 0;
    while (m--) {
        while (v[heap.top().p]) heap.pop();

        item_type t = heap.top();
        heap.pop();

        ans += t.val;

        update(t);
    }

    printf("%d\n", ans);

    return 0;
}
```

## 重定向

&emsp;&emsp;考虑贪心。

&emsp;&emsp;记 $minx[i]$ 为 $i$ 的后缀最小值，$minv$ 为当前“可填的数字”的最小值。

&emsp;&emsp;由于字典序的性质，我们只要从前往后找到一个能让字典序变小的位置，就可以得到最小的字典序。

&emsp;&emsp;分三种情况：

- $0 < a[i + 1] < a[i]$，此时删去 $a[i]$ 即可。

- $a[i + 1] = 0 \wedge a[i] > minv$，此时删去 $a[i]$。

- $a[i] = 0 \wedge minx[i] < minv$，此时删去 $minx[i]$。

```cpp
#include <cstdio>
#include <cstring>
#include <queue>
using namespace std;

const int MAXN = 2e5 + 10;
int a[MAXN], pos[MAXN];
int minx[MAXN];
bool v[MAXN];
priority_queue<int, vector<int>, greater<int>> heap;

int main()
{
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    // freopen("out", "w", stdout);
#endif
    int T;
    scanf("%d", &T);
    
    while (T--) {
        int n;
        scanf("%d", &n);

        // initialize
        memset(v, false, sizeof(bool) * (n + 1));
        memset(minx, 0x3f, sizeof(int) * (n + 2)); // set minx[n + 1] to 0x3f3f3f3f
        memset(pos, 0, sizeof(int) * (n + 1));
        heap = priority_queue<int, vector<int>, greater<int>>();

        for (int i = 1; i <= n; i++) {
            scanf("%d", &a[i]);
            v[a[i]] = true;
            if (a[i]) pos[a[i]] = i;
        }

        for (int i = n; i >= 1; i--) {
            minx[i] = min(minx[i + 1], a[i] ? a[i] : 0x3f3f3f3f);
        }

        for (int i = 1; i <= n; i++) {
            if (!v[i]) {
                heap.push(i);
            }
        }

        int p = n;
        for (int i = 1; i < n; i++) {
            if (!a[i]) {
                if (minx[i] > heap.top()) {
                    a[i] = heap.top();
                    heap.pop();
                }
                else {
                    a[i] = minx[i];
                    p = pos[a[i]];
                    break;
                }
                continue;
            }
            if (a[i + 1]) {
                if (a[i] > a[i + 1]) {
                    p = i;
                    heap.push(a[i]);
                    break;
                }
            }
            else {
                if (a[i] > heap.top()) {
                    p = i;
                    heap.push(a[i]);
                    break;
                }
            }
        }
        
        for (int i = 1; i <= n; i++) {
            if (a[i] || i == p) continue;
            a[i] = heap.top();
            heap.pop();
        }

        for (int i = 1; i <= n; i++) {
            if (i == p) continue;
            printf("%d ", a[i]);
        }
        printf("\n");
    }

    return 0;
}
```

## 斯坦纳树

&emsp;&emsp;涉及虚树，超出笔者知识范围，不做讲解。

## 直径

&emsp;&emsp;感觉比连通图还复杂的一道“巨大的 DP”（出题人题解原话）。笔者水平与时间有限，不做讲解。