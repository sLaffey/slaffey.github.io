# 「NOIP 2015」推销员

## Solution

&emsp;&emsp;从小到大考虑答案，记当前走的最远距离为 $maxp$。可以发现每个住户贡献为 $A_i + \max \lbrace (S_i - maxp) \times 2, 0 \rbrace$。我们贪心地每次选出一个权值最大的来统计答案，就得到了 $\Theta(n^2)$ 的暴力。

&emsp;&emsp;考虑优化，可以发现在更新前后，以 $maxp$ 为分界线的前后住户贡献相对大小顺序不变，因此我们建立两个堆动态维护前后两部分住户即可。

## Code

```cpp
#include <cstdio>
#include <queue>
using namespace std;

const int MAXN = 1e5 + 10;

int maxp, p;
struct resi_type {
    int s, x, p;

    int cal() const
    {
        return (x - maxp) > 0 ? (x - maxp) * 2 : 0;
    }

    inline const bool operator < (const resi_type &a) const
    {
        return s + cal() < a.s + a.cal();
    }
} a[MAXN];

priority_queue<resi_type> geap, leap;

bool v[MAXN];

int main()
{
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    // freopen("out", "w", stdout);
#endif
    int n;
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) {
        scanf("%d", &a[i].x);
        a[i].p = i;
    }
    for (int i = 1; i <= n; i++) {
        scanf("%d", &a[i].s);
    }

    for (int i = 1; i <= n; i++) {
        geap.push(a[i]);
    }
    leap.push({0, 0, 0}); // 其实这行不加也能避免 RE

    int ans = 0;
    for (int i = 1; i <= n; i++) {
        for (p; a[p].x <= maxp && p <= n; p++) {
            if (v[a[p].p]) continue;
            leap.push(a[p]);
        }
        while (!geap.empty() && geap.top().x <= maxp) {
            geap.pop();
        }

        resi_type t;
        if (leap.top() < geap.top()) {
            t = geap.top();
            geap.pop();
        }
        else {
            t = leap.top();
            leap.pop();
        }

        ans += t.s + t.cal();
        maxp = max(maxp, t.x);
        v[t.p] = true;

        printf("%d\n", ans);
    }

    return 0;
}
```