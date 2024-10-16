# 「ZJOI 2008」骑士

### 前言

众所周知，[骑士](https://www.luogu.com.cn/problem/P2607)这道题是一个很裸但很恶心的基环树 dp。并且 dfs 找环比较恶心，有很多选手都挂在了找环上~~比如我~~。所以本文来介绍一个很简单的找环方式。即**拓扑排序**。

仔细一想，拓扑排序确实很适合解决这道基环树。不需考虑连通块，自动遍历所有不在环上的点，可以拿来执行树形 dp……一切一切的优点都决定了使用拓扑排序的本题代码十分简洁。

### 思路

简单讲下思路。环形 dp 就省略了。

这道题的树形 dp 是从叶子到根转移的，所以拓扑排序时用当前的节点更新下一个点即可。

难点是怎么解决统计入度问题。

如果我们**把这个基环树看做一个内向树**，那么每个点的入度很明显是自身**直接连通**的点的个数减一。因此我们可以直接统计与每个点直接连通的点的数目，并以此代替入度执行拓扑排序即可。

这样我们可以在拓扑排序的时候给每个点打上标记，表示**这个点不在环内**。对于每个连通块都执行简单的 dfs 即可直接找环。

### Code

代码不吸氧 `600ms`，吸氧 `200ms`，总体速度还是可以的。

如果不用 STL queue 的话，不吸氧就可以跑到 `300ms`。
```cpp
#include <cstdio>
#include <queue>
using namespace std;

namespace QuickRead {
    const int BUFFER_SIZE = 1e6 + 10;
    const int MAX_READ = 1e6;
    char ReadBuffer[BUFFER_SIZE];
    int p = MAX_READ;

    const char& get()
    {
        if (p == MAX_READ) {
            fread(ReadBuffer + 1, 1, MAX_READ, stdin);
            p = 0;
        }
        return ReadBuffer[++p];
    }

    const int read()
    {
        int ans = 0;
        char c = get();
        while (c < '0' || c > '9') {
            c = get();
        }
        while (c >= '0' && c <= '9') {
            ans *= 10;
            ans += c - 48;
            c = get();
        }
        return ans;
    }
}

typedef long long ll;

const ll& max(const ll &a, const ll &b) { return a > b ? a : b; }

const int MAXN = 1e6 + 10;
int Head[MAXN], Next[MAXN << 1], to[MAXN << 1], tot;

void add(const int &x, const int &y)
{
    to[++tot] = y;
    Next[tot] = Head[x];
    Head[x] = tot;
}

bool v[MAXN];
int we[MAXN];
ll f[MAXN][2];
int son[MAXN];
int n;

queue<int> q;

void Topsort()
{
    for (int i = 1; i <= n; i++) {
        son[i]--;
        f[i][1] = we[i];
        if (!son[i]) {
            q.push(i);
        }
    }

    while (!q.empty()) {
        int x = q.front();
        q.pop();
        v[x] = true;

        for (int i = Head[x]; i; i = Next[i]) {
            int y = to[i];
            if (!son[y]) continue;

            f[y][0] += max(f[x][1], f[x][0]);
            f[y][1] += f[x][0];

            son[y]--;
            if (!son[y]) {
                q.push(y);
            }
        }
    }

    return;
}

int circle[MAXN], top;

void dfs(int x)
{
    circle[++top] = x;
    v[x] = true;
    for (int i = Head[x]; i; i = Next[i]) {
        int y = to[i];
        if (v[y]) continue;

        dfs(y);
    }
}

ll g[MAXN][2][2];

ll dp()
{
    g[2][0][1] = f[circle[1]][0] + f[circle[2]][1];
    g[2][1][0] = f[circle[1]][1] + f[circle[2]][0];
    g[2][1][1] = g[2][1][0];
    g[2][0][0] = f[circle[1]][0] + f[circle[2]][0];

    for (int i = 3; i <= top; i++) {
        g[i][0][0] = max(g[i - 1][0][1], g[i - 1][0][0]) + f[circle[i]][0];
        g[i][0][1] = g[i - 1][0][0] + f[circle[i]][1];
        g[i][1][0] = max(g[i - 1][1][0], g[i - 1][1][1]) + f[circle[i]][0];
        g[i][1][1] = g[i - 1][1][0] + f[circle[i]][1];
    }

    return max(g[top][0][0], max(g[top][0][1], g[top][1][0]));
}

int main()
{
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
#endif
    using QuickRead::read;
    n = read();
    for (int i = 1; i <= n; i++) {
        we[i] = read();
        int y = read();
        add(i, y);
        add(y, i);
        son[i]++, son[y]++;
    }

    Topsort();
    
    ll ans = 0;
    for (int i = 1; i <= n; i++) {
        if (!v[i]) {
            top = 0;
            dfs(i);
            ans += dp();
        }
    }

    printf("%lld\n", ans);

    return 0;
}
```