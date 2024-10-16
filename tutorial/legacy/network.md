# 「AcWing #364」网络

### Solution

&emsp;&emsp;明显是一道边双的题，考虑先缩点，容易发现在同一个边双内加边不会对答案造成影响。

&emsp;&emsp;在不同边双间加边，会把这两个边双之间的路径全部变成非桥边。因此我们求出边双并缩点，由于得到的图一定是一个树，直接向上查询 LCA 即可，可以用并查集优化此过程。

### Code
```cpp
#include <cstdio>
#include <cstring>
using namespace std;

const int& min(const int &a, const int &b) { return a > b ? b : a; }
void swap(int &a, int &b) { int t = a; a = b, b = t; }

const int MAXN = 1e5 + 10, MAXM = 2e5 + 10;

namespace Graph {
    int Head[MAXN], Next[MAXM << 1], to[MAXM << 1], tot = 1;

    void add(const int &x, const int &y)
    {
        to[++tot] = y;
        Next[tot] = Head[x];
        Head[x] = tot;
    }
}

int dfn[MAXN], low[MAXN];
int times;

bool bridge[MAXM << 1];
int bc;

void tarjan(int x, int from)
{
    using namespace Graph;
    dfn[x] = low[x] = ++times;
    for (int i = Head[x]; i; i = Next[i]) {
        int y = to[i];
        if (i == (from ^ 1)) continue;

        if (!dfn[y]) {
            tarjan(y, i);
            low[x] = min(low[x], low[y]);

            if (dfn[x] < low[y]) {
                bridge[i] = bridge[i ^ 1] = true;
                bc++;
            }
        }
        else {
            low[x] = min(low[x], dfn[y]);
        }
    }
}

int dcc[MAXN];
int dcc_cnt;

void dfs(int x)
{
    using namespace Graph;
    dcc[x] = dcc_cnt;
    for (int i = Head[x]; i; i = Next[i]) {
        int y = to[i];
        if (dcc[y] || bridge[i]) continue;

        dfs(y);
    }
}

namespace Tree {
    int Head[MAXN], Next[MAXN << 1], to[MAXN << 1], tot;

    void add(const int &x, const int &y)
    {
        to[++tot] = y;
        Next[tot] = Head[x];
        Head[x] = tot;
    }
}

int fat[MAXN];
int dep[MAXN];

void dfs_fa(int x, int f)
{
    using namespace Tree;
    fat[x] = f;
    dep[x] = dep[f] + 1;
    for (int i = Head[x]; i; i = Next[i]) {
        int y = to[i];
        if (y == f) continue;
        dfs_fa(y, x);
    }
}

int fa[MAXN];

const int& Find(const int &x) { return x == fa[x] ? fa[x] : fa[x] = Find(fa[x]); }

void query(int x, int y)
{
    if (dep[x] < dep[y]) swap(x, y);
    
    while (dep[x] > dep[y]) {
        bc--;
        fa[x] = fat[x];
        x = Find(fat[x]);
    }

    if (x == y) return;

    while (x != y) {
        bc -= 2;
        fa[x] = fat[x], fa[y] = fat[y];
        x = Find(fat[x]), y = Find(fat[y]);
    }

    return;
}

void init()
{
    memset(Graph::Head, 0, sizeof Graph::Head);
    Graph::tot = 1;

    memset(Tree::Head, 0, sizeof Tree::Head);
    Tree::tot = 0;

    memset(dfn, 0, sizeof dfn);
    memset(low, 0, sizeof low);
    memset(bridge, false, sizeof bridge);
    memset(dcc, 0, sizeof dcc);
    bc = dcc_cnt = times = 0;

    memset(fat, 0, sizeof fat);
    memset(dep, 0, sizeof dep);
}

int main()
{
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
#endif
    int n, m;
    int cs = 0;
    while (scanf("%d%d", &n, &m) != EOF && n) {
        init();
        cs++;
        printf("Case %d:\n", cs);
        for (int i = 1; i <= m; i++) {
            using Graph::add;
            int a, b;
            scanf("%d%d", &a, &b);
            add(a, b);
            add(b, a);
        }

        for (int i = 1; i <= n; i++) {
            if (!dfn[i]) {
                tarjan(i, 0);
            }
        }

        for (int i = 1; i <= n; i++) {
            if (!dcc[i]) {
                dcc_cnt++;
                dfs(i);
            }
        }

        for (int x = 1; x <= n; x++) {
            using Graph::Head, Graph::Next, Graph::to;
            using Tree::add;
            for (int i = Head[x]; i; i = Next[i]) {
                if (!bridge[i]) continue;
                add(x, to[i]);
            }
        }

        for (int i = 1; i <= dcc_cnt; i++) fa[i] = i;

        dfs_fa(1, 0);

        int q;
        scanf("%d", &q);
        while (q--) {
            int x, y;
            scanf("%d%d", &x, &y);
            x = dcc[x], y = dcc[y];
            x = Find(x), y = Find(y);

            query(x, y);

            printf("%d\n", bc);
        }
        printf("\n");
    }

    return 0;
}
```