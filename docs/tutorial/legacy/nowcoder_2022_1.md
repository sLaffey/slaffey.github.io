# 「牛客竞赛」赛前集训营 2022 第一场

## 前言

&emsp;&emsp;报名了，也打了，写篇题解罢。

&emsp;&emsp;成绩 90 + 10 + 20 + 0 = 110，排名 39/421。不算好啊。在 HE 都够不到省一线。

## 光

&emsp;&emsp;作为第一道题，单调性这么明显，我居然没想到二分去，属实该打。（不过后来贪心骗了 90 pts，算是一点安慰）

&emsp;&emsp;答案很明显有单调性，所以我们考虑二分答案后枚举判定。

&emsp;&emsp;考虑枚举对角两个格子，然后再枚举剩下两个中一个来判定答案，虽然理论复杂度有些高，但是足以通过本题。为了减小复杂度，可以考虑估计剩下的格子要填的剩余功耗，在估计值附近枚举即可。

```cpp
#include <cstdio>
using namespace std;

const int& max(const int &a, const int &b) { return a > b ? a : b; }
const int& min(const int &a, const int &b) { return a > b ? b : a; }

int A, B, C, D;

inline bool check(int x)
{
    for (int a = 0; a <= x; a++) {
        for (int d = 0; d <= x - a; d++) {
            int na = max(0, A - a - d / 4), nd = max(0, D - d - a / 4);
            int need = max(na, nd);
            int rest = x - a - d;
            if (need > rest / 2) {
                continue;
            }
            int nb = max(0, B - a / 2 - d / 2), nc = max(0, C - a / 2 - d / 2);
            for (int b = 0; b <= rest; b++) {
                int c = rest - b;
                if (b + c / 4 >= nb && c + b / 4 >= nc && b / 2 + c / 2 >= na && b / 2 + c / 2 >= nd) {
                    return true;
                }
            }
        }
    }

    return false;
}

int main()
{
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    // freopen("out", "w", stdout);
#endif
    scanf("%d%d%d%d", &A, &B, &C, &D);

    int r = A + B + C + D, l = 0;
    while (l < r) {
        int mid = (l + r) >> 1;
        if (check(mid)) {
            r = mid;
        }
        else {
            l = mid + 1;
        }
    }
    printf("%d\n", l);

    return 0;
}
```

&emsp;&emsp;来自出题人的估算代码（我不是很懂）。在牛客的测试中显示，加上估算后可以从 `493ms` 跑到 `78ms`。

```cpp
int aa = max(0, (tempa - now / 4) * 4 / 3);
for(int k = max(0, aa - 5); k <= min(now, aa + 5); k++){
    if(k + (now - k) / 4 >= tempa && k / 4 + now - k >= tempd){
        return true;
    }
}
```

## 爬

&emsp;&emsp;挺恶心的一道组合计数题。需要思路清晰才行。

&emsp;&emsp;可以发现这道题中所有点对答案的贡献跟自己的儿子有关。我们考虑逐个节点统计答案。又因为异或属于位运算，直接统计不好计算，我们考虑按位统计。

&emsp;&emsp;考虑一个枚举每一位后的节点（假设不是根节点），我们统计子树中权值为 $1$ 的节点数目，记为 $m$，那么它们产生的有贡献的方案数为 $C^1_m + C^3_m + C^5_m + \ldots = 2^{m - 1}$。再算上剩下的权值为 $0$ 的节点可以任意爬或不爬，总的方案数为 $2^{m - 1} \times 2^{tot - m} = 2^{tot - 1}$，其中 $tot$ 为子节点数目。另外整棵树中其它的节点（除根节点）也可以选择爬或不爬，所以需要再累乘一个 $2^{n - tot - 1}$。另外，由于只留下一个蚂蚁的时候不会产生贡献，所以还要减去这些情况。分为自己留在这个节点与某个子节点上的蚂蚁留在这个节点两种情况。

