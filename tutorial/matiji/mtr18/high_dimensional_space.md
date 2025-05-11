# 高维空间

## 题目大意

给一个 $d$ 维空间，第 $i$ 个维度的尺度为 $a_i$。在这个空间中有 $n = \prod^d_{i=1} a_i$ 个点。定义两个点 $(p_1, p_2, \ldots, p_d)$ 和 $(q_1, q_2, \ldots, q_d)$ 是相邻的，当且仅当 $\sum^d_{i = 1} |p_i - q_i| = 1$。从 $(a_1, a_2, \ldots, a_d)$ 出发，每次只允许走相邻的点，且图中有一些点不可达，问走到 $(1, 1, \ldots, 1)$ 至少需要几步。

数据范围：$1 \leq d \leq 20, 1 \leq n \leq 10^5$。

## 解题思路

按照字典序将所有点编号，得到一张无向图，在这张图上 BFS 即可得到答案。

建边与否需要额外判断，详见代码。

## 参考代码

```cpp
#include <iostream>
#include <queue>
#include <cstring>
using namespace std;

const int MAXN = 1e5 + 10, MAXD = 21;
bool v[MAXN];
int a[MAXD], mul[MAXD];
int d, n = 1;

queue<int> q;
int dis[MAXN];

inline void bfs()
{
    memset(dis, 0x3f, sizeof dis);
    q.push(n);
    dis[n] = 0;
    while (!q.empty()) {
        int x = q.front();
        q.pop();
        for (int i = 1; i <= d; i++) {
            auto update = [&](const int &y) {
                if (y < 1 || y > n) return;
                if (!v[y] && dis[y] > dis[x] + 1) {
                    dis[y] = dis[x] + 1;
                    q.push(y);
                }
            };
            if ((x - 1) / mul[i] % a[i] != a[i] - 1) update(x + mul[i]);
            if ((x - 1) / mul[i] % a[i] != 0) update(x - mul[i]);
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
    cin >> d;
    for (int i = 1; i <= d; i++) {
        cin >> a[i];
        n *= a[i];
    }

    mul[d] = 1;
    for (int i = d - 1; i >= 1; i--) {
        mul[i] = mul[i + 1] * a[i + 1];
    }
    mul[0] = n;

    string s;
    cin >> s;
    for (int i = 1; i <= n; i++) {
        v[i] = (s[i - 1] == '1');
    }

    bfs();

    cout << (dis[1] == 0x3f3f3f3f ? -1 : dis[1]) << endl;
    return 0;
}
```