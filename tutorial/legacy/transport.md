# 「NOIP 2013」货车运输

[洛谷题目链接](https://www.luogu.com.cn/problem/P1967)

[AcWing题目链接](https://www.acwing.com/problem/content/508/)

## 题目大意

一个城市交通网络为一个 $N$ 个点，$M$ 条边的无向图。道路可双向通行。每条道路都有一个限重。询问司机从 $A$ 地到 $B$ 地所能承载的最大重量。

## 思路分析

首先要保证对于每一对 $A, B$ 我们都能找出连通的路径，且这个路径上的所有边权都是最大的。联系到最小生成树，我们可以想到这个问题可以转化为**求最大生成树**的问题。

根据最大生成树的定义（类比最小生成树），很明显 $A, B$ 间的所有路径中只有树上的路径能保证所有边权都最大。因为我们在求最大生成树时总是选出权值较大的边。这和最长路问题应加以区分，因为最长路关心的是总和最大，而最大生成树是每一条边的权值最大，在下面的图里可以看出区别。

![graph.png](https://s2.loli.net/2022/05/02/USvqOIz95XnJ3Fh.png)

在上面的图中，很明显 `1->3` 的最长路径和最大生成树上的路径是不同的。而这时我们应该选择的路径就是最大生成树而不是最长路。

于是我们要做的就是利用 Kruskal 求出最大生成树，然后**在询问时输出任意两点间最大生成树路径中的边权最小值**。

维护边权最小值相信肯定很熟悉了。利用 LCA 树上倍增的思想，推出状态转移方程即可。类比求 LCA 时的 $F$ 数组，这里用 $G[i, j]$ 表示从节点 $i$ 开始到向上跳 $2^j$ 步到达的节点处这一路径中边权的最大值。则状态转移方程如下。

$$
G[i, \ j] = \min \{G[i, \ j - 1], \ G[F[i, j - 1], \ j - 1]\}
$$

最后询问的时候也是和 LCA 一样的思路往上边跳边求最小值即可。

## 代码实现
```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

const int MAXN = 1e4 + 10, MAXM = 5e4 + 10;
int n, m;

const int min(const int & a, const int & b)
{
    return (a > b) ? b : a;
}

const int min(const int & a, const int & b, const int & c)
{
    return min(a, min(b, c));
}

// 前向星
int Head[MAXN], to[MAXN << 1], Next[MAXN << 1], we[MAXN << 1], tot;

void add(const int & x, const int & y, const int & w)
{
    to[++tot] = y;
    we[tot] = w;
    Next[tot] = Head[x];
    Head[x] = tot;
}

// 并查集
int fa[MAXN];

int Find(int x)
{
    if (fa[x] == x) {
        return x;
    }
    return fa[x] = Find(fa[x]);
}

void Union(int x, int y)
{
    x = Find(x), y = Find(y);
    fa[y] = x;
}

// Kruskal 求最大生成树
struct edge {
    int x, y, w;
    bool flag;
} edges[MAXM];

bool cmp(const edge & a, const edge & b)
{
    return a.w > b.w;
}

void Kruskal()
{
    sort(edges + 1, edges + m + 1, cmp);
    for (int i = 1; i <= n; i++) {
        fa[i] = i;
    }
    for (int i = 1; i <= m; i++) {
        int x = edges[i].x, y = edges[i].y, w = edges[i].w;
        int px = Find(x), py = Find(y);
        if (px == py) {
            continue;
        }
        Union(px, py);
        edges[i].flag = true;
        add(x, y, w);
        add(y, x, w);
    }
}

// 利用 LCA 的思想，树上倍增求最小边权
int f[MAXN][21];
int g[MAXN][21];
int dep[MAXN];

void dfs(int x)
{
    for (int i = 1; i <= 20; i++) {
        f[x][i] = f[f[x][i - 1]][i - 1];
        g[x][i] = min(g[x][i - 1], g[f[x][i - 1]][i - 1]);
    }

    for (int i = Head[x]; i; i = Next[i]) {
        int y = to[i], w = we[i];
        if (y == f[x][0]) {
            continue;
        }
        f[y][0] = x;
        dep[y] = dep[x] + 1;
        g[y][0] = w;
        dfs(y);
    }
}

int ask(int x, int y)
{
    if (dep[x] > dep[y]) {
        swap(x, y);
    }

    int ans = 0x3f3f3f3f;
    for (int i = 20; i >= 0; i--) {
        if (dep[x] <= dep[f[y][i]]) {
            ans = min(ans, g[y][i]);
            y = f[y][i];
        }
    }

    if (x == y) {
        // return ans;
        // 虽然按下面那么写才更合理些，但是按上面的写也能过（数据比较水
        return ans == 0 ? -1 : ans;
    }

    for (int i = 20; i >= 0; i--) {
        if (f[x][i] != f[y][i]) {
            ans = min(ans, g[x][i], g[y][i]);
            x = f[x][i], y = f[y][i];
        }
    }
    ans = min(ans, g[x][0], g[y][0]);

    return ans == 0 ? -1 : ans;
}

int main()
{
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= m; i++) {
        scanf("%d%d%d", &edges[i].x, &edges[i].y, &edges[i].w);
    }
    Kruskal();
    dep[1] = 1;
    dfs(1);
    int q;
    scanf("%d", &q);
    while (q--) {
        int x, y;
        scanf("%d%d", &x, &y);
        printf("%d\n", ask(x, y));
    }
    return 0;
}
```

### 关于 MarkDown 和 LaTeX（夹带私货

[排版参考](https://studyingfather.blog.luogu.org/blog-written-guide)

不过在数组书写时我更喜欢采取进阶指南上的**大写字母做数组名+中括号与逗号列出数组下标**的方式

### 关于 AcWing 上的 hack 数据

这段代码提交 AcWing 是不能过的，还是哪道题都要去 AcWing 做的 [@Star](http://oj.hyoicloud.cn/user/176) 发现了这个问题。

hack 数据如下。

```
6 4
1 2 10
1 3 20
4 5 10
4 6 30
3
2 3
5 6
1 4
```

这个数据的输出就不粘了，画出图我们可以发现**这个图不是连通的**。

![image.png](https://s2.loli.net/2022/05/02/GgcHRDaJfSjuAOx.png)

的确，原题中并没有提到关于这个图是否连通的问题，这也启发我们**认真读题**。

那么这个数据怎么处理呢？其实也很简单，既然不连通，那我们就把整张图划分为一系列连通块，分别进行一次 dfs 即可。

代码仅提供修改部分。

```cpp
for (int i = 1; i <= n; i++) {
    if (fa[i] == i) {
        dep[i] = 1;
        dfs(i);
    }
}
```