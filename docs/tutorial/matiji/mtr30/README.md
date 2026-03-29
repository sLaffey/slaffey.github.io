# 码蹄周赛第三十场（提高组）题解

懒得再给每个题单开一页了，主要这场强度低也没什么好说的。

其实倒也正适合，毕竟去年打完区域赛后一直没怎么练习，这场就当热热手了。

## 1、璇玑图

### 题目大意

给定一个 $N \times N$ 的字符方阵，你需要计算：

从任意一个位置出发，沿横向（左/右）、纵向（上/下）、斜向（左上、右上、左下、右下）这八个单一方向，延伸任意长度（至少包含一个字符）所形成的本质不同字符串数量。

$2 \leq N \leq 29$，仅包含小写英文字母。

### 解题思路

没啥好说的，数据范围很小，直接枚举扔一个 `set/map` 里去一下重就可以了。

实现起来可能稍微麻烦些，主要需要用到一个经典的处理网格上不同方向的技巧。

### 参考代码

```cpp
#include <iostream>
#include <set>
#include <string>
using namespace std;

const int MAXN = 35;
const int dx[8] = {-1, -1, 0, 1, 1, 1, 0, -1};
const int dy[8] = {0, 1, 1, 1, 0, -1, -1, -1};
char c[MAXN][MAXN];
set<string> v;
int cnt = 0;

int main()
{
#ifndef ONLINE_JUDGE
    freopen("./in", "r", stdin);
#endif
    ios_base::sync_with_stdio(false);
    cin.tie(nullptr), cout.tie(nullptr);
    int n;
    cin >> n;
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= n; j++) {
            cin >> c[i][j];
        }
    }
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= n; j++) {
            string s;
            s += c[i][j];
            v.insert(s);
            for (int p = 0; p < 8; p++) {
                s = "";
                s += c[i][j];
                int x = i + dx[p], y = j + dy[p];
                while (x > 0 && x <= n && y > 0 && y <= n) {
                    s += c[x][y];
                    v.insert(s);
                    x += dx[p], y += dy[p];
                }
            }
        }
    }
    cout << v.size() << endl;
    return 0;
}
```

## 2、树上染色

### 题目大意

给定一棵 $n$ 个结点的树，树上结点编号为 $1 \sim n$，且根结点编号为 $1$。

你需要对这棵树上的结点染色，共有三种颜色（红、绿、蓝）。

染色要求如下：

1. 相邻结点不能同色；

2. 若一个结点不是红色，则它至多拥有一个孩子结点为红色；

3. 要求最后红色结点数量最多；

求满足以上三个要求的情况下，红色结点数量的最大值。

$3\leq n \leq 10^5.$

### 解题思路

简单的树上 DP。

设计状态 $f[x, c]$ 表示结点 $x$ 染色 $c$ 时子树中红色结点数的最大值。考虑结点 $x$。

当 $x$ 自身染红色时，其所有孩子可以在绿色和蓝色中自由选择，统计其较大者即可。状态转移方程为：

$$
f[x, \text{red}] = \sum_{y \in \text{son}(x)} \max \{ f[y, \text{green}], f[y, \text{blue}]\} + 1
$$

当 $x$ 自身染绿色或蓝色时，此处不妨取绿色，蓝色同理。其所有孩子中至多有一个染红色，其余孩子直接染蓝色即可。枚举哪个孩子选红色（或者没有孩子选红色），即可写出状态转移方程：

$$
f[x, \text{green}] = \max \{ \max_{y \in \text{son}(x)} \{ \sum_{z \in \text{son}(x), z \neq y} f[z, \text{blue}] + f[y, \text{red}]\}, \sum_{y \in \text{son}(x)} f[y, \text{blue}] \}
$$

这需要两层循环求解，是不太能接受的。因此可以先让每个孩子都选蓝色，再在最后补上选红色和选蓝色的差：

$$
f[x, \text{green}] = \sum_{y \in \text{son}(x)} f[y, \text{blue}] + \max \{0, \max_{y \in \text{son}(x)} \{f[y, \text{red}] - f[y, \text{blue}]\}\}
$$

这样就可以用一层循环转移。

