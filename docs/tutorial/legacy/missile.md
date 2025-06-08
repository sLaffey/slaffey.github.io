# 「KDOI-02」一个弹的投

## 前言

&emsp;&emsp;看到都没人交题解，我就来一发吧。

&emsp;&emsp;蒟蒻考场上写这道题写了快三个小时，结果因为没开 long long 挂大分了（悲

## Solution

&emsp;&emsp;首先读题，很明显，我们只需要快速计算出每颗导弹落地的杀伤力，再由此计算出每颗导弹能被反制武器减少的杀伤力，从大到小选择就行了。所以这道题难点其实在于如何计算杀伤力。

&emsp;&emsp;我们首先考虑所有 $y_i$ 均相等，即所有导弹都在同一水平面上的情况。这样就转化成了线段求交的问题。设某个导弹的初始横坐标为 $x$，落地横坐标为 $x^{\prime}$，则很明显有当且仅当 $x_i < x_j$ 且 $x^{\prime}_i > x^{\prime}_j$ 时这两个导弹的轨迹会相交。画成图就是下面这样子。

![](https://cdn.luogu.com.cn/upload/image_hosting/wra6mj03.png)

&emsp;&emsp;所以我们只需要对于每个导弹 $i$ 找出所有导弹 $j$ 满足 $x_i < x_j$ 且 $x^{\prime}_i > x^{\prime}_j$ 或者反过来满足的计算入杀伤力。

&emsp;&emsp;那我们如何在 $\Theta(n \log n)$ 的复杂度内计算出碰撞数量呢？考虑对每个导弹统计答案，我们要做的是对于每个导弹，查询所有落点坐标小于它的导弹中，起始坐标大于它的数量。因此我们考虑将落点坐标排序，从小到大考虑每个导弹，用一个树状数组维护当前大于或小于某个坐标的导弹数量。对于每个导弹，我们先查询起始坐标大于它的导弹数，再将它的起始坐标插入树状数组中。这样就可以在题目规定的时间内求解了。不过为了能用树状数组维护信息，我们需要将横坐标离散化。另外还要注意我们还需要反过来再统计一遍。

&emsp;&emsp;考虑将这个解法推广到一般，根据平抛运动的性质，不在同一水平面上投放的导弹肯定不会相交，所以我们按照纵坐标排序，就可以对于每层纵坐标相同的导弹依次求解了。

## Code

&emsp;&emsp;本蒻解法略复杂些，需要排好几遍序，还需要初始化很多遍树状数组，复杂度常数略大。放在这里供大家参考。

&emsp;&emsp;代码是考场代码略加修改和精简的版本。

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <cmath>
#include <cstring>
using namespace std;

typedef long long ll;

const int MAXN = 5e5 + 10;
const double g = 9.8;

int n, m;

struct bul_type {
    int x, y, v;
    double hit;
    ll power, delta;
    int index;
} bul[MAXN];

int a[MAXN];

struct BIT {
    ll c[MAXN];

    void init()
    {
        memset(c + 1, 0, sizeof(long long) * n);
    }

    void modify(int x)
    {
        while (x <= n) {
            c[x]++;
            x += x & -x;
        }
    }

    ll query(int x)
    {
        ll ans = 0;
        while (x) {
            ans += c[x];
            x -= x & -x;
        }
        return ans;
    }
} tr;

signed main()
{
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    // freopen("out", "w", stdout);
#endif
    ios_base::sync_with_stdio(false);
    cin.tie(nullptr);

    cin >> n >> m;
    for (int i = 1; i <= n; i++) {
        cin >> bul[i].x >> bul[i].y >> bul[i].v;
        bul[i].hit = bul[i].x + sqrt(2 * bul[i].y / g) * bul[i].v;
        bul[i].index = i;
    }
    for (int i = 1; i <= n; i++) {
        cin >> a[i];
    }

    sort(bul + 1, bul + n + 1, [](const bul_type &a, const bul_type &b)
    -> const bool {
        return a.x < b.x;
    });
    int top = 0;
    for (int i = 1; i <= n; i++) {
        top++;
        while (bul[i + 1].x == bul[i].x) {
            bul[i].x = top;
            i++;
        }
        bul[i].x = top;
    }

    sort(bul + 1, bul + n + 1, [](const bul_type &a, const bul_type &b)
    -> const bool {
        return a.y < b.y;
    });

    for (int l = 1, r = 1; l <= n; l = r) {
        while (bul[l].y == bul[r].y) r++;
        sort(&bul[l], &bul[r], [](const bul_type &a, const bul_type &b)
        -> const bool {
            return a.hit < b.hit;
        });
        tr.init();
        for (int i = r - 1; i >= l; i--) {
            int p = bul[i].x;
            bul[i].power = tr.query(p);
            tr.modify(p);
        }
        tr.init();
        for (int i = l; i < r; i++) {
            int p = bul[i].x;
            bul[i].power += tr.query(n) - tr.query(p);
            tr.modify(p);
            bul[i].delta = min(bul[i].power, (long long) a[bul[i].index]);
        }
    }

    sort(bul + 1, bul + n + 1, [](const bul_type &a, const bul_type &b)
    -> const bool {
        return a.delta > b.delta;
    });

    ll ans = 0;
    for (int i = 1; i <= m; i++) {
        ans += bul[i].power - bul[i].delta;
    }
    for (int i = m + 1; i <= n; i++) {
        ans += bul[i].power;
    }

    cout << ans << endl;

    return 0;
}
```