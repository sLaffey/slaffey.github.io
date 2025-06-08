# 「2024 昆明邀请赛」Relearn through Review

## 题意描述

给定正整数序列 $a_1, \ldots, a_n$ 以及一个非负整数 $k$，你可以做出以下操作**最多一次**：选定两个整数 $l, r$ 满足 $1 \leq l \leq r \leq n$，并将每个 $l \leq i \leq r$，将 $a_i$ 赋值为 $a_i + k$。请最大化数列中所有数的最大公因数。

## 解题思路

对于一个确定的 $l, r$，操作后数列的最大公因数可以写成以下几项的最大公因数：

$$
\gcd(a_1, a_2, \ldots, a_l) \\
\gcd(a_{l + 1} + k, a_{l + 2} + k, \ldots, a_r + k) \\
\gcd(a_{r + 1}, \ldots, a_n)
$$

由最大公因数的性质，以及更相减损术可以得到：

$$
\begin{aligned}
 & \gcd(a_{l + 1} + k, a_{l + 2} + k, \ldots, a_r + k) \\
= & \gcd(a_{l + 1} + k, |a_{l + 2} - a_{l + 1}|, \ldots, |a_r - a_{r - 1}|) 
\end{aligned}
$$

也就是说，操作后数列的最大公因数可以写成以下几项的最大公因数：

$$
\gcd(a_1, a_2, \ldots, a_l) \\
a_{l + 1} + k \\
\gcd(|a_{l + 2} - a_{l + 1}|, |a_{l + 3} - a_{l + 2}|, \ldots, |a_r - a_{r - 1}|) \\
\gcd(a_{r + 1}, \ldots, a_n)
$$

其中第一项与第四项均可以用前后缀维护。不妨枚举 $l$，考虑 $r$。

考虑最后一项公因数，我们知道随着 $r$ 变化后缀最大公约数的变化大约在对数级别。考虑每个后缀最大公约数相同的区间，则第四项固定，要使此时答案最大，那么就应使第三项尽可能大，也就是说，令第三项项数尽可能少，换句话说，只要令 $r$ 尽可能小即可。所以我们每次取 $r$ 为使后缀产生变化的值，统计答案即可得出。

## 参考代码

```cpp
#include <iostream>
#include <cstring>
using namespace std;

const long long gcd(const long long &a, const long long &b)
{
    if (a == 0) return b;
    if (b == 0) return a;
    return gcd(b, a % b);
}

template <typename T, typename ...Args>
const long long gcd(const T &a, const Args& ...args)
{
    if (sizeof...(args) == 0) return a;
    return gcd(a, gcd(args...));
}

const int MAXN = 3e5 + 10;
long long a[MAXN];
int n;
long long k;
long long prefix[MAXN], suffix[MAXN];
int Next[MAXN];

int main()
{
#ifndef ONLINE_JUDGE
    freopen("./in", "r", stdin);
#endif
    ios_base::sync_with_stdio(false);
    int t;
    cin >> t;
    while (t--) {
        cin >> n >> k;
        for (int i = 1; i <= n; i++) {
            cin >> a[i];
        }

        prefix[1] = a[1];
        for (int i = 2, p = 1; i <= n; i++) {
            prefix[i] = gcd(prefix[i - 1], a[i]);
            if (prefix[i] != prefix[i - 1]) {
                Next[p] = i;
                p = i;
            }
        }
        
        suffix[n + 1] = 0;
        suffix[n] = a[n];
        for (int i = n - 1; i >= 1; i--) {
            suffix[i] = gcd(a[i], suffix[i + 1]);
        }

        long long ans = 0;
        for (int l = 1; l; l = Next[l]) {
            ans = max(ans, gcd(prefix[l - 1], a[l] + k, suffix[l + 1]));
            long long g = 0;
            for (int r = l + 1; r <= n; r++) {
                g = gcd(g, abs(a[r - 1] - a[r]));
                ans = max(ans, gcd(prefix[l - 1], g, a[r] + k, suffix[r + 1]));
            }
        }

        cout << ans << endl;

        memset(Next + 1, 0, sizeof(int) * n);
        memset(a + 1, 0, sizeof(long long) * n);
        memset(prefix + 1, 0, sizeof(long long) * n);
        memset(suffix + 1, 0, sizeof(long long) * (n + 1));
    }
    
    return 0;
}
```