这道题挂了一次，挂在输入时候边数没写对，警钟长鸣。

### 参考代码

```cpp
#include <iostream>
using namespace std;

const int MAXN = 1e5 + 10, INF = 0x3f3f3f3f;
int Head[MAXN], Next[MAXN << 1], to[MAXN << 1], tot;

inline void add(const int &x, const int &y)
{
    to[++tot] = y;
    Next[tot] = Head[x];
    Head[x] = tot;
}

int f[MAXN][3];
// red green blue

void dfs(int x, int fa)
{
    for (int i = Head[x]; i; i = Next[i]) {
        int y = to[i];
        if (y == fa) continue;
        dfs(y, x);
    }

    int max1 = -INF, max2 = -INF;
    f[x][0] = 1;
    for (int i = Head[x]; i; i = Next[i]) {
        int y = to[i];
        if (y == fa) continue;
        f[x][0] += max(f[y][1], f[y][2]);
        f[x][1] += f[y][2];
        f[x][2] += f[y][1];
        max1 = max(max1, f[y][0] - f[y][1]);
        max2 = max(max2, f[y][0] - f[y][2]);
    }
    f[x][1] += max(0, max2);
    f[x][2] += max(0, max1);
}

int main()
{
#ifndef ONLINE_JUDGE
    freopen("./in", "r", stdin);
#endif
    ios_base::sync_with_stdio(false);
    cin.tie(nullptr), cout.tie(nullptr);

    int n;
    cin >> n;
    for (int i = 1; i < n; i++) {
        int u, v;
        cin >> u >> v;
        add(u, v);
        add(v, u);
    }

    dfs(1, 0);

    cout << max(f[1][0], max(f[1][1], f[1][2])) << endl;
    return 0;
}
```

## 3、蓬莱

### 题目大意

你进入了一个蓬莱游戏，它可以抽象成一张 $n$ 个点 $m$ 条边的无向连通图，其中结点 $1$ 为起点，结点 $n$ 为终点，边有边权，边权表示经过这条边的代价。

八仙过海，各显神通，在这场游戏中你可以使用两种仙法：

1. 仙法 1：持有此仙法时，所经过路径的代价，固定减去 $x$；
2. 仙法 2：持有此仙法时，所经过路径的代价，除以 $2$ 上取整；

你可以在起点处选择一种仙法带上，并可以在途中任意一个点切换一种仙法，具体规则如下：

1. 在起点时可以选择一种仙法带上，此时没有代价；
2. 在非起点位置，可以有**至多一次**切换仙法的机会，在点 $i$ 时切换仙法的代价是 $c_i$；

你需要找到从结点 $1$ 到结点 $n$ 的最短路径长度（即你所经过的所有边的边权之和最小）。

$2 \le n \le 2 \times 10^5, \ n - 1 \le m \le 4 \times 10^5, \ 1 \le u, v \le n, \ 1 \le x \le w \le 10^9, \ 1 \le c_i \le 10^{12}.$

### 解题思路

简单的最短路问题。由于路上只能切换一次，所以考虑枚举在哪个点切换。分别以结点 $1$ 和结点 $n$ 为起点，固定携带一种仙法，跑单源最短路，再枚举在哪个点切换仙法，组合出答案后取最小即可。

这道题挂了一次，挂在没考虑不切换的情况，以及太久没写最短路板子写错了，警钟长鸣。

有一个容易想到的变式是代价不变，但不限制切换仙法的次数。也可以在相同的时间复杂度内求解。这样就是一道标准的分层最短路问题，建两层图表示两种仙法，在相同的点中间连权值为 $c_i$ 的边，跑一遍最短路即可。

### 参考代码

感觉分层图版本可能实现起来还更简单一些。

