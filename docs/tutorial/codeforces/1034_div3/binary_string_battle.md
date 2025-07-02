# Binary String Battle

赛时铸币了想了个错的做法。

显然若 $1$ 的数量小于等于 $k$，则 Alice 一定能赢。

否则若 $2k > n$，此时无论 Bob 如何选取区间 $[l, r]$，一定有 $l \leq k \leq r$，则 Alice 只要每次选取除 $a_k$ 外的 $k$ 个 $1$ 变成 $0$，就能保证 $1$ 的数量严格递减，在充分多步后一定有 Alice 赢。

除此之外的情况都是 Bob 赢。

```cpp
#include <iostream>
#include <cstring>
using namespace std;

const int MAXN = 2e5 + 10;
int a[MAXN];
int s[MAXN];

int main()
{
#ifndef ONLINE_JUDGE
    freopen("./in", "r", stdin);
#endif
    ios_base::sync_with_stdio(false);
    cin.tie(nullptr), cout.tie(nullptr);
    int t;
    cin >> t;
    while (t--) {
        int n, k;
        cin >> n >> k;
        for (int i = 1; i <= n; i++) {
            char c;
            cin >> c;
            a[i] = c - '0';
            s[i] = s[i - 1] + a[i];
        }
        if (s[n] <= k || k * 2 > n) {
            cout << "Alice\n";
        }
        else {
            cout << "Bob\n";
        }
    }
    return 0;
}
```