&emsp;&emsp;考虑根节点。根节点无法往上爬，所以方案数会减半，但是如果根节点的所有子节点的权值都为 $0$，只有根节点自己为 $1$ 的话，方案数又会加回来。

&emsp;&emsp;最后我们统计完了答案，就可以愉快地切了这道题了。

```cpp
#include <cstdio>
#include <vector>
using namespace std;

typedef long long ll;

const int MAXN = 1e5 + 10;
const unsigned int MOD = 1e9 + 7;

ll power[MAXN];
int n;
vector<int> g[MAXN];
int we[MAXN];

int main()
{
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    // freopen("out", "w", stdout);
#endif
    scanf("%d", &n);
    power[0] = 1;
    for (int i = 1; i <= n; i++) {
        power[i] = (power[i - 1] << 1) % MOD;
    }

    for (int i = 1; i <= n; i++) {
        scanf("%d", &we[i]);
    }
    for (int i = 2; i <= n; i++) {
        int fa;
        scanf("%d", &fa);
        g[fa].push_back(i);
    }

    ll ans = 0;
    for (int x = 1; x <= n; x++) {
        int tot = g[x].size() + 1;
        if (tot == 1) continue;

        ll od = power[tot - 1]; // 2^(n - 1) * 2^(tot - n)
        ll exc = power[n - tot];
        if (x != 1) exc = power[n - tot - 1];
        else od = power[tot - 2];

        for (int p = 0; p < 30; p++) {
            int cnt = ((we[x] >> p) & 1);
            for (int j : g[x]) {
                if ((we[j] >> p) & 1) {
                    cnt++;
                }
            }
            if (!cnt) continue;

            int odd = od;
            if (x == 1 && ((we[x] >> p) & 1) && cnt == 1) odd = odd * 2 % MOD;

            ans += power[p] * odd % MOD * exc % MOD;
            ans %= MOD;
        }

        ans = (ans - we[x] * exc % MOD + MOD) % MOD;
        if (x != 1) {
            for (int y : g[x]) {
                ans = (ans - we[y] * exc % MOD + MOD) % MOD;
            }
        }
    }

    printf("%lld\n", ans);

    return 0;
}
```

## 字符串

&emsp;&emsp;首先枚举有多少个切换点。然后考虑贪心。

&emsp;&emsp;每个切换点填 $c$ 个 $B$ 和一个 $A$。我们贪心地先把所有能切换的点用 $1$ 点代价拿到权值，然后将 $B$ 补全到 $b + 1$ 个点，这时随便选一个地方把 $B$ 用完即可。之后我们再将 $A$ 补全，就得到了当前的权值和。用它来更新答案。

```cpp
#include <cstdio>
using namespace std;

inline const int& min(const int &a, const int &b) { return a > b ? b : a; }
inline const int& max(const int &a, const int &b) { return a > b ? a : b; }

const int MAXN = 1e5 + 10;

int main()
{
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    int T;
    scanf("%d", &T);

    while (T--) {
        int n, m, a, b, c;
        scanf("%d%d%d%d%d", &n, &m, &a, &b, &c);

        int ans = (n - 1) / a + (m - 1) / b + 2, lim = min(n, m / c);
        for (int i = 1; i <= lim; i++) {
            int sum = i * 2;
            if (c > b) {
                sum += (c - 1) / b * i;
            }
            if (n - i >= 1) {
                sum++;
                sum += (n - i - 1) / a;
            }
            if (m - c * i >= 1) {
                sum++;
                int rest = m - c * i - 1;
                if (c <= b) {
                    int t = min(i, rest / (b + 1 - c));
                    rest -= t * (b + 1 - c);
                    sum += t;
                }
                else {
                    int t = min(i, rest / (b - (c - 1) % b));
                    rest -= t * ((b - (c - 1) % b));
                    sum += t;
                }
                sum += rest / b;
            }
            ans = max(ans, sum);
        }

        printf("%d\n", ans);
    }

    return 0;
}
```

## 奇怪的函数

&emsp;&emsp;线段树（数据结构一生之敌），水平和时间有限，不写题解了。