```cpp
#include <iostream>
#include <cmath>
#include <queue>
#include <cstring>
using namespace std;

const int MAXN = 2e5 + 10, MAXM = 4e5 + 10;
typedef long long ll;
const ll INF = 0x3f3f3f3f3f3f3f3f;
int Head[MAXN], Next[MAXM << 1], to[MAXM << 1], we[MAXM << 1], tot;

inline void add(const int &x, const int &y, const int &w)
{
    to[++tot] = y;
    Next[tot] = Head[x];
    Head[x] = tot;
    we[tot] = w;
}

ll c[MAXN];
int x;

inline int f1(const int &w) { return w - x; }
inline int f2(const int &w) { return (w + 1) / 2; }

ll dis[2][2][MAXN];

struct node {
    int x;
    ll w;

    const bool operator < (const node &a) const {
        return w > a.w;
    }
};

int n, m;
bool v[MAXN];

void Dijkstra(int s, ll *dis, int f(const int&))
{
    priority_queue<node> q;
    q.push({s, 0});
    memset(dis + 1, 0x3f, sizeof(ll) * n);
    memset(v + 1, 0, sizeof(bool) * n);
    dis[s] = 0;
    while (!q.empty()) {
        int x = q.top().x;
        q.pop();

        if (v[x]) continue;
        v[x] = true;

        for (int i = Head[x]; i; i = Next[i]) {
            int y = to[i], w = f(we[i]);
            if (dis[x] + w < dis[y]) {
                dis[y] = dis[x] + w;
                q.push({y, dis[y]});
            }
        }
    }
}

int main()
{
#ifndef ONLINE_JUDGE
    freopen("./in", "r", stdin);
#endif
    ios_base::sync_with_stdio(false);
    cin.tie(nullptr), cout.tie(nullptr);

    cin >> n >> m >> x;
    for (int i = 2; i <= n - 1; i++) {
        cin >> c[i];
    }
    for (int i = 1; i <= m; i++) {
        int u, v, w;
        cin >> u >> v >> w;
        add(u, v, w);
        add(v, u, w);
    }

    Dijkstra(1, dis[0][0], f1);
    Dijkstra(1, dis[0][1], f2);
    Dijkstra(n, dis[1][0], f1);
    Dijkstra(n, dis[1][1], f2);

    ll ans = min(dis[0][0][n], dis[0][1][n]);
    for (int i = 2; i <= n - 1; i++) {
        ans = min(ans, 
            min(
                dis[0][0][i] + dis[1][1][i] + c[i],
                dis[0][1][i] + dis[1][0][i] + c[i]
            ));
    }
    cout << ans << endl;

    return 0;
}
```

## 4、广寒宫

### 题目大意

广寒宫中共有 $n$ 种月兔，第 $i$ 种月兔的数量为 $a_i$。

你需要总共捕捉 $k$ 只兔子，过程如下：

**第一阶段**

执行最多 $m$ 轮。

每一轮中，按**编号从小到大**的顺序，对每一种月兔尝试捕捉 $1$ 只：

若该种月兔剩余数量大于 $0$，则捕捉 $1$ 只；

若数量为 $0$，则跳过。

若在某个时刻已累计捕捉满 $k$ 只，则立即停止整个过程。

**第二阶段**

若在第一阶段后 仍未捕捉满 $k$ 只，则继续进行：

每次从所有月兔中选择**剩余数量最多**的品种捕捉 $1$ 只；若有并列，则选择**编号最小**的。

直到累计捕捉满 $k$ 只为止。

最终在第二阶段结束后，输出所有月兔剩余数量。

$1 \le T \le 10^5, \ 1 \le n \le 10^5, \ 1 \le \sum n \le 5 \times 10^5, \ 1 \le a_i, m \le 10^9, 1 \le k \le \sum a_i.$

### 解题思路

火速做完三道，以为最后一道能给点惊喜，结果来一道构式模拟题是何意味，出题人没活了？

一阶段可以发现兔子是按照数量顺序逐个被取完的。因此把兔子按照数量从小到大排序。逐个考虑兔子 $1 \leq j \leq i - 1$ 都已经被取完时，第 $i$ 个兔子完全被取完的过程。可以发现在取完兔子 $i$ 的过程中，每一轮都会把所有兔子 $i \leq j \leq n$ 取掉一个。每一轮对 $k$ 的贡献固定为 $n - i + 1$，因此可以一次计算取完兔子 $i$ 的所有轮的总贡献。接下来考虑 $k$ 取完的情况，此时可以先考虑 $\lfloor \frac{k}{n - i + 1} \rfloor$ 轮的总贡献，剩余的部分由于一定不超过 $n - i + 1$，暴力模拟即可。考虑轮数超过 $m$ 的情况，此时一次性计算最后所有轮的总贡献，再结算 $a_i$ 的值即可。

