# Left and Down

## 题目大意

有一个机器人在 $(x, y)$ 点。每次操作可以指定一个 $0 \leq dx, dy \leq k$，并将其移动到 $(x - dx, y - dy)$。你的目标是将它移动到原点。若 $(dx, dy)$ 在之前的操作中出现过，则该操作代价为 $0$，否则代价为 $1$。你需要最小化总代价。

## 解题思路

有一种显然的总代价为 $2$ 的思路是先指定 $(dx, dy)$ 为 $(1, 0)$，将横坐标先减为 $0$，再指定为 $(0, 1)$，将纵坐标也减为 $0$。

那么只需考虑代价为 $1$ 的情况。记 $g = \gcd(x, y)$，则指定 $(dx, dy)$ 为 $(x/g, y/g)$，并操作 $g$ 次即可到达原点。这种方案可行需要 $\max\{dx/g, dy/g\} \leq k$，判断一下即可。

## 参考代码

```cpp
#include <iostream>
using namespace std;

#define int long long
const int gcd(const int &a, const int &b) { return b ? gcd(b, a % b) : a; }

signed main()
{
#ifndef ONLINE_JUDGE
    freopen("./in", "r", stdin);
#endif
    ios_base::sync_with_stdio(false);
    cin.tie(nullptr), cout.tie(nullptr);

    int t;
    cin >> t;
    while (t--) {
        int a, b, k;
        cin >> a >> b >> k;
        int g = gcd(a, b);
        cout << ((a / g <= k && b / g <= k) ? 1 : 2) << '\n';
    }
    return 0;
}
```