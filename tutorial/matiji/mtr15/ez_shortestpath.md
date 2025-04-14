# 简单最短路

## 题目大意

给一张无向图 $G$，定义一条长度大于等于 $3$ 的路径 $P$ 的代价为 $w(P) = \min \limits _{x, y, z \in P}\{w(x)w(y)+w(x)w(z)+w(y)w(z)\}$，其中 $P$ 表示路径上所有边的集合，$w(x)$ 表示边 $x$ 的权值。题中路径不一定为简单路径。

$q$ 组询问两点 $A, B$ 间路径的最小代价。若不存在符合条件的路径，则输出 $-1$。

数据范围：$1 \leq n \leq 10^5, 1 \leq m \leq 2 \times 10^5, 1 \leq q \leq 10^5, 1\leq w \leq 10^5$。

## 解题思路

要最小化路径的代价，只需要最小化路径中权值最小的三条边的权值即可。既然不要求为简单路径，那么在同一连通块内任意两点间必定存在可以经过该连通块内所有边的路径。因此，只需要用并查集维护连通块中权值最小的三条边的权值，即可高效回答询问。

理论复杂度 $O((n + q)\log n)$。

## 参考代码

```cpp
#include <iostream>
#include <cstring>
using namespace std;

const int MAXN = 1e5 + 10, MAXM = 2e5 + 10;

int fa[MAXN];

inline int Find(int x)
{
    while (fa[x] != x) {
        x = fa[x] = fa[fa[x]];
    }
    return x;
}

struct Triad {
    long long a, b, c;

    void update(const int &x)
    {
        if (x < c) c = x;
        if (a > c) swap(a, c);
        if (b > c) swap(b, c);
    }

    void update(const Triad &x)
    {
        update(x.a);
        update(x.b);
        update(x.c);
    }
} we[MAXN];

int main()
{
#ifndef ONLINE_JUDGE
    freopen("./in", "r", stdin);
#endif
    ios_base::sync_with_stdio(false);
    int n, m;
    cin >> n >> m;
    for (int i = 1; i <= n; i++) {
        fa[i] = i;
    }
    memset(we, 0x3f, sizeof we);
    for (int i = 1; i <= m; i++) {
        int u, v, w;
        cin >> u >> v >> w;
        u = Find(u), v = Find(v);
        fa[v] = u;
        if (u != v) we[u].update(we[v]);
        we[u].update(w);
        w ++;
    }
    int q;
    cin >> q;
    while (q--) {
        int a, b;
        cin >> a >> b;
        a = Find(a), b = Find(b);
        if (a != b || we[a].c == 0x3f3f3f3f) {
            cout << -1 << '\n';
        }
        else {
            cout << we[a].a * we[a].b + we[a].a * we[a].c + we[a].b * we[a].c << '\n';
        }
    }
    return 0;
}
```