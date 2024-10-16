# [USACO18OPEN] Disruption P

## Solution

&emsp;&emsp;我们思考一下在建出树后，每条附加边对答案的贡献。可以发现，每条附加边只会更新这条边的两个端点在树上的路径上的边的答案。因此我们可以维护树上边的答案，初始值均为无穷大。

&emsp;&emsp;不难得到一个使用树剖动态维护最小值的在线做法。时间复杂度 $\Theta(n \log^2 n)$。洛谷的题解中提到似乎可以使用 LCT 优化成 $\Theta(n \log n)$，不过笔者不会 LCT。

&emsp;&emsp;当然，我们也可以思考更简单的解法。

&emsp;&emsp;很明显，每条树边只会被最小的附加边更新一次。因此我们考虑将所有附加边排序，按照顺序不重复地更新树边。这时可以采用并查集维护“需要更新的树边”。时间复杂度同样为 $\Theta(n \log n)$。瓶颈主要在于排序。在实际测试中表现优异。

&emsp;&emsp;树剖做法在洛谷评测 905ms（吸氧），并查集做法在洛谷评测 261ms（吸氧），加上快读可以跑到 175ms。截至目前（2022.10.14）是最优解第一。

![截图 2022-10-14 19-07-32.png](https://s2.loli.net/2022/10/14/9jPs2o6fdmlkvau.png)

## Code

### 树剖

```cpp
#include <cstdio>
using namespace std;

void swap(int &a, int &b) { int t = a; a = b, b = t; }
const int& min(const int &a, const int &b) { return a < b ? a : b; }

const int MAXN = 5e4 + 10, INF = 0x3f3f3f3f;

int Head[MAXN], Next[MAXN << 1], to[MAXN << 1], tot;

inline int index(const int &i) { return (i >> 1) + (i & 1); }

inline void add(const int &x, const int &y)
{
    to[++tot] = y;
    Next[tot] = Head[x];
    Head[x] = tot;
}

int node_id[MAXN];

int Size[MAXN], dep[MAXN];

int son[MAXN], fa[MAXN], top[MAXN];

int id[MAXN], id_top;

void dfs_size(int x)
{
    Size[x] = 1;

    for (int i = Head[x]; i; i = Next[i]) {
        int y = to[i];
        if (y == fa[x]) continue;

        fa[y] = x;
        dep[y] = dep[x] + 1;
        node_id[index(i)] = y;

        dfs_size(y);

        Size[x] += Size[y];
        if (Size[son[x]] < Size[y]) {
            son[x] = y;
        }
    }
}

void dfs_top(int x, int v)
{
    top[x] = v;
    id[x] = ++id_top;

    if (son[x]) dfs_top(son[x], v);
    else return;

    for (int i = Head[x]; i; i = Next[i]) {
        int y = to[i];
        if (y == son[x] || y == fa[x]) continue;

        dfs_top(y, y);
    }
}

struct stree {
    int val = INF, upd = INF;
    int l, r;
} tree[MAXN << 2];

#define ls (p << 1)
#define rs (p << 1 | 1)
#define self tree[p]
#define lson tree[ls]
#define rson tree[rs]

void st_build(int p, int l, int r)
{
    self.l = l, self.r = r;
    if (l == r) {
        return;
    }

    int mid = l + r >> 1;
    st_build(ls, l, mid);
    st_build(rs, mid + 1, r);

    self.val = min(lson.val, rson.val);
}

void pushdown(int p)
{
    if (self.upd != INF) {
        lson.val = min(lson.val, self.upd);
        lson.upd = min(lson.upd, self.upd);

        rson.val = min(rson.val, self.upd);
        rson.upd = min(rson.upd, self.upd);

        self.upd = INF;
    }
}

void st_modify(int p, int l, int r, int v)
{
    if (l <= self.l && self.r <= r) {
        self.val = min(self.val, v);
        self.upd = min(self.upd, v);
        return;
    }

    pushdown(p);

    if (l <= lson.r) st_modify(ls, l, r, v);
    if (r >= rson.l) st_modify(rs, l, r, v);

    self.val = min(lson.val, rson.val);
}

int st_query(int p, int x)
{
    if (self.l == self.r) {
        return self.val;
    }

    pushdown(p);

    int ans = INF;
    if (x <= lson.r) ans = st_query(ls, x);
    else ans = st_query(rs, x);

    return ans;
}

void modify(int x, int y, int v)
{
    while (top[x] != top[y]) {
        if (dep[top[x]] < dep[top[y]]) {
            swap(x, y);
        }
        st_modify(1, id[top[x]], id[x], v);
        x = fa[top[x]];
    }
    if (dep[x] > dep[y]) {
        swap(x, y);
    }
    if (id[x] < id[y]) st_modify(1, id[x] + 1, id[y], v);
}

int query(int x)
{
    return st_query(1, id[node_id[x]]);
}

int main()
{
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    // freopen("out", "w", stdout);
#endif
    int n, m;
    scanf("%d%d", &n, &m);
    for (int i = 1; i < n; i++) {
        int a, b;
        scanf("%d%d", &a, &b);
        add(a, b);
        add(b, a);
    }

    dfs_size(1);
    dfs_top(1, 1);
    st_build(1, 1, n);

    for (int i = 1; i <= m; i++) {
        int a, b, w;
        scanf("%d%d%d", &a, &b, &w);
        modify(a, b, w);
    }

    for (int i = 1; i < n; i++) {
        int res = query(i);
        printf("%d\n", res == INF ? -1 : res);
    }

    return 0;
}
```

### 并查集

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

const int MAXN = 5e4 + 10, INF = 0x3f3f3f3f;

int Head[MAXN], Next[MAXN << 1], to[MAXN << 1], tot;

inline void add(const int &x, const int &y)
{
    to[++tot] = y;
    Next[tot] = Head[x];
    Head[x] = tot;
}

struct edge_type {
    int x, y, w;
    
    inline const bool operator < (const edge_type &a) const
    {
        return w < a.w;
    }
} edge[MAXN];

inline const int index(const int &x) { return (x >> 1) + (x & 1); }

int fat[MAXN];
int node_we[MAXN];
int node_id[MAXN];
int dep[MAXN];

void dfs(int x)
{
    for (int i = Head[x]; i; i = Next[i]) {
        int y = to[i];
        if (y == fat[x]) continue;

        fat[y] = x;
        node_we[y] = INF;
        node_id[index(i)] = y;
        dep[y] = dep[x] + 1;

        dfs(y);
    }
}

int fa[MAXN];

const int& Find(const int &x) { return fa[x] == x ? fa[x] : fa[x] = Find(fa[x]); }

void modify(int x, int y, int v)
{
    x = Find(x), y = Find(y);
    while (x != y) {
        if (dep[x] < dep[y]) {
            swap(x, y);
        }
        node_we[x] = v;
        fa[x] = fat[x];
        x = Find(x);
    }
}

int main()
{
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    // freopen("out", "w", stdout);
#endif
    int n, m;
    scanf("%d%d", &n, &m);
    for (int i = 1; i < n; i++) {
        int a, b;
        scanf("%d%d", &a, &b);
        add(a, b);
        add(b, a);
    }
    for (int i = 1; i <= m; i++) {
        scanf("%d%d%d", &edge[i].x, &edge[i].y, &edge[i].w);
    }

    dfs(1);
    sort(edge + 1, edge + m + 1);
    for (int i = 1; i <= n; i++) {
        fa[i] = i;
    }

    for (int i = 1; i <= m; i++) {
        modify(edge[i].x, edge[i].y, edge[i].w);
    }

    for (int i = 1; i < n; i++) {
        printf("%d\n", node_we[node_id[i]] == INF ? -1 : node_we[node_id[i]]);
    }

    return 0;
}
```