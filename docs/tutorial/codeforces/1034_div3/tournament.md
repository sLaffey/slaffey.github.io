# Tournament

只需每次指定最强的选手和其它选手对局就可以淘汰至只剩两人，特别地，如果需要留下来的选手自身就是最强的，那么可以淘汰至只剩他一人。

```cpp
#include <iostream>
using namespace std;

const int MAXN = 2e5 + 10;
int a[MAXN];

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
        int n, j, k;
        cin >> n >> j >> k;
        int maxa = -1;
        for (int i = 1; i <= n; i++) {
            cin >> a[i];
            maxa = max(a[i], maxa);
        }
        if (k > 1 || (k == 1 && a[j] == maxa)) {
            cout << "YES\n";
        }
        else {
            cout << "NO\n";
        }
    }
    return 0;
}
```