二阶段仍旧按照数量顺序。把兔子按照数量从小到大排序。先不考虑编号顺序，则取到兔子 $i$ 时，一定是所有兔子 $i < j \leq n$ 的数量都已经被取成 $a_i$ 了。因此从大到小考虑每个兔子 $i$，在将兔子 $i \leq j \leq n$ 都取成 $a_{i - 1}$ 的过程中，每一轮都会取掉 $n - i + 1$ 个兔子。因此仍然可以一次计算完总贡献，在 $k$ 用完时最后的部分模拟即可。

具体看代码。

这道题挂了三次，挂在最后输出时没有按照编号排序上，警钟长鸣。

### 参考代码

```cpp
#include <iostream>
#include <queue>
#include <algorithm>
using namespace std;

const int MAXN = 1e5 + 10;
typedef long long ll;
struct rab {
    int id;
    ll c;

    const bool operator < (const rab &a) const {
        return c != a.c ? c < a.c : id > a.id;
    }
} a[MAXN];

int main()
{
#ifndef ONLINE_JUDGE
    freopen("./in", "r", stdin);
#endif
    ios_base::sync_with_stdio(false);
    cin.tie(nullptr), cout.tie(nullptr);

    int T;
    cin >> T;
    while (T--) {
        int n;
        ll m, k;
        cin >> n >> m >> k;
        for (int i = 1; i <= n; i++) {
            cin >> a[i].c;
            a[i].id = i;
        }

        sort(a + 1, a + n + 1, [](const rab &a, const rab &b) -> const bool {
            return a.c != b.c ? a.c < b.c : a.id < b.id;
        });
        ll base = 0;
        for (int i = 1; i <= n; i++) {
            ll cnt = (a[i].c - base) * (n - i + 1);
            if (k > cnt) {
                if (a[i].c >= m) {
                    k -= (m - base) * (n - i + 1);
                    for (int i = 1; i <= n; i++) {
                        a[i].c = max(0LL, a[i].c - m);
                    }
                    break;
                }
                k -= cnt;
            }
            else {
                ll t = k / (n - i + 1);
                if (base + t >= m) {
                    k -= (m - base) * (n - i + 1);
                    for (int i = 1; i <= n; i++) {
                        a[i].c = max(0LL, a[i].c - m);
                    }
                    break;
                }
                k %= (n - i + 1);
                base += t;
                for (int i = 1; i <= n; i++) {
                    a[i].c = max(0LL, a[i].c - base);
                }
                sort(a + 1, a + n + 1, [](const rab &a, const rab &b) -> const bool {
                    return a.id < b.id;
                });
                for (int i = 1; i <= n; i++) {
                    if (a[i].c && k) {
                        a[i].c--;
                        k--;
                    }
                }
                break;
            }
            base = a[i].c;
        }

        if (k) {
            sort(a + 1, a + n + 1, [](const rab &a, const rab &b) -> const bool {
                return a.c < b.c;
            });
            for (int i = n; i >= 1; i--) {
                if (a[i].c == a[i - 1].c) continue;
                ll cnt = (a[i].c - a[i - 1].c) * (n - i + 1);
                if (k > cnt) {
                    k -= cnt;
                }
                else {
                    ll base = a[i].c - k / (n - i + 1);
                    for (int j = i; j <= n; j++) {
                        a[j].c = base;
                    }
                    k %= (n - i + 1);
                    sort(a + i, a + n + 1, [](const rab &a, const rab &b) -> const bool {
                        return a.id < b.id;
                    });
                    for (int j = i; j <= n; j++) {
                        if (k) {
                            a[j].c--;
                            k--;
                        }
                    }
                    break;
                }
            }
        }

        sort(a + 1, a + n + 1, [](const rab &a, const rab &b) -> const bool {
            return a.id < b.id;
        });
        for (int i = 1; i <= n; i++) {
            cout << a[i].c << ' ';
        }
        cout << endl;
    }

    return 0;